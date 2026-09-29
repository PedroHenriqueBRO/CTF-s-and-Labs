# 🎯 Relatório de Invasão: Injectics

> [!abstract] Resumo 
> **Objetivo:** Explorar a aplicação web injectics
> **Vetor de Acesso Inicial:** SQL injection no form de login .
> **Vetor de Elevação de Privilégios:** Dropar table users e conseguir logar com super admin com suas credenciais padrões e depois abusar de um camp que permitia SSTI e assim logarmos no servidor como www-data.

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- ffuf
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap -sS


![[Pasted image 20260928202319.png]]

> [!example] Output do Nmap -sU

![[Pasted image 20260928203626.png]]

> [!example] Output do Nmap com -sC e -sV
> 

![[Pasted image 20260928203859.png]]

> [!info] Análise do Reconhecimento
>Vemos aqui a presença de um host com ssh aberto na porta 22 e um servidor web na porta 80 indicando um servidor provavelmente php com apache (LAMP) rodando em um ubuntu server , sem http only permtindo roubo de cookies via XSS.

---

## 🌐 2. Enumeração Web & VHOSTs

`gobuster dir -u http://[IP] -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,log`

> [!quote] Dados e reconhecimento
>

![[Pasted image 20260928204934.png]]

### Página inicial
![[Pasted image 20260928204740.png]]
![[Pasted image 20260928204801.png]]
>[!info] Reconhecimento
>Vemos que no page source de index.php temos informações expostas nele , como um email (dev@injectics.thm) e que os emails são guardados em mail.log.
### mail.log
![[Pasted image 20260928205122.png]]

>[!info] Reconhecimento
>Aqui vemos a existência de dois emails sendo um deles já visto na page source da página inicial e agora vemos o email do superadmin. Porém essas credenciais foram mudadas , mas temos agora dois emails válidos para pode tentar fazer bypass de autenticação ou algo do tipo.Sabemos que tem um serviço rodando a cada minuto monitorando para adicionar credenciais padrões nos users do mail.log caso os dados do banco sejam corrompidos ou deletados. Vemos também que o nome da tabela é users.
### script.js
![[Pasted image 20260928212403.png]]

### dashboard.php
![[Pasted image 20260928235836.png]]

### edit_leaderboard.php
![[Pasted image 20260928235932.png]]

>[!info] Reconhecimento
>Aqui vemos uma rota de update para os registros dessa tabela no banco que faz controle da quantidade de medalhas de ouro , prata  e bronze para um certo país. Essa rota é vulnerável a sql injection permitindo que dropemos a tabela users e assim loguemos com a credencial padrão do superadmin.

### update_profile.php
![[Pasted image 20260929000148.png]]

>[!info] Reconhecimento
>Essa rota se mostrou vulnerável a SSTI ao permitirmos violar o twig e executarmos um payload de reverse shell.


---

## ⚡ 3. Exploração & Acesso Inicial (Foothold)

>[!failure] Vulnerabilidade encontrada
>**Título**: Injeção SQL 
>**Risk Rating**: 10 DREAD
>**Resumo**: O form de login possui vulnerabilidade de SQL injection onde somente certos filtros são aplicados na parte do cliente em forma de black list,assim não filtrando tudo.
>**Background(Contexto adicional)**: SQL injection é a capacidade de fazer uma entrada ,em um campo que recebe entrada de usuário, ser parte de uma consulta SQL podendo bypassar autenticação , recuperar dados e demais coisas.
>**Impacto**: Podemos logar sem utilizar das credenciais de login , permitindo logar e recuperar dados do usuário retornado e/ou executar solicitações em seu nome ao estar em sua conta.
>**Conselhos e Remediações**: Não utilizar de consultar sql em formato raw e sim usar solicitações statement com bind_param.
>**Detalhes Técnicos e Evidências**:

1. Primeiro vamos para o form de login.
![[Pasted image 20260928234251.png]]
2. Depois colocamos qualquer coisa no form de login com o burp ouvindo no proxy e mandamos a request , salvamos ela no repeater e mandamos esse payload no email. Assim fazemos login como user dev.
![[Pasted image 20260928222526.png]]
![[Pasted image 20260928234615.png]]

