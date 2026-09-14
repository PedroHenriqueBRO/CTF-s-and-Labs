# Tópico
Opacity
# Tags

#tryhackme 

# Resumo
## Nmap
Utilizei 3 nmaps diferentes:
- nmap -sS -O para não fazer aperto de mão de três vias via TCP e ver a versão do SO
```bash
	nmap -sS -O 10.65.141.245 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-12 10:02 -0400
Nmap scan report for 10.65.141.245
Host is up (0.15s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.98%E=4%D=5/12%OT=22%CT=1%CU=35979%PV=Y%DS=3%DC=I%G=Y%TM=6A0332F
OS:4%P=x86_64-pc-linux-gnu)SEQ(SP=101%GCD=1%ISR=10F%TI=Z%CI=Z%II=I%TS=A)SEQ
OS:(SP=105%GCD=1%ISR=108%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=105%GCD=1%ISR=10B%TI=Z%
OS:CI=Z%II=I%TS=A)SEQ(SP=106%GCD=1%ISR=10C%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=106%G
OS:CD=1%ISR=10D%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M4E8ST11NW7%O2=M4E8ST11NW7%O3=M4
OS:E8NNT11NW7%O4=M4E8ST11NW7%O5=M4E8ST11NW7%O6=M4E8ST11)WIN(W1=F4B3%W2=F4B3
OS:%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F507%O=M4E8NNSNW7%C
OS:C=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%
OS:T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD
OS:=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S
OS:=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK
OS:=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 3 hops

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.33 seconds
```
- nmap -sV para procurar melhor sobre os serviços rodando nas portas descobertas
```bash
nmap -sV -O 10.65.141.245
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-12 10:03 -0400
Nmap scan report for 10.65.141.245
Host is up (0.15s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.98%E=4%D=5/12%OT=22%CT=1%CU=32362%PV=Y%DS=3%DC=I%G=Y%TM=6A03334
OS:7%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)SEQ
OS:(SP=105%GCD=1%ISR=109%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=105%GCD=1%ISR=10B%TI=Z%
OS:CI=Z%II=I%TS=A)SEQ(SP=106%GCD=1%ISR=109%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=FF%GC
OS:D=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M4E8ST11NW7%O2=M4E8ST11NW7%O3=M4E
OS:8NNT11NW7%O4=M4E8ST11NW7%O5=M4E8ST11NW7%O6=M4E8ST11)WIN(W1=F4B3%W2=F4B3%
OS:W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F507%O=M4E8NNSNW7%CC
OS:=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T
OS:=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=
OS:0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=
OS:Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=
OS:G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.08 seconds

```
- nmap -sC para utilizar um script que é executado nos serviços rodando nessas portas para saber mais ainda sobre eles.
```bash 
nmap -sC 10.65.141.245
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-12 10:04 -0400
Nmap scan report for 10.65.141.245
Host is up (0.15s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
| ssh-hostkey: 
|   3072 92:17:5b:1f:a0:78:af:08:7b:2e:ef:0c:2c:d0:27:df (RSA)
|   256 65:c2:86:61:63:ec:74:11:6b:db:50:cb:a2:a0:34:52 (ECDSA)
|_  256 20:62:41:57:42:60:56:4c:bc:4a:70:77:eb:63:8f:2c (ED25519)
80/tcp  open  http
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-title: Login
|_Requested resource was login.php
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
|_clock-skew: -1s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-05-12T14:04:13
|_  start_date: N/A
|_nbstat: NetBIOS name: , NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Nmap done: 1 IP address (1 host up) scanned in 37.79 seconds

```
Dessas informações podemos ter em mente se acharmos um nome de usuário podemos tentar fazer brute force via hydra para o ssh , podemos fazer listagem de dir pelo gobuster no servidor web para achar paginas que podem nos mostrar algo ou permitir ataques e a porta 139/445 são de servidores samba na qual a 445 na verdade é do linux e não da microsoft e nos deu uma informação importante com -sC que ao ter "Message signing enabled but not required" podemos fazer ataques man in the middle , mas por enquanto como tem essa porta aberta podemos usar o smbclient -L "ip" para tentar achar pastas publicas.
## smbclient -L ip
Não nos deu informações importantes , assim pulei para outro comando que tenta listar e fazer descobertas de usuários no sistema alvo que seria o comando enum4linux .

