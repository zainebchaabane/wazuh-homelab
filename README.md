# Wazuh Homelab

## Overview
This repository documents a homelab built with Wazuh to monitor Windows 10 endpoint activity, simulate RDP brute-force attempt, and observe how Wazuh detects it.It configures also  both Wazuh capabilities File Integrity Monitoring (FIM) and automated threat mitigation via Active Response.

---

## Lab Architecture

This lab was inspired by the following [tutorial](https://youtu.be/KTAqe2fJmsE)
## Lab Architecture
![Lab Architecture](/screenshots/wazuh-lab.png)

This lab was hosted in Oracle VM VirtualBox.
| Component | Version | IP Address|
|---|---|---|
| Wazuh Server | Pre-built [OVA](https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html) — 4.14.9 | 192.168.10/24 |
| Endpoint | Windows 10 Version 22H2  | 192.168.13/24 |
| Kali Linux | 2026.3 |192.168.11/24 | 
## Steps 
### 1. Environment Setup 
- Configure static IP addresses within the 192.168.10.0/24 network
- Install the Wazuh Agent on the Windows 10 VM linking it to the Wazuh Manager
![](/screenshots/wazuh-agent.png)
- Confirm connectivity 
### 2. Windows Authentication Monitoring
Create a Windows user named TestUser using PowerShell with administrator privileges.
``` 
net user TestUser "Password123" /add
``` 
### 3. RDP Brute-Force Simulation
Generated a RDP brute force attack from Kali Linux using Hydra Windows 10 target's Remote Desktop service on user TestUser.
```
hydra -V -l TestUser -P passwords.txt rdp://192.168.10.13
``` 
## Results

### Scenario 1: Baseline Windows Authentication
Perform manual sign-ins with TestUser, monitoring Event ID 4624 (Successful Logon) and Event ID 4625 (Failed Logon).
### Scenario 2: RDP Brute-Force Detection
- Wazuh collected Windows 10 endpoint telemetry showing repeated authentication failures identified by **Event ID 4625** (Failed log on )
![](/screenshots/result-1.png)
The event details captured:
     * **Target User:** `TestUser`
     * **Source Workstation:** `kali`
     * **Source IP:** `192.168.10.11`
     * **Triggered Alert:** `Rule 60204` (Multiple Windows Logon Failures)
- Wazuh automatically mapped the correlated event telemetry to the MITRE ATT&CK framework under : 
     * **Tactic:** Credential Access 
     * **Technique:** Brute Force (`T1110`)
![](/screenshots/result-2.png)
![](/screenshots/result-3.png)

## Recommended Mitigations( MITRE ATT&CK )
-**Account Lockout Policy**: Configure appropriate account lockout thresholds and durations to limit repeated password-guessing attempts.
-**Multi-Factor Authentication**
### File Integrity Monitoring (FIM)
Wazuh tracked real-time modifications in `C:\Secret` direcotry on Windows 10 VM.
### Active Response
Configured Wazuh to automatically mitigate  RDP attack by isolating the attacker's source IP address upon alert. 
![](/screenshots/active-response.png)