>[!failure] Vulnerabilidade encontrada
>**Título**: SQL injection 
>**Risk Rating**: 10 DREAD
>**Resumo**:Foi encontrado um sql injection na rota de edit_leaderboard.php onde podemos executar um sql injection nos campos de form para update do registro no banco de dados.
>**Background(Contexto adicional)**:SQL injection é a capacidade de fazer uma entrada ,em um campo que recebe entrada de usuário, ser parte de uma consulta SQL podendo bypassar autenticação , recuperar dados e demais coisas.
>**Impacto**: Dropamos a tabela users para resetar a senha do usuário superadmin e logar com ele no admin panel, permitindo que possamos usufruir de suas funcionalidades.
>**Conselhos e Remediações**: Qualquer entrada nesse formulário deve ser colocada na consulta sql como valor e não como parte da consulta SQL , exigindo que utilize consultas sql statement com bind_param.
>**Detalhes Técnicos e Evidências**:
1. Estamos no dashboard como dev.
![[Pasted image 20260928235015.png]]
2. Apertamos para editar qualquer registro.
![[Pasted image 20260928235032.png]]
3. No campo gold colocaremos o seguinte payload.
![[Pasted image 20260928235056.png]]
4. Apertamos update e o servidor crash dizendo que irá restaurar o banco para recuperar a tabela excluída.
![[Pasted image 20260928235114.png]]
5. Agora podemos logar como superadmin com a senha encontrada em mail.log
![[Pasted image 20260928235243.png]]
![[Pasted image 20260928235250.png]]


>[!failure] Vulnerabilidade encontrada
>**Título**: SSTI
>**Risk Rating**:10 DREAD 
>**Resumo**:Foi possível fazer um SSTI em um parâmetro firstname no update profile do admin usando o twig, assim conseguimos injetar um reverse shell
>**Background(Contexto adicional)**: Um SSTI é um injeção feita em mecanismos de template para gerar conteúdo dinâmico em uma página html , porém os dados que normalmente chegam nele não são bem tratados permitindo injeção de entradas maliciosas.
>**Impacto**: Com o SSTI foi possível gerar reverse shell e entrar no servidor web como www-data , possibilitando possíveis escalações de privilégio , exfiltração de dados de usuários e do contexto da aplicação web.
>**Conselhos e Remediações**:Esse campo não deve permitir que a entrada seja processada como código , assim se for utilizado {{}} na entrada então deve ser tratado como texto e não comando.
>**Detalhes Técnicos e Evidências**:

1. Quando logamos como admin aparecemos no dashboard
![[Pasted image 20260928233446.png]]
2.  Depois temos de ir em Profile.
![[Pasted image 20260928233533.png]]
3. Injetamos esse payload da foto a seguir onde ip pode ser substituído pelo ip da máquina da qual você está ouvindo e a porta pode ser modificada.
![[Pasted image 20260928233632.png]]
4. Apertamos submit e depois vamos para home.A tela irá traver e o shell será criado.
![[Pasted image 20260928233715.png]]

---

## 🔑 Tabela de Credenciais Capturadas

| Serviço / Local | Usuário                  | Senha / Hash         | Origem do Achado |
| :-------------- | :----------------------- | :------------------- | :--------------- |
| User web        | dev@injectics.thm        | devPasswd123         | mail.log         |
| User web        | superadmin@injectics.thm | superSecurePasswd101 | mail.log         |

---

## 📚 Considerações Técnicas & Cheat Sheet

> [!info] Lições Aprendidas
> 1. Testar sempre consultas codificadas e não codificadas , perdi muito tempo no sql injection para dropar users pois eu estava enviando as consultas de forma codificada e utilizando ' , gerando erros desnecessários.
> 2. Procurar sempre ver na aplicação se há campos dinâmicos que mudam de acordo com um valor mudado em um form por exemplo , o html pode estar usando mecanismos de template como twig e podem ser campos vulneráveis.