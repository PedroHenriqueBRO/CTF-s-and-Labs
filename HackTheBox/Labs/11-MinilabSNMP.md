# Tópico

# Tags

#Hackthebox 

# Resumo
Fui instruído a enumerar um serviço snmp na maquina de destino e fiz isso primeiramente com o snmpwalk que enumera os OID's através de uma chave que a gente passa para ele .
``` 
snmpwalk -v2c -c public 10.129.42.195                
iso.3.6.1.2.1.1.1.0 = STRING: "Linux NIX02 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (38887) 0:06:28.87
iso.3.6.1.2.1.1.4.0 = STRING: "devadmin <devadmin@inlanefreight.htb>"
iso.3.6.1.2.1.1.5.0 = STRING: "NIX02"
iso.3.6.1.2.1.1.6.0 = STRING: "InFreight SNMP v0.91"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.1.8.0 = Timeticks: (5) 0:00:00.05
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
iso.3.6.1.2.1.1.9.1.4.1 = Timeticks: (4) 0:00:00.04
iso.3.6.1.2.1.1.9.1.4.2 = Timeticks: (4) 0:00:00.04
iso.3.6.1.2.1.1.9.1.4.3 = Timeticks: (4) 0:00:00.04
iso.3.6.1.2.1.1.9.1.4.4 = Timeticks: (4) 0:00:00.04
iso.3.6.1.2.1.1.9.1.4.5 = Timeticks: (4) 0:00:00.04
iso.3.6.1.2.1.1.9.1.4.6 = Timeticks: (4) 0:00:00.04
iso.3.6.1.2.1.1.9.1.4.7 = Timeticks: (4) 0:00:00.04
iso.3.6.1.2.1.1.9.1.4.8 = Timeticks: (4) 0:00:00.04
iso.3.6.1.2.1.1.9.1.4.9 = Timeticks: (5) 0:00:00.05
iso.3.6.1.2.1.1.9.1.4.10 = Timeticks: (5) 0:00:00.05
iso.3.6.1.2.1.2.1.0 = INTEGER: 2
...
```
Com essa parte conseguimos responder duas perguntas que são elas a versao personalizada do snmp (InFreight SNMP v0.91) e o email do admin(devadmin@inlanefreight.htb).
Devemo enumerar o script personalizado que está rodando no sistema.
```
iso.3.6.1.2.1.25.1.7.1.2.1.2.4.70.76.65.71 = STRING: "/usr/share/flag.sh"
iso.3.6.1.2.1.25.1.7.1.2.1.3.4.70.76.65.71 = ""
iso.3.6.1.2.1.25.1.7.1.2.1.4.4.70.76.65.71 = ""
iso.3.6.1.2.1.25.1.7.1.2.1.5.4.70.76.65.71 = INTEGER: 5
iso.3.6.1.2.1.25.1.7.1.2.1.6.4.70.76.65.71 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.2.1.7.4.70.76.65.71 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.2.1.20.4.70.76.65.71 = INTEGER: 4
iso.3.6.1.2.1.25.1.7.1.2.1.21.4.70.76.65.71 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.3.1.1.4.70.76.65.71 = STRING: "HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}"
iso.3.6.1.2.1.25.1.7.1.3.1.2.4.70.76.65.71 = STRING: "HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}"
iso.3.6.1.2.1.25.1.7.1.3.1.3.4.70.76.65.71 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.3.1.4.4.70.76.65.71 = INTEGER: 0
iso.3.6.1.2.1.25.1.7.1.4.1.2.4.70.76.65.71.1 = STRING: "HTB{5nMp_fl4g_uidhfljnsldiuhbfsdij44738b2u763g}"
iso.3.6.1.2.1.25.2.2.0 = INTEGER: 2034728

```
Depois de um tempo executando apareceu esses resultados e aí utilizei essa flag para responder a última pergunta.



