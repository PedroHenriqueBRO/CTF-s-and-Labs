# Tópico

# Tags

#Hackthebox 

# Resumo
Foi nos dado um ip e foi pedido que utilizassemos ele em pesquisas dig para enumerar informações sobre o domínio inlanefreight.htb e nisso comecei utilizando ele para any.
```
dig any inlanefreight.htb @10.129.42.195

; <<>> DiG 9.20.20-1-Debian <<>> any inlanefreight.htb @10.129.42.195
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52671
;; flags: qr aa rd; QUERY: 1, ANSWER: 5, AUTHORITY: 0, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: d479e04c47150fee010000006a8e42287a9561da8e4227b8 (good)
;; QUESTION SECTION:
;inlanefreight.htb.             IN      ANY

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.      604800  IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.      604800  IN      TXT     "MS=ms97310371"
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       127.0.0.1

;; Query time: 500 msec
;; SERVER: 10.129.42.195#53(10.129.42.195) (TCP)
;; WHEN: Tue Aug 25 21:32:25 EDT 2026
;; MSG SIZE  rcvd: 437

```

A partir disso consegui descobrir o FQDN para o domínio nos dado , a resposta é ns.inlanefreight.htb , pois ele é do tipo A e é um domínio bem qualificado tendo o tld , domain e subdomain/host.
Depois disso pediram para ver se era possívei fazer uma transferência de zona , nisso executei o comando a cima mas trocando de any para axfr.
``` 
dig axfr inlanefreight.htb @10.129.42.195

; <<>> DiG 9.20.20-1-Debian <<>> axfr inlanefreight.htb @10.129.42.195
;; global options: +cmd
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      TXT     "MS=ms97310371"
inlanefreight.htb.      604800  IN      TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.      604800  IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
dev.inlanefreight.htb.  604800  IN      A       10.12.0.1
internal.inlanefreight.htb. 604800 IN   A       10.129.1.6
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
ns.inlanefreight.htb.   604800  IN      A       127.0.0.1
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 500 msec
;; SERVER: 10.129.42.195#53(10.129.42.195) (TCP)
;; WHEN: Tue Aug 25 21:37:34 EDT 2026
;; XFR size: 11 records (messages 1, bytes 560)

```
O despejo de um SOA no início e depois SOA no final envelopando os registros restantes no meio indica um sucesso na tranferência de zona.Ainda sim não cheguei no objetivo , mas nisso vendo o internal ali utilizei ele para fazer outro axfr.
```
dig axfr internal.inlanefreight.htb @10.129.42.195

; <<>> DiG 9.20.20-1-Debian <<>> axfr internal.inlanefreight.htb @10.129.42.195
;; global options: +cmd
internal.inlanefreight.htb. 604800 IN   SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
internal.inlanefreight.htb. 604800 IN   TXT     "MS=ms97310371"
internal.inlanefreight.htb. 604800 IN   TXT     "HTB{DN5_z0N3_7r4N5F3r_iskdufhcnlu34}"
internal.inlanefreight.htb. 604800 IN   TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
internal.inlanefreight.htb. 604800 IN   TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
internal.inlanefreight.htb. 604800 IN   NS      ns.inlanefreight.htb.
dc1.internal.inlanefreight.htb. 604800 IN A     10.129.34.16
dc2.internal.inlanefreight.htb. 604800 IN A     10.129.34.11
mail1.internal.inlanefreight.htb. 604800 IN A   10.129.18.200
ns.internal.inlanefreight.htb. 604800 IN A      127.0.0.1
vpn.internal.inlanefreight.htb. 604800 IN A     10.129.1.6
ws1.internal.inlanefreight.htb. 604800 IN A     10.129.1.34
ws2.internal.inlanefreight.htb. 604800 IN A     10.129.1.35
wsus.internal.inlanefreight.htb. 604800 IN A    10.129.18.2
internal.inlanefreight.htb. 604800 IN   SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 504 msec
;; SERVER: 10.129.42.195#53(10.129.42.195) (TCP)
;; WHEN: Tue Aug 25 21:54:28 EDT 2026
;; XFR size: 15 records (messages 1, bytes 677)
```

Assim respondi mais duas perguntas devido a flag ali no TXT e o ip de dc1.
No fim tive de passar por vários brute forces nesses subdomains e achei o ip terminado em 203 ao verificar o dev.inlanefreight.htb
``` 
gobuster dns --domain dev.inlanefreight.htb --resolver 10.129.174.239 -w /usr/share/seclists/Discovery/DNS/fierce-hostlist.txt -t 100 --no-error
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     dev.inlanefreight.htb
[+] Threads:    100
[+] Resolver:   10.129.174.239
[+] Timeout:    1s
[+] Wordlist:   /usr/share/seclists/Discovery/DNS/fierce-hostlist.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
Progress: 0 / 1 (0.00%)[INFO] [-] Unable to validate base domain: dev.inlanefreight.htb (lookup dev.inlanefreight.htb on 10.0.2.3:53: no such host)
dev1.dev.inlanefreight.htb 10.12.3.6
ns.dev.inlanefreight.htb 127.0.0.1
win2k.dev.inlanefreight.htb 10.12.3.203
Progress: 2280 / 2280 (100.00%)
===============================================================
Finished
===============================================================

```
A resposta é win2k.dev.inlanefreight.htb . 
