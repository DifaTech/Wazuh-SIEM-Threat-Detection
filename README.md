# 🛡️ Wazuh SIEM Deployment & Threat Detection Lab

## 📌 Project Overview

This project demonstrates the deployment of a **Security Information and Event Management (SIEM)** environment using **Wazuh** in a virtualized laboratory environment.

The goal was to build a small SOC-style monitoring environment that could collect security logs from different operating systems, detect suspicious activities, generate security alerts, and support basic incident investigation.

The lab was implemented using **Oracle VirtualBox** and consisted of a Wazuh Manager running on Ubuntu Server, a Windows 10 endpoint, and a Kali Linux endpoint. The endpoints were connected to the Wazuh Manager using Wazuh agents.

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

## 🏗️ Lab Architecture

The environment consisted of three virtual machines connected through a **VirtualBox Host-Only network**.

| Component | Operating System | Role | IP Address |
|---|---|---|---|
| Wazuh Manager & Dashboard | Ubuntu Server | SIEM Manager | `192.168.56.103` |
| Windows Endpoint | Windows 10 | Monitored Endpoint | `192.168.56.105` |
| Linux Endpoint | Kali Linux | Monitored Endpoint / Attack Source | `192.168.56.104` |

---

## 📌 Network Design

The VirtualBox Host-Only network allowed the virtual machines to communicate with each other within an isolated laboratory environment.

This setup made it possible to perform the security simulations without targeting external or unauthorized systems.

### Basic Architecture

```text
+-----------------------------------------------------+
|                    Wazuh Manager                    |
|                    Ubuntu Server                    |
|                   192.168.56.103                    |
|                                                     |
|            Wazuh Manager + Wazuh Dashboard          |
+--------------------------+--------------------------+
                           |
                   Host-Only Network
                           |
             +-------------+-------------+
             |                           |
             v                           v
+-------------------------+ +-------------------------+
|       Kali Linux        | |       Windows 10        |
|     192.168.56.104      | |     192.168.56.105      |
|                         | |                         |
| Wazuh Agent             | | Wazuh Agent             |
| Security Testing        | | Windows Logs            |
+-------------------------+ +-------------------------+
```
---

## 📊Log Sources

The Windows endpoint forwarded:

- System Event Logs
- Security Event Logs
- Application Event Logs

The Kali Linux endpoint forwarded:

- Syslog events
- Authentication logs

These logs were collected by the Wazuh Manager for centralized monitoring and analysis.

---

## 🔍 Security Simulations & Detection

The SIEM deployment was validated through three controlled simulations performed within the isolated laboratory environment.

### 1️⃣ SSH Brute-Force Attempt

The SSH brute-force simulation was launched from the Kali Linux endpoint against the Ubuntu Wazuh Manager.

The purpose was to generate multiple failed SSH authentication attempts and observe how Wazuh collected and analyzed the resulting authentication events.

**Command used:**

```
hydra -l fakeuser -p "pass1,pass2,pass3,pass4,pass5" ssh://192.168.56.103
```

The activity generated multiple failed SSH authentication events on the Ubuntu host, which were collected and analyzed by Wazuh.

**Wazuh Detection**

The resulting authentication events generated alerts including:

- Rule ID ``5710`` — ``sshd: Attempt to login using a non-existent user``
- Rule ID ``5503`` — ``PAM: User login failed``

**MITRE ATT&CK:** ``T1110`` – Brute Force

### 2️⃣ Unauthorized Administrator Account Creation

The second simulation was performed on the Windows 10 endpoint to test Wazuh's ability to detect changes involving local user accounts and administrator privileges.

A local Windows account named `hackeradmin` was created and then added to the local **Administrators** group.

**Commands used:**
```
net user hackeradmin Password123! /add
net localgroup Administrators hackeradmin /add
```

The first command created the local user account, while the second command added the account to the local Administrators group.

The resulting Windows Security events were collected by the Wazuh agent and forwarded to the Wazuh Manager for analysis.

**Wazuh Detection**

The activity generated alerts related to the creation of the local account and the modification of the Administrators group:

- Rule ID `60109` — `User account enabled or created`
Level: 8
- Rule ID `60154` — `Administrators Group Changed`
Level: 12

The alerts allowed the account creation and subsequent administrator-group modification to be identified and investigated through the Wazuh Dashboard.

**MITRE ATT&CK Mapping**

- T1136 — Create Account
- T1136.001 — Local Account
- T1098 — Account Manipulation

This simulation demonstrated how changes to local accounts and privileged groups can be monitored as potential indicators of unauthorized activity.


### 3️⃣ Windows Security Event Log Clearing

The Windows Security Event Log was cleared to simulate a **defense evasion** activity that an attacker may attempt after compromising a system.

**Command used:**

```
wevtutil cl Security
```
This command was used to clear the Windows Security Event Log in the controlled laboratory environment.

**Wazuh Detection**

Wazuh detected the resulting Windows security event and generated:

- Rule ID `63103` — `The audit log was cleared`
Level: 5

This demonstrated that even an attempt to remove security records could itself generate a detectable event.

**MITRE ATT&CK Mapping**

T1070.001 — Clear Windows Event Logs

This technique falls under Indicator Removal, where an attacker attempts to remove or modify evidence of their activity.

**Evidence**

Screenshots showing the command execution and the corresponding Wazuh alert are included in this repository.

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

One of the main activities investigated during the project was the creation of an unauthorized local administrator account on the Windows endpoint.

The activity involved two main actions:

1. Creating the ``hackeradmin`` local account.
2. Adding ``hackeradmin`` to the local Administrators group.

The Windows endpoint generated security events for these activities. The Wazuh agent collected the events and forwarded them to the Wazuh Manager, where they were analyzed and displayed as alerts in the Wazuh Dashboard.

The account creation activity generated **Wazuh Rule ID `60109`**, while the subsequent modification of the Administrators group generated **Wazuh Rule ID `60154`**.

The second alert had a **Level 12** severity, making the administrator-group modification the higher-level alert observed during this activity.

This investigation demonstrated how related Windows account-management events can be monitored together to provide a clearer understanding of suspicious activity on an endpoint.

---

## 📊 Detection Summary

Activity |	Wazuh Rule |	Level |	Description
|---|---|---|---|
Local account creation |	``60109`` |	8 |	User account enabled or created
Administrator group modification |	``60154`` |	12 | Administrators Group Changed
Windows Security Log clearing |	``63103`` |	5 |	The audit log was cleared
SSH authentication failure	| ``5710``	| 5	| Attempt to login using a non-existent user
SSH authentication failure	| ``5503`` |	5 |	PAM: User login failed

---

## 🧭 MITRE ATT&CK Mapping

The detected activities were mapped to relevant MITRE ATT&CK techniques.

Security Activity |	MITRE ATT&CK Technique |	Technique Name
|---|---|---|
SSH authentication attacks |	``T1110`` |	Brute Force
Local account creation |	``T1136``	| Create Account
Local account creation |	``T1136.001``	| Local Account
Administrator group modification |	``T1098`` |	Account Manipulation
Windows Security Event Log clearing	| ``T1070.001``|	Clear Windows Event Logs

The mapping helped provide a standard way of describing the activities observed during the lab

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


