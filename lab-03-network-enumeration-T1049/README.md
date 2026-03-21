# Lab 03 — Network Enumeration Detection (T1049)

## MITRE ATT&CK
- **Tactic:** Discovery
- **Technique:** T1049 — System Network Connections Discovery
- **Detection:** Windows Security Event ID 4688 (Process Creation)

---

## Objetivo

Simular comandos de enumeração de rede executados por um atacante após comprometer uma máquina Windows, e detectar o comportamento através do Wazuh SIEM monitorando o Event ID 4688.

---

## Ambiente do Lab

| Máquina | Role | IP |
|---|---|---|
| Kali Linux | Attacker | 192.168.56.104 |
| Windows 10 | Target | 192.168.56.103 |
| Wazuh Server | SIEM | — |

---

## Explicação Técnica

Após comprometer um sistema, atacantes executam comandos de reconhecimento para mapear a rede local — identificando hosts ativos, conexões estabelecidas e sessões abertas. Essa fase é crítica para planejar movimentação lateral.

Neste lab, o atacante executa comandos nativos do Windows para coletar informações de rede. O Wazuh captura cada processo via Event ID 4688, registrando o nome do processo e os argumentos da linha de comando.

---

## Passo a Passo do Ataque

### 1. Executar comandos de enumeração no Windows

No Windows 10, abrir o `cmd.exe` e executar:

```cmd
arp -a
query user
net session
```

**O que cada comando revela:**

| Comando | Informação coletada |
|---|---|
| `arp -a` | Tabela ARP — hosts ativos na rede local |
| `query user` | Sessões de usuário ativas no sistema |
| `net session` | Sessões de rede abertas com outros hosts |

---

### 2. Detecção no Wazuh — Event ID 4688

No Wazuh Dashboard, os eventos foram capturados com o filtro:

```
data.win.eventdata.commandLine: *arp*
```

O Wazuh registrou o processo `arp.exe` sendo iniciado com o argumento `-a`, confirmando a execução do comando de enumeração.

![Wazuh ARP Event](images/16-wazuh-arp-event.png)

---

### 3. Análise dos campos do evento

Com o evento expandido no Wazuh, os campos confirmam a execução:

| Campo | Valor |
|---|---|
| `eventID` | `4688` |
| `newProcessName` | `C:\Windows\System32\arp.exe` |
| `commandLine` | `arp -a` |
| `parentProcessName` | `C:\Windows\System32\cmd.exe` |
| `subjectUserName` | `target` |

![Wazuh Event Details](images/17-additional-event.png)

---

## Sequência do Ataque

![Attack Sequence](images/15-attack-sequence.png)

---

## Indicadores de Comprometimento (IOCs)

| Indicador | Valor |
|---|---|
| Processos | `arp.exe`, `query.exe`, `net.exe` |
| Argumentos | `-a`, `user`, `session` |
| Processo pai | `cmd.exe` |
| Event ID | `4688` |

---

## O que o SOC deve observar

- Execução sequencial de múltiplos comandos de reconhecimento em curto intervalo
- `arp.exe`, `net.exe`, `query.exe` iniciados a partir de `cmd.exe` ou `powershell.exe`
- Comandos de enumeração executados por contas de usuário padrão
- Padrão de discovery seguido de tentativas de conexão lateral

---

## Estrutura de Arquivos

```
lab-03-network-enumeration-T1049/
├── README.md
└── images/
    ├── 15-attack-sequence.png
    ├── 16-wazuh-arp-event.png
    └── 17-additional-event.png
```

---

## Referências

- [MITRE ATT&CK T1049 — System Network Connections Discovery](https://attack.mitre.org/techniques/T1049/)
- [Windows Event ID 4688 — Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)
- [Wazuh Documentation](https://documentation.wazuh.com)
