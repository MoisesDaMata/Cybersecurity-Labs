# Cybersecurity Labs — Moises da Mata

![Wazuh](https://img.shields.io/badge/Wazuh-v4.x-blue?style=flat-square&logo=wazuh&logoColor=white)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-red?style=flat-square)
![Windows](https://img.shields.io/badge/Platform-Windows_10-0078D6?style=flat-square&logo=windows&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

---

## 🇺🇸 English

### Overview

Hands-on cybersecurity labs simulating a full attack kill chain and detecting each technique using **Wazuh SIEM** on a Windows 10 endpoint.

Each lab covers a real-world attacker technique mapped to **MITRE ATT&CK**, documented with attack simulation steps, Wazuh detection evidence, and key IOCs for SOC analysts.

### Lab Environment

| Machine | Role | OS |
|---|---|---|
| Kali Linux | Attacker | Kali Linux |
| Windows 10 | Target + Wazuh Agent | Windows 10 |
| Wazuh Server | SIEM | Ubuntu Server |

### Labs

| # | Lab | Tactic | Technique | Detection |
|---|---|---|---|---|
| 01 | [Brute Force Detection](./lab-01-bruteforce-T1110/) | Credential Access | T1110 | Event ID 4625 |
| 02 | [Encoded PowerShell](./lab-02-encoded-powershell-T1059/) | Execution | T1059.001 | Event ID 4688 |
| 03 | [Network Enumeration](./lab-03-network-enumeration-T1049/) | Discovery | T1049 | Event ID 4688 |
| 04 | [Pass-the-Hash](./lab-04-pass-the-hash-T1550/) | Lateral Movement | T1550.002 | Event ID 4624 |
| 05 | [Privilege Escalation](./lab-05-privilege-escalation-T1078/) | Privilege Escalation | T1078.003 | Event IDs 4720, 4732 |
| 06 | [PowerShell Download Cradle](./lab-06-download-cradle-T1105/) | Command & Control | T1105 | Event ID 4688 |
| 07 | [Scheduled Task Creation](./lab-07-scheduled-task-T1053/) | Persistence | T1053 | Event ID 4688 |
| 08 | [Disable Windows Defender](./lab-08-disable-defender-T1562/) | Defense Evasion | T1562.001 | Event ID 4688 |

### Prerequisites

- Wazuh v4.x (Manager + Agent configured)
- Windows 10 with Wazuh Agent installed
- Kali Linux (attacker machine)
- Process Creation auditing enabled (Event ID 4688 + Command Line logging)

---

## 🇧🇷 Português

### Visão Geral

Labs práticos de cibersegurança simulando uma kill chain completa e detectando cada técnica com **Wazuh SIEM** em um endpoint Windows 10.

Cada lab cobre uma técnica real de ataque mapeada ao **MITRE ATT&CK**, documentada com passo a passo da simulação, evidências de detecção no Wazuh e principais IOCs para analistas SOC.

### Ambiente do Lab

| Máquina | Papel | SO |
|---|---|---|
| Kali Linux | Atacante | Kali Linux |
| Windows 10 | Alvo + Wazuh Agent | Windows 10 |
| Wazuh Server | SIEM | Ubuntu Server |

### Labs

| # | Lab | Tática | Técnica | Detecção |
|---|---|---|---|---|
| 01 | [Brute Force Detection](./lab-01-bruteforce-T1110/) | Acesso a Credenciais | T1110 | Event ID 4625 |
| 02 | [Encoded PowerShell](./lab-02-encoded-powershell-T1059/) | Execução | T1059.001 | Event ID 4688 |
| 03 | [Network Enumeration](./lab-03-network-enumeration-T1049/) | Descoberta | T1049 | Event ID 4688 |
| 04 | [Pass-the-Hash](./lab-04-pass-the-hash-T1550/) | Movimento Lateral | T1550.002 | Event ID 4624 |
| 05 | [Privilege Escalation](./lab-05-privilege-escalation-T1078/) | Escalada de Privilégios | T1078.003 | Event IDs 4720, 4732 |
| 06 | [PowerShell Download Cradle](./lab-06-download-cradle-T1105/) | Comando e Controle | T1105 | Event ID 4688 |
| 07 | [Scheduled Task Creation](./lab-07-scheduled-task-T1053/) | Persistência | T1053 | Event ID 4688 |
| 08 | [Disable Windows Defender](./lab-08-disable-defender-T1562/) | Evasão de Defesa | T1562.001 | Event ID 4688 |

### Pré-requisitos

- Wazuh v4.x (Manager + Agent configurados)
- Windows 10 com Wazuh Agent instalado
- Kali Linux (máquina atacante)
- Auditoria de criação de processos habilitada (Event ID 4688 + Command Line logging)

---

## Repository Structure

```
Cybersecurity-Labs/
├── README.md
├── lab-01-bruteforce-T1110/
├── lab-02-encoded-powershell-T1059/
├── lab-03-network-enumeration-T1049/
├── lab-04-pass-the-hash-T1550/
├── lab-05-privilege-escalation-T1078/
├── lab-06-download-cradle-T1105/
├── lab-07-scheduled-task-T1053/
└── lab-08-disable-defender-T1562/
```

---

## Connect

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/moisesdamata/)

*Developed by Moises da Mata*
