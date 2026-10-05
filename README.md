# 🚩 Offensive Security Journey & Practical Labs

[![GitHub last commit](https://img.shields.io/github/last-commit/PedroHenriqueBRO/CTF-s-and-Labs?style=flat-square&color=crimson)](https://github.com/PedroHenriqueBRO/CTF-s-and-Labs)
[![Hack The Box](https://img.shields.io/badge/HackTheBox-Certified%20Junior%20Cybersecurity%20Analyst-brightgreen?style=flat-square&logo=hackthebox)](https://academy.hackthebox.com/)
[![TryHackMe](https://img.shields.io/badge/TryHackMe-Pathways%20Completed-red?style=flat-square&logo=tryhackme)](https://tryhackme.com/)
[![Study Time](https://img.shields.io/badge/Dedication-4h%2Fday-blueviolet?style=flat-square)]()
[![Writeups](https://img.shields.io/badge/Writeups-36-orange?style=flat-square)]()

Repositório de documentação prática da transição de **Software Developer** para **Cybersecurity / Offensive Security (Red Team)**.

Cada pasta é um CTF ou lab resolvido e documentado — da enumeração à pós-exploração — com anotações, comandos e lições aprendidas. A ordem numerada abaixo (#1 → #36) reflete a sequência cronológica dos commits: serve para acompanhar a **evolução da documentação** e da **metodologia de pentest**, não só o conteúdo técnico.

---

## 🎯 Rotina e Foco

* **Carga horária:** pelo menos **4h/dia** (teoria + labs + máquinas).
* **Objetivo atual:** trilha **CJCA (Certified Junior Cybersecurity Analyst)** no Hack The Box Academy.
* **Método:** protocolo → enumeração manual → exploração → documentar o que funcionou e o que travou.

---

## 🧠 O que já consolidei na prática

### Reconhecimento & Network Enumeration
* Varreduras TCP/UDP com Nmap (`-sS`, `-sV`, `-sC`, `-sA`, `-sU`, `-O`), leitura de portas *filtered* vs *open*.
* Evasão de IDS/IPS (ex.: `--source-port 53`), fingerprinting sob restrição e NSE scripts.
* Enumeração de DNS (`dig`), NetBIOS e mapeamento de superfície antes de atacar.

### Footprinting de serviços
* **SMB/Samba:** `smbclient`, null sessions, shares, RPC/RID.
* **NFS:** `showmount`, montagem de exports e caça a credenciais em arquivos.
* **SMTP / POP3 / IMAP:** open relay, banners, exfiltração de e-mail e chaves privadas.
* **SNMP:** `snmpwalk`, community strings e scripts de gestão vazando senhas.
* **MySQL / MSSQL / Oracle TNS:** login, queries, SIDs, `impacket-mssqlclient`, módulos MSF.
* **IPMI:** dump de hash RAKP + crack com Hashcat (modo 7300).

### Web & Application Attacks
* Dir/vhost enum com Gobuster/ffuf; leitura de `info.php` e headers inseguros.
* **LFI → RCE** (wrappers `php://filter`, `file://`, Apache/SSH log poisoning + Burp User-Agent).
* File upload bypass (ex.: `arquivo.php .jpg`), RFI controlado via servidor local.
* **SQLi** manual: UNION, auth bypass, boolean-blind e time-based (`SLEEP`); dump e abuso de tabelas.
* **SSRF** para rotas internas, port scan em localhost e recuperação de credenciais/admin APIs.
* **JWT / MFA:** forgery de claims, reset de senha frágil e bypass de segundo fator.
* **SSTI** em templates; mass assignment (`isAdmin`); XSS sem HttpOnly + CSRF contra bots admin.
* WordPress: WPScan, temas indexáveis, webshell via `404.php`, Metasploit multi/handler.
* LFD em APIs/configs, sessão frágil (MD5) e escalada de conta web (user → admin).

### Privilege Escalation & Pós-exploração
* Linux: SUID, cronjobs, PATH hijacking, escrita em scripts de outros usuários, `sudo -u` / `sudo find`, GTFOBins (`less`), wildcard injection em `tar`.
* Windows: Meterpreter, dump NTLM, exploração de serviços expostos (ex.: FortiLogger).
* KeePass → `keepass2john` → John/Hashcat; reutilização de senhas entre serviços.
* Cadeias multi-usuário (ex.: Jump: `recon → dev → monitor → ops → root`).

### Frameworks & tooling
* Metasploit (exploits remotos/locais, sessions, Baron Samedit / sudo).
* Impacket, Hydra (quando faz sentido), Hashcat, John, Burp Suite, CyberChef, ffuf, BloodHound.

---

## 📈 Evolução da jornada (ordem cronológica)

Cada entrada é um writeup commitado. O número **#N / 36** é a posição na linha do tempo — útil para comparar como a escrita e o raciocínio de pentest mudaram do primeiro CTF ao mais recente.

| Fase | Writeups | Foco |
|------|----------|------|
| Base THM | #1–#2 | Máquinas completas, narrativa livre |
| HTB Academy — Nmap & serviços | #3–#15 | Enumeração sistemática por protocolo |
| Footprinting integrado | #16–#18 | Encadear vários serviços até a flag |
| Exploração & MSF | #19–#21 | WordPress + Metasploit (Linux/Windows) |
| Web + Privesc documentados | #22–#25 | SQLi, LFD/API, relatório mais estruturado |
| Active Directory | #26–#27 | Coerção NTLM, delegação Kerberos e RBCD |
| Relatórios THM (cadeia completa) | #28–#32 | Portal → AD/DMZ → SSRF/cron → sudo |
| AppSec avançado | #33–#36 | JWT/MFA, SSTI, SSRF+LFI, XSS+CSRF |

---

## 📂 Estrutura das pastas

A organização separa a plataforma (**HackTheBox** vs **TryHackMe**) e, no THM, cada sala em sua pasta. Nos labs da Academy HTB, os writeups ficam agrupados em `Labs/` porque são módulos curtos da mesma trilha.

```text
.
├── README.md                              # Este arquivo — mapa da jornada e skills
├── HackTheBox/
│   └── Labs/                              # Trilha Academy (Nmap, Footprinting, MSF, WP…)
│       ├── 03-Lab NMAP Fácil.md
│       ├── 04-Lab NMAP Médio.md
│       ├── 05-Lab NMAP Difícil.md
│       ├── 06-MiniLabSmb.md
│       ├── 07-MiniLabNFS.md
│       ├── 08-MiniLabDns.md
│       ├── 09-MiniLabSMTP.md
│       ├── 10-MiniLabPop3Imap.md
│       ├── 11-MinilabSNMP.md
│       ├── 12-MiniLabMySql.md
│       ├── 13-MiniLabMSSQL.md
│       ├── 14-MiniLabOracleTNS.md
│       ├── 15-MiniLabIPMI.md
│       ├── 16-FootPrintingLabEasy.md
│       ├── 17-FootPrintingLabMedium.md
│       ├── 18-FootPrintingLabHard.md
│       ├── 19-LabWordpress.md
│       ├── 20-MiniLabMSFconsole.md
│       └── 21-MiniLabMSFConsoleV2.0.md
│
└── TryHackMe/
    ├── Archangel/01-Archangel.md          # LFI / log poison / PATH hijack
    ├── Opacity/02-Opacity.md              # upload bypass / KeePass / root
    ├── minilabsPath/
    │   └── 22-SQLInjectionLab.md          # SQLi manual (4 vetores)
    ├── Recrutamento/23-Recrutamento.md    # file:// + SQLi → admin
    ├── Support/24-Support.md              # LFD / sessão / relatório estruturado
    ├── jump/25-Jump.md                    # privesc multi-usuário encadeado
    ├── Proxy/26-Proxy.md                  # AD / coerção NTLM / constrained delegation
    ├── Forward/27-Forward.md              # lateral / KeePass / RBCD
    ├── Domino/28-Domino.md                # wordlist login / devops cron → root
    ├── SilentMonitor/29-SilentMonitor.md  # SQLi / RCE / KeePass → root
    ├── DeadDrop/30-DeadDrop.md            # AD / DMZ pivot / Domain Admin
    ├── OperationPromotion/31-OperationPromotion.md  # RCE web / sudo find
    ├── OperationColdStart/32-OperationColdStart.md  # SSRF / tar wildcard → root
    ├── Hammer/33-Hammer.md                # reset frágil / JWT forgery / MFA
    ├── Injectics/34-Injectics.md          # SQLi / SSTI → www-data
    ├── Include/35-Include.md              # mass assignment / SSRF / LFI log poison
    └── WhatsYourName/36-WhatsYourName.md  # XSS cookie theft / CSRF admin bot
```

**Por que essa estrutura?**  
Plataforma → contexto (lab de módulo vs sala completa) → um Markdown por desafio. Os arquivos usam o prefixo `NN-` (mesmo id da linha do tempo `#N / 36`), amarrando nome do writeup à ordem real de execução e deixando visível a progressão de “anotações soltas” (#1–#2) para template Objetivo/Ferramentas (#12+) e, depois, relatório de invasão com cadeia de ataque (#24–#36).
