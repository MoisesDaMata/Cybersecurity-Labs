# Lab 03 — Network Enumeration Detection (T1049)

## MITRE ATT&CK
- **Tactic:** Discovery
- **Technique:** T1049 — System Network Connections Discovery
- **Detection:** Windows Security Event ID 4688 (Process Creation)

---

## Objective

Simulate network enumeration commands executed by an attacker after compromising a Windows machine, and detect the behavior through Wazuh SIEM by monitoring Event ID 4688.

---

## Lab Environment

| Machine | Role | IP |
|---|---|---|
| Kali Linux | Attacker | 192.168.56.104 |
| Windows 10 | Target | 192.168.56.103 |
| Wazuh Server | SIEM | — |

---

## Technical Background

After compromising a system, attackers run reconnaissance commands to map the local network — identifying active hosts, established connections, and open sessions. This phase is critical for planning lateral movement.

In this lab, the attacker executes native Windows commands to collect network information. Wazuh captures each process via Event ID 4688, recording the process name and command-line arguments.

---

## Attack Walkthrough

### 1. Execute enumeration commands on Windows

On Windows 10, open `cmd.exe` and run:

```cmd
arp -a
query user
net session
```

**What each command reveals:**

| Command | Information collected |
|---|---|
| `arp -a` | ARP table — active hosts on the local network |
| `query user` | Active user sessions on the system |
| `net session` | Open network sessions with other hosts |

---

### 2. Detection in Wazuh — Event ID 4688

In the Wazuh Dashboard, the events were captured using the filter:

```
data.win.eventdata.commandLine: *arp*
```

Wazuh logged the `arp.exe` process being launched with the `-a` argument, confirming the execution of the enumeration command.

![Wazuh ARP Event](images/16-wazuh-arp-event.png)

---

### 3. Event field analysis

With the event expanded in Wazuh, the fields confirm the execution:

| Field | Value |
|---|---|
| `eventID` | `4688` |
| `newProcessName` | `C:\Windows\System32\arp.exe` |
| `commandLine` | `arp -a` |
| `parentProcessName` | `C:\Windows\System32\cmd.exe` |
| `subjectUserName` | `target` |

![Wazuh Event Details](images/17-additional-event.png)

---

## Attack Sequence

![Attack Sequence](images/15-attack-sequence.png)

---

## Indicators of Compromise (IOCs)

| Indicator | Value |
|---|---|
| Processes | `arp.exe`, `query.exe`, `net.exe` |
| Arguments | `-a`, `user`, `session` |
| Parent process | `cmd.exe` |
| Event ID | `4688` |

---

## What the SOC Should Look For

- Sequential execution of multiple reconnaissance commands in a short timeframe
- `arp.exe`, `net.exe`, `query.exe` launched from `cmd.exe` or `powershell.exe`
- Enumeration commands executed by standard user accounts
- Discovery pattern followed by lateral movement connection attempts

---

## File Structure

```
lab-03-network-enumeration-T1049/
├── README.md
└── images/
    ├── 15-attack-sequence.png
    ├── 16-wazuh-arp-event.png
    └── 17-additional-event.png
```

---

## References

- [MITRE ATT&CK T1049 — System Network Connections Discovery](https://attack.mitre.org/techniques/T1049/)
- [Windows Event ID 4688 — Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)
- [Wazuh Documentation](https://documentation.wazuh.com)
