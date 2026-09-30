# 🎯 Relatório de Invasão: Include

> [!abstract] Resumo 
> **Objetivo:** Explorar a aplicação web Include
> **Cadeia de ataque completa**:
> Loguei como usuário guest e fui em editar seu perfil , executei mudança de isAdmin false para true através de modificação insegura de propriedade do objeto e assim virei admin. Como admin eu acessei um path que mostrava duas rotas de api , sendo uma delas a rota que devolvia os admins e suas senhas porém era localhost. 
> Com isso achei uma rota settings que permitia SSRF e assim coloquei a rota que devolvia as infos dos admins e recuperei nomes e senhas. A partir disso descobri uma aplicação chamada sysmon mas ela não estava dentro das portas 1 a 10000 , com isso procurei ver se essa aplicação estava rodando em uma porta mais alta , fiz usando ffuf através da rota de settings fazendo busca em localhost que consegue ser mais rápido que buscas com nmap para um ip externo. 
> Nisso eu descobri a aplicação sysmon rodando na porta 50000 e nela explorei uma falha de LFI na rota de ver a imagem do perfil , bypassei o filtro de path relativo e assim acessei arquivos dentro do servidor.
> Mas para executar algo relevante tiver de fazer Log Poison no arquivo /var/log/auth.log colocando um payload como nome de usuário ao tentar logar via ssh . Com o payload injetado eu simplesmente reiniciei a página e ele foi executado, mostrando o arquivo escondido que recuperei o conteúdo executando o LFI.

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- ffuf
- openssl
- netcat
- remina
- burp
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap -sS

![[Pasted image 20260930100126.png]]

> [!example] Output do Nmap com -sC e -sV
> 

![[Pasted image 20260930100406.png]]
![[Pasted image 20260930100422.png]]


> [!info] Análise do Reconhecimento
>Aqui vemos que temos um servidor web rodando na porta 4000 utilizando nodejs com express, o domínio é filepath.lab . Temos imap e pop3 rodando em outras portas , com suas versões normais e com TLS. Além disso temos um servidor ssh rodando na porta 22.
### IMAP & POP3 
![[Pasted image 20260930102246.png]]

### IMAPS & POP3S
#### IMAPS
![[Pasted image 20260930102606.png]]
#### POP3S
![[Pasted image 20260930102635.png]]

> [!info] Reconhecimento
> Foi visto que esses serviços não permitem login com o mesmo user da aplicação web (guest) e nem como anônimo. Irei explorar a aplicação web e voltar a esses serviços se caso não achar nada que eu explorar lá.

---

## 🌐 2. Enumeração Web & VHOSTs
```
ffuf -u http://10.65.156.83:4000/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 -e .js,.txt,.js.bak,.md -fs 1295
```
![[Pasted image 20260930103428.png]]
> [!quote] Dados e reconhecimento
>Encontramos uma rota que chama profile.php que o query param ?img= permite LFI e com isso o utilizamos para encadear um ataque de log poisoning com RCE.
### Index
![[Pasted image 20260930103502.png]]

### friend/id
![[Pasted image 20260930103814.png]]

### FFUF na rota de /admin/settings para enumerar portas
![[Pasted image 20260930115529.png]]

#### Enumeração Sysmon como administrator
![[Pasted image 20260930115928.png]]
##### index.php
![[Pasted image 20260930115952.png|719]]
##### login.php
![[Pasted image 20260930120117.png]]

##### templates
![[Pasted image 20260930120150.png]]

##### dashboard.php
![[Pasted image 20260930120239.png]]

##### profile.php
![[Pasted image 20260930134659.png]]

---

## ⚡ 3. Exploração & Acesso Inicial (Foothold)

>[!failure] Vulnerabilidade encontrada
>**Título**: Modificação de propriedade de forma insegura
>**Risk Rating**: 10 DREAD 
>**Resumo**: Na tela /friend/1 vemos o perfil do guest que é nosso usuário e vemos as propriedades do objeto dele , o isAdmin pode ser trocado de false para true através do formulário de recomenda atividade pro guest, permitindo que viremos admin.
>**Background(Contexto adicional)**: Essa vulnerabilidade parte do fato de que podemos alterar um valor já existente em um objeto de usuário de forma não intencional já que não deveria ser possível fazer isso.
>**Impacto**: Podemos como qualquer usuário virar admin e obter suas funcionalidades .
>**Conselhos e Remediações**: Deve se filtrar a entrada do usuário e não permitir que ela modifique valores já existentes do objeto , essa funcionalidade de recomandar tarefas deve ser restrita a adicionar novos valores de tarefas na qual isAdmin deveria ser impedido de ser validado como uma entrada legítima , além de que a propriedade isAdmin não deveria ser visível nas informações dos objetos dos usuários.
>**Detalhes Técnicos e Evidências**:
1. Primeiro vamos para a tela de /friend/1 (nosso usuário)
![[Pasted image 20260930105253.png]]
2. Em recomenda atividade colocamos isAdmin no tipo da atividade e no valor colocamos true, apertando em recomendar atividade viramos admin automaticamente.
![[Pasted image 20260930105343.png]]

