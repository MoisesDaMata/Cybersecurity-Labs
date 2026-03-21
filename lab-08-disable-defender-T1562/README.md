# Lab 08 — Disable Windows Defender Detection (T1562)

## MITRE ATT&CK
- **Tactic:** Defense Evasion
- **Technique:** T1562.001 — Impair Defenses: Disable or Modify Tools
- **Detection:** Windows Security Event ID 4688 (Process Creation)

---

## Objetivo

Simular a desativação do Windows Defender via linha de comando, técnica usada por atacantes para remover a principal camada de proteção do endpoint antes de executar ferramentas ou payloads maliciosos, e detectar o comportamento através do Wazuh SIEM.

---

## Ambiente do Lab

| Máquina | Role | IP |
|---|---|---|
| Windows 10 | Target | 192.168.56.103 |
| Wazuh Server | SIEM | — |

---

## Explicação Técnica

Antes de executar ferramentas ofensivas em um sistema comprometido, atacantes frequentemente desativam ou modificam soluções de segurança para evitar detecção. A desativação do Windows Defender via `Set-MpPreference` é uma das técnicas mais documentadas e utilizadas por malwares, ransomware e grupos APT.

Neste lab, o atacante:
1. Abre o `cmd.exe` com privilégios de Administrador
2. Chama o `powershell.exe` explicitamente como processo filho, passando o comando de desativação via `-Command`
3. O Wazuh captura o Event ID 4688 com o `commandLine` completo, expondo a tentativa de evasão

A escolha de invocar o PowerShell a partir do `cmd.exe` é intencional — garante a criação de um novo processo filho, gerando o evento 4688 com toda a cadeia de execução visível: processo pai (`cmd.exe`) → processo filho (`powershell.exe`) → argumento (`Set-MpPreference -DisableRealtimeMonitoring $true`).

---

## Passo a Passo do Ataque

### 1. Desabilitar o Windows Defender via cmd.exe

No Windows 10, com `cmd.exe` aberto como Administrador:

```cmd
powershell.exe -ExecutionPolicy Bypass -Command "Set-MpPreference -DisableRealtimeMonitoring $true"
```

O comando retorna sem erro — o Defender está desativado.

![Defender Disabled](images/01-defender-disabled.png)

---

### 2. Detecção no Wazuh — Event ID 4688

No Wazuh Dashboard, o evento foi localizado com o filtro:

```
data.win.eventdata.commandLine: *Set-MpPreference*
```

O Wazuh retornou **1 hit** — o processo `powershell.exe` iniciado pelo `cmd.exe` no agente `DESKTOP-8LF9GEI`, rule ID `67027`, às `Mar 20, 2026 @ 19:56:57`.

![Wazuh 4688 Detection](images/02-wazuh-4688-defender.png)

---

### 3. Análise dos campos do evento

Com o evento expandido no Wazuh, os campos confirmam toda a cadeia de execução:

| Campo | Valor |
|---|---|
| `eventID` | `4688` |
| `newProcessName` | `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` |
| `parentProcessName` | `C:\Windows\System32\cmd.exe` |
| `commandLine` | `powershell.exe -ExecutionPolicy Bypass -Command "Set-MpPreference -DisableRealtimeMonitoring $true"` |
| `subjectUserName` | `target` |
| `subjectDomainName` | `DESKTOP-8LF9GEI` |

![Wazuh Event Details](images/03-wazuh-event-details.png)

---

## Indicadores de Comprometimento (IOCs)

| Indicador | Valor |
|---|---|
| Processo | `powershell.exe` |
| Processo pai | `cmd.exe` |
| Comando | `Set-MpPreference -DisableRealtimeMonitoring $true` |
| Flag suspeita | `-ExecutionPolicy Bypass` |
| Event ID | `4688` |

---

## O que o SOC deve observar

- `powershell.exe` iniciado a partir de `cmd.exe` com `-ExecutionPolicy Bypass`
- `commandLine` contendo `Set-MpPreference` combinado com `-Disable*`
- Qualquer modificação nas preferências do Defender via linha de comando
- Processos filhos do `cmd.exe` ou `powershell.exe` executando cmdlets de segurança
- Sequência suspeita: desativação do Defender seguida de download ou execução de novo processo

---

## Cleanup

```cmd
powershell.exe -ExecutionPolicy Bypass -Command "Set-MpPreference -DisableRealtimeMonitoring $false"
```

---

## Estrutura de Arquivos

```
lab-08-t1562-disable-defender/
├── README.md
└── images/
    ├── 01-defender-disabled.png
    ├── 02-wazuh-4688-defender.png
    └── 03-wazuh-event-details.png
```

---

## Referências

- [MITRE ATT&CK T1562.001 — Impair Defenses](https://attack.mitre.org/techniques/T1562/001/)
- [Windows Event ID 4688 — Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)
- [Wazuh Documentation](https://documentation.wazuh.com)
