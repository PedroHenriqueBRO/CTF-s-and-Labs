# Tópico

# Tags

#Hackthebox 

# Resumo
Foi nos dado a missão de ver se conseguimos encontrar a versao do servidor dns do alvo pelo ip nos dado.
- IP Alvo: 10.129.171.114
Primeiramente eu executei uma varredura -sS , na qual ela nos mostrou certas coisas interessantes>.
```
sudo nmap -sS 10.129.171.114 -F                   
[sudo] password for kali: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-24 15:06 -0400
Nmap scan report for 10.129.171.114
Host is up (0.24s latency).
Not shown: 92 closed tcp ports (reset)
PORT    STATE    SERVICE
21/tcp  open     ftp
22/tcp  open     ssh
53/tcp  filtered domain
80/tcp  open     http
110/tcp open     pop3
139/tcp open     netbios-ssn
143/tcp open     imap
445/tcp filtered microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 4.83 seconds
```
Vemos por esse -sS que temos os seguintes serviços :
- 21 , ftp
- 22,ssh
- 53 , o domain (dns server)
- 80, http
- 110 , pop 3
- 139 , netbios-ssn
- 143 ,  imap
- 445 ,  microsoft-ds
Nisso vemos que a porta 53 e 445 são portas que estão filtrada , assim podendo estarem abertas ou fechadas .
Penso que podemos direto já envia um -sV para a porta 53 pedindo a versão tendo como porta de origem nossa a 53.
```
sudo nmap -sV 10.129.171.114 --source-port 53 -p 53
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-24 15:19 -0400
Nmap scan report for 10.129.171.114
Host is up (0.37s latency).

PORT   STATE    SERVICE VERSION
53/tcp filtered domain

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.21 seconds
```
Aqui vemos que simplesmente só mudar a porta para 53 não adianta , devemos ver mais o que podemos fazer. 
Utilizando o -sC nas portas gerais recebi informações na 139 (netbios) sobre nome do host e algumas informações.
```
nmap -sS -sC -F 10.129.171.114
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-24 15:16 -0400
Stats: 0:00:46 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.16% done; ETC: 15:17 (0:00:00 remaining)
Stats: 0:00:49 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.16% done; ETC: 15:17 (0:00:00 remaining)
Stats: 0:00:50 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.16% done; ETC: 15:17 (0:00:00 remaining)
Stats: 0:00:50 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.16% done; ETC: 15:17 (0:00:00 remaining)
Stats: 0:00:51 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.16% done; ETC: 15:17 (0:00:00 remaining)
Stats: 0:00:51 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.16% done; ETC: 15:17 (0:00:00 remaining)
Stats: 0:00:52 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.16% done; ETC: 15:17 (0:00:00 remaining)
Nmap scan report for 10.129.171.114
Host is up (0.23s latency).
Not shown: 92 closed tcp ports (reset)
PORT    STATE    SERVICE
21/tcp  open     ftp
22/tcp  open     ssh
| ssh-hostkey: 
|   2048 71:c1:89:90:7f:fd:4f:60:e0:54:f3:85:e6:35:6c:2b (RSA)
|   256 e1:8e:53:18:42:af:2a:de:c0:12:1e:2e:54:06:4f:70 (ECDSA)
|_  256 1a:cc:ac:d4:94:5c:d6:1d:71:e7:39:de:14:27:3c:3c (ED25519)
53/tcp  filtered domain
80/tcp  open     http
|_http-title: Apache2 Ubuntu Default Page: It works
110/tcp open     pop3
|_pop3-capabilities: RESP-CODES PIPELINING AUTH-RESP-CODE SASL TOP CAPA UIDL
139/tcp open     netbios-ssn
143/tcp open     imap
|_imap-capabilities: LITERAL+ LOGIN-REFERRALS ID IMAP4rev1 post-login IDLE OK listed LOGINDISABLEDA0001 ENABLE Pre-login capabilities have more SASL-IR
445/tcp filtered microsoft-ds

Host script results:
|_nbstat: NetBIOS name: HTB984NIFN97CBO, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: nix-nmap-medium
|   NetBIOS computer name: HTB984NIFN97CBO783QBNJCPAS984UIN\x00
|   Domain name: \x00
|   FQDN: nix-nmap-medium
|_  System time: 2026-08-24T21:16:20+02:00
|_clock-skew: mean: -40m00s, deviation: 1h09m15s, median: -1s
| smb2-time: 
|   date: 2026-08-24T19:16:20
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

Nmap done: 1 IP address (1 host up) scanned in 66.57 seconds
```
- Tentei fazer login ftp como anonymous e não deu certo .
	- ftp 10.129.171.114
- Tentei fazer curl para o ip com flag -i e -I para ver se era devolvido o domain server mas não deu certo também
	- curl -I 10.129.171.114
Finalmente encontrei uma forma de mostrar se a porta esta online ou não , usei -sU para mandar pacotes UDP e assim a porta 53 deu como aberta , logo o tráfico udp nela se mostra permitido.
```
nmap -sU 10.129.171.114 -F
Nmap scan report for 10.129.171.114
Host is up (0.28s latency).
Not shown: 96 closed udp ports (port-unreach)
PORT    STATE         SERVICE
53/udp  open          domain
68/udp  open|filtered dhcpc
137/udp open          netbios-ns
138/udp open|filtered netbios-dgm

Nmap done: 1 IP address (1 host up) scanned in 115.23 seconds
```
Com isso ficou muito mais fácil , bypassando com o -sU foi possível depois executar -sC ou podemos utilizar -sV para receber a versao.

