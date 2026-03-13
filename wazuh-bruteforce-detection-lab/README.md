# Wazuh SIEM Lab: Brute Force Detection and Investigation
![Platform](https://img.shields.io/badge/Platform-Windows%2010-blue)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-green)
![Attack%20Type](https://img.shields.io/badge/Attack-Brute%20Force-red)
![MITRE](https://img.shields.io/badge/MITRE-T1110-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

Status: Completed | Attack Successfully Detected | Incident Investigated | MITRE ATT&CK Mapped
## Executive Summary

Simulated a real-world RDP brute force attack against a Windows system and detected it using Wazuh SIEM correlation rules.

Attack generated multiple Event ID 4625 logs which triggered Wazuh rule 18107 (Level 10 – High Severity), mapped to MITRE ATT&CK T1110.

The incident was investigated following SOC triage procedures and validated as a confirmed brute force attempt.
## Objective

Demonstrate hands-on detection and investigation of a brute force attack using Wazuh SIEM in a controlled lab environment, replicating real SOC operational workflow.
## MITRE ATT&CK Mapping

- Technique: T1110 – Brute Force  
- Tactic: Credential Access

This technique is commonly used by attackers to gain unauthorized access through repeated authentication attempts.

Official reference: https://attack.mitre.org/techniques/T1110/
## Lab Architecture

The lab environment consisted of three virtual machines:

| Machine        | Role                              | IP Address       | OS            |
|---------------|-----------------------------------|------------------|--------------|
| Wazuh Server  | SIEM                              | 192.168.56.102   | Ubuntu Server |
| Windows Target| Victim Machine (Wazuh Agent)      | 192.168.56.103   | Windows 10    |
| Kali Linux    | Attacker Machine                  | 192.168.56.101   | Kali Linux    |
## Network Architecture Diagram

```
               [ Kali Linux ]
        Attacker - 192.168.56.101
                    |
                    | Attack traffic
                    v
             [ Windows Target ]
         Victim - 192.168.56.103
             (Wazuh Agent)
                    |
                    | Log forwarding
                    v
             [ Wazuh Server ]
          SIEM - 192.168.56.102
           (Detection & Alerts)
```

## Attack flow:

Kali Linux → Windows Target → Wazuh SIEM detection
## Log Flow

Windows Target → Wazuh Agent → Wazuh Server → Alert generated
## Tools Used

- Wazuh SIEM (Security Information and Event Management)
  
- Kali Linux (attacker machine)
  
- Hydra (brute force attack tool)
  
- Windows Event Viewer (log analysis)
  
- VirtualBox (lab virtualization environment)

## Attack Simulation
A brute-force attack was completed using _Hydra_ from the Kali Linux system and the Windows Remote Desktop Protocol (RDP) service was the target.


Attack command executed from attacker machine:

```bash
hydra -l Administrator -P rockyou.txt rdp://192.168.56.103
```

This command will generate multiple variations of passwords against the Administrator account with the _Rockyou.txt_ word list.


This simulation replicates a real-world brute-force attack scenario in a controlled lab environment.
## Detection in Wazuh SIEM

The Wazuh Agent installed on the Windows machine collected logs and forwarded them to the Wazuh Server.

Wazuh analyzed the logs using its detection rules and generated alerts based on suspicious activity.

**Alert characteristics observed:**

- Multiple failed login attempts in short time period

- Same source IP address

- Same target account

- Security Event Log correlation

- High severity alert level
### Detection Rule Details

Wazuh triggered rule ID 18107, which detects multiple Windows authentication failures.

Rule logic summary:

- Monitors Windows Security Event ID 4625
- Correlates multiple failed login attempts
- Identifies repeated failures from same source
- Triggers alert when threshold exceeded

Rule classification:

- Rule ID: 18107
- Level: 10 (High Severity)
- Framework: MITRE ATT&CK
- Technique: T1110 – Brute Force
## Alert Analysis

The Wazuh alert provided key forensic information required for incident investigation.

Key fields analyzed:

- Rule ID: 18107
- Rule Level: 10 (High Severity)
- Agent Name: Windows Target
- Source IP Address: 192.168.56.101
- Target Account: Administrator
- Event ID: 4625
- Log Source: Windows Security Event Log

Analysis findings:

- Multiple authentication failures originated from a single source IP
- Failures occurred within a very short timeframe
- Target account was a privileged account (Administrator)
- Pattern matches brute force attack behavior

Conclusion:

The alert represents a confirmed brute force attack attempt against a Windows system.
**Example Wazuh alert details:**

Rule ID: **_18107_**  
Rule Level: **_10_**  
Rule Description: **_Multiple Windows login failures detected_**  
MITRE Technique: **_T1110 – Brute Force_** 

This rule is triggered when multiple failed login attempts are detected within a short timeframe, indicating potential brute force activity.

**Source IP identified in alert:**

> 192.168.56.101 (Kali Linux attacker machine)

**Target account:**

> Administrator

This confirms the attack originated from the attacker machine and targeted the Windows system.

**Relevant Windows Event ID:**

Event ID **4625** — Failed login attempt
## Detection workflow

1. Attack executed from Kali Linux
2. Windows generated Event ID 4625 (failed authentication)
3. Wazuh Agent collected event
4. Log forwarded to Wazuh Server
5. Correlation rule 18107 triggered
6. High-severity alert generated in dashboard
## Screenshots Overview

The following evidence demonstrates the detection and investigation process:

- Wazuh SIEM alerts triggered by brute force attack
- Alert details showing source IP and rule ID
- Windows Event Viewer logs confirming Event ID 4625
## Evidence

### Wazuh Alert Showing Failed Login Attempts

![Wazuh Alert](Brute_Force_evidence/wazuh_alert.png)
![Wazuh Alert](Brute_Force_evidence/wazuh_alert_details.png)

These alerts show multiple failed login attempts detected by Wazuh from the attacker machine.

---

### Windows Event Viewer Showing Event ID 4625

![Windows Event Logs](Brute_Force_evidence/Windows_Event_logs.png)
![Windows Event Logs](Brute_Force_evidence/Windows_Event_logs_2.png)

These logs confirm authentication failures generated during the brute force attack.

## Investigation Process

The investigation followed standard SOC analyst workflow:

Step 1 - Alert Identification
Wazuh SIEM generated alerts indicating multiple failed login attempts.

Step 2 - Log Analysis
Analysis of Windows Security Logs confirmed Event ID **_4625_**.

Step 3 - Source Identification
Source IP address was identified as:

**_192.168.56.101 (Kali Linux attacker machine)_**

Step 4 - Pattern Confirmation
Multiple login failures within a short timeframe confirmed brute force activity.

Step 5 - Threat Classification
Attack mapped to MITRE ATT&CK technique:

**_T1110 – Brute Force_**
## SOC Investigation Questions Answered

During the investigation, the following key questions were addressed:

- What happened?
  > Multiple failed authentication attempts detected.

- When did it happen?
  > During the attack simulation timeline.

- Where did it originate?
  > Source IP: 192.168.56.101 (Kali Linux attacker machine)

- What was targeted?
  > Windows Administrator account via RDP.

- Was the attack successful?
  > No. All authentication attempts failed.

- What is the severity?
  > High severity due to attack pattern targeting privileged account.

- What is the MITRE ATT&CK classification?
  > T1110 – Brute Force
## Timeline of Events

| Time       | Event |
|------------|------|
| 13:59:21 | First failed login detected (Event ID 4625) |
| 13:59:22 | Multiple failed login attempts detected |
| 13:59:23 | Wazuh correlation rule triggered (Rule ID 18107) |
| 13:59:24 | Alert generated in SIEM dashboard |
| 13:59:30 | Investigation initiated |
| 14:02:00 | Incident classified as brute force attack |
  
This timeline demonstrates how SIEM enables rapid detection and investigation of malicious activity.
## Security Impact

If this were a real environment, this attack could result in:

- Unauthorized access

- Privilege escalation

- Lateral movement

- Data exfiltration

- Full system compromise

Early detection by SIEM is critical to prevent escalation.
## Relevance to Real-World SOC Operations

Brute force attacks are one of the most common attack techniques observed in enterprise environments.

SOC analysts must be able to:

- Identify authentication attack patterns
- Analyze SIEM alerts
- Correlate logs from multiple sources
- Identify attacker IP addresses
- Determine attack severity
- Recommend mitigation actions

This lab demonstrates hands-on experience performing these tasks using a real SIEM platform.
## False Positive Consideration

During analysis, false positive scenarios were considered.

Legitimate causes of multiple failed logins may include:

- User entering incorrect password multiple times
- Misconfigured services using outdated credentials
- Automated scripts attempting authentication

However, before confirming the alert as malicious, additional validation steps were performed.

Validation included checking for successful login events (Event ID 4624) to confirm no account compromise occurred.

No successful authentication events were observed.

The alert was confirmed as malicious due to:

- High frequency of login attempts
- Originating from attacker machine (Kali Linux)
- Use of brute force tool (Hydra)
- Clear attack simulation context
## Recommended Mitigation

The following mitigation actions are recommended to prevent or reduce brute force attacks:

- Block the attacker IP address at the firewall
- Enable account lockout policy after multiple failed login attempts
- Enable Multi-Factor Authentication (MFA)
- Restrict RDP access to trusted IP addresses only
- Monitor authentication logs continuously using SIEM

These measures significantly reduce the risk of unauthorized access.
## Skills Demonstrated

This project demonstrates practical SOC analyst skills including:

- SIEM monitoring and alert analysis

- Brute force attack detection

- Windows Event Log analysis

- Security incident investigation

- Threat detection using Wazuh

- MITRE ATT&CK framework mapping

- Log correlation and threat identification
  
- Security event correlation
  
- SIEM rule analysis and alert validation
  
- Alert triage and validation

- Understanding of authentication log behavior
  
- Basic detection engineering concepts
## Conclusion

This lab demonstrates how authentication-based attacks generate detectable patterns in Windows logs and how SIEM correlation rules transform raw log data into actionable security alerts.

The investigation process validated the alert, ruled out false positives, and confirmed malicious activity aligned with MITRE ATT&CK T1110.

This project reflects practical SOC-level experience in detection, triage, log analysis, and incident validation within a realistic monitoring environment.
## Author

**Moises da Mata**  
Junior SOC Analyst | Cybersecurity Enthusiast  

LinkedIn: https://www.linkedin.com/in/moisesdamata/  
GitHub: https://github.com/MoisesDaMata  

This project is part of my cybersecurity portfolio demonstrating hands-on SIEM monitoring, threat detection, and incident investigation skills aligned with real-world SOC operations.




