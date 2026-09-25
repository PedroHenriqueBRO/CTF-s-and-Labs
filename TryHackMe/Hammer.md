# 🎯 Relatório de Invasão:  Hammer

> [!abstract] Resumo 
> **Objetivo:** Iremos explorar uma aplicação web que possui mecanismos de autenticação , devemos burlar eles.
> **Vetor de Acesso Inicial:** Rota reset_password.php vulnerável permitindo que trocassemos a senha de uma conta e acessássemos o dashboard com ela .
> **Vetor de Elevação de Privilégios:** Comando de reverse shell na rota de RCE ao modifcar o JWT para sermos admin.

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- ffuf
- jwted.py
- cracking_mfa.py
- penelope
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap -sS -p1-10000

| PORT | SERVICE |
| ---- | ------- |
| 22   | ssh     |
| 1337 | http    |

> [!example] Output do Nmap com -sC e -sV -p22,1337
> 

| PORT | SERVICE/VERSION                                                       | INFOS                                                                                                                                                                                                                              |
| ---- | --------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 22   | ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0) | \| ssh-hostkey: <br>\|   3072 d3:aa:e4:80:e6:56:28:d5:6c:a2:95:57:b7:11:24:05 (RSA)<br>\|   256 1e:7f:1c:89:49:e5:9e:da:7f:a5:44:be:e2:ef:20:c8 (ECDSA)<br>\|_  256 78:72:c6:55:86:9a:fd:30:d1:5f:3e:2d:73:e4:82:62 (ED25519)      |
| 1337 | http    Apache httpd 2.4.41 ((Ubuntu))                                | \|_http-title: Login<br>\|_http-server-header: Apache/2.4.41 (Ubuntu)<br>\| http-cookie-flags: <br>\|   /: <br>\|     PHPSESSID: <br>\|_      httponly flag not set<br>Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel<br> |

> [!info] Análise do Reconhecimento
>Vemos aqui um servidor rodando um serviço ssh na porta 22 sem exploit para a versão.
>Além disso um servidor web apache rodando na porta 1337 com httponly não setado permitindo XSS para acessar cookies.

---

## 🌐 2. Enumeração Web & VHOSTs

> [!quote] Dados e reconhecimento
>

| Path's             | Análise                                                                                                                                |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| index.php          | Página inicial de login contendo informações no page source dizendo sobre a convenção de nomes dos diretórios , dizendo que é hmr_DIR. |
| javascript/        | Sem permissão de acesso                                                                                                                |
| logout.php         | Página de logout                                                                                                                       |
| vendor/            | Indica que o servidor alvo utiliza de tokens jwt .                                                                                     |
| config.php         | Não foi preciso nessa sala.                                                                                                            |
| dashboard.php      | Página após faz login , havia um formulário que só podíamos como user utilizar do comando ls.11                                        |
| phpmyadmin         | Página de administração do banco de dados php.                                                                                         |
| reset_password.php | Rota para resetar senha , onde podemos usar ele para enumerar usuários .                                                               |
### index.php
![[Pasted image 20260924205048.png]]
![[Pasted image 20260924205104.png]]

### ffuf
/hmr_FUZZ
![[Pasted image 20260924205253.png]]
/FUZZ
![[Pasted image 20260924205759.png]]

### hmr_logs
![[Pasted image 20260924210043.png]]
Por essa foto vemos a existência de um usuário chamado tester@hammer.thm, isso nos da dois indicativos que temos um usuário talvez válido e o nome do domínio que podemos usar para enumerar usuários.

### reset_password.php
![[Pasted image 20260924210550.png]]
![[Pasted image 20260924210559.png]]
Aqui vemos que ao colocar um email válido iremos ter x segundos para colocar o código . Depois de alguns testes vemos que a rota de reset_password.py tem rate limiting , porém somente para um PHPSESSID na qual permitiu a gente criar um script que solicitava o reset e fazia 6 envios de otp , depois matava a sessão , criava outra com outro PHPSESSID e fazia mais 6 envios , assim verificando se o tamanho da resposta era diferente de 2202 (resposta com token inválido) a gente recuperou o PHPSESSID dessa sessão e usamos ele para criar a nova senha e entrar com tester@hammer.thm. Passo a passo na exploração.

