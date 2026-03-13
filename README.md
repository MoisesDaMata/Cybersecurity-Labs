# Cybersecurity Labs - Moises Franco

## 🇺🇸 English

Hands-on cybersecurity labs focused on threat detection with Wazuh and PowerShell.

### Prerequisites

- Wazuh v4.x (Manager + Agent configured)
- Windows 10/11 or Windows Server 2019/2022 (for agent)
- PowerShell 5.1+
- Python 3.8+ (optional, for auxiliary scripts)

### Available Labs

| Lab | Description | Status |
|-----|-------------|--------|
| [bruteforce-detection](./bruteforce-detection/) | Brute force attack detection rules | ✅ Completed |
| [suspicious-powershell](./suspicious-powershell/) | Encoded command monitoring | ✅ Completed |
| [pass-the-hash](./pass-the-hash/) | MITRE ATT&CK T1550.002 implementation | ✅ Completed |

### Repository Structure

```
cybersecurity-labs/
├── bruteforce-detection/
├── suspicious-powershell/
├── pass-the-hash/
└── README.md
```

### Roadmap

- [ ] Privilege Escalation Detection (T1068)
- [ ] Lateral Movement Monitoring (T1021)
- [ ] Ransomware Behavior Detection

---

## 🇧🇷 Português

Labs práticos de cibersegurança com detecção de ameaças via Wazuh e PowerShell.

### Pré-requisitos

- Wazuh v4.x (Manager + Agent configurados)
- Windows 10/11 ou Windows Server 2019/2022 (para o agente)
- PowerShell 5.1+
- Python 3.8+ (opcional, para scripts auxiliares)

### Labs Disponíveis

| Lab | Descrição | Status |
|-----|-----------|--------|
| [bruteforce-detection](./bruteforce-detection/) | Regras contra ataques de força bruta | ✅ Concluído |
| [suspicious-powershell](./suspicious-powershell/) | Detecção de comandos codificados | ✅ Concluído |
| [pass-the-hash](./pass-the-hash/) | Detecção MITRE T1550.002 | ✅ Concluído |

### Estrutura do Repositório

```
cybersecurity-labs/
├── bruteforce-detection/
├── suspicious-powershell/
├── pass-the-hash/
└── README.md
```

### Roadmap

- [ ] Detecção de Escalada de Privilégios (T1068)
- [ ] Monitoramento de Movimento Lateral (T1021)
- [ ] Detecção de Comportamento de Ransomware

---

![Wazuh v4.x](https://img.shields.io/badge/Wazuh-v4.x-blue)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-red)
![GitHub Labs](https://img.shields.io/badge/GitHub-Labs-black?logo=github)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)
