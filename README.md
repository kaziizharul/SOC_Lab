# DDL Injection Attack Detection and Investigation Using Wazuh SIEM

A hands-on detection-engineering lab demonstrating how to detect, alert on,
and investigate a **DDL Injection attack** (`DROP` / `ALTER` / `CREATE` /
`TRUNCATE` statements smuggled through an unsanitized SQL Injection point)
using **Wazuh SIEM**, **DVWA**, and **MySQL general query log** as the
evidence source.

> DDL injection is a more destructive sibling of classic SQL injection —
> instead of just reading data (`SELECT`), the attacker rewrites the
> database schema itself (drops tables, alters columns, creates rogue
> tables for persistence). This project shows the full attack-to-alert
> pipeline: exploit → log → decode → rule match → alert → investigate.

---

## Lab Topology

| Role | Hostname | IP Address | Notes |
|---|---|---|---|
| Attacker / Wazuh Manager | `kali` | `192.168.121.128` | Kali Linux — runs the SQLi/DDLi payloads and hosts the Wazuh manager + dashboard |
| Target / Wazuh Agent | `windows` (agent **001**) | `192.168.121.129` | Wazuh agent `v4.13.1`, monitored endpoint running the vulnerable web app stack |

This mirrors the agent registration shown in the Wazuh **Endpoints summary**
view below (agent `001`, status `active`, registered against manager node
`node01`).

![Wazuh agent overview](screenshots/individual/agent_overview_reference.png)

---

## Architecture

```
 ┌────────────────────┐        SQLi / DDLi payload         ┌─────────────────────────┐
 │   Attacker (Kali)   │ ───────────────────────────────▶  │  Target Web Server      │
 │  192.168.121.128    │   id=1'; DROP TABLE users; --      │  192.168.121.129         │
 │  Wazuh Manager      │                                    │  DVWA + MySQL            │
 └─────────▲───────────┘                                    │  Wazuh Agent (001)       │
           │                                                 └────────────┬─────────────┘
           │              alerts (rule 100002, level 12)                  │
           │                                                  mysql.log (general_log)
           │                                                              │
           └──────────────────────  Wazuh Manager  ◀──────────── Wazuh Agent ships logs
                       (decoder: mysql_general_log → rule: 100002)
```

---

## Detection Logic

| Component | File | Purpose |
|---|---|---|
| Decoder | [`wazuh-config/local_decoder.xml`](wazuh-config/local_decoder.xml) | Parses MySQL general query log lines into `data.query`, `data.user`, `data.srcip` |
| Rule | [`wazuh-config/local_rules.xml`](wazuh-config/local_rules.xml) | Rule `100002` (level 12) fires when `data.query` matches `^(DROP\|ALTER\|CREATE\|TRUNCATE)\s+TABLE` |
| Log source | [`wazuh-config/ossec_localfile_snippet.xml`](wazuh-config/ossec_localfile_snippet.xml) | Ships `/var/log/mysql/mysql.log` from the agent to the manager |

Rule `100002` maps to **MITRE ATT&CK T1190 — Exploit Public-Facing
Application**, and is tagged for **PCI DSS 6.5.1** (injection flaws).

---

## Attack Simulation

The full attack chain — confirm injection point → `DROP TABLE` →
`ALTER TABLE` → `CREATE TABLE` (backdoor) — is scripted in
[`scripts/simulate_ddl_injection.sh`](scripts/simulate_ddl_injection.sh)
for repeatable testing against a DVWA instance set to **security level: low**.

```bash
./scripts/simulate_ddl_injection.sh 192.168.121.129 "PHPSESSID=...; security=low" "<csrf_token>"
```

⚠️ Run only in an isolated lab you own. DVWA is intentionally vulnerable —
never expose it to a real network.

---

## Walkthrough

### 1 — Dashboard overview
Wazuh **Security events → Dashboard** showing the spike in alert volume
during the attack window: total events, level-12+ alerts, and the
top-talking agents.

