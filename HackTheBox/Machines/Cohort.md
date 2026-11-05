# 🎯 Relatório de Invasão: Cap

> [!abstract] Resumo 
> **Cadeia de ataque completa**: Ao acessar o servidor web nos deparamos com uma rota que permite que recuperemos dados de outros usuários (IDOR), assim recuperamos um .pcap de um usuário chamado nathan que nesse .pcap expoe dados de user e senha dele do ftp , então reutilizamos essas credenciais para o ssh e conseguimos shell com sucesso. Ao ter shell como nathan pesquisamos os cap em binários que ele pode usar e nisso vemos que ele pode fazer setuid com python3.8 , assim criamos um shell como root com um comando python e assim terminamos de escalar privilégio e dominar a máquina.

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- ftp
- ffuf
- wireshark
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap com -sC e -sV
> 

![[Pasted image 20261105103859.png]]

![[Pasted image 20261105103953.png]]


> [!info] Análise do Reconhecimento
> Vemos uma porta rodando um serviço ftp que podemos enumerar e explorar para ver se o user anonymous é permitido nele, possivelmente permitindo a gente acesso a arquivos que não deveríamos. Temos um ssh que pode nos gerar um shell caso achemos user e password de user de sistema. Além disso tem uma aplicação web rodando onde iremos enumerar e explorar por vulnerabilidades para ganhar acesso ao sistema , a usuários normais e admins.

### FTP
![[Pasted image 20261105104538.png]]
Vemos que o usuário padrão anonymous não tem acesso com senha padrão, iremos por enquanto explorar a web e depois voltar para o ftp caso achemos algo de interessante.

### Pcap com wireshark
Após usarmos o IDOR para recuperar um .pcap de um outro usuário usamos o wireshark para visualizar os dados desse .pcap.
![[Pasted image 20261105111241.png]]
Aqui conseguimos visualizar o processo de login do usuário nathan no ftp , com user e senha expostos sem segurança. Com isso podemos logar ftp e tentar ver se no ssh ele reutilizou a mesma senha.

---

## 🌐 2. Enumeração Web & VHOSTs

> [!quote] Dados e reconhecimento
>


![[Pasted image 20261105105616.png]]
Vemos um nome de usuário sendo ele chamado Nathan que podemos usar para tentar fazer login no ftp.

![[Pasted image 20261105111015.png]]
Nessa tela temos dados referentes a scans feitos em nível de rede para capturar pacotes , com isso vemos a presença na url de um número depois de /data, essa tela tem um IDOR que foi testado onde colocando 0 podemos acessar um scan importante de uma outra pessoa.


---

## ⚡ 3. Exploração & Acesso Inicial (Foothold)

>[!failure] Vulnerabilidade encontrada
>**Título**: IDOR em url /data/{id}
>**Risk Rating**: CVSS 7.5 
>**Resumo**: Foi encontrado um IDOR na url /data/{id} que retornava uma tela que nos gerava um botão de download , na qual o arquivo baixado era um .pcap para scans feitos em nível de rede pelo usuário. Com isso o IDOR permitia que capturassemos .pcap de outros usuários.
>**Background(Contexto adicional)**: IDOR é uma vulnerabilidade que permite que acessemos dados de outras pessoas sem ter a permissão para isso , devido a falhas de verificação de autorização nesse recurso .
>**Impacto**: Podemos extrair os .pcap de todos os usuários na plataforma apenas mudando o id para certo valor.
>**Conselhos e Remediações**: Para esse recurso deve se verificar se o id do usuário que está solicitando certo recurso é o mesmo id de usuário que possui esse recurso, assim impedindo que outros usuários recuperem dados de outras pessoas.
>**Detalhes Técnicos e Evidências**:

1. A baixo teremos um print de um recurso que é devidamente meu
![[Pasted image 20261105112632.png]]
2. Com isso vemos na url que possui um id de valor 4 , iremos mudar para 0 que é o valor de um .pcap que possui vários dados de outra pessoa.
![[Pasted image 20261105112717.png]]
3. Concluindo , vemos que podemos acessar dados de outros usuários somente trocando o id na url.

>[!failure] Vulnerabilidade encontrada
>**Título**: Binário setado com cap de setuid para usuário nathan
>**Risk Rating**: CVSS 7.8
>**Resumo**: Foi setado permissão para o usuário nathan para executar o binário do python3.8 com setuid , permitindo a criação de um shell como root.
>**Background(Contexto adicional)**: Cap são capacidades que podemos setar em binários para dar permissões a mais a um usuário a invés de setar permissões de sudo , assim limitando privilégio a certo binário.
>**Impacto**: Após spawnar o shell como root podemos fazer toda e qualquer modificação no servidor e extrair dados de TODOS os usuários .
>**Conselhos e Remediações**: Rever a necessidade de colocar cap de setuid nesse binário e assim confirmar se realmente é necessário , caso não seja colocar então cap's menos privilegiadas .
>**Detalhes Técnicos e Evidências**:
1. Primeiro rodamos o comando de getcap.
![[Pasted image 20261105113944.png]]
2. Pegamos o script do gtfobins para python com setui para criar um shell como root e o executamos.
![[Pasted image 20261105114028.png]]
3. Pronto , agora temos acesso como root e podemos fazer qualquer ação no servidor.

---

## 🔑 Tabela de Credenciais Capturadas

| Serviço / Local | Usuário | Senha / Hash    | Origem do Achado |
| :-------------- | :------ | :-------------- | :--------------- |
| ftp             | nathan  | Buck3tH4TF0RM3! | .pcap            |
| ssh             | nathan  | Buck3tH4TF0RM3! | .pcap            |

---

## 📚 Considerações Técnicas & Cheat Sheet

> [!info] Lições Aprendidas
> 1. Ficar mais atento a toda url que contenha id's , pode ser que tenha um IDOR.
