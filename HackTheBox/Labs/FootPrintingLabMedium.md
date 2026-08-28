## Alvo
- IP -> 10.129.202.41

## Objetivo
- Esse lab diz que existe um servidor nessa máquina que todos na rede interna tem acesso, na qual esse tipo de servidor normalmente é um dos mais escolhidos para ser invadidos, irei enumerar e estudar os serviços para ver qual serviço se trata. -> Nesse caso era o nfs , que foi o estopim para o restantes das enumerações/explorações.
- Enumere o servidor com cuidado e encontre o nome de usuário "HTB" e sua senha. Em seguida, envie a senha deste usuário como resposta. -> lnch7ehrdn43i7AoqVPK4zWR

## Ferramentas e scripts utilizados
- nmap
- showmount
- mount
- rpcclient
- smbclient
- xfreerdp

## Resultados obtidos
```
sudo nmap -sS 10.129.202.41    
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-28 14:08 -0400
Nmap scan report for 10.129.202.41
Host is up (0.83s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE SERVICE
111/tcp  open  rpcbind
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2049/tcp open  nfs
3389/tcp open  ms-wbt-server
5985/tcp open  wsman

Nmap done: 1 IP address (1 host up) scanned in 8.21 seconds
```
Aqui vemos que temos duas versões do samba rodando , uma sendo a netbios porta 139 e a cifs na porta 445. Além de um network file system na 2049, um rpcbind rodando na porta 111 , um wmi rodando na porta 135 , um rdp na porta 3389 e um winrm rodando na 5985.

--- 
```
sudo nmap -sV -sC 10.129.202.41
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-28 14:23 -0400
Nmap scan report for 10.129.202.41
Host is up (0.46s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
111/tcp  open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
2049/tcp open  nlockmgr      1-4 (RPC #100021)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=WINMEDIUM
| Not valid before: 2026-08-27T18:05:13
|_Not valid after:  2027-02-26T18:05:13
|_ssl-date: 2026-08-28T18:25:18+00:00; +1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: WINMEDIUM
|   NetBIOS_Domain_Name: WINMEDIUM
|   NetBIOS_Computer_Name: WINMEDIUM
|   DNS_Domain_Name: WINMEDIUM
|   DNS_Computer_Name: WINMEDIUM
|   Product_Version: 10.0.17763
|_  System_Time: 2026-08-28T18:25:10+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-08-28T18:25:10
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 146.94 seconds
```
Aqui rodamos scripts para saber mais informações sobre o alvo , descobrimos que o sistema alvo é windows , o smb utiliza versão 3.1.1 e o rpcbind nos retornou os protocolos que os serviços de nfs suportam e vemos que a porta 2049 se apresentou como nlockmgr , assim mudando nossa visão sobre o que realmente estava rodando naquela porta.

Não descobri o suficiente de cada um , entao vou rodar script\* para cada um e tentar obter informações melhores.

---
*MSRPC* Porta 135
Não conseguir rodar scripts nessa porta devido a algum IDS/IPS , drop de pacotes pelo firewall ou algo do tipo.

---
*SMB* netbios 139 e cifs 445
```
sudo nmap -sS --script smb* -p 139,445 10.129.202.41 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-28 14:40 -0400
Nmap scan report for 10.129.202.41
Host is up (0.31s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
|_smb-enum-services: ERROR: Script execution failed (use -d to debug)
445/tcp open  microsoft-ds
|_smb-enum-services: ERROR: Script execution failed (use -d to debug)

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_smb-vuln-ms10-061: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
| smb2-time: 
|   date: 2026-08-28T18:40:26
|_  start_date: N/A
|_smb-flood: ERROR: Script execution failed (use -d to debug)
| smb-mbenum: 
|_  ERROR: Failed to connect to browser service: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
|_smb-print-text: false
| smb2-capabilities: 
|   2.0.2: 
|     Distributed File System
|   2.1: 
|     Distributed File System
|     Leasing
|     Multi-credit operations
|   3.0: 
|     Distributed File System
|     Leasing
|     Multi-credit operations
|   3.0.2: 
|     Distributed File System
|     Leasing
|     Multi-credit operations
|   3.1.1: 
|     Distributed File System
|     Leasing
|_    Multi-credit operations
|_smb-vuln-ms10-054: false
| smb-protocols: 
|   dialects: 
|     2.0.2
|     2.1
|     3.0
|     3.0.2
|_    3.1.1

Nmap done: 1 IP address (1 host up) scanned in 67.10 seconds

```
Recebemos poucas informações úteis , não mostrou vulnerabilidades disponíveis mas mostrou as versoes de protocolos smb que aceitam.

