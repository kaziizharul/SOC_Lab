# DDL Injection Attack Detection and Investigation Using Wazuh SIEM

## Project Overview

This project demonstrates the detection, monitoring, and investigation of a Data Definition Language (DDL) Injection attack using the Wazuh Security Information and Event Management (SIEM) platform.

A vulnerable web application (DVWA) was used to simulate SQL Injection attacks targeting a MySQL database. Security events generated during the attack were collected and analyzed through Wazuh, allowing detection of malicious DDL statements such as DROP TABLE, ALTER TABLE, and CREATE TABLE.

The project highlights the role of SIEM technologies in database attack detection, log correlation, threat monitoring, and incident investigation.

---

# Project Objectives

* Deploy and configure Wazuh SIEM
* Simulate DDL Injection attacks
* Monitor MySQL database activities
* Generate and analyze security alerts
* Investigate Indicators of Compromise (IOCs)
* Map attack behavior to MITRE ATT&CK
* Produce incident investigation findings

---

# Lab Environment

| Component        | Description          |
| ---------------- | -------------------- |
| SIEM Platform    | Wazuh                |
| Attacker Machine | Kali Linux           |
| Victim Server    | DVWA Web Application |
| Database         | MySQL                |
| Operating System | Linux                |
| Browser          | Firefox              |

---

# Architecture Diagram

Kali Linux (Attacker)

↓

DVWA Web Application

↓

MySQL Database

↓

Wazuh Agent

↓

Wazuh Manager

↓

Wazuh Dashboard

---

# Attack Scenario

The attacker exploits a SQL Injection vulnerability in DVWA and attempts to execute malicious DDL statements against the backend database.

### Payload Used

```sql
1'; DROP TABLE users; --
```

### Additional DDL Commands Observed

```sql
DROP TABLE users;
ALTER TABLE users ADD COLUMN test VARCHAR(50);
CREATE TABLE backdoor (id INT);
```

These actions are designed to manipulate database structures and potentially destroy or alter sensitive information.

---

# Security Monitoring

## Data Sources

* MySQL Logs
* Web Server Logs
* Wazuh Agent Logs
* Security Events

## Monitored Activities

* Database Structure Changes
* Unauthorized SQL Commands
* Suspicious Query Execution
* High Severity Security Alerts

---

# Detection Methodology

Wazuh continuously monitors MySQL log files and generates alerts when suspicious DDL keywords are detected.

### Detection Keywords

* DROP
* ALTER
* CREATE
* TRUNCATE

### Alert Trigger Logic

DDL statements detected in database logs generate a high-severity security event for analyst review.

---

# Alert Investigation

## Alert Name

DDL Injection Attempt Detected

## Rule Level

High (Level 12)

## Rule ID

100002

## Detection Source

MySQL Log File

## Alert Status

Successfully Detected

---

# Indicators of Compromise (IOCs)

| IOC Type              | Evidence              |
| --------------------- | --------------------- |
| SQL Injection         | DROP TABLE users      |
| Database Modification | ALTER TABLE users     |
| Database Creation     | CREATE TABLE backdoor |
| Source IP             | 192.168.56.105        |
| Application           | DVWA                  |
| Log Source            | MySQL                 |

---

# Log Analysis

The database logs confirmed execution attempts involving multiple DDL statements.

### Observed Events

```sql
DROP TABLE users;
ALTER TABLE users ADD COLUMN test VARCHAR(50);
CREATE TABLE backdoor (id INT);
```

These commands indicate an attempt to manipulate database schema and establish persistence through unauthorized database objects.

---

# MITRE ATT&CK Mapping

| Technique ID | Technique                         |
| ------------ | --------------------------------- |
| T1190        | Exploit Public-Facing Application |
| T1059        | Command and Scripting Interpreter |
| T1505        | Server Software Component         |
| T1078        | Valid Accounts                    |

---

# Investigation Findings

## Detection Result

YES

## Alert Generated

YES

## Database Activity Logged

YES

## Attack Successfully Identified

YES

## Analyst Visibility

YES

Wazuh successfully detected suspicious DDL statements and generated actionable security alerts for investigation.

---

# Screenshots
<img width="1536" height="1024" alt="Image" src="https://github.com/user-attachments/assets/57ac161f-2990-4e1f-9275-0306641a176f" />


---

# Recommendations

* Implement parameterized queries
* Validate all user inputs
* Deploy a Web Application Firewall (WAF)
* Restrict database permissions
* Enable continuous log monitoring
* Review SIEM alerts regularly
* Follow secure coding practices

---

# Conclusion

This project successfully demonstrated the detection and investigation of DDL Injection attacks using Wazuh SIEM. Security events generated during the attack were collected, analyzed, and correlated to produce actionable alerts. The investigation confirmed that Wazuh provides effective visibility into database-focused attacks and can significantly improve an organization's threat detection and incident response capabilities.

---

# Skills Demonstrated

* Security Monitoring
* SIEM Operations
* Wazuh Administration
* Log Analysis
* Threat Detection
* Incident Investigation
* SQL Injection Analysis
* MITRE ATT&CK Mapping

---

# Author

Kazi Izharul Islam

SOC Analyst | Blue Team Learner

GitHub Security Project Portfolio