<img width="3072" height="1600" alt="Image" src="https://github.com/user-attachments/assets/1d9449e1-8adc-4efd-835e-552efed24f2a" />

### 2 — Payload delivery (DVWA)
The injection payload submitted through DVWA's SQL Injection page. The
`User ID` field carries a stacked query (`1'; DROP TABLE users; --`) instead
of a single ID.

<img width="2360" height="1440" alt="Image" src="https://github.com/user-attachments/assets/67c2d86f-9e6f-4c58-897b-e4e7b20d7d2d" />

### 3 — Alert detail
Wazuh's expanded alert view for rule `100002` — **"DDL Injection Attempt
Detected"** at severity **High** (level 12), with source IP, user, and the
exact query that triggered the rule.

<img width="2360" height="1520" alt="Image" src="https://github.com/user-attachments/assets/f6880b66-3089-4bf2-9cf6-3ad8b5ca0250" />

### 4 — Log analysis
The raw evidence on the target host: `mysql.log` (general query log)
showing the `SELECT` baseline followed by the `DROP TABLE`, `ALTER TABLE`,
and `CREATE TABLE` statements as they actually executed.

<img width="2040" height="1120" alt="Image" src="https://github.com/user-attachments/assets/db8839ca-af57-4cc7-ad68-4bf39b1b1eed" />

### 5 — Investigation (expanded document)
The fully parsed Wazuh event document — `agent.id`, `data.srcip`,
`data.user`, `rule.id`, `rule.level`, and `full_log` — used to confirm
attribution and timeline during incident investigation.

<img width="2360" height="1520" alt="Image" src="https://github.com/user-attachments/assets/2af9bd8a-4645-466c-badb-b74804a77d93" />

### Combined view
All five panels together, for quick reference:

<img width="1562" height="888" alt="Image" src="https://github.com/user-attachments/assets/3a6171da-0455-4272-81a4-fb11d18b06ce" />
---

## Sample Artifacts

- [`docs/sample_mysql_general.log`](docs/sample_mysql_general.log) — raw log lines that produced the alert
- [`docs/sample_alert.json`](docs/sample_alert.json) — the parsed Wazuh alert document

---

## Reproducing This Lab

1. Stand up Wazuh manager + indexer + dashboard (Kali host, `192.168.121.128`, or any manager host).
2. Install the Wazuh agent on the target web server and register it (agent `001`, `192.168.121.129`).
3. Deploy DVWA on the target, backed by MySQL/MariaDB.
4. Enable MySQL's general query log:
   ```sql
   SET GLOBAL general_log = 'ON';
   SET GLOBAL general_log_file = '/var/log/mysql/mysql.log';
   ```
5. Copy `wazuh-config/local_decoder.xml` → `/var/ossec/etc/decoders/` on the manager.
6. Copy `wazuh-config/local_rules.xml` → `/var/ossec/etc/rules/` on the manager.
7. Add the `<localfile>` block from `wazuh-config/ossec_localfile_snippet.xml` to the agent's `ossec.conf`.
8. Restart the manager (`systemctl restart wazuh-manager`) and the agent.
9. Run `scripts/simulate_ddl_injection.sh` against the target and watch the alert land in **Security events**.

---

## Key Takeaways

- **DDL statements in application logs are a strong, low-noise signal.**
  Legitimate web apps almost never issue `DROP`/`ALTER`/`CREATE` through
  user-facing input — a single match is high-confidence.
- **Decoder + rule pairing turns an unstructured log into a queryable
  field** (`data.query`), which is what makes the regex-based detection
  reliable instead of a blind substring grep.
- **The investigation view (raw → parsed → alert) closes the loop**
  between "something fired" and "here is exactly what happened, from
  where, as whom."

---

## Disclaimer

This repository is for educational and authorized security-testing
purposes only (home lab / CTF / training environment). Do not use these
techniques against systems you do not own or have explicit written
permission to test.