## enum4linux
Ele basicamente de primeira tenta fazer o mesmo que o smbclient -L mas como ele não achou nada então foi para o brute discover de usuários e assim tenta achar coisas na porta 139 e achou informações sobre password ages do sistema alvo . Em seguida ele busca grupos no sistema alvo e depois faz o brute de usuários na qual ele procura por SID. No fim ele achou isso:
```bash
enum4linux -a 10.65.141.245
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue May 12 10:16:54 2026

 =========================================( Target Information )=========================================

Target ........... 10.65.141.245
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ===========================( Enumerating Workgroup/Domain on 10.65.141.245 )===========================


[+] Got domain/workgroup name: WORKGROUP


 ===============================( Nbtstat Information for 10.65.141.245 )===============================

Looking up status of 10.65.141.245
        ..__MSBROWSE__. <01> - <GROUP> B <ACTIVE>  Master Browser
                        <00> -         B <ACTIVE> 
                        <03> -         B <ACTIVE> 
                        <20> -         B <ACTIVE> 
        WORKGROUP       <00> - <GROUP> B <ACTIVE>  Domain/Workgroup Name
        WORKGROUP       <1d> -         B <ACTIVE>  Master Browser
        WORKGROUP       <1e> - <GROUP> B <ACTIVE>  Browser Service Elections

        MAC Address = 00-00-00-00-00-00

 ===================================( Session Check on 10.65.141.245 )===================================


[+] Server 10.65.141.245 allows sessions using username '', password ''


 ================================( Getting domain SID for 10.65.141.245 )================================

Domain Name: WORKGROUP
Domain Sid: (NULL SID)

[+] Can't determine if host is part of domain or part of a workgroup


 ==================================( OS information on 10.65.141.245 )==================================


[E] Can't get OS info with smbclient


[+] Got OS info for 10.65.141.245 from srvinfo: 
        IP-10-65-141-24Wk Sv PrQ Unx NT SNT ip-10-65-141-245 server (Samba, Ubuntu)
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03


 =======================================( Users on 10.65.141.245 )=======================================
                                                                                                                                                                                                                                                                                   
Use of uninitialized value $users in print at ./enum4linux.pl line 972.                                                                                                                                                                                                            
Use of uninitialized value $users in pattern match (m//) at ./enum4linux.pl line 975.

Use of uninitialized value $users in print at ./enum4linux.pl line 986.
Use of uninitialized value $users in pattern match (m//) at ./enum4linux.pl line 988.

 =================================( Share Enumeration on 10.65.141.245 )=================================
                                                                                                                                                                                                                                                                                   
smbXcli_negprot_smb1_done: No compatible protocol selected by server.                                                                                                                                                                                                              

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        IPC$            IPC       IPC Service (ip-10-65-141-245 server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.
Protocol negotiation to server 10.65.141.245 (for a protocol between LANMAN1 and NT1) failed: NT_STATUS_INVALID_NETWORK_RESPONSE
Unable to connect with SMB1 -- no workgroup available

[+] Attempting to map shares on 10.65.141.245                                                                                                                                                                                                                                      
                                                                                                                                                                                                                                                                                   
//10.65.141.245/print$  Mapping: DENIED Listing: N/A Writing: N/A                                                                                                                                                                                                                  

[E] Can't understand response:                                                                                                                                                                                                                                                     
                                                                                                                                                                                                                                                                                   
NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*                                                                                                                                                                                                                                         
//10.65.141.245/IPC$    Mapping: N/A Listing: N/A Writing: N/A

 ===========================( Password Policy Information for 10.65.141.245 )===========================
                                                                                                                                                                                                                                                                                   
Password:                                                                                                                                                                                                                                                                          




[+] Attaching to 10.65.141.245 using a NULL share

[+] Trying protocol 139/SMB...

[+] Found domain(s):

        [+] IP-10-65-141-245
        [+] Builtin

[+] Password Info for Domain: IP-10-65-141-245

        [+] Minimum password length: 5
        [+] Password history length: None
        [+] Maximum password age: 136 years 37 days 6 hours 21 minutes 
        [+] Password Complexity Flags: 000000

                [+] Domain Refuse Password Change: 0
                [+] Domain Password Store Cleartext: 0
                [+] Domain Password Lockout Admins: 0
                [+] Domain Password No Clear Change: 0
                [+] Domain Password No Anon Change: 0
                [+] Domain Password Complex: 0

        [+] Minimum password age: None
        [+] Reset Account Lockout Counter: 30 minutes 
        [+] Locked Account Duration: 30 minutes 
        [+] Account Lockout Threshold: None
        [+] Forced Log off Time: 136 years 37 days 6 hours 21 minutes 



[+] Retieved partial password policy with rpcclient:                                                                                                                                                                                                                               
                                                                                                                                                                                                                                                                                   
                                                                                                                                                                                                                                                                                   
Password Complexity: Disabled                                                                                                                                                                                                                                                      
Minimum Password Length: 5


 ======================================( Groups on 10.65.141.245 )======================================
                                                                                                                                                                                                                                                                                   
                                                                                                                                                                                                                                                                                   
[+] Getting builtin groups:                                                                                                                                                                                                                                                        
                                                                                                                                                                                                                                                                                   
                                                                                                                                                                                                                                                                                   
[+]  Getting builtin group memberships:                                                                                                                                                                                                                                            
                                                                                                                                                                                                                                                                                   
                                                                                                                                                                                                                                                                                   
[+]  Getting local groups:                                                                                                                                                                                                                                                         
                                                                                                                                                                                                                                                                                   
                                                                                                                                                                                                                                                                                   
[+]  Getting local group memberships:                                                                                                                                                                                                                                              
                                                                                                                                                                                                                                                                                   
                                                                                                                                                                                                                                                                                   
[+]  Getting domain groups:                                                                                                                                                                                                                                                        
                                                                                                                                                                                                                                                                                   
                                                                                                                                                                                                                                                                                   
[+]  Getting domain group memberships:                                                                                                                                                                                                                                             
                                                                                                                                                                                                                                                                                   
                                                                                                                                                                                                                                                                                   
 ==================( Users on 10.65.141.245 via RID cycling (RIDS: 500-550,1000-1050) )==================
                                                                                                                                                                                                                                                                                   
                                                                                                                                                                                                                                                                                   
[I] Found new SID:                                                                                                                                                                                                                                                                 
S-1-22-1                                                                                                                                                                                                                                                                           

[I] Found new SID:                                                                                                                                                                                                                                                                 
S-1-5-32                                                                                                                                                                                                                                                                           

[I] Found new SID:                                                                                                                                                                                                                                                                 
S-1-5-32                                                                                                                                                                                                                                                                           

[I] Found new SID:                                                                                                                                                                                                                                                                 
S-1-5-32                                                                                                                                                                                                                                                                           

[I] Found new SID:                                                                                                                                                                                                                                                                 
S-1-5-32                                                                                                                                                                                                                                                                           

[+] Enumerating users using SID S-1-5-32 and logon username '', password ''                                                                                                                                                                                                        
                                                                                                                                                                                                                                                                                   
S-1-5-32-544 BUILTIN\Administrators (Local Group)                                                                                                                                                                                                                                  
S-1-5-32-545 BUILTIN\Users (Local Group)
S-1-5-32-546 BUILTIN\Guests (Local Group)
S-1-5-32-547 BUILTIN\Power Users (Local Group)
S-1-5-32-548 BUILTIN\Account Operators (Local Group)
S-1-5-32-549 BUILTIN\Server Operators (Local Group)
S-1-5-32-550 BUILTIN\Print Operators (Local Group)

[+] Enumerating users using SID S-1-5-21-2096130846-3104434501-109344026 and logon username '', password ''                                                                                                                                                                        
                                                                                                                                                                                                                                                                                   
S-1-5-21-2096130846-3104434501-109344026-501 IP-10-65-141-245\nobody (Local User)                                                                                                                                                                                                  
S-1-5-21-2096130846-3104434501-109344026-513 IP-10-65-141-245\None (Domain Group)

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''                                                                                                                                                                                                        
                                                                                                                                                                                                                                                                                   
S-1-22-1-1000 Unix User\sysadmin (Local User)                                                                                                                                                                                                                                      
S-1-22-1-1001 Unix User\ubuntu (Local User)

 ===============================( Getting printer info for 10.65.141.245 )===============================
                                                                                                                                                                                                                                                                                   
No printers returned.                                                                                                                                                                                                                                                              


enum4linux complete on Tue May 12 10:28:37 2026

```
Basicamente temos dois usuários nesse IP alvo que podemos utilizar para brute force que são sysadmin e ubuntu,mas antes vou fazer buscar de dir's com gobuster.