---
*NFS* porta 2049
Novamente não aceitou scripts do nmap , logo vou testar com alguns comandos.
```
showmount -e 10.129.202.41                          
Export list for 10.129.202.41:
/TechSupport (everyone)
```
Aqui vemos uma pasta que pode ser acessada por qualquer um, podendo ser uma grande falha de configuração permitir isso, a não ser que não tenha informações sensíveis.

--- 
*SMB* 139 e 445 manual
```
(kali㉿kali)-[~]
└─$ smbclient -L 10.129.202.41 -p 445 -U ''      
Password for [WORKGROUP\]:
                                                                                                                      
┌──(kali㉿kali)-[~]
└─$ smbclient -L 10.129.202.41 -p 139 -U ''      
do_connect: Connection to 10.129.202.41 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)

smbclient -L 10.129.202.41 -p 445 -U 'guest' 
Password for [WORKGROUP\guest]:
session setup failed: NT_STATUS_ACCOUNT_DISABLED


```
Não aceitam login anonimo e a 139 esta dando mesmo colocando o nome do servidor.

--- 
*NFS* 2049
```
sudo mount -t nfs 10.129.202.41:/TechSupport ./mountalvo -o nolock
```
Fiz a montagem do alvo para a minha máquina na pasta mountalvo e dei ls -alh onde no meio de 99% de arquivos de ticket vazios havia esse abaixo que havia conteúdo, nisso eu dei cat nele.
```
sudo cat mountalvo/ticket4238791283782.txt
Conversation with InlaneFreight Ltd

Started on November 10, 2021 at 01:27 PM London time GMT (GMT+0200)
---
01:27 PM | Operator: Hello,. 
 
So what brings you here today?
01:27 PM | alex: hello
01:27 PM | Operator: Hey alex!
01:27 PM | Operator: What do you need help with?
01:36 PM | alex: I run into an issue with the web config file on the system for the smtp server. do you mind to take a look at the config?
01:38 PM | Operator: Of course
01:42 PM | alex: here it is:

 1smtp {
 2    host=smtp.web.dev.inlanefreight.htb
 3    #port=25
 4    ssl=true
 5    user="alex"
 6    password="lol123!mD"
 7    from="alex.g@web.dev.inlanefreight.htb"
 8}
 9
10securesocial {
11    
12    onLoginGoTo=/
13    onLogoutGoTo=/login
14    ssl=false
15    
16    userpass {      
17      withUserNameSupport=false
18      sendWelcomeEmail=true
19      enableGravatarSupport=true
20      signupSkipLogin=true
21      tokenDuration=60
22      tokenDeleteInterval=5
23      minimumPasswordLength=8
24      enableTokenJob=true
25      hasher=bcrypt
26      }
27
28     cookie {
29     #       name=id
30     #       path=/login
31     #       domain="10.129.2.59:9500"
32            httpOnly=true
33            makeTransient=false
34            absoluteTimeoutInMinutes=1440
35            idleTimeoutInMinutes=1440
36    }   


```
Nisso foi possível a gente saber duas coisas , user alex e password lol123!mD . Podemos agora utilizar essas credenciais para acessar outros serviços.