### Dashboard.php
![[Pasted image 20260925112628.png]]
![[Pasted image 20260925112638.png]]
![[Pasted image 20260925112655.png]]
![[Pasted image 20260925112826.png]]
Vemos que pelo dashboard a gente consegue usar o ls e com isso recebemos uma chave.Junto a isso olhando os cookies a percebemos que o token é um jwt , podemos olhar o conteúdo dele.
#### jwted.py
Vendo o conteúdo do jwt nos devolvido percebemos o seguinte.
![[Pasted image 20260925113012.png]]
O token contém um kid que mostra onde está a chave que o assinou , sendo uma vulnerabilidade já que qualquer um pode ler seu próprio token e saber por ele onde está a chave no servidor . Se pudessemos usar cat , more , less ou algo do tipo voltando diretórios com ../ poderíamos ler essa chave , porém a ideia aqui é testar se a chave 188ade1.key está listada como um kid válido e trocar o role de user para admin.Essa exploração foi completa e com sucesso , passo a passo na exploração.

---

## ⚡ 3. Exploração & Acesso Inicial (Foothold)

>[!failure] Vulnerabilidade encontrada
>**Título**:Rate Limiting falho em reset_password.php
>**Risk Rating**:10 DREAD
>**Resumo**: Através de um exploit criado para essa vulnerabilidade a gente consegue para qualquer usuários válido recuperar sua senha fazendo um brute forcing no reset_password, através de uma falha que o número de request para a rota é contabilizado pelo PHPSESSID , com isso é possível criar um script que burla isso criando novas sessões , manda requisição para resetar password e faz 6 tentativas de brute forcing de otp . Com isso quando uma response retorna um content lenght de body diferente de 2202 a gente printa o response.request.headers e recupera o PHPSESSID e troca o atual no navegador por ele e acessamos reset_password para colocar uma senha de nossa vontade.
>**Background(Contexto adicional)**: Rate limiting é uma restrição utilizada em rotas para limitar a quantidade de requests que um usuário pode mandar em um certo intervalo de tempo.
>**Impacto**: Para qualquer usuário válido podemos executar esse exploit e trocar a senha dele , assim permitindo acesso a qualquer conta.
>**Conselhos e Remediações**:O rate limiting deve ser aplicado ao ip do usuário que manda a requisição e não no PHPSESSID.
>**Detalhes Técnicos e Evidências**:
1. Primeiro deve se acessar a rota de reset_password.php.
![[Pasted image 20260925120513.png]]
2. Digite um valor válido de email e aperte submit.
![[Pasted image 20260925120631.png]]
3. Aqui entramos na etapa de enviar os otps , iremos enviar 1 muita vezes.
![[Pasted image 20260925120717.png]]
4. Enviamos 7 vezes e na oitava deu rate limiting , com isso iremos apagar nosso PHPSESSID e recarregar.
![[Pasted image 20260925120809.png]]
5. Após apagar e resetar a página ela nos devolve um novo PHPSESSID , com ele podemos fazer mais 7 tentativas , e depois mais 7 , mais 7 e vai indo , porém fazer isso manualmente iria demorar muito , com isso o seguinte script foi criado.
![[Pasted image 20260925121014.png]]
6. Esse script basicamente utiliza um for começando de 1000 e pula 6 a cada iteração e dentro dele é criado uma sessão onde fazer um post solicitando resetar senha e depois um for aninhado dentro que faz de i até i+6 , assim enviamos 6 otps ao total para cada sessão. Com isso verificamos o tamanho do content da response e se for diferente de 2202 (opt inválido) a gente printa o header da request dessa response e o i (opt).Execução a baixo.
![[Pasted image 20260925121402.png]]
7. Recuperamos o token referente a esse pedido de reset , agora copiamos e colamos ele no cookie correspondente no storage e resetamos a página.
![[Pasted image 20260925121517.png]]
8. Vou colocar a senha pass12 , confirmar e logar.
![[Pasted image 20260925121613.png]]
9. Com isso exploit feito e pode ser executado para qualquer usuário válido , assim permitindo o roubo de qualquer conta.