## gobuster
Pelo gobuster dir foi encontrado várias rotas que direcionam para o login.php , mas duas outras não fazem isso que é a css (não tem tanta importância mas vale a pena buscar dentro) e a rota cloud que essa tem bastante importância visto que ela é uma rota de upload de imagem , se ela tiver uma vulnerabilidade de executar uma imagem como arquivo php podemos fazer ataque de RFI criando um servidor python local com python3 -m http.server 80 e nesse dir colocar um arquivo php , mas aí temos de verificar os tipos de filtros , se vamos ter que somente alterar o mimo ou o hexeditor , penso que por ser RFI teria de ser o hexeditor pois a imagem seria buscada na parte do servidor e não do cliente.
```bash
gobuster dir -u http://10.65.141.245 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 -x js,php,txt,html
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.65.141.245
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              txt,html,js,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
login.php            (Status: 200) [Size: 848]
index.php            (Status: 302) [Size: 0] [--> login.php]
css                  (Status: 301) [Size: 312] [--> http://10.65.141.245/css/]
logout.php           (Status: 302) [Size: 0] [--> login.php]
cloud                (Status: 301) [Size: 314] [--> http://10.65.141.245/cloud/]
Progress: 398227 / 1102795 (36.11%)^C
```

## Hydra
É inviável fazer brute force nesse servidor ssh visto que é possível abrir poucas tasks por server e assim demoraria muito para achar ou não a senha desses usuários (155 dias).Vamos tentar no http-form.Como só sei a mensagem de erro que é Invalid Login Details acaba que o hydra retorna só falsos positivos e como não tenho condição de fazer um login correto para saber a mensagem ou status de acerto então vou para exploração do /cloud.

