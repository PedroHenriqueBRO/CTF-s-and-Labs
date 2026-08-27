# Tópico

# Tags

#Hackthebox 

# Resumo
Foi pedido para a gente enumerar os serviços POP3 e IMAP no servidor de destino e para isso utilizarei o nmap -sS para descobrir primeiramente as portas sendo utilizadas e seus serviços.
``` 
sudo nmap -sS 10.129.42.195                              
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-26 20:57 -0400
Nmap scan report for 10.129.42.195
Host is up (0.44s latency).
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

Nmap done: 1 IP address (1 host up) scanned in 6.85 seconds
```
Aqui vemos que tem o Imap e Pop3 rodando suas versões com tls(993 imaps e 995 pop3) e sem tls (110 pop3 e 143 tls).
Agora iremos utilizar o -sV e -sC para melhor entender esses serviços.
``` 
sudo nmap -sV -sC 10.129.42.195 -p 110,143,993,995
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-26 20:59 -0400
Nmap scan report for 10.129.42.195
Host is up (0.36s latency).

PORT    STATE SERVICE  VERSION
110/tcp open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dev.inlanefreight.htb/organizationName=InlaneFreight Ltd/stateOrProvinceName=London/countryName=UK
| Not valid before: 2021-11-08T23:10:05
|_Not valid after:  2295-08-23T23:10:05
|_pop3-capabilities: AUTH-RESP-CODE STLS SASL RESP-CODES CAPA TOP PIPELINING UIDL
143/tcp open  imap     Dovecot imapd
| ssl-cert: Subject: commonName=dev.inlanefreight.htb/organizationName=InlaneFreight Ltd/stateOrProvinceName=London/countryName=UK
| Not valid before: 2021-11-08T23:10:05
|_Not valid after:  2295-08-23T23:10:05
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: more IMAP4rev1 IDLE LOGINDISABLEDA0001 STARTTLS ID ENABLE have capabilities LOGIN-REFERRALS LITERAL+ listed Pre-login OK SASL-IR post-login
993/tcp open  ssl/imap Dovecot imapd
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: IMAP4rev1 IDLE more have AUTH=PLAINA0001 ENABLE LITERAL+ capabilities LOGIN-REFERRALS listed Pre-login OK SASL-IR ID post-login
| ssl-cert: Subject: commonName=dev.inlanefreight.htb/organizationName=InlaneFreight Ltd/stateOrProvinceName=London/countryName=UK
| Not valid before: 2021-11-08T23:10:05
|_Not valid after:  2295-08-23T23:10:05
995/tcp open  ssl/pop3 Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dev.inlanefreight.htb/organizationName=InlaneFreight Ltd/stateOrProvinceName=London/countryName=UK
| Not valid before: 2021-11-08T23:10:05
|_Not valid after:  2295-08-23T23:10:05
|_pop3-capabilities: AUTH-RESP-CODE USER SASL(PLAIN) RESP-CODES CAPA TOP PIPELINING UIDL

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.56 seconds
```
Aqui vemos o nome em comum sendo dev.inlanefreight.com e a organização sendo InlaneFreight Ltd e assim respondemos a primeira pergunta que é o nome da organização e a segunda é o FQDN que os dois serviços estao atribuidos que é dev.inlanefreight.htb.
Agora devemos enumerar melhor o serviço do IMAP.
```
curl -k "imaps://10.129.42.195" --user robin:robin -v 
*   Trying 10.129.42.195:993...
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* SSL Trust: peer verification disabled
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / x25519 / RSASSA-PSS
* Server certificate:
*   subject: C=UK; ST=London; L=London; O=InlaneFreight Ltd; OU=DevOps DepÃartment; CN=dev.inlanefreight.htb; emailAddress=cto.dev@dev.inlanefreight.htb
*   start date: Nov  8 23:10:05 2021 GMT
*   expire date: Aug 23 23:10:05 2295 GMT
*   issuer: C=UK; ST=London; L=London; O=InlaneFreight Ltd; OU=DevOps DepÃartment; CN=dev.inlanefreight.htb; emailAddress=cto.dev@dev.inlanefreight.htb
*   Certificate level 0: Public key type RSA (2048/112 Bits/secBits), signed using sha256WithRSAEncryption
*  SSL certificate verification failed, continuing anyway!
* Established connection to 10.129.42.195 (10.129.42.195 port 993) from 10.10.17.37 port 40722 
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
< * OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LITERAL+ AUTH=PLAIN] HTB{roncfbw7iszerd7shni7jr2343zhrj}
> A001 CAPABILITY
< * CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LITERAL+ AUTH=PLAIN
< A001 OK Pre-login capabilities listed, post-login capabilities have more.
> A002 AUTHENTICATE PLAIN AHJvYmluAHJvYmlu
< * CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE SORT SORT=DISPLAY THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND URL-PARTIAL CATENATE UNSELECT CHILDREN NAMESPACE UIDPLUS LIST-EXTENDED I18NLEVEL=1 CONDSTORE QRESYNC ESEARCH ESORT SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS BINARY MOVE SNIPPET=FUZZY PREVIEW=FUZZY LITERAL+ NOTIFY SPECIAL-USE
< A002 OK Logged in
> A003 LIST "" *
< * LIST (\Noselect \HasChildren) "." DEV
* LIST (\Noselect \HasChildren) "." DEV
< * LIST (\Noselect \HasChildren) "." DEV.DEPARTMENT
* LIST (\Noselect \HasChildren) "." DEV.DEPARTMENT
< * LIST (\HasNoChildren) "." DEV.DEPARTMENT.INT
* LIST (\HasNoChildren) "." DEV.DEPARTMENT.INT
< * LIST (\HasNoChildren) "." INBOX
* LIST (\HasNoChildren) "." INBOX
< A003 OK List completed (0.002 + 0.000 + 0.001 secs).
* Connection #0 to host 10.129.42.195:993 left intact

```
```
curl -k "pop3s://10.129.42.195" --user robin:robin -v
*   Trying 10.129.42.195:995...
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* SSL Trust: peer verification disabled
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / x25519 / RSASSA-PSS
* Server certificate:
*   subject: C=UK; ST=London; L=London; O=InlaneFreight Ltd; OU=DevOps DepÃartment; CN=dev.inlanefreight.htb; emailAddress=cto.dev@dev.inlanefreight.htb
*   start date: Nov  8 23:10:05 2021 GMT
*   expire date: Aug 23 23:10:05 2295 GMT
*   issuer: C=UK; ST=London; L=London; O=InlaneFreight Ltd; OU=DevOps DepÃartment; CN=dev.inlanefreight.htb; emailAddress=cto.dev@dev.inlanefreight.htb
*   Certificate level 0: Public key type RSA (2048/112 Bits/secBits), signed using sha256WithRSAEncryption
*  SSL certificate verification failed, continuing anyway!
* Established connection to 10.129.42.195 (10.129.42.195 port 995) from 10.10.17.37 port 58952 
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
< +OK InFreight POP3 v9.188
> CAPA
< +OK
< CAPA
< TOP
< UIDL
< RESP-CODES
< PIPELINING
< AUTH-RESP-CODE
< USER
< SASL PLAIN
< .
> AUTH PLAIN
< + 
> AHJvYmluAHJvYmlu
< +OK Logged in.
> LIST
< +OK 0 messages:

* Connection #0 to host 10.129.42.195:995 left intact

```
Eu enumerei melhor o IMAP com o curl fazendo login e pegando informações do servidor imap e nisso peguei a flag e em seguida fiz o mesmo pro pop3 e peguei a versão persoanlizada dele , assim respondi mais duas perguntas.
As duas últimas perguntas eu acessei o servidor POP3S e tentei utilizar o comando stat e list para solicitar o numeros de emails salvos do servidor e depois listar o número e o tamanho de todos os emails mas recebi 0 nos dois comandos. Fui para o IMAP e usei o openssl s_client para conectar ao servidor IMAP .