>[!failure] Vulnerabilidade encontrada
>**Título**: SSRF
>**Risk Rating**: 10 DREAD
>**Resumo**: Podemos fazer o servidor buscar informações da api sobre todos os admins e suas senhas a partir de /admin/api ao coletar as urls de get Admins API e indo em /admin/settings que possui uma entrada que a gente coloca uma url para dar update no banner image , mas ao colocar as url's que coletamos essa rota nos devolve valores em base64 do resultado do get.
>**Background(Contexto adicional)**: SSRF é a capacidade de você usar uma funcionalidade de uma aplicação para induzir ela a fazer requisições a recursos internos do servidor alvo , permitindo que enumeremos recursos , rotas , exfiltremos dados e demais coisas.
>**Impacto**: Conseguimos coletar os nomes de admins e senhas da aplicação , assim podemos logar como eles e utilizar de suas funcionalidades.
>**Conselhos e Remediações**: Essa rota deve possuir um filtro para não permitir qualquer busca interna ao servidor filtrando url que contenham 127.0.0.1 , 127.1 ou qualquer derivação que comece com 127. , localhost ::1 e o que seja considerado localhost. Além disso deve ser filtrado buscas feitas para o ip público do servidor , o 127. somente não consegue restringir 100% e também restringir buscas a ips privados da rede .
>**Detalhes Técnicos e Evidências**:
1. A gente acessa a rota /admin/api e visualizamos as rotas que podemos exfiltrar dados.
![[Pasted image 20260930111315.png]]
2. Agora podemos ir em /admin/settings e colocar uma dessas url's.
![[Pasted image 20260930111440.png]]
3. Colocando a url http://127.0.0.1:5000/getAllAdmins101099991 a gente consegue receber esse resultado json a cima , copiamos o que vem depois da "," após base64 e colocamos isso em um decodificador de base64.
![[Pasted image 20260930111553.png]]
4. Conseguimos os nomes de usuários admin e suas senhas.

>[!failure] Vulnerabilidade encontrada
>**Título**: Remote code execution via LFI e Log Poisoning
>**Risk Rating**: 10 DREAD
>**Resumo**: É possível fazer execução de código de forma remota via poison feito no arquivo /var/log/auth.log e usando LFI na rota /profile.php?img= para depois ver o resultado do poison.
>**Background(Contexto adicional)**: Remote code execution é a capacidade de conseguirmos executar código dentro do servidor , LFI é podermos bypassar caminhos relativos ou absolutos e conseguir visualizar outros arquivos que não deveríamos conseguir ver e LogPoisoning é injetar payloads em arquivos de log que junto de um LFI causa RCE.
>**Impacto**: Podemos conseguir executar comandos como www-data , exfiltrar dados e atrapalhar o funcionamento do servidor web.
>**Conselhos e Remediações**: Deve se fazer filtros que executam dentro de um loop para buscar todos os ../ , pois ../ e ....// não funcionam mas ....../// funciona , visto que não foi feito de forma dinâmica esse filtro. Além disso deve se fazer verificações quanto a . e / com seus valores codificados e com dupla codificação , também feito de forma dinâmica para não permitir tripla e nem quadrupla codificação.
>**Detalhes Técnicos e Evidências**:
1. Vemos aqui a rota que permite LFI que bypassei o caminho relativo com ....../// , assim permitindo que eu voltasses diretórios e podemos ver conteúdos como passwd.
![[Pasted image 20260930133744.png]]
2. Olhamos /var/log/auth.log 
![[Pasted image 20260930133818.png]]
3. Está vazio mas iremos agora popular esse arquivo com nosso payload. Iremos usar o remmina para colocar um código php no nome de usuário .
![[Pasted image 20260930133952.png]]
4. Enviamos sem senha mesmo , agora iremos olhar o auth.log denovo.
![[Pasted image 20260930134459.png]]
5. Vemos aqui que o ls foi executado corretamente, agora podemos pegar o valor do arquivo escondido.
![[Pasted image 20260930134550.png]]

---

## 🔑 Tabela de Credenciais Capturadas

| Serviço / Local   | Usuário       | Senha / Hash      | Origem do Achado                            |
| :---------------- | :------------ | :---------------- | :------------------------------------------ |
| --------------    | ---------     | superSecretKey123 | /admin/api, texto puro                      |
| ReviewAppUsername | admin         | admin@!!!         | http://127.0.0.1:5000/getAllAdmins101099991 |
| SysMonAppUsername | administrator | S$9$qk6d#**LQU    | http://127.0.0.1:5000/getAllAdmins101099991 |

---

## 📚 Considerações Técnicas & Cheat Sheet

> [!info] Lições Aprendidas
> 1. Verificar se o user web tem permissão para ver os logs em /var/log , pois assim podemos executar log poisoning e executar se tiver LFI na aplicação.
> 	1. Arquivos de log importantes -> /var/log/auth.log , /var/log/apache2/acess.log ou error.log
> 2. Sempre enumerar o máximo possível , exploração boa vem de uma enumeração boa e eficiente.