---
Com o user e senha que recebi comecei a usar essas credenciais , primeiro foi no rpccliente mas não tinha permissão de fazer nada 
```
rpcclient -U "alex" 10.129.202.41
Password for [WORKGROUP\alex]:
rpcclient $> ls
command not found: ls
rpcclient $> srvinfo
        10.129.202.41  Wk Sv Sql NT SNT     
        platform_id     :       500
        os version      :       10.0
        server type     :       0x9007
rpcclient $> enumdomains
result was NT_STATUS_CONNECTION_DISCONNECTED
rpcclient $> netshareenumall
result was WERR_ACCESS_DENIED
rpcclient $> enumdomusers
result was NT_STATUS_CONNECTION_DISCONNECTED
rpcclient $> netshareenumall
result was WERR_ACCESS_DENIED
rpcclient $> ^C
```
Mas depois listei o smb com o user alex e vi um share lá .
```
smbclient -L 10.129.202.41 -p 445 -U 'alex'               
Password for [WORKGROUP\alex]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        devshare        Disk      
        IPC$            IPC       Remote IPC
        Users           Disk      

```
Nisso acessei ele e dei ls , tendo um txt chamado important.
```
smbclient //10.129.202.41/devshare -p 445 -U 'alex' 
Password for [WORKGROUP\alex]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Nov 10 11:12:22 2021
  ..                                  D        0  Wed Nov 10 11:12:22 2021
  important.txt                       A       16  Wed Nov 10 11:12:55 2021

                10328063 blocks of size 4096. 6100614 blocks available
smb: \> cat important.xt
cat: command not found
smb: \> get important.txt \
getting file \important.txt of size 16 as \ (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \> get important.txt 
getting file \important.txt of size 16 as important.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \> exit
```
Dei get nele e dei cat para ver o que havia dentro.
```
cat important.txt 
sa:87N1ns@slls83
```
O sa é um usuário de mssql , com isso eu havia acessado o smbclient na pasta compartilhada User e acessado o desktop de alex e vi um Microsoft sql server management studio, com isso usei o xfreerdp .
```
xfreerdp /v:10.129.202.41 /u:alex /p:'lol123!mD' /dynamic-resolution
```
Assim agora vou tentar logar como sa. Não tive como logar como sa no microsoft sql server management studio mas essa senha dele eu testei para administrador e ela foi reaproveitada como a senha do admin. Nisso rodei esse studio como admin e loguei como admin nele , nisso tem um db chamado accounts , com uma tabela chamada dbo.devsacc, nisso utilizei a seguinte query:
```
select name, password from dbo.devsacc where name like 'HTB';
```
Assim selecionei o que o lab queria e a resposta era lnch7ehrdn43i7AoqVPK4zWR, a senha do user HTB.
![[Pasted image 20260828165047.png]]
## Considerações
rpcbind -> Funciona como se fosse uma lista telefonica que mostra os programas que utilizam RPC para portas específicas da máquin
msrpc -> Mapeia interface DCOM e serviços internos do windows
netbios -> Transporte de sessão legado para smb sobre netbios
smb (microsoft-ds)-> compartilhamento nativo de arquivos , impressoras e pipes nomeados
NFS -> compartilhamento de pasta na rede
RDP -> Acesso gráfico interativo a área de trabalho
WinRM -> Administração remota via linha de comando / powershell

Sempre olhar o NFS pois nele pode haver diretórios compartilhados que podemos montar e visualizar arquivos com conteudos importantes nele . Após eu visualizar o ticket onde continha o user e senha do alex eu consegui acessar todos os serviços praticamente com esse login e senha , nisso acessei o rpcclient na porta 445 mas não tinha privilégio algum para executar comandos de enumeração. Mas acessando o smb na porta 445 e dando ls havia um arquivo que continha a senha que no início e pela identificação tudo condizia ser do usuário sa do mssql , e assim já obtive no smb mais informações importantes para a escadala de privilégio , na qual usei o xfreerdp agora para acessar o windows pela porta 3389 e loguei como alex. Nisso abri o Microsoft MYSQL server management studio e la havia de padrão o sa para logar e coloquei seco a senha que havia no arquivo , mas não obtive o resultado que queria . Com isso pensei e pensei , resolvi abrir como admin e colocar a senha do sa no admin pois podia ser uma senha reaproveitada , assim deu certo e loguei com admin nesse software , de resto só listei o que eu queria na tabela de usuários . 

- Lições aprendidas 
	- reutilizar senhas para outros usuários principalmente se uma senha de sa ( maior privilégio no mssql) pode ser a mesma do admin por serem de maiores privilégios dentro de seus contextos.
	- Listar tudo em mount's feitos de diretórios de nfs, de primeira listei com um simples ls e nem vi que vários arquivos estavam vazios .
	- Arquivos podem conter usuários e senhas 
	- Nem sempre scripts automatizados funcionam , devemos saber utilizar os comandos manualmente e testar 


