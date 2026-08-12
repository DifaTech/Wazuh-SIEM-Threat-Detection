# 🔐 Wazuh SIEM Deployment & Threat Detection — Detailed Walkthrough

## 1. Introduction

This document provides a step-by-step walkthrough of the practical implementation of the Wazuh SIEM laboratory.

The project was built from scratch using Oracle VirtualBox. The lab was designed to simulate a small Security Operations Center (SOC) environment where security events from Windows and Linux systems could be collected centrally, analyzed, and detected through Wazuh.

The environment consisted of:

- Ubuntu Server — Wazuh Manager and Dashboard
- Windows 10 — monitored endpoint
- Kali Linux — monitored endpoint and security testing machine

The three virtual machines communicated through a VirtualBox Host-Only network, keeping the security testing within an isolated laboratory environment.

---

# 2. Virtual Lab Architecture

## 2.1 Virtual Machine Provisioning

The first stage was creating the virtual machines required for the SIEM environment using Oracle VirtualBox.

The resources allocated to each virtual machine were based on the requirements of the laboratory and the available system resources.

| Virtual Machine | Operating System | RAM | Storage | Processors | Role |
|---|---|---:|---:|---:|---|
| Ubuntu Server | Ubuntu Server | 4 GB | 40.52 GB | 2 | Wazuh Manager & Dashboard |
| Windows Endpoint | Windows 10 | 4 GB | 50 GB | 2 | Monitored Endpoint |
| Kali Linux | Kali Linux | 2 GB | 39.06 GB | 2 | Monitored Endpoint / Security Testing |

The Ubuntu Server was given the highest memory allocation because it was responsible for running the Wazuh Manager and Dashboard and processing logs received from the monitored endpoints.

Windows 10 was allocated 4 GB of RAM to support the operating system and security monitoring components, while Kali Linux was allocated 2 GB of RAM because it was primarily used as the Linux monitoring endpoint and security testing machine.

---

## 2.2 Ubuntu Server

The Ubuntu Server was deployed as the central SIEM server.

Its main responsibilities were:

- Running the Wazuh Manager
- Running the Wazuh Dashboard
- Receiving logs from monitored endpoints
- Processing security events
- Generating alerts
- Providing a centralized interface for security monitoring

The Ubuntu Server was assigned:

```text
IP Address: 192.168.56.103
```

---

## 2.3 Windows 10 Endpoint

A Windows 10 virtual machine was created as the Windows endpoint.

The machine was configured with:

```text
RAM: 4 GB
Storage: 50 GB
Processors: 2
IP Address: 192.168.56.105
```

The Windows machine was later configured with a Wazuh Agent so that Windows security-related events could be forwarded to the Wazuh Manager.

The Windows endpoint provided visibility into:

- System events
- Security events
- Application events
- User and account activities
- Other operating system activities monitored by Wazuh

---

## 2.4 Kali Linux Endpoint

Kali Linux was deployed as the Linux monitoring endpoint and security testing machine.

The virtual machine was configured with:

```text
RAM: 2 GB
Storage: 39.06 GB
Processors: 2
IP Address: 192.168.56.104
```

Kali Linux was configured with a Wazuh Agent and used to forward authentication and system log information to the Wazuh Manager.

It was also used as the source system for the controlled SSH authentication attack simulation.

---

# 3. Network Configuration

The three virtual machines were connected using the **VirtualBox Host-Only Adapter**.

The Host-Only network provided communication between the virtual machines while keeping the laboratory isolated from external systems.

The IP addressing used in the laboratory was:

| System | IP Address |
|---|---|
| Ubuntu Server / Wazuh Manager | `192.168.56.103` |
| Kali Linux | `192.168.56.104` |
| Windows 10 | `192.168.56.105` |

The resulting architecture was:

```text
                         ┌─────────────────────────────┐
                         │       Ubuntu Server         │
                         │        Wazuh Manager        │
                         │       Wazuh Dashboard       │
                         │       192.168.56.103        │
                         └──────────────┬──────────────┘
                                        │
                              VirtualBox Host-Only
                                   Network
                                        │
                         ┌──────────────┴──────────────┐
                         │                             │
                         │                             │
              ┌──────────▼─────────┐       ┌──────────▼─────────┐
              │     Kali Linux     │       │     Windows 10     │
              │   192.168.56.104   │       │   192.168.56.105   │
              │    Wazuh Agent     │       │    Wazuh Agent     │
              │  Security Testing  │       │  Security Events   │
              └────────────────────┘       └────────────────────┘
```

