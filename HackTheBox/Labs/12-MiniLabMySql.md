## Alvo
- IP -> 10.129.178.31

## Objetivo
-  Enumere o servidor MySQL e determine a versão em uso. (Formato: MySQL X.X.XX) -> MySQL 8.0.27
-  Durante o nosso teste de penetração, encontramos credenciais fracas "robin:robin". Devemos tentar contra o servidor MySQL. Qual é o endereço de e-mail do cliente "Otto Lang"? -> 

## Ferramentas e scripts utilizados
- nmap
- mysql

## Resultados obtidos
-  ```
	sudo nmap -sS 10.129.178.31
	Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-27 10:36 -0400
	Nmap scan report for 10.129.178.31
	Host is up (0.43s latency).
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
	
	Nmap done: 1 IP address (1 host up) scanned in 7.04 seconds
  ```
- ```
	sudo nmap -sV -sC 10.129.178.31 -p 3306
	Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-27 10:39 -0400
	Nmap scan report for 10.129.178.31
	Host is up (0.84s latency).
	
	PORT     STATE SERVICE VERSION
	3306/tcp open  mysql   MySQL 8.0.27-0ubuntu0.20.04.1
	| ssl-cert: Subject: commonName=MySQL_Server_8.0.27_Auto_Generated_Server_Certificate
	| Not valid before: 2021-11-09T13:29:25
	|_Not valid after:  2031-11-07T13:29:25
	|_ssl-date: TLS randomness does not represent time
	| mysql-info: 
	|   Protocol: 10
	|   Version: 8.0.27-0ubuntu0.20.04.1
	|   Thread ID: 9
	|   Capabilities flags: 65535
	|   Some Capabilities: Support41Auth, LongPassword, DontAllowDatabaseTableColumn, SupportsLoadDataLocal, ConnectWithDatabase, FoundRows, IgnoreSpaceBeforeParenthesis, Speaks41ProtocolOld, IgnoreSigpipes, SupportsTransactions, SwitchToSSLAfterHandshake, LongColumnFlag, InteractiveClient, SupportsCompression, ODBCClient, Speaks41ProtocolNew, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
	|   Status: Autocommit
	|   Salt: g\x02\x10dLIi\x0Fxr(8+hj\x04\x1AD+I
	|_  Auth Plugin Name: caching_sha2_password
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 68.92 seconds

  ```
- ```
	mysql -u robin -probin -h 10.129.178.31 --skip-ssl-verify-server-cert
	Welcome to the MariaDB monitor.  Commands end with ; or \g.
	Your MySQL connection id is 19
	Server version: 8.0.27-0ubuntu0.20.04.1 (Ubuntu)
	
	Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
	
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
	
	MySQL [(none)]> show database;
	ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'database' at line 1
	MySQL [(none)]> show databases;
	+--------------------+
	| Database           |
	+--------------------+
	| customers          |
	| information_schema |
	| mysql              |
	| performance_schema |
	| sys                |
	+--------------------+
	5 rows in set (0.286 sec)
	
	MySQL [(none)]> use customers;
	Reading table information for completion of table and column names
	You can turn off this feature to get a quicker startup with -A
	
	Database changed
	MySQL [customers]> show tables;
	+---------------------+
	| Tables_in_customers |
	+---------------------+
	| myTable             |
	+---------------------+
	1 row in set (0.237 sec)
	
	MySQL [customers]> select * from myTable;
	+----+---------------------+------------------------------------------+--------------------+-------------+-------------------------------+-----------------------------------+---------------------+------+
	| id | name                | email                                    | country            | postalZip   | city                          | address                           | pan                 | cvv  |
	+----+---------------------+------------------------------------------+--------------------+-------------+-------------------------------+-----------------------------------+---------------------+------+
	|  1 | Emery Reyes         | diam.eu@icloud.htb                       | Spain              | 26-579      | Quảng Ngãi                    | 675-4432 Nunc Av.                 | 519358 9482346334   | 144  |
	|  2 | Kristen Trujillo    | tellus.id@google.htb                     | Costa R.htb        | 376420      | Chiclayo                      | 101-8154 Ac Rd.                   | 546871 777532 7590  | 125  |
	|  3 | Fletcher Jimenez    | lobortis@outlook.htb                     | Germany            | 3515        | Timaru                        | 9562 Dui, St.                     | 559 47883 93145 224 | 550  |
	|  4 | Boris Sharp         | donec@protonmail.htb                     | Pakistan           | 1317        | Jönköping                     | 728-7809 Cras Road                | 4716447833847468    | 536  |
	|  5 | Ruth Carson         | suspendisse.aliquet@yahoo.htb            | Pakistan           | 14945       | Oviedo                        | 324-8221 Ut Road                  | 5164782453566544    | 449  |
	|  6 | Caryn Porter        | neque@aol.htb                            | New Zealand        | 3798        | Vichy                         | Ap #770-5801 Donec Rd.            | 4916 475 64 6748    | 590  |
	|  7 | G.htbe Stein        | eget@google.htb                          | Spain              | 80125       | Jeonju                        | 305-2770 Lectus. Rd.              | 485 51263 38542 847 | 402  |
	|  8 | Emery Watson        | non@icloud.htb                           | Austria            | 23940       | Secunderabad                  | Ap #542-2284 Mauris, Street       | 4716 5544 1838 4955 | 411  |
	|  9 | Silas Holder        | metus@hotmail.htb                        | Netherlands        | 63-851      | San Cristóbal de la Laguna    | Ap #604-7295 Duis St.             | 5186 1252 1481 7760 | 863  |
	| 10 | Patrick Walls       | aliquet@yahoo.htb                        | Indonesia          | 776718      | Weelde                        | 118-5893 Rhoncus. Ave             | 513 51634 82351 610 | 430  |

  ```
- ``` 
	MySQL [customers]> select email from myTable where name like "Otto%";
	+---------------------+
	| email               |
	+---------------------+
	| ultrices@google.htb |
	+---------------------+
	1 row in set (0.241 sec)

  ```


## Considerações
Utilizei primeiramente o nmap com flag -sS para poder enumerar o host de destino e assim recuperei todas as portas e seus serviços rodando .Em seguida utilizei -sV junto do -sC para executar os scripts padrões em cima da porta do mysql (3306), com isso recuperei o banner de versão dele e respondi a primeira pergunta. 
Logo em seguida a questão me deu a ideia de logar com user robin e password robin , assim utilizei o comando mysql e loguei com essas credenciais. Dentro do servidor já eu utilizei o show databases para mostrar quais schemas estavam disponíveis (schema no caso de mysql , em outros sgbds como postgree e sql server um schema é um subgrupo ou namespace para organizar tabelas). 
Dei use no customers e usei o show tables para mostrar as tabelas , aí usei o select \* para mostrar tudo dessa tabela e depois fiz mais direcionado para o nome like Otto% recebendo de volta o email dele para responder a segunda pergunta.


