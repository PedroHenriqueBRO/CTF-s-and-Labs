# Tópico

# Tags

#Hackthebox 

# Resumo
Foi nos dado outro desafio mas agora difícil , na qual agora o sistema ta melhor verificado e devemos pegar as versões do serviços em execução.
- Alvo : 10.129.172.134
Vamos fazer um -sS inicial para entender como está o sistema inicialmente.
```
sudo nmap -sS 10.129.172.134
Nmap scan report for 10.129.172.134
Host is up (0.24s latency).
Not shown: 869 closed tcp ports (reset), 129 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 13.50 seconds
```
Vemos aqui que de 1000 portas 2 estão abertas , 869 fechadas e 129 filtradas , realmente o sistema está melhor defensivamente.

Vamos tentar utilizar o Ack e o UDP para visualizar mais resultados.
```
sudo nmap -sA 10.129.172.134 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-24 15:57 -0400
Nmap scan report for 10.129.172.134
Host is up (0.23s latency).
All 1000 scanned ports on 10.129.172.134 are in ignored states.
Not shown: 871 unfiltered tcp ports (reset), 129 filtered tcp ports (no-response)

Nmap done: 1 IP address (1 host up) scanned in 8.57 seconds
```
Aqui vemos que escaneamos ACK não estão funcionando pois estão em estados de ignore, na qual as unfiltered são portas que são acessíveis mas não conseguimos dizer se estão abertas ou fechadas , e as 129 portas filtradas continuaram não respondendo.

Vamos agora mandar pacotes UDP para tentar entender o comportamento em cima .
```
sudo nmap -sU -F 10.129.172.134
Nmap scan report for 10.129.172.134
Host is up (0.31s latency).
Not shown: 97 closed udp ports (port-unreach)
PORT    STATE         SERVICE
68/udp  open|filtered dhcpc
137/udp open          netbios-ns
138/udp open|filtered netbios-dgm
```
Aqui vemos que utilizar o UDP mostrou informações melhores , pois assim conseguimos ver mais uma porta aberta , as outras duas ainda podem estar abertas ou fechadas. Irei rodar o -sC na 137 para ver .
```
sudo nmap -sU -sC -p 137 10.129.172.134   
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-24 16:07 -0400
Stats: 0:00:03 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 100.00% done; ETC: 16:07 (0:00:00 remaining)
Nmap scan report for 10.129.172.134
Host is up (0.24s latency).

PORT    STATE SERVICE
137/udp open  netbios-ns
| nbns-interfaces: 
|   hostname: NIX-NMAP-HARD
|   interfaces: 
|_    10.129.172.134

Host script results:
|_nbstat: NetBIOS name: NIX-NMAP-HARD, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Nmap done: 1 IP address (1 host up) scanned in 11.67 seconds
```
Vemos aqui o hostname e interfaces que é mostrando o ip da interface pelo qual ja temos conhecimento. O -sV mostra o seguinte: 
``` 
sudo nmap -sU -sV -p 137 10.129.172.134 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-24 16:08 -0400
Nmap scan report for 10.129.172.134
Host is up (0.29s latency).

PORT    STATE SERVICE    VERSION
137/udp open  netbios-ns Samba nmbd netbios-ns (workgroup: WORKGROUP)
Service Info: Host: NIX-NMAP-HARD

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.64 seconds

```
Vemos que é um samba rodando no netbios .

Aqui rodamos o -sV e -sC para as portas 22 e 80 vistas antes para ver se achamos algo:
``` 
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sS -sV -p 22,80 10.129.172.134 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-24 16:10 -0400
Nmap scan report for 10.129.172.134
Host is up (0.34s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.17 seconds
                                                                                                                      
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sS -sC -p 22,80 10.129.172.134 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-24 16:11 -0400
Nmap scan report for 10.129.172.134
Host is up (0.23s latency).

PORT   STATE SERVICE
22/tcp open  ssh
| ssh-hostkey: 
|   2048 71:c1:89:90:7f:fd:4f:60:e0:54:f3:85:e6:35:6c:2b (RSA)
|   256 e1:8e:53:18:42:af:2a:de:c0:12:1e:2e:54:06:4f:70 (ECDSA)
|_  256 1a:cc:ac:d4:94:5c:d6:1d:71:e7:39:de:14:27:3c:3c (ED25519)
80/tcp open  http
|_http-title: Apache2 Ubuntu Default Page: It works

Nmap done: 1 IP address (1 host up) scanned in 11.27 seconds
```
Vemos informações normais , como o openssh rodando em ubuntu com versão 7.6p1 
e o servidor web é um apache 2.4.29. 
Rodei novamente o -sS mas agora sem flag de -p ou -F e rodei com --source-port 53 , achei na porta 50000 o serviço ibm-db2 , estou agora vendo formas de mostrar o serviço dele.
```
sudo nmap -sS --source-port 53 10.129.172.134
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-24 16:46 -0400
Stats: 0:00:13 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 100.00% done; ETC: 16:46 (0:00:00 remaining)
Nmap scan report for 10.129.172.134
Host is up (0.26s latency).
Not shown: 869 closed tcp ports (reset), 128 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
50000/tcp open  ibm-db2

Nmap done: 1 IP address (1 host up) scanned in 13.46 seconds
```
Porém o hachthebox estava mostrando ibm-db2 por motivos de firewall , o nmap estava somente associando a ibm porque não conseguia dizer certamente qual serviço estava rodando na porta e quando era utilizado -sV ou -sC dava timeout pois ele achando que era ibm mandava scripts errado para o serviço errado.
Nisso utilizei o ncat para conectar na porta e pegar a flag de versão.
```
sudo ncat -nv --source-port 53 10.129.172.166 50000                   
Ncat: Version 7.99 ( https://nmap.org/ncat )
Ncat: Connected to 10.129.172.166:50000.
220 HTB{kjnsdf2n982n1827eh76238s98di1w6}
```

- Alvo:10.129.172.166
- Hostname: NIX-NMAP-HARD
- So : Linux(Ubuntu)
- Lições aprendidas: Utilizamos o --source-port 53 para burlar o ids/ips , mas isso nos trouxe um dado incorreto pois o nmap nos devolveu que era ibm-db2 , sendo que não era e isso fez eu ficar perdido devido ao fato de que tentando rodar -sV e -sC nessa porta acabou gerando timeouts por scripts rodando em uma porta dada como serviço errado. A lição aprendida é se ela permite tráfego pela --source-port 53 e o nmap não está conseguindo lidar com resolver versão e script , podemos usar o sudo ncat com --source-port 53 no ip e porta indicada que provavelmente iremos conseguir o banner.