``` 
openssl s_client -connect 10.129.42.195:imaps
Connecting to 10.129.42.195
CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 C=UK, ST=London, L=London, O=InlaneFreight Ltd, OU=DevOps DepÃartment, CN=dev.inlanefreight.htb, emailAddress=cto.dev@dev.inlanefreight.htb
verify error:num=18:self-signed certificate
verify return:1
depth=0 C=UK, ST=London, L=London, O=InlaneFreight Ltd, OU=DevOps DepÃartment, CN=dev.inlanefreight.htb, emailAddress=cto.dev@dev.inlanefreight.htb
verify return:1
---
Certificate chain
 0 s:C=UK, ST=London, L=London, O=InlaneFreight Ltd, OU=DevOps DepÃartment, CN=dev.inlanefreight.htb, emailAddress=cto.dev@dev.inlanefreight.htb
   i:C=UK, ST=London, L=London, O=InlaneFreight Ltd, OU=DevOps DepÃartment, CN=dev.inlanefreight.htb, emailAddress=cto.dev@dev.inlanefreight.htb
   a:PKEY: RSA, 2048 (bit); sigalg: sha256WithRSAEncryption
   v:NotBefore: Nov  8 23:10:05 2021 GMT; NotAfter: Aug 23 23:10:05 2295 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIEUzCCAzugAwIBAgIUDf35PqFuv6Uv0EECM8dFmNSZoY8wDQYJKoZIhvcNAQEL
BQAwgbcxCzAJBgNVBAYTAlVLMQ8wDQYDVQQIDAZMb25kb24xDzANBgNVBAcMBkxv
bmRvbjEaMBgGA1UECgwRSW5sYW5lRnJlaWdodCBMdGQxHDAaBgNVBAsME0Rldk9w
cyBEZXDDg2FydG1lbnQxHjAcBgNVBAMMFWRldi5pbmxhbmVmcmVpZ2h0Lmh0YjEs
MCoGCSqGSIb3DQEJARYdY3RvLmRldkBkZXYuaW5sYW5lZnJlaWdodC5odGIwIBcN
MjExMTA4MjMxMDA1WhgPMjI5NTA4MjMyMzEwMDVaMIG3MQswCQYDVQQGEwJVSzEP
MA0GA1UECAwGTG9uZG9uMQ8wDQYDVQQHDAZMb25kb24xGjAYBgNVBAoMEUlubGFu
ZUZyZWlnaHQgTHRkMRwwGgYDVQQLDBNEZXZPcHMgRGVww4NhcnRtZW50MR4wHAYD
VQQDDBVkZXYuaW5sYW5lZnJlaWdodC5odGIxLDAqBgkqhkiG9w0BCQEWHWN0by5k
ZXZAZGV2LmlubGFuZWZyZWlnaHQuaHRiMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEAxvMwFE6m+iBUSujb5d6DUy1xDYR5awzQRwddyvq6iBrMxbnptSrn
+j0UOKWHCOpD5LREwP26ghUg0lVJzfo+v5pQJGnxEXKg0OFlzWEd8xgx/JWW/z1/
rDsWlNa2yYZkCy68YWJlC7UZxvcDFrI0V0pDJIkrjForw26laoYDkrh1A5F8uUXD
1TwRLLYo+NGmtNHT3BADJpv6aFUZ4CGrqBQNi7XpsTZ948WLhUwQvWmebiK06Dai
TvMNKBctjWAiNI4xvq34W9hIUaPxT1JJzuujRslep6nHGHW00QEWTWgyOMYThc3b
HtKIHMfDLTUMz7s8RhVVwlWE6+ly1DMRgQIDAQABo1MwUTAdBgNVHQ4EFgQUGDTC
9B5KCKPWT7vXbnMunL/mEE4wHwYDVR0jBBgwFoAUGDTC9B5KCKPWT7vXbnMunL/m
EE4wDwYDVR0TAQH/BAUwAwEB/zANBgkqhkiG9w0BAQsFAAOCAQEADh0v5XWCf3KO
atrWcoiIOC67Z0ZIO7yEF+fQo8z+Wx1dWzmCFVu7u4+l7slcdJICCGBbOX8eItWS
chwzgnWJToyX8PWY8lSaB8ifMDQcr457Y7O6NmvgU35sRcLnYYqXzu2oh0lxsFLR
vL1wpyDLPhhoI++j1fELhiJ3GWiUQrb0vfJPcbSkHTgzf0hm7mLJTaqt3WfS/Gr2
8Oh7vSfzvqvHLE7HHAO0G5Q81zo+wWsrQF0s40HEF/raEMfOy2Htm79YjyjAlLWf
ueS+u8rX2smOYdRIpL3UPx7+yZPGu47vYoetde1Z5cfTCgmeS05BQ2qMOp6Tw6+G
xUuqg8nK1Q==
-----END CERTIFICATE-----
subject=C=UK, ST=London, L=London, O=InlaneFreight Ltd, OU=DevOps DepÃartment, CN=dev.inlanefreight.htb, emailAddress=cto.dev@dev.inlanefreight.htb
issuer=C=UK, ST=London, L=London, O=InlaneFreight Ltd, OU=DevOps DepÃartment, CN=dev.inlanefreight.htb, emailAddress=cto.dev@dev.inlanefreight.htb
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: rsa_pss_rsae_sha256
Peer Temp Key: X25519, 253 bits
---
SSL handshake has read 1667 bytes and written 1740 bytes
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
    Session-ID: 883BD7FB3EE4B4DB6DD5837EA1EF17217666B77A20747A568389B11DE23314A1
    Session-ID-ctx: 
    Resumption PSK: 731105D8C5266C6F42FF030BD4EFB84A5854BBB7F37481B98B4CFD87ECE1F4972C65E9AD480038FFC2AC11A09A8FE7F7
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - fe f2 6c eb c2 e4 53 12-f6 8b d6 41 31 81 0e 4e   ..l...S....A1..N
    0010 - 78 32 5e aa 00 f4 87 f3-bf 9e bc c1 a8 04 48 77   x2^...........Hw
    0020 - 94 7a 60 c5 b3 a7 cf a4-41 5c c4 be 65 81 c3 dd   .z`.....A\..e...
    0030 - fc d6 0b 5f a0 98 af 8a-59 fe 37 ed c7 ca e3 67   ..._....Y.7....g
    0040 - ba 6f 97 bf ec b2 bd aa-a3 7a 1a 12 19 fc 0e cd   .o.......z......
    0050 - 21 0f 40 65 c2 fa cf fe-f7 bc df b4 aa 2e 12 f1   !.@e............
    0060 - 92 f3 9f c0 5b d1 f3 a9-6b 5d 85 6a 9d 41 44 51   ....[...k].j.ADQ
    0070 - b6 a9 3f 09 ea e1 91 7e-99 4d 60 8e 2c 14 98 83   ..?....~.M`.,...
    0080 - 97 7c b3 38 3e 63 19 ea-c6 4a 15 d2 ca cd 10 b1   .|.8>c...J......
    0090 - 35 58 9d cb 0f 64 7a ec-db 0e 3c d5 54 d2 f6 57   5X...dz...<.T..W
    00a0 - 8a d2 37 3a 52 4f 6c d3-b8 fa 8c b2 ed d0 df 07   ..7:ROl.........
    00b0 - 15 52 da be ff 17 de 2a-e7 2e da 36 23 15 b0 5d   .R.....*...6#..]

    Start Time: 1787794068
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
    Session-ID: 73EDE6AFF55A2636F832A4F674D8C6E3E0DC00355F771C891744F571228E99EA
    Session-ID-ctx: 
    Resumption PSK: 760146A854B1EABF441A4C246E40849119405A92C9029CC7E0B8F5CCE6E6F57EFC0C44A9D9D62FA4DC0A06930A0B1476
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - fe f2 6c eb c2 e4 53 12-f6 8b d6 41 31 81 0e 4e   ..l...S....A1..N
    0010 - 94 cd b8 bc 45 9d 63 76-f5 63 73 3f e0 db d5 53   ....E.cv.cs?...S
    0020 - b1 55 0b 86 a0 a1 9a 91-78 58 d0 0c 0c 77 b0 e4   .U......xX...w..
    0030 - d9 d2 fb 71 4d 36 c2 d4-8e 18 5f 40 27 65 1c d6   ...qM6...._@'e..
    0040 - 75 b1 06 40 c2 87 52 d8-90 48 84 2c a9 57 60 ca   u..@..R..H.,.W`.
    0050 - 7c 6c 13 73 c5 48 89 b2-fb 69 a9 5a ac 05 f1 be   |l.s.H...i.Z....
    0060 - 89 c0 dc 73 78 55 60 50-c3 d2 76 35 d3 cf e8 cb   ...sxU`P..v5....
    0070 - 24 d6 8f 37 4e 55 16 ec-bb 7f e5 0b ad eb e3 f5   $..7NU..........
    0080 - 09 5e 94 89 35 60 3d f9-17 ad 82 a2 c1 60 e0 b6   .^..5`=......`..
    0090 - dd 6c c7 aa 3a 39 f0 bd-47 f2 f5 07 50 92 8f 35   .l..:9..G...P..5
    00a0 - 56 57 5b 26 59 80 30 d8-f2 e2 fd cf c7 45 12 dd   VW[&Y.0......E..
    00b0 - 43 fd 88 4d a8 44 29 b3-ca c0 22 68 ff 3e eb 02   C..M.D)..."h.>..

    Start Time: 1787794068
    Timeout   : 7200 (sec)
    Verify return code: 18 (self-signed certificate)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
* OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LITERAL+ AUTH=PLAIN] HTB{roncfbw7iszerd7shni7jr2343zhrj}
1 LOGIN robin robin
1 OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE SORT SORT=DISPLAY THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND URL-PARTIAL CATENATE UNSELECT CHILDREN NAMESPACE UIDPLUS LIST-EXTENDED I18NLEVEL=1 CONDSTORE QRESYNC ESEARCH ESORT SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS BINARY MOVE SNIPPET=FUZZY PREVIEW=FUZZY LITERAL+ NOTIFY SPECIAL-USE] Logged in
1 LIST "" *
* LIST (\Noselect \HasChildren) "." DEV
* LIST (\Noselect \HasChildren) "." DEV.DEPARTMENT
* LIST (\HasNoChildren) "." DEV.DEPARTMENT.INT
* LIST (\HasNoChildren) "." INBOX
1 OK List completed (0.001 + 0.000 secs).
A01 SELECT "DEV.DEPARTMENT.INT"
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 1 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 1636414279] UIDs valid
* OK [UIDNEXT 2] Predicted next UID
A01 OK [READ-WRITE] Select completed (0.003 + 0.000 + 0.002 secs).
A02 SEARCH ALL
* SEARCH 1
A02 OK Search completed (0.001 + 0.000 secs).
A03 FETCH 1 (BODY[])
* 1 FETCH (BODY[] {167}
Subject: Flag
To: Robin <robin@inlanefreight.htb>
From: CTO <devadmin@inlanefreight.htb>
Date: Wed, 03 Nov 2021 16:13:27 +0200

HTB{983uzn8jmfgpd8jmof8c34n7zio}
)
A03 OK Fetch completed (0.010 + 0.000 + 0.009 secs).
```
Aqui matei as duas ultimas perguntas que eram qual o email do admin e a flag de um email dentro do servidor.

Comandos vistos :
openssl s_client --conect ip:pop3s|imaps

STAT e LIST no pop3s 

1 LIST "" \* -> Lista os diretórios dentro de inbox

1 SELECT "DEV.DEPARTMENT.INT"-> Acesso a caixa de correio especificada

1 SEARCH ALL -> busco todas os emails desse inbox

1 FETCH 1 all -> faço uma listagem de dados gerais desse email 1 selecionado

1 FETCH 1 (BODY\[\]) -> Comando que usei para listar o corpo do email e visualizar o email do admin e flag

1 SEARCH FROM "admin" -> Perguntei um jeito mais fácil de buscar certas keys em emails e IA me deu esse comando que busca em cabeçalhos a palavra admin ,se eu substituir FROM port TEXT vai ser buscado no texto a palavra admin.

1 FETCH 1:\* (BODY\[\]) -> Seleciona todos os emails buscados pelo search from e mostra seus corpos.