```
sudo nmap -sU -sV 10.129.171.114 -p 53
Nmap scan report for 10.129.171.114
Host is up (0.24s latency).

PORT   STATE SERVICE VERSION
53/udp open  domain  (unknown banner: HTB{GoTtgUnyze9Psw4vGjcuMpHRp})
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-UDP:V=7.98%I=7%D=8/24%Time=6A8C9F5D%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReq,57,"\0\x06\x85\0\0\x01\0\x01\0\x01\0\0\x07version\x04bind
SF:\0\0\x10\0\x03\xc0\x0c\0\x10\0\x03\0\0\0\0\0\x1f\x1eHTB{GoTtgUnyze9Psw4
SF:vGjcuMpHRp}\xc0\x0c\0\x02\0\x03\0\0\0\0\0\x02\xc0\x0c")%r(DNSStatusRequ
SF:est,C,"\0\0\x90\x04\0\0\0\0\0\0\0\0")%r(DNS-SD,101,"\0\0\x80\x80\0\x01\
SF:0\0\0\r\0\0\t_services\x07_dns-sd\x04_udp\x05local\0\0\x0c\0\x01\0\0\x0
SF:2\0\x01\x006\xee\x80\0\x14\x01D\x0cROOT-SERVERS\x03NET\0\0\0\x02\0\x01\
SF:x006\xee\x80\0\x04\x01J\xc0;\0\0\x02\0\x01\x006\xee\x80\0\x04\x01C\xc0;
SF:\0\0\x02\0\x01\x006\xee\x80\0\x04\x01I\xc0;\0\0\x02\0\x01\x006\xee\x80\
SF:0\x04\x01A\xc0;\0\0\x02\0\x01\x006\xee\x80\0\x04\x01H\xc0;\0\0\x02\0\x0
SF:1\x006\xee\x80\0\x04\x01B\xc0;\0\0\x02\0\x01\x006\xee\x80\0\x04\x01K\xc
SF:0;\0\0\x02\0\x01\x006\xee\x80\0\x04\x01M\xc0;\0\0\x02\0\x01\x006\xee\x8
SF:0\0\x04\x01F\xc0;\0\0\x02\0\x01\x006\xee\x80\0\x04\x01G\xc0;\0\0\x02\0\
SF:x01\x006\xee\x80\0\x04\x01L\xc0;\0\0\x02\0\x01\x006\xee\x80\0\x04\x01E\
SF:xc0;")%r(RPCCheck,C,"r\xfe\x98\x01\0\0\0\0\0\0\0\0")%r(NBTStat,105,"\x8
SF:0\xf0\x80\x90\0\x01\0\0\0\r\0\0\x20CKAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\0\0
SF:!\0\x01\0\0\x02\0\x01\x006\xee\x80\0\x14\x01I\x0cROOT-SERVERS\x03NET\0\
SF:0\0\x02\0\x01\x006\xee\x80\0\x04\x01L\xc0\?\0\0\x02\0\x01\x006\xee\x80\
SF:0\x04\x01J\xc0\?\0\0\x02\0\x01\x006\xee\x80\0\x04\x01G\xc0\?\0\0\x02\0\
SF:x01\x006\xee\x80\0\x04\x01C\xc0\?\0\0\x02\0\x01\x006\xee\x80\0\x04\x01A
SF:\xc0\?\0\0\x02\0\x01\x006\xee\x80\0\x04\x01B\xc0\?\0\0\x02\0\x01\x006\x
SF:ee\x80\0\x04\x01E\xc0\?\0\0\x02\0\x01\x006\xee\x80\0\x04\x01H\xc0\?\0\0
SF:\x02\0\x01\x006\xee\x80\0\x04\x01F\xc0\?\0\0\x02\0\x01\x006\xee\x80\0\x
SF:04\x01D\xc0\?\0\0\x02\0\x01\x006\xee\x80\0\x04\x01K\xc0\?\0\0\x02\0\x01
SF:\x006\xee\x80\0\x04\x01M\xc0\?");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.40 seconds
```
``` 
sudo nmap -sU -sC 10.129.171.114 -p 53
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-24 15:42 -0400
Nmap scan report for 10.129.171.114
Host is up (0.28s latency).

PORT   STATE SERVICE
53/udp open  domain
| dns-nsid: 
|_  bind.version: HTB{GoTtgUnyze9Psw4vGjcuMpHRp}

Nmap done: 1 IP address (1 host up) scanned in 11.79 seconds

```

- Alvo:10.129.171.114
- Hostname: nix-nmap-medium
- So : Linux(Ubuntu)
- Samba: 4.7.6( emulando windows 6.1)
- Lições aprendidas: Quando a gente bypassa um IDS/IPS utilizando -sS , -sA , -sU ou o que seja podemo utilizar o -sC ou -sV caso queiramos mais informações mas devemos levar em consideração que eles dois só são executados se as portas forem entedidas como abertas , logo -sS com -sC não executou os scripts corretamente na porta 53 , visto que foi dada como filtrada e somente com -sU ela foi dada como aberta , o nmap entendeu como executaria o -sC , que seria com pacotes udp que é o que a porta permite e assim descobrimos o banner e isso serve também para -sV.



