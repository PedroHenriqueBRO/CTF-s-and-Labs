# Tópico

# Tags

#Hackthebox 

# Resumo
Agora vamos ser práticos. Uma empresa nos contratou para testar suas defesas de segurança de TI, incluindo suas `IDS`e `IPS`sistemas. Nosso cliente quer aumentar sua segurança de TI e, portanto, fará melhorias específicas em sua `IDS/IPS`sistemas após cada teste bem sucedido. Não sabemos, no entanto, segundo quais diretrizes essas mudanças serão feitas. Nosso objetivo é descobrir informações específicas das situações dadas.

Nós só somos fornecidos com uma máquina protegida por `IDS/IPS`sistemas e pode ser testado. Para fins de aprendizagem e para ter uma ideia de como `IDS/IPS`pode se comportar, temos acesso a uma página web de status em:

http://alvo/status.php

Esta página mostra-nos o número de alertas. Sabemos que se recebermos uma quantidade específica de alertas, seremos `banned`. Portanto, temos que testar o sistema de destino como `quietly`o mais possível.


## Tentativas
Mandei via -sS e com/sem --source-port 53 , deu que o sistema final utilizava linux mas não era. Depois mandei um -sA mas deu somente resultados de 95% , impossibilitando que eu saiba realmente qual deles era realmente o SO.
``` 
sudo nmap -sS -O 10.129.114.181 -F         
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-23 10:17 -0400
Nmap scan report for 10.129.114.181
Host is up (0.18s latency).
Not shown: 94 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
110/tcp open  pop3
139/tcp open  netbios-ssn
143/tcp open  imap
445/tcp open  microsoft-ds
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.10 seconds

```
Mas provavlemente essa resposta vem de padrão para enganar os digitalizadores que utilizam o -sS.
``` 
sudo nmap -sA -O 10.129.114.181 -p 139,445 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-23 10:19 -0400
Nmap scan report for 10.129.114.181
Host is up (0.17s latency).

PORT    STATE      SERVICE
139/tcp unfiltered netbios-ssn
445/tcp unfiltered microsoft-ds
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Aruba IAP-105 WAP (95%), AVM FRITZ!Box FON WLAN 7240 WAP (95%), Belkin N600 DB WAP (95%), Buffalo LinkStation NAS device (95%), Buffalo LS-WXL NAS device (95%), Check Point VPN-1 UTM appliance (95%), Cisco CP 8945 VoIP phone (95%), D-Link DSR-1000N WAP (95%), EnGenius ESR-9250 WAP (95%), Android 2.3.5 (Linux 2.6.35) (95%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.75 seconds

```
Aqui tentamos via -sA como eu havia dito e gerou vários SO com 95% de chance de ser , vou tentar usa -sC em serviços específicos para tentar achar.
```
sudo nmap -sS -sC 10.129.114.181 -p 139     
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-23 10:21 -0400
Nmap scan report for 10.129.114.181
Host is up (0.14s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: nix-nmap-easy
|   NetBIOS computer name: NIX-NMAP-EASY\x00
|   Domain name: \x00
|   FQDN: nix-nmap-easy
|_  System time: 2026-08-23T16:21:12+02:00
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_clock-skew: mean: -40m00s, deviation: 1h09m16s, median: -1s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: NIX-NMAP-EASY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-time: 
|   date: 2026-08-23T14:21:12
|_  start_date: N/A

Nmap done: 1 IP address (1 host up) scanned in 17.67 seconds
```
Depois dessa tentativa fui banido mas consegui a resposta , não estava entre nenhuma das tentativas de 95% , aqui a análise minha de primeira foi errada , achei que era um windows 6.1 , mas devemos analisar como esse samba nos responde nesse service do netbios , ele basicamente diz que está emulando o comportamento de um windows 6.1 , equivalente ao windows 7 , para manter compatibilidade , sendo ele (o samba) de versão 4.7.6 e rodando em um server ubuntu.

Basicamente a primeira análise com -sS foi correta pois era um ambiente linux porque mediu a pilha tcp/ip do kernel , mas a partir disso só não conseguiu dizer corretamente a distribuição que era o que a atividade pedia.

O ack não funcionou nada bem pois ele precisa encontrar ao menos um serviço que responda se a porta está fechada e outro que está aberta para o -O funcionar assim medindo a diferença de resposta da pilha de rede.

- Alvo:10.129.114.181
- Hostname: nix-nmap-ease
- So : Linux(Ubuntu)
- Kernel : 4.15 - 5.19
- Samba: 4.7.6( emulando windows 6.1)
- Lições aprendidas: -O no -sS acertou o sistema mas não conseguiu descobrir a distribuição.A flag -O não deve ser combinada com -sA , pois a ausência de uma porta como open invalida a precisão do nmap para dizer qual o SO.



