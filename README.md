# 🛡️ Enterprise SIEM Deployment & Threat Detection Lab

## 📌 Executive Summary
This project demonstrates the end-to-end deployment of an enterprise-grade **Security Information and Event Management (SIEM)** environment using **Wazuh**. The primary goal of this lab was to establish centralized log collection across multiple operating systems, simulate real-world security threats, and perform incident triage using the **MITRE ATT&CK** framework.

---

## 🏗️ Architecture & Network Setup
* **SIEM Manager:** Wazuh Manager (Central Monitoring & Alerting Engine)
* **Monitored Endpoints:** 
  * Windows Workstation (`Windows-target` with Wazuh Agent + Sysmon for granular process logging)
  * Linux Server (`Ubuntu` with Wazuh Agent)
* **Virtualization:** VMware / VirtualBox in an isolated host-only network environment

---

## 🎯 Key Objectives
1. **Infrastructure Provisioning:** Configured a multi-node virtual lab setup.
2. **Telemetry Ingestion:** Forwarded system and security event logs from Windows and Linux endpoints to the Wazuh Manager.
3. **Endpoint Visibility:** Integrated **Sysmon** on Windows to capture process creation, network connections, and account modifications.
4. **Adversary Simulation:** Simulated controlled attack vectors to trigger alerts:
   * SSH Brute-Force Authentication Attacks
   * Unauthorized Local Account Creation & Privilege Escalation
   * Windows Audit Log Clearing
5. **Incident Response & Triage:** Mapped alerts to **MITRE ATT&CK**, analyzed raw log JSON, and documented remediation steps.

---

## 🔍 Attack Simulations & Detection Proof

### Scenario 1: SSH Authentication Failures (Brute-Force Vector)
* **Vector:** Generated high-frequency failed SSH authentication requests against an Ubuntu host using non-existent usernames.
* **Detection:** Captured via Wazuh Rule ID `5710` (`sshd: Attempt to login using a non-existent user`) and Rule ID `5503` (`PAM: User login failed`).
* **MITRE ATT&CK Mapping:** T1110 (Brute Force)

### Scenario 2: Unauthorized Local Account Creation & Escalation
* **Vector:** Created a local user account on a Windows endpoint and elevated its privileges to the local Administrators group.
* **Detection:** Triggered Wazuh Rule ID `60109` (`User account enabled or created`, Level 8) followed by high-severity Rule ID `60154` (`Administrators Group Changed`, Level 12).
* **MITRE ATT&CK Mapping:** T1136 (Create Account) & T1098 (Account Manipulation)

### Scenario 3: Windows Audit Log Clearing (Defense Evasion)
* **Vector:** Executed log suppression commands to clear the Windows Security Event Log on the target host.
* **Detection:** Triggered Wazuh Rule ID `63103` (`The audit log was cleared`, Level 5).
* **MITRE ATT&CK Mapping:** T1070.001 (Indicator Removal: Clear Windows Event Logs)

---

## 📑 Sample Incident Triage Report

| Field | Details |
| :--- | :--- |
| **Incident ID** | INC-2026-001 |
| **Target Host** | `Windows-target` |
| **Severity / Alert Level** | **High** (Wazuh Rule Level 12) |
| **Detection Source** | Wazuh Ruleset (Rule IDs: `60109`, `60154`, `63103`) & Windows Event Logs |
| **MITRE ATT&CK Tactics** | Persistence (TA0003), Privilege Escalation (TA0004), Defense Evasion (TA0005) |
| **MITRE Techniques** | T1136 (Create Account), T1098 (Account Manipulation), T1070.001 (Clear Windows Logs) |
| **Observed Activity** | Unauthorized local user creation followed immediately by elevation to the local `Administrators` group and clearing of audit logs. |
| **Containment Action** | Network isolation of `Windows-target`, removal of rogue administrative account, audit log backup restoration, and host integrity scan. |

---

## 🛠️ Tools & Technologies
* **SIEM:** Wazuh 
* **Telemetry:** Sysmon, Windows Event Log, Linux Syslog
* **Frameworks:** MITRE ATT&CK
* **Virtualization:** Oracle VirtualBox / VMware Workstation
* **Operating Systems:** Windows 10, Ubuntu Linux
