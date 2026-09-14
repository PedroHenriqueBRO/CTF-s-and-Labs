# 🚩 Offensive Security Journey & Practical Labs

[![GitHub last commit](https://img.shields.io/github/last-commit/PedroHenriqueBRO/CTF-s-and-Labs?style=flat-square&color=crimson)](https://github.com/PedroHenriqueBRO/CTF-s-and-Labs)
[![Hack The Box](https://img.shields.io/badge/HackTheBox-Certified%20Junior%20Cybersecurity%20Analyst-brightgreen?style=flat-square&logo=hackthebox)](https://academy.hackthebox.com/)
[![TryHackMe](https://img.shields.io/badge/TryHackMe-Pathways%20Completed-red?style=flat-square&logo=tryhackme)](https://tryhackme.com/)
[![Study Time](https://img.shields.io/badge/Dedication-4h%2Fday-blueviolet?style=flat-square)]()
[![Writeups](https://img.shields.io/badge/Writeups-25-orange?style=flat-square)]()

Repositório de documentação prática da transição de **Software Developer** para **Cybersecurity / Offensive Security (Red Team)**.

Cada pasta é um CTF ou lab resolvido e documentado — da enumeração à pós-exploração — com anotações, comandos e lições aprendidas. A ordem numerada abaixo (#1 → #25) reflete a sequência cronológica dos commits: serve para acompanhar a **evolução da documentação** e da **metodologia de pentest**, não só o conteúdo técnico.

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
* **LFI → RCE** (wrappers `php://filter`, `file://`, Apache log poisoning + Burp User-Agent).
* File upload bypass (ex.: `arquivo.php .jpg`), RFI controlado via servidor local.
* **SQLi** manual: UNION, auth bypass, boolean-blind e time-based (`SLEEP`).
* WordPress: WPScan, temas indexáveis, webshell via `404.php`, Metasploit multi/handler.
* LFD em APIs/configs, sessão frágil (MD5) e escalada de conta web (user → admin).

### Privilege Escalation & Pós-exploração
* Linux: SUID, cronjobs, PATH hijacking, escrita em scripts de outros usuários, `sudo -u`, GTFOBins (`less`).
* Windows: Meterpreter, dump NTLM, exploração de serviços expostos (ex.: FortiLogger).
* KeePass → `keepass2john` → John; reutilização de senhas entre serviços.
* Cadeias multi-usuário (ex.: Jump: `recon → dev → monitor → ops → root`).

### Frameworks & tooling
* Metasploit (exploits remotos/locais, sessions, Baron Samedit / sudo).
* Impacket, Hydra (quando faz sentido), Hashcat, John, Burp Suite, CyberChef.

---

## 📈 Evolução da jornada (ordem cronológica)

Cada entrada é um writeup commitado. O número **#N / 25** é a posição na linha do tempo — útil para comparar como a escrita e o raciocínio de pentest mudaram do primeiro CTF ao mais recente.

| Fase | Writeups | Foco |
|------|----------|------|
| Base THM | #1–#2 | Máquinas completas, narrativa livre |
| HTB Academy — Nmap & serviços | #3–#15 | Enumeração sistemática por protocolo |
| Footprinting integrado | #16–#18 | Encadear vários serviços até a flag |
| Exploração & MSF | #19–#21 | WordPress + Metasploit (Linux/Windows) |
| Web + Privesc documentados | #22–#25 | SQLi, LFD/API, relatório mais estruturado |

### Linha do tempo dos writeups

#### `#1 / 25` — 2026-05-09 14:54:47 -0300
**[Archangel](TryHackMe/Archangel/01-Archangel.md)** · TryHackMe  
Nmap + Gobuster, LFI com bypass de path, `php://filter`, Apache log poisoning → reverse shell, cronjob e PATH hijacking até root.

#### `#2 / 25` — 2026-05-12 15:16:43 -0300
**[Opacity](TryHackMe/Opacity/02-Opacity.md)** · TryHackMe  
Enum SMB/web, upload bypass (`.php .jpg`), KeePass cracking (John), privesc via script/lib sobrescrita como root.

#### `#3 / 25` — 2026-08-25 09:56:21 -0300
**[Lab NMAP Fácil](HackTheBox/Labs/03-Lab%20NMAP%20Fácil.md)** · HackTheBox  
Scanning “quieto” sob IDS/IPS; contraste entre `-sS -O` e `-sA` quando o fingerprint mente.

#### `#4 / 25` — 2026-08-25 09:56:30 -0300
**[Lab NMAP Médio](HackTheBox/Labs/04-Lab%20NMAP%20Médio.md)** · HackTheBox  
Portas filtradas (DNS/SMB), version detection com source-port e scripts NSE em NetBIOS.

#### `#5 / 25` — 2026-08-25 09:56:41 -0300
**[Lab NMAP Difícil](HackTheBox/Labs/05-Lab%20NMAP%20Difícil.md)** · HackTheBox  
Superfície reduzida; ACK inútil; UDP + NBNS revelando hostname e serviços escondidos.

#### `#6 / 25` — 2026-08-25 17:04:48 -0300
**[MiniLab SMB](HackTheBox/Labs/06-MiniLabSmb.md)** · HackTheBox  
Samba 4 / SMB2, `smbclient -L`, shares e baseline de enum Windows/Linux file sharing.

#### `#7 / 25` — 2026-08-25 18:40:16 -0300
**[MiniLab NFS](HackTheBox/Labs/07-MiniLabNFS.md)** · HackTheBox  
rpcbind + NFS (111/2049), listagem e montagem de exports.

#### `#8 / 25` — 2026-08-25 23:43:36 -0300
**[MiniLab DNS](HackTheBox/Labs/08-MiniLabDns.md)** · HackTheBox  
`dig` (ANY e tipos específicos) contra domínio interno HTB.

#### `#9 / 25` — 2026-08-26 10:27:44 -0300
**[MiniLab SMTP](HackTheBox/Labs/09-MiniLabSMTP.md)** · HackTheBox  
Scripts `smtp*` do Nmap; identificação de open relay.

#### `#10 / 25` — 2026-08-26 22:48:39 -0300
**[MiniLab POP3 / IMAP](HackTheBox/Labs/10-MiniLabPop3Imap.md)** · HackTheBox  
Enum 110/143/993/995 com `-sV -sC` e leitura de serviços de e-mail.

#### `#11 / 25` — 2026-08-26 23:54:41 -0300
**[MiniLab SNMP](HackTheBox/Labs/11-MinilabSNMP.md)** · HackTheBox  
`snmpwalk` por OIDs e descoberta de scripts/processos via management plane.

#### `#12 / 25` — 2026-08-27 12:03:43 -0300
**[MiniLab MySQL](HackTheBox/Labs/12-MiniLabMySql.md)** · HackTheBox  
Primeiro writeup no formato Objetivo → Ferramentas → Resultados; MySQL 8 + credenciais fracas.

#### `#13 / 25` — 2026-08-27 13:21:26 -0300
**[MiniLab MSSQL](HackTheBox/Labs/13-MiniLabMSSQL.md)** · HackTheBox  
Hostname via enum; `impacket-mssqlclient` e banco não padrão.

#### `#14 / 25` — 2026-08-27 22:49:04 -0300
**[MiniLab Oracle TNS](HackTheBox/Labs/14-MiniLabOracleTNS.md)** · HackTheBox  
SID XE, brute via MSF `oracle_login`, sysdba e hash de usuário (DBSNMP).

#### `#15 / 25` — 2026-08-28 11:16:09 -0300
**[MiniLab IPMI](HackTheBox/Labs/15-MiniLabIPMI.md)** · HackTheBox  
UDP 623, dump RAKP no Metasploit, crack Hashcat → `admin:trinity`.

#### `#16 / 25` — 2026-08-28 15:07:01 -0300
**[FootPrinting Lab Easy](HackTheBox/Labs/16-FootPrintingLabEasy.md)** · HackTheBox  
Lab integrado: encadear o que os mini-labs ensinaram em um alvo único.

#### `#17 / 25` — 2026-08-28 17:11:47 -0300
**[FootPrinting Lab Medium](HackTheBox/Labs/17-FootPrintingLabMedium.md)** · HackTheBox  
NFS → tickets/creds → SMB/RDP/MSSQL; reutilização de senha `sa`/admin.

#### `#18 / 25` — 2026-08-29 11:51:29 -0300
**[FootPrinting Lab Hard](HackTheBox/Labs/18-FootPrintingLabHard.md)** · HackTheBox  
SNMP vazando user/senha → IMAP (chave privada) → SSH → `users.sql` como root.

#### `#19 / 25` — 2026-08-31 18:56:22 -0300
**[Lab WordPress](HackTheBox/Labs/19-LabWordpress.md)** · HackTheBox  
Vhost pelo domínio do e-mail, WPScan/ExploitDB, webshell em tema, handler MSF.

#### `#20 / 25` — 2026-09-01 23:29:02 -0300
**[MiniLab MSFconsole](HackTheBox/Labs/20-MiniLabMSFconsole.md)** · HackTheBox  
Exploit remoto → `www-data` → privesc local Baron Samedit (sudo) → root.

#### `#21 / 25` — 2026-09-02 10:45:52 -0300
**[MiniLab MSFconsole V2.0](HackTheBox/Labs/21-MiniLabMSFConsoleV2.0.md)** · HackTheBox  
Alvo Windows (FortiLogger etc.), shell SYSTEM e dump do NTLM do `htb-student`.

#### `#22 / 25` — 2026-09-06 11:34:11 -0300
**[SQL Injection Lab](TryHackMe/minilabsPath/22-SQLInjectionLab.md)** · TryHackMe  
UNION, auth bypass, boolean-blind e time-based — tudo manual, sem sqlmap.

#### `#23 / 25` — 2026-09-07 10:50:30 -0300
**[Recrutamento](TryHackMe/Recrutamento/23-Recrutamento.md)** · TryHackMe  
DNS/vhost, `file://` para leitura local, SQLi com `#` vs `--` até admin.

#### `#24 / 25` — 2026-09-09 23:47:11 -0300
**[Support](TryHackMe/Support/24-Support.md)** · TryHackMe  
Relatório no formato de invasão: brute force → LFD (`api.php`/`config.php`) → admin; reflexão explícita sobre organização do writeup.

#### `#25 / 25` — 2026-09-14 12:57:44 -0300
**[Jump](TryHackMe/jump/25-Jump.md)** · TryHackMe  
Escalada encadeada por scripts/FTP anonymous/`sudo -u`/`less`; tabela de credenciais + retrospectiva do que travou.

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
    └── jump/25-Jump.md                    # privesc multi-usuário encadeado
```

**Por que essa estrutura?**  
Plataforma → contexto (lab de módulo vs sala completa) → um Markdown por desafio. Os arquivos usam o prefixo `NN-` (mesmo id da linha do tempo `#N / 25`), amarrando nome do writeup à ordem real de execução e deixando visível a progressão de “anotações soltas” (#1–#2) para template Objetivo/Ferramentas (#12+) e, depois, relatório de invasão com cadeia de ataque (#24–#25).
