# Tópico

# Tags

#Hackthebox 

# Resumo
Foi dado no final da seção 6 perguntas quanto a enumerar e explorar um servidor smb rodando na máquina alvo, iremos começar agora pelo nmap para enumerar as portas e serviços desse servidor e achar as do smb.
## Nmap
``` 
sudo nmap -sS 10.129.174.162          
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-25 15:38 -0400
Nmap scan report for 10.129.174.162
Host is up (0.24s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2049/tcp open  nfs

Nmap done: 1 IP address (1 host up) scanned in 10.16 seconds
```
Aqui vemos que está rodando o netbios-ssn na porta 139 e o microsoft-sd na porta 445 (smb mais novo que atua na 445).
Como podemos obter várias informações quanto a essas portas utilizando o nmap , vamos utilizar ele de primeira mão para ver ate onde conseguimos responder as perguntas da seção.
``` 
sudo nmap -sS -sV -sC 10.129.174.162 -p 139,445
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-25 15:40 -0400
Nmap scan report for 10.129.174.162
Host is up (0.32s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4

Host script results:
| smb2-time: 
|   date: 2026-08-25T19:40:56
|_  start_date: N/A
|_nbstat: NetBIOS name: DEVSMB, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.00 seconds

```
Não recebemos quase nada de informação útil , somente que tem o smb-security-mode na versão 3.1.1 e a versão do samba é a Samba smbd 4 , logo respondemos a primeira pergunta.

## Smbclient
Depois utilizei o smblclient -L para listar as partilhas no alvo e responder a pergunta 2 que queria saber qual a partilha disponível no sistema alvo.
``` 
smbclient -L 10.129.174.162
Password for [WORKGROUP\kali]:

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        sambashare      Disk      InFreight SMB v3.1
        IPC$            IPC       IPC Service (InlaneFreight SMB server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.
smbXcli_negprot_smb1_done: No compatible protocol selected by server.
Protocol negotiation to server 10.129.174.162 (for a protocol between LANMAN1 and NT1) failed: NT_STATUS_INVALID_NETWORK_RESPONSE
Unable to connect with SMB1 -- no workgroup available
```
sambashare era a resposta .
Depois disso foi requerido da gente acesar o compartilhamento sambashare e encontrassemos o arquivo flag.txt , nisso eu acessei o compartilhamento e achei
``` 
smbclient '//10.129.174.162/sambashare'                  
Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Nov  8 08:43:14 2021
  ..                                  D        0  Mon Nov  8 10:53:19 2021
  .profile                            H      807  Tue Feb 25 07:03:22 2020
  contents                            D        0  Mon Nov  8 08:43:45 2021
  .bash_logout                        H      220  Tue Feb 25 07:03:22 2020
  .bashrc                             H     3771  Tue Feb 25 07:03:22 2020

                4062912 blocks of size 1024. 414232 blocks available
smb: \> cd contents\
smb: \contents\> ls
  .                                   D        0  Mon Nov  8 08:43:45 2021
  ..                                  D        0  Mon Nov  8 08:43:14 2021
  flag.txt                            N       38  Mon Nov  8 08:43:45 2021

                4062912 blocks of size 1024. 414232 blocks available
```
Depois dei get flag.txt e na minha máquina dei cat no flag para responder a pergunta com o conteúdo de flag.txt . 

## rpcclient
Utilizei agora o rpc client para buscar informações do server , domains e querydominfo, assim respondendo a próxima pergunta de qual dominio o server pertence.
``` 
rpcclient -U "" 10.129.174.162
Password for [WORKGROUP\]:
rpcclient $> svrinfo
command not found: svrinfo
rpcclient $> srvinfo
        DEVSMB         Wk Sv PrQ Unx NT SNT InlaneFreight SMB server (Samba, Ubuntu)
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03
rpcclient $> enumdomains
name:[DEVSMB] idx:[0x0]
name:[Builtin] idx:[0x1]
rpcclient $> querydominfo
Domain:         DEVOPS
Server:         DEVSMB
Comment:        InlaneFreight SMB server (Samba, Ubuntu)
Total Users:    0
Total Groups:   0
Total Aliases:  0
Sequence No:    1787687737
Force Logoff:   4294967295
Domain Server State:    0x1
Server Role:    ROLE_DOMAIN_PDC
Unknown 3:      0x1

```
Aqui vemos que o server é o DEVSMB e o domínio dele é o DEVOPS . 
Depois utilizei o netshareenumall para responder a próxima pergunta que era sobre a versão em específico desse compartilhamento (*sambashare*).
``` 
rpcclient $> netshareenumall
netname: print$
        remark: Printer Drivers
        path:   C:\var\lib\samba\printers
        password:
netname: sambashare
        remark: InFreight SMB v3.1
        path:   C:\home\sambauser\
        password:
netname: IPC$
        remark: IPC Service (InlaneFreight SMB server (Samba, Ubuntu))
        path:   C:\tmp
        password:
```
Por fim pediu para mostrar informações específicas do path dessa partilha , nisso utilizei o comando netsharegetinfo sambashare.
``` 
netsharegetinfo sambashare
netname: sambashare
        remark: InFreight SMB v3.1
        path:   C:\home\sambauser\
        password:
        type:   0x0
        perms:  0
        max_uses:       -1
        num_uses:       1
revision: 1
type: 0x8004: SEC_DESC_DACL_PRESENT SEC_DESC_SELF_RELATIVE 
DACL
        ACL     Num ACEs:       1       revision:       2
        ---
        ACE
                type: ACCESS ALLOWED (0) flags: 0x00 
                Specific bits: 0x1ff
                Permissions: 0x1f01ff: SYNCHRONIZE_ACCESS WRITE_OWNER_ACCESS WRITE_DAC_ACCESS READ_CONTROL_ACCESS DELETE_ACCESS 
                SID: S-1-1-0

```
O path foi respondido em forma /home/sambauser . 