### Architecture Evidence

The repository contains the topology diagram used to document the final laboratory architecture.

![SIEM Topology](https://github.com/DifaTech/Wazuh-SIEM-Threat-Detection/blob/main/images/setup/SIEM%20Topolgy.jpg)

---

# 4. Wazuh Manager Deployment on Ubuntu Server

## 4.1 Wazuh Installation

After preparing the Ubuntu Server virtual machine, Wazuh was deployed as the central SIEM platform.

The Wazuh installation assistant was downloaded using `curl` and executed with administrative privileges.

The following command was used:

```bash
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```
The -a option was used to perform the all-in-one Wazuh installation.

This installation provided the main components required for the SIEM environment, including the Wazuh Manager, Wazuh Indexer, and Wazuh Dashboard.

After the installation completed, the Ubuntu Server served as the central monitoring system for the laboratory.

The Wazuh Manager was assigned: 

`IP Address: 192.168.56.103`

---

## 4.2 Starting and Verifying Wazuh Services

The Wazuh services were started on the Ubuntu Server using:

```bash
sudo systemctl start wazuh-manager
sudo systemctl start wazuh-indexer
sudo systemctl start wazuh-dashboard
```

The Wazuh Manager service status was then checked using:

```bash
sudo systemctl status wazuh-manager
```

This was used to confirm that the Wazuh Manager service was running correctly before connecting the monitored endpoints.

---

# 5. Installing and Configuring the Wazuh Agents

The monitored systems were connected to the Wazuh Manager using Wazuh agents.

Two agents were deployed:

1. Windows 10 Wazuh Agent
2. Kali Linux Wazuh Agent

After installation and registration, both agents successfully appeared as **Active** in the Wazuh Dashboard.

---

# 6. Windows 10 Wazuh Agent Installation

## 6.1 Downloading and Installing the Agent

The Windows 10 endpoint was configured with a Wazuh Agent using PowerShell running with Administrator privileges.

The Wazuh Agent MSI package was downloaded using:

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.6-1.msi -OutFile ${env:TEMP}\wazuh-agent.msi
```

---

## 6.2 Configuring Windows Telemetry & Event Channels

By default, the Windows agent automatically captures core Windows Event Logs. The default configuration in C:\Program Files (x86)\ossec-agent\ossec.conf was verified to collect:

- System Event Logs: Tracks authentication events (logon/logoff), user account management (creation, privilege escalation), and security policy changes.
- Security Event Logs: Tracks operating system services, driver updates, and system level events.
- Application Event Logs: Tracks application errors and software event logs.

---

# 6.3 Starting the Windows Agent

After installation, the Windows Wazuh Agent service was started using:

```powershell
NET START WazuhSvc
```

This started the Wazuh Agent service and allowed the Windows endpoint to begin sending telemetry to the Wazuh Manager.

The successful registration of the Windows agent was verified through the Wazuh Dashboard.

### Windows Agent Evidence

![Windows Telemetry](https://github.com/DifaTech/Wazuh-SIEM-Threat-Detection/blob/main/images/setup/Windows%20telemetry.png)

---

# 7. Kali Linux Wazuh Agent Installation

## 7.1 Downloading the Wazuh Agent

The Wazuh Agent package was downloaded on the Kali Linux endpoint using:

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.6-1_amd64.deb
```

The downloaded Debian package was then installed while specifying the Wazuh Manager IP address and the agent name.

```bash
sudo WAZUH_MANAGER='192.168.56.103' WAZUH_AGENT_NAME='Kali-Endpoint' dpkg -i ./wazuh-agent_4.14.6-1_amd64.deb
```

The following values were used during installation:

```text
Wazuh Manager: 192.168.56.103
Agent Name: Kali-Endpoint
```

This configured the Kali Linux machine to communicate with the Wazuh Manager.

---

## 7.2 Enabling and Starting the Kali Agent

After installation, the Wazuh Agent service was reloaded, enabled, and started:

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

Enabling the service allowed the Wazuh Agent to start automatically with the system.

Starting the service allowed the agent to begin communicating with the Wazuh Manager.

---

## 7. Configuring Kali Linux Log Collection

To ensure Kali's authentication and system events were forwarded to the Wazuh Manager, custom log collection paths were added to the local agent configuration using `nano`:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Inserted the following XML blocks under the <ossec_config> section:

```bash
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>
</localfile>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/syslog</location>
</localfile>
```

After adding the configuration, I saved the file and restarted the Wazuh agent so that the changes could take effect:

```bash
sudo systemctl restart wazuh-agent
```

I then verified that the Wazuh agent was running:

```bash
sudo systemctl status wazuh-agent
```

This configuration allowed the Kali Linux endpoint to forward authentication and system log information to the Wazuh Manager for centralized monitoring and analysis.

---

### Explanation

The first configuration monitors:

```text
/var/log/auth.log
```

This file contains authentication-related events, including SSH login attempts and authentication failures.

The second configuration monitors:

```text
/var/log/syslog
```

This provides broader system-level event information generated by the Linux operating system.

The purpose of adding these log sources was to make authentication and system activities visible to the Wazuh Manager.

### Kali Telemetry Evidence

![Kali Telemetry](https://github.com/DifaTech/Wazuh-SIEM-Threat-Detection/blob/main/images/setup/kali%20telemetry.png))

---

# 8. Verifying Agent Registration

After configuring the Wazuh agents, the Wazuh Dashboard was used to verify that the endpoints successfully connected to the Manager.

The expected state for the agents was:

```text
Status: Active
```

The successful Active status confirmed that:

1. The Wazuh agents were registered.
2. The agents could communicate with the Wazuh Manager.
3. The Manager was receiving telemetry from the endpoints.

The project report confirms that both Windows and Kali agents became active and continuously forwarded logs to the Manager.

![Proof of Setup](https://github.com/DifaTech/Wazuh-SIEM-Threat-Detection/blob/main/images/setup/Real%20proof%20of%20setup.png)

---

# 9. Verifying Log Ingestion

Agent registration alone was not enough. The next step was to verify that actual security events were reaching the Wazuh Manager.

The Wazuh Dashboard was used to inspect events received from both endpoints.

## Windows Log Ingestion

Windows System, Security, and Application Event Logs were successfully received and parsed by Wazuh.

This confirmed that the Windows endpoint was successfully sending telemetry to the SIEM.

## Kali Linux Log Ingestion

The Kali endpoint provided authentication and system log information through the configured:

```bash
/var/log/auth.log
/var/log/syslog
```

This provided visibility into Linux authentication activity, system processes, and administrative activities.

The successful ingestion from both operating systems demonstrated that the SIEM was functioning as a centralized monitoring platform.

---

# 10. SSH Brute-Force Simulation

## 10.1 Objective

The first security simulation was an SSH authentication attack.

The purpose was to generate multiple failed SSH authentication attempts and determine whether Wazuh could detect the resulting authentication events.

The simulation was performed from Kali Linux against the Ubuntu Wazuh Manager.

Target:

```text
192.168.56.103
```

---

## 10.2 Attack Command

The following Hydra command was used:

```bash
hydra -l fakeuser -p "pass1,pass2,pass3,pass4,pass5" ssh://192.168.56.103
```

The command generated multiple invalid authentication attempts against the SSH service.

This simulated an attacker repeatedly attempting to authenticate using an invalid username and multiple passwords.

### Command Evidence

![Brute Force Command](../images/ssh-detection/Brute-force%20command.png)

---

## 10.3 Wazuh Detection

The failed authentication attempts generated events that were collected and analyzed by Wazuh.

The relevant Wazuh detections observed during the practical exercise included:

```text
Rule ID 5710
sshd: Attempt to login using a non-existent user

Rule ID 5503
PAM: User login failed
```

These alerts demonstrated that the authentication failures generated by the simulation were visible through the SIEM.

### Detection Evidence

![Brute Force Attack](../images/ssh-detection/Brute-force%20attack.png)

---

## 10.4 MITRE ATT&CK Mapping

The SSH authentication attack was mapped to:

```text
T1110 — Brute Force
```

The technique describes attempts to gain access by repeatedly guessing authentication credentials.

---

# 11. Unauthorized Administrator Account Creation

## 11.1 Objective

The second simulation focused on unauthorized account creation and modification of local administrator privileges.

A local Windows account named:

```text
hackeradmin
```

was created on the Windows 10 endpoint.

The account was then added to the local Administrators group.

The purpose was to simulate a situation where an attacker creates an additional privileged account after gaining access to a Windows system.

---

## 11.2 Account Creation

The following command was used to create the local account:

```cmd
net user hackeradmin Password123! /add
```

The account was then added to the local Administrators group:

```cmd
net localgroup Administrators hackeradmin /add
```

### Command Evidence

![Account Creation Command](../images/account-creation/account%20creation%20command.png)

---

## 11.3 Wazuh Detection

The Windows actions generated Security Event Logs.

The Windows Wazuh Agent forwarded the events to the Wazuh Manager, where they were analyzed and matched against Wazuh detection rules.

The observed alerts included:

```text
Rule ID 60109
User account enabled or created
Level: 8
```

and:

```text
Rule ID 60154
Administrators Group Changed
Level: 12
```

The second alert was particularly important because it showed that the newly created account had been added to the local Administrators group.

### Detection Evidence

![Account Creation Attack](../images/account-creation/account%20creation%20attack.png)

---

## 11.4 Raw Log Analysis

The raw Wazuh event data was also reviewed to understand the information contained within the alert.

The raw event provided additional details about the detected activity and the affected Windows endpoint.

### Raw Log Evidence

![Raw Log Snippet](../images/account-creation/Raw%20log%20snippet.png)

---

## 11.5 MITRE ATT&CK Mapping

The account creation activity was mapped to:

```text
T1136 — Create Account
T1136.001 — Local Account
```

The activity was associated with the **Persistence** tactic because unauthorized accounts can be created to maintain access to a compromised system.

The administrator group modification was also analyzed as account manipulation activity.

---

# 12. Windows Security Event Log Clearing

## 12.1 Objective

The third simulation focused on defense evasion.

The Windows Security Event Log was cleared to simulate an attacker attempting to remove evidence of activity from the compromised system.

---

## 12.2 Command Used

The following Windows command was executed:

```cmd
wevtutil cl Security
```

The command clears the Windows Security Event Log.

This was performed only within the isolated laboratory environment.

### Command Evidence

![Clearing Windows Command](../images/log-clearing/Clearing%20windows%20command.png)

---

## 12.3 Wazuh Detection

The log-clearing activity generated a Windows security event.

Wazuh collected the resulting event and generated the following alert:

```text
Rule ID 63103
The audit log was cleared
Level: 5
```

This demonstrated an important SIEM capability: an attempt to remove security evidence can itself generate a detectable event.

### Detection Evidence

![Clearing Windows Attack](../images/log-clearing/Clearing%20windows%20attack.png)

---

## 12.4 MITRE ATT&CK Mapping

The activity was mapped to:

```text
T1070.001 — Clear Windows Event Logs
```

This technique falls under the **Indicator Removal** category and is associated with **Defense Evasion**.

---

# 13. Alert Investigation

After generating the simulated security events, the Wazuh Dashboard was used to investigate the resulting alerts.

The investigation focused on:

- Alert description
- Rule ID
- Alert level
- Affected endpoint
- Event information
- User involved
- Timestamp
- Detection source
- MITRE ATT&CK mapping

This process helped demonstrate how a SOC analyst can move from a raw security event to an understandable security finding.

---

# 14. Detection Summary

| Activity | Wazuh Rule | Level | Detection |
|---|---:|---:|---|
| SSH authentication failure | `5710` | 5 | `sshd: Attempt to login using a non-existent user` |
| SSH authentication failure | `5503` | 5 | `PAM: User login failed` |
| Local account creation | `60109` | 8 | `User account enabled or created` |
| Administrator group modification | `60154` | 12 | `Administrators Group Changed` |
| Windows Security Log clearing | `63103` | 5 | `The audit log was cleared` |

---

# 15. MITRE ATT&CK Mapping

| Observed Activity | Technique | Technique Name | Tactic |
|---|---|---|---|
| SSH authentication attack | `T1110` | Brute Force | Credential Access |
| Local account creation | `T1136` | Create Account | Persistence |
| Local account creation | `T1136.001` | Local Account | Persistence |
| Administrator group modification | `T1098` | Account Manipulation | Persistence |
| Windows Security Event Log clearing | `T1070.001` | Clear Windows Event Logs | Defense Evasion |

The MITRE ATT&CK framework was used to provide a standardized way of describing the security activities observed during the laboratory exercises.

---

# 16. Incident Timeline

The account creation simulation can be summarized as follows:

| Step | Activity |
|---:|---|
| 1 | Wazuh agents connected successfully to the Wazuh Manager |
| 2 | `hackeradmin` was created on the Windows endpoint |
| 3 | `hackeradmin` was added to the local Administrators group |
| 4 | Windows generated Security Event Logs |
| 5 | The Wazuh Agent forwarded the events to the Manager |
| 6 | Wazuh analyzed the incoming events |
| 7 | Wazuh generated security alerts |
| 8 | The alerts were reviewed through the Wazuh Dashboard |

---

# 17. Key Findings

The practical deployment produced several important observations.

### Finding 1 — Centralized Monitoring

The Wazuh Manager successfully collected security telemetry from both Windows and Kali Linux.

This provided a single location for reviewing events from different operating systems.

### Finding 2 — Account Activity Detection

The creation of a new Windows account and its addition to the Administrators group generated separate Wazuh alerts.

This showed how multiple related events can provide more context during an investigation.

### Finding 3 — Authentication Monitoring

The SSH simulation generated authentication failures that were collected and detected by Wazuh.

### Finding 4 — Detection of Log Tampering

Clearing the Windows Security Event Log generated an event that Wazuh detected.

This is important because attackers may attempt to remove local evidence after carrying out malicious activities.

---

# 18. Security Recommendations

Based on the activities observed during the laboratory exercise, the following defensive measures are recommended:

- Apply the principle of least privilege.
- Regularly review local administrator accounts.
- Enable Multi-Factor Authentication for privileged accounts where applicable.
- Implement account lockout policies to reduce brute-force attempts.
- Continuously monitor Windows Security Event Logs.
- Maintain centralized logging so that security events remain available even if local logs are deleted.
- Keep operating systems and security software updated.
- Conduct regular vulnerability assessments.
- Restrict administrative tools to authorized users.
- Perform periodic security audits.

---

# 19. What This Project Demonstrated

This project provided practical experience in building a SIEM environment from the ground up.

The major skills demonstrated include:

- Virtual machine provisioning
- VirtualBox Host-Only networking
- Ubuntu Server deployment
- Wazuh Manager deployment
- Wazuh Dashboard deployment
- Wazuh Agent installation and registration
- Windows endpoint monitoring
- Linux endpoint monitoring
- Linux log-source configuration
- Centralized log collection
- Security alert investigation
- SSH authentication attack simulation
- Windows account activity monitoring
- Detection of Windows Security Event Log clearing
- MITRE ATT&CK mapping
- Basic SOC-style incident analysis

---

# 20. Conclusion

The laboratory successfully demonstrated the deployment and operation of a Wazuh-based SIEM environment using three virtual machines.

Ubuntu Server operated as the central Wazuh Manager and Dashboard, while Windows 10 and Kali Linux served as monitored endpoints.

The successful registration of both agents and the subsequent ingestion of Windows and Linux logs confirmed that the centralized monitoring environment was functioning correctly.

Controlled security simulations were then performed to validate the detection capability of the SIEM. These included an SSH authentication attack, unauthorized local administrator account creation, and Windows Security Event Log clearing.

Wazuh successfully generated alerts for the simulated activities, allowing the events to be investigated and mapped to relevant MITRE ATT&CK techniques.

Overall, the project provided hands-on experience with SIEM deployment, endpoint monitoring, centralized logging, security alert analysis, attack detection, and basic incident response in a controlled SOC-style laboratory.

---

## ⚠️ Disclaimer

All security simulations documented in this project were performed within an isolated virtual laboratory created for educational purposes.

No unauthorized external systems or real-world networks were targeted.