>[!failure] Vulnerabilidade encontrada
>**Título**: Chave de assinatura de jwt exposta / servidor aceitando mais de uma chave de assinatura
>**Risk Rating**: 10 DREAD
>**Resumo**: Uma chave de assinatura foi encontrada no path / da aplicação , permitindo que assinássemos um jwt arbitrário de role admin e especificando o kid para o path dela, assim permitindo acesso a rota de RCE como admin.
>**Background(Contexto adicional)**: Uma chave de assinatura é utilizada para criar uma assinatura para um header.payload de um jwt (composto por header.payload.signature) e assim permitindo que jwt modificados ,sem consciência da chave de assinatura para criar uma nova assinatura, sejam invalidados e servidores podem acabar aceitando uma chave somente uma ou mais de uma .
>**Impacto**: Com a posse de uma chave de assinatura válida podemos assinar qualquer jwt arbitrário derivado do original devolvido pela plataforma e mudar valores para nos tornar admin , assim permitindo controle das funcionalidades do servidor web de forma totalmente privilegiada.
>**Conselhos e Remediações**: Um servidor pode aceitar mais de uma chave ( embora isso aumente a superfície de ataque) porém essas chaves não devem estar em lugares expostos, com isso essa chave exposta deve ser trocada de lugar e de valor.
>**Detalhes Técnicos e Evidências**:
1. Ao logar e ir pro dashboard podemos olhar no storage do navegador a presença de um token , esse token é um jwt que podemos decodificar.
 ![[Pasted image 20260925122924.png]]
 2. Pegamos esse token e usamos o script jwted.py para decodificar ele .
![[Pasted image 20260925123107.png]]
3. Vemos o kid da chave jwt que iremos modificar para o path de onde achamos a outra chave, iremos mudar o role de user para admin e assim exploit concluído.
![[Pasted image 20260925135127.png]]
4. Aqui mudamos os valores do jwt para role:admin , o exp e o kid para apontar para onde a chave estava.
![[Pasted image 20260925135419.png]]
5. Em seguida a gente assina o novo jwt e pegamos o jwt gerado para utilizar na rota de RCE.
![[Pasted image 20260925135311.png]]
6. Com essa chave assinada a gente pode acessar a rota de RCE e executar id que é só permitido para admin.

>[!failure] Vulnerabilidade encontrada
>**Título**:RCE
>**Risk Rating**:10 (DREAD)
>**Resumo**: Quando somos admin temos a mesma funcionalidade de user para enviar comandos para o servidor mas com permissões diferentes , com isso tendo acesso como admin conseguimos fazer um reverse shell pelo RCE.
>**Background(Contexto adicional)**: RCE é a capacidade de pela interface web usar um formulário que a entrada dele é um comando que é processado pelo servidor e nos devolve uma resposta.
>**Impacto**:Com esse RCE conseguimos acesso ao servidor como www-data e poderíamos exfiltrar todos os dados permitidos a esse usuário e tentar escalar privilégios( Porém não era o objetivo dessa sala , como www-data conseguimos recuperar a flag em /home/ubuntu).
>**Conselhos e Remediações**: Limitar a capacidade de executar comandos mesmo como admin , comandos que podem criar conexões com outro host (nc, mkfifo e bash com o tcp embutido do /dev/tcp ) devem ser filtrados e proibidos. Além de que o RCE permitia cd .. , isso deve ser filtrado também para que algo fique somente no contexto do diretório raiz onde é executado os comandos.
>**Detalhes Técnicos e Evidências**:
7. Supondo que já executamos a mudança do JWT para admin e assinamos o jwt , agora podemos enviar comandos como admin.
![[Pasted image 20260925114214.png]]
![[Pasted image 20260925114242.png]]
2. Confirmamos o RCE como admin dando ls para testar o token e depois id( não era permitido como user), agora podemos iniciar um penelope ouvindo na porta 4444 e executar um bash.
![[Pasted image 20260925114420.png]]
3. Agora é só clicar em send e esperar a conexão no penelope.
![[Pasted image 20260925112557.png]]
4. Entramos como www-data e foi possível recuperar a flag.

---

## 📚 Considerações Técnicas & Cheat Sheet

> [!info] Lições Aprendidas
> 1. Quando envolver jwt mudar o token no Authorization: Bearer , esqueci de fazer isso e fiquei sendo negado de fazer RCE como admin.

