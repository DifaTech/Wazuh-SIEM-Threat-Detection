# 🛡️ Wazuh SIEM Deployment & Threat Detection Lab

## 📌 Project Overview

This project demonstrates the deployment of a **Security Information and Event Management (SIEM)** environment using **Wazuh** in a virtualized laboratory environment.

The goal was to build a small SOC-style monitoring environment that could collect security logs from different operating systems, detect suspicious activities, generate security alerts, and support basic incident investigation.

The lab was implemented using **Oracle VirtualBox** and consisted of a Wazuh Manager running on Ubuntu Server, a Windows 10 endpoint, and a Kali Linux endpoint. The endpoints were connected to the Wazuh Manager using Wazuh agents.

---

## 🏗️ Lab Architecture

The environment consisted of three virtual machines connected through a **VirtualBox Host-Only network**.

| Component | Operating System | Role | IP Address |
|---|---|---|---|
| Wazuh Manager & Dashboard | Ubuntu Server | SIEM Manager | `192.168.56.103` |
| Windows Endpoint | Windows 10 | Monitored Endpoint | `192.168.56.105` |
| Linux Endpoint | Kali Linux | Monitored Endpoint / Attack Source | `192.168.56.104` |

### Log Sources

The Windows endpoint forwarded:

- System Event Logs
- Security Event Logs
- Application Event Logs

The Kali Linux endpoint forwarded:

- Syslog events
- Authentication logs

These logs were collected by the Wazuh Manager for centralized monitoring and analysis.

---

## 🎯 Project Objectives

The main objectives of the project were to:

1. Deploy a centralized Wazuh SIEM Manager using Ubuntu Server.
2. Connect Windows 10 and Kali Linux endpoints to the Wazuh Manager using Wazuh agents.
3. Configure the endpoints to forward security-related logs to the SIEM.
4. Simulate controlled security activities within the isolated laboratory.
5. Verify that Wazuh could detect and record the activities.
6. Analyze generated alerts using the Wazuh Dashboard.
7. Map relevant detected activities to the MITRE ATT&CK framework.
8. Document observations and security recommendations.

---

## 🔍 Security Simulations & Detection

The SIEM deployment was validated through three controlled simulations performed within the isolated laboratory environment.

### 1. SSH Brute-Force Attempt

The SSH brute-force simulation was launched from the Kali Linux endpoint against the Ubuntu Wazuh Manager.

The purpose was to generate multiple failed SSH authentication attempts and observe how Wazuh collected and analyzed the resulting authentication events.

**Command used:**

``
hydra -l fakeuser -p "pass1,pass2,pass3,pass4,pass5" ssh://192.168.56.103``

The action generated a Windows security event which was collected and detected by Wazuh.

**MITRE ATT&CK:** T1110 – Brute Force

### 2. Unauthorized Administrator Account Creation

A local Windows account named `hackeradmin` was created on the Windows 10 endpoint and subsequently added to the local Administrators group.

**Commands used:**
```
net user hackeradmin Password123! /add
net localgroup Administrators hackeradmin /add
```

The actions generated Windows Security Event Logs which were forwarded to the Wazuh Manager for analysis.

The activity was detected through Wazuh alerts related to account creation and changes to the Administrators group.

**MITRE ATT&CK:**

- T1136 – Create Account
- T1136.001 – Local Account

---

### 3. Windows Security Event Log Clearing

The Windows Security Event Log was cleared to simulate a **defense evasion** activity that an attacker may attempt after compromising a system.

**Command used:**

```
wevtutil cl Security
```

The action generated a Windows security event which was collected and detected by Wazuh.

---

## 📊 Alert Analysis

The Wazuh Dashboard was used to review the alerts generated during the simulations.

The investigation included reviewing:

- Alert descriptions
- Affected endpoints
- Event information
- Security event categories
- Wazuh rule information
- Relevant MITRE ATT&CK mappings

Screenshots and supporting evidence are provided in this repository to demonstrate the actual detection results.

---

## 📑 Incident Analysis

One of the main activities investigated during the project was the creation of an unauthorized local administrator account.

The activity involved creating the `hackeradmin` account and adding it to the local Administrators group. Windows generated Security Event Logs for these actions, which were forwarded to the Wazuh Manager.

Wazuh analyzed the incoming events and generated alerts that allowed the activity to be identified and reviewed through the Dashboard.

This demonstrated how centralized log collection can provide visibility into account-management activities that may indicate unauthorized access or persistence.

### MITRE ATT&CK Mapping

| Category | Details |
|---|---|
| Tactic | Persistence |
| Technique | T1136 – Create Account |
| Sub-technique | T1136.001 – Local Account |
| Platform | Windows |
| Detection Source | Windows Security Event Logs collected by Wazuh |

---

## 🛠️ Tools & Technologies

- **SIEM:** Wazuh
- **Virtualization:** Oracle VirtualBox
- **SIEM Server:** Ubuntu Server
- **Endpoints:** Windows 10, Kali Linux
- **Network:** VirtualBox Host-Only Network
- **Attack Simulation:** Hydra
- **Security Analysis:** Wazuh Dashboard
- **Framework:** MITRE ATT&CK

---

## 📚 What I Learned

Through this project, I gained practical experience in:

- Deploying a SIEM solution in a virtual laboratory.
- Installing and registering Wazuh agents.
- Collecting logs from Windows and Linux systems.
- Monitoring authentication and system events.
- Simulating controlled security activities.
- Investigating SIEM alerts.
- Understanding how security events are used for threat detection.
- Mapping detected activities to the MITRE ATT&CK framework.
- Understanding the importance of centralized security monitoring in a SOC environment.

---

## ⚠️ Disclaimer

All security simulations in this project were performed in an isolated virtual laboratory created for educational purposes. No unauthorized systems or real-world networks were targeted.


