## Alvo
- IP -> 10.129.76.211

## Objetivo
-  Enumerar o alvo usando os conceitos ensinados nesta seção. Liste o nome do host do servidor MSSQL. -> ILF-SQL-01
-  Conecte-se à instância do MSSQL em execução no destino usando a conta (backdoor:Password1) e, em seguida, liste o banco de dados não padrão presente no servidor. -> Employees 

## Ferramentas e scripts utilizados
- Nmap
- impacket-mssqlclient

## Resultados obtidos
- ```
	sudo nmap -sS 10.129.76.211
	Nmap scan report for 10.129.76.211
	Host is up (0.32s latency).
	Not shown: 994 closed tcp ports (reset)
	PORT     STATE SERVICE
	135/tcp  open  msrpc
	139/tcp  open  netbios-ssn
	445/tcp  open  microsoft-ds
	1433/tcp open  ms-sql-s
	3389/tcp open  ms-wbt-server
	5985/tcp open  wsman
	
	Nmap done: 1 IP address (1 host up) scanned in 287.91 seconds

  ```
- ```
	sudo nmap -sV -sC -p 1433 10.129.76.211
	Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-27 11:52 -0400
	Nmap scan report for 10.129.76.211
	Host is up (0.26s latency).
	
	PORT     STATE SERVICE  VERSION
	1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM
	| ms-sql-info: 
	|   10.129.76.211\MSSQLSERVER: 
	|     Instance name: MSSQLSERVER
	|     Version: 
	|       name: Microsoft SQL Server 2019 RTM
	|       number: 15.00.2000.00
	|       Product: Microsoft SQL Server 2019
	|       Service pack level: RTM
	|       Post-SP patches applied: false
	|     TCP port: 1433
	|     Named pipe: \\10.129.76.211\pipe\sql\query
	|_    Clustered: false
	| ms-sql-ntlm-info: 
	|   10.129.76.211\MSSQLSERVER: 
	|     Target_Name: ILF-SQL-01
	|     NetBIOS_Domain_Name: ILF-SQL-01
	|     NetBIOS_Computer_Name: ILF-SQL-01
	|     DNS_Domain_Name: ILF-SQL-01
	|     DNS_Computer_Name: ILF-SQL-01
	|_    Product_Version: 10.0.17763
	| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
	| Not valid before: 2026-08-27T15:43:11
	|_Not valid after:  2056-08-27T15:43:11
	|_ssl-date: 2026-08-27T15:52:55+00:00; 0s from scanner time.
	
	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 14.35 seconds

  ```
- ```
	impacket-mssqlclient backdoor:'Password1'@10.129.76.211 -windows-auth
	Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 
	
	[*] Encryption required, switching to TLS
	[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
	[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
	[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
	[*] INFO(ILF-SQL-01): Line 1: Changed database context to 'master'.
	[*] INFO(ILF-SQL-01): Line 1: Changed language setting to us_english.
	[*] ACK: Result: 1 - Microsoft SQL Server 2019 RTM (15.0.2000)
	[!] Press help for extra shell commands
	SQL (ILF-SQL-01\backdoor  dbo@master)> select name from sys.database;
	ERROR(ILF-SQL-01): Line 1: Incorrect syntax near the keyword 'database'.
	SQL (ILF-SQL-01\backdoor  dbo@master)> select name from sys.databases
	name        
	---------   
	master      
	tempdb      
	model       
	msdb        
	Employees   
	SQL (ILF-SQL-01\backdoor  dbo@master)> 
  ```
  
## Considerações
Primeiro utilizei o nmap com flag -sS para fazer a enumeração de portas e serviços , aí achei rodando na porta 1433 o mssql. Assim novamente utilizei o nmap para agora pegar a versão dele (-sV) e rodar scripts (-sC) , assim recebi os resultados a cima na qual com o campo Target_Name eu respondi a primeira pergunta.Em seguida loguei com as credenciais ,dadas pela seção , e acessei o mssql server com o script impacket-mssqlclient com autenticação windows(-windows-auth). No fim listei com o comando select name from sys.databases os nomes dos databases presentes , na qual os 4 primeiros são padrões dos servidores mssql e o Employees é o db que responde  a nossa pergunta.


