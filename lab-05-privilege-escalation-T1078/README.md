# Lab 05 — Privilege Escalation Detection (T1078)

## MITRE ATT&CK
- **Tactic:** Privilege Escalation
- **Technique:** T1078.003 — Valid Accounts: Local Accounts
- **Detection:** Windows Security Event IDs 4720, 4732

---

## Objetivo

Simular uma escalada de privilégios via conta local válida, técnica usada por atacantes para obter acesso administrativo sem explorar vulnerabilidades, e detectar o comportamento através do Wazuh SIEM monitorando eventos de gerenciamento de contas do Windows.

---

## Ambiente do Lab

| Máquina | Role | IP |
|---|---|---|
| Windows 10 | Target | 192.168.56.103 |
| Wazuh Server | SIEM | — |

---

## Explicação Técnica

Diferente de exploits, essa técnica abusa de funcionalidades legítimas do sistema operacional. O atacante cria uma conta de usuário padrão e em seguida a adiciona ao grupo local de Administradores — obtendo privilégios máximos sem acionar detecções baseadas em exploração.

Os dois eventos críticos gerados são:
- **Event ID 4720** — nova conta de usuário criada
- **Event ID 4732** — conta adicionada ao grupo local de Administradores

A correlação entre os dois eventos pelo mesmo SID confirma a escalada de privilégios.

---

## Passo a Passo do Ataque

### 1. Criar a conta e escalar privilégios

No Windows 10, com `cmd.exe` aberto como Administrador:

```powershell
# Criar usuário padrão
net user attacker Password123! /add

# Adicionar ao grupo Administradores
net localgroup Administrators attacker /add

# Confirmar membros do grupo
net localgroup Administrators
```

![User Creation](images/event_4720.png)

---

### 2. Confirmar adição ao grupo Administradores

```powershell
net localgroup Administrators
```

![Administrator Group Addition](images/administrator_group_addition.png)

---

### 3. Detecção no Wazuh — Event IDs 4720 e 4732

O Wazuh capturou os dois eventos em sequência:

**Event ID 4720 — Conta criada:**

| Campo | Valor |
|---|---|
| `eventID` | `4720` |
| `newTargetUserName` | `attacker` |
| `subjectUserName` | `target` |

**Event ID 4732 — Adicionada ao grupo Administradores:**

| Campo | Valor |
|---|---|
| `eventID` | `4732` |
| `memberSid` | SID correspondente à conta `attacker` |
| `targetUserName` | `Administrators` |
| `subjectUserName` | `target` |

![Wazuh Event 4732](images/event_4732.png)

---

## Correlação por SID

A chave da detecção é correlacionar o SID entre os dois eventos:

```
Event 4720 — Conta criada
  Usuário: attacker
  SID:     S-1-5-21-...-XXXX

Event 4732 — Adicionada ao grupo Administradores
  Member SID: S-1-5-21-...-XXXX

→ O mesmo SID confirma que a conta recém-criada foi
  imediatamente escalada para Administradores.
```

---

## Indicadores de Comprometimento (IOCs)

| Indicador | Valor |
|---|---|
| Event IDs | `4720`, `4732` |
| Conta criada | `attacker` |
| Grupo alvo | `Administrators` |
| Comando | `net user /add` + `net localgroup Administrators /add` |

---

## O que o SOC deve observar

- Event ID 4720 seguido rapidamente de Event ID 4732 para o mesmo SID
- Contas adicionadas ao grupo Administradores fora do horário comercial
- Criação de contas com nomes genéricos ou suspeitos
- `net.exe` sendo usado para gerenciamento de grupos por usuários não administrativos
- Event ID 4672 (privilégios especiais) logo após o logon da nova conta

---

## Estrutura de Arquivos

```
lab-05-privilege-escalation-T1078/
├── README.md
└── images/
    ├── event_4720.png
    ├── administrator_group_addition.png
    └── event_4732.png
```

---

## Referências

- [MITRE ATT&CK T1078.003 — Valid Accounts: Local Accounts](https://attack.mitre.org/techniques/T1078/003/)
- [Windows Event ID 4720 — User Account Created](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4720)
- [Windows Event ID 4732 — Member Added to Security Group](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4732)
- [Wazuh Documentation](https://documentation.wazuh.com)
