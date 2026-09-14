# Tópico

# Tags

#Hackthebox 

# Resumo
Devemo enumerar um serviço smtp para responder duas perguntas , irei rodar o nmap para visualizar as portas abertas primeiro.
```
sudo nmap -sS 10.129.176.3                            
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-26 08:47 -0400
Nmap scan report for 10.129.176.3
Host is up (0.45s latency).
Not shown: 992 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
53/tcp   open  domain
110/tcp  open  pop3
143/tcp  open  imap
993/tcp  open  imaps
995/tcp  open  pop3s
3306/tcp open  mysql

Nmap done: 1 IP address (1 host up) scanned in 8.85 seconds
```

Vemos que na porta 25 esta rodando o servidor smtp.
``` 
sudo nmap -sV --script smtp* 10.129.176.3 -p 25
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-26 08:48 -0400
Stats: 0:00:00 elapsed; 0 hosts completed (0 up), 1 undergoing Ping Scan
Parallel DNS resolution of 1 host. Timing: About 0.00% done
Stats: 0:00:01 elapsed; 0 hosts completed (0 up), 1 undergoing Ping Scan
Parallel DNS resolution of 1 host. Timing: About 0.00% done
Stats: 0:00:01 elapsed; 0 hosts completed (0 up), 1 undergoing Ping Scan
Parallel DNS resolution of 1 host. Timing: About 0.00% done
Stats: 0:00:02 elapsed; 0 hosts completed (0 up), 1 undergoing Ping Scan
Parallel DNS resolution of 1 host. Timing: About 0.00% done
Stats: 0:00:04 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 50.00% done; ETC: 08:48 (0:00:02 remaining)
Stats: 0:00:05 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 99.99% done; ETC: 08:48 (0:00:00 remaining)
Stats: 0:00:38 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 100.00% done; ETC: 08:49 (0:00:00 remaining)
Nmap scan report for 10.129.176.3
Host is up (0.24s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp
| smtp-vuln-cve2010-4344: 
|_  The SMTP server is not Exim: NOT VULNERABLE
|_smtp-commands: mail1, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
|_smtp-open-relay: Server is an open relay (16/16 tests)
| fingerprint-strings: 
|   Hello: 
|     220 InFreight ESMTP v2.11
|_    Syntax: EHLO hostname
| smtp-enum-users: 
|   root
|   admin
|   administrator
|   webadmin
|   sysadmin
|   netadmin
|   guest
|   user
|   web
|_  test
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port25-TCP:V=7.99%I=7%D=8/26%Time=6A8EE0C0%P=x86_64-pc-linux-gnu%r(Hell
SF:o,36,"220\x20InFreight\x20ESMTP\x20v2\.11\r\n501\x20Syntax:\x20EHLO\x20
SF:hostname\r\n");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 62.48 seconds
```
Aqui fiz uma verificação na porta 25 do smtp com todos os scripts para smtp e -sV mas o -sV nao conseguiu retornar um valor no campo version , porém no fingerprint-strings da para ver o banner de versão "InFreight ESMTP v2.11".Depois pediu para eu achar o nome de usuário que existe no sistema , existem 10 usuários de acordo com o script de enum mas podem existir falsos positivos entre eles. Na verdade na dica da sessão fala para utilizar uma wordlist de enumeração para brute force de vrfy.
Utilizei um modulo do metasploit para isso e achei o usuário robin
``` 
msf auxiliary(scanner/smtp/smtp_enum) > show options

Module options (auxiliary/scanner/smtp/smtp_enum):

   Name       Current Setting                   Required  Description
   ----       ---------------                   --------  -----------
   RHOSTS                                       yes       The target host(s), see https://docs.metasploit.com/docs/u
                                                          sing-metasploit/basics/using-metasploit.html
   RPORT      25                                yes       The target port (TCP)
   THREADS    1                                 yes       The number of concurrent threads (max one per host)
   UNIXONLY   true                              yes       Skip Microsoft bannered servers when testing unix users
   USER_FILE  /usr/share/metasploit-framework/  yes       The file that contains a list of probable users accounts.
              data/wordlists/unix_users.txt


View the full module info with the info, or info -d command.

msf auxiliary(scanner/smtp/smtp_enum) > set RHOST 10.129.176.3
RHOST => 10.129.176.3
msf auxiliary(scanner/smtp/smtp_enum) > set user_file /home/kali/Desktop/wordlist.txt
user_file => /home/kali/Desktop/wordlist.txt
msf auxiliary(scanner/smtp/smtp_enum) > exploit
[*] 10.129.176.3:25       - 10.129.176.3:25 Banner: 220 InFreight ESMTP v2.11
[+] 10.129.176.3:25       - 10.129.176.3:25 Users found: , robin
[*] 10.129.176.3:25       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution complete
```