## SQLMAP
Antes da exploração do Cloud vou tentar achar vulnerabilidades de sql injection no formulário , para isso eu barrei uma requisição no burp suite e peguei a requisição para por em um arquivo , nisso coloquei sqlmap -r request.txt -dbs para tentar acha os bancos desse ip , não deu certo mas tentei subir o --level , --risk e colocar --tamper=space2comment para tentar achar a vulnerabilidade básica de //.
## Cloud
No Cloud iremos fazer a inclusão de uma url externa que será de um servidor aberto em nossa máquina com um arquivo que conterá um comando de abertura de shell, podendo ser um simples nc ou um mkfifo com nc.Criei um arquivo .php em um diretório meu , criei um servidor python http.serve e mandei a url no cloud mas essa url eu coloquei archive.php .jpg no final pois o servidor ele baixava o arquivo .php mesmo que tinha o .jpg no final pois esse .jpg foi para burlar o filtro de extensão , assim depois eu acessei a rota http://10.65.141.245/cloud/images/archive.php e recebi a shell.

## Dentro do Linux
Tem os dois usuários vistos anteriormente que é o sysadmin e o ubuntu , com o usuário que peguei shell não consigo fazer escalação com getcap, suid , crontab , ld preload , nfs e nem sudo -l , assim fui vasculhando o sistema e achei um arquivo em /opt que era um data base do tipo keepass , nisso transformei o conteudo em base64 , copiei , coloquei em um arquivo na minha maquina , usei o decode denovo para outro arquivo , desse arquivo usei o keepass2john que me devolveu um arquivo do tipo dataset:hash, usei o john em cima dele e recebi a senha , usei keepass no arquivo .kdbx, coloquei a senha e consegui a senha do sysadmin.Com sysadmin eu acessei a pasta scripts que ele tinha na home dele e eu tinha permissão de ler o script.php que puxava uma pasta que estava em /lib e também havia um backup desses arquivos , com isso descobri que se na pasta que tinha o script e a lib eu jogasse o backup a lib seria substituída e eu seria o novo donos dos arquivos na qual eu poderia agora mudar o arquivo de backup que estava na lib e nele colocar uma chamada de shell para a minha máquina , esse script rodava como root , logo o shell que abria era para o usuário root, assim consegui a shell como root e peguei a última flag.


## Observações
Sempre vasculhar todos os serviços resultantes do nmap mesmo que tenha arranjado uma rota melhor , isso para aprendizado já que em uma situação real é melhor explorar o caminho mais eficiente.Descobri outro jeito de burlar filtro de arquivo que é colocar arquivo.php .jpg que assim o servidor entende o .jpg como uma substring da url mas ele não faz parte da extensão do arquivo , assim ele é upado como archive.php e executado como php no servidor de destino.Quando conectar em uma maquina que não tem caminhos triviais procurar buscar em todos os lugares arquivos importante mas antes ver se não tem caminhos de privilégio mais fáceis , como sudo -l , arquivos com permissão de sudo sem dar senha , arquivo suid , getcap , nfs , PATH , exploit de versão do linux , cronjobs , ld preload e outras funcionalidades legítimas de comandos que podem ser utilizadas para isso.



	  
	  




