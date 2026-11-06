# 🎯 Relatório de Invasão:  Cohort

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- ffuf 
- exploit.py 1 CVE-2026-39987
- exploit.py 2 CVE-2026-4165
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap com -sC e -sV
> 

![[Pasted image 20261105181720.png]]

> [!info] Análise do Reconhecimento
> Vemos a presença de uma porta 22 aberta com ssh , permitindo que conectemos caso arranjemos credenciais. Além disso há dois servidores web , um sendo na porta 80(http) e outro na porta 443(https).

---

## 🌐 2. Enumeração Web & VHOSTs

> [!quote] Dados e reconhecimento
>


### Exploração manual
![[Pasted image 20261105181945.png]]
![[Pasted image 20261105181957.png]]
Vemos que não há nada demais nessas telas e no page source .

Assim comecei a explorar a funcionalidade de validação , vista acima ao clicar no botão Open Client Insights.
![[Pasted image 20261105194358.png]]
Essa funcionalidade eu comecei testando meu servidor usando python3 -m http.server e a requisição chegou , com isso pensei que poderia colocar um script mas o servidor alvo não executa , ele só mostra o conteúdo. Com isso pensei em enumerar internamente o servidor alvo. Com isso coloquei 127.0.0.1 mas como a própria pagina diz o loopback é proibido mas aí pensei que eles poderiam ter feito uma verificação ruim e assim coloquei 127.1 e deu certo. Assim enumerei as portas internas. Nisso achei a porta 8888 que retornava uma página de login.
![[Pasted image 20261105194713.png]]
Enumerando na porta 8888 não achei nada de interessante , assim comecei a enumerar diretamente no 127.1 mesmo e descobri a rota status.
![[Pasted image 20261105194850.png]]
Ela nos deu um novo vhost que ao acessar chegamos na rota de login da porta 8888.
![[Pasted image 20261105194920.png]]
Nessa tela vemos o title sendo marimo , assim fui buscar por cve e achei uma poc de uma cve . Essa cve é um pre auth RCE , assim com o código consegui um shell no alvo.

---

## ⚡ 3. Exploração & Acesso Inicial (Foothold)

>[!failure] Vulnerabilidade encontrada
>**Título**: SSRF na rota https://cohort.htb/portal.html
>**Risk Rating**: 8.6
>**Resumo**: Um SSRF permitiu que coletemos dados internos , como portas abertas e nisso descobrirmos dados de dentro do servidor. Assim conseguimos descobrir uma aplicação rodando em um certo vhost descoberto em uma rota.
>**Background(Contexto adicional)**: SSRF é uma vulnerabilidade que permite que forjemos uma requisição para que o servidor alvo busque dados internamente ou em serviços de terceiros e nos devolva esses dados.
>**Impacto**: Podemos enumerar a infra do servidor alvo e pegar dados de arquivos confidenciais .
>**Conselhos e Remediações**: A aplicação restringiu 127.0.0.1 mas não restringiu 0.0.0.0 e 127.1 , provavelmente existem mais jeito de acessar o loopback . Assim é preciso restringir mais especificamente para todas as variações dos dispositivos de rede interno ou fazer um whitelist impondo quais tipos de url's podem ser acessadas.
>**Detalhes Técnicos e Evidências**:
1. Primeiramente acessamos a página que contém a função vulnerável.
![[Pasted image 20261105203253.png]]
2. No campo de url colocamos http://127.1:8888/.
![[Pasted image 20261105203334.png]]
3. Vemos o SSRF , visto que fizemos a aplicação buscar dados de uma interface interna do servidor e certa porta e nos entregar.

>[!failure] Vulnerabilidade encontrada
>**Título**: Marimo Pre-Auth RCE via Unauthenticated WebSocket Terminal (CVE-2026-39987)
>**Risk Rating**: 9.8
>**Resumo**: Podemos de forma não autenticada executar esse exploit desse CVE no vhost nb-1be3782a8afd3ad5.cohort.htb alvo , assim permitindo que possamos fazer RCE e ganhar reverse shell.
>**Background(Contexto adicional)**: Essa CVE é sobre a vulnerabilidade que permite que de forma não autenticada estabeleçamos um web socket com o alvo e assim podemos executar códigos diretamente no servidor alvo.
>**Impacto**: Podemos de maneira não autenticada ganhar um reverse shell como user marimo no servidor alvo e assim executar ações em seu nome , exfiltrar dados e escalarmos privilégios para root caso haja como.
>**Conselhos e Remediações**: Essa CVE afeta uma versão antiga do marimo , assim deve-se atualizar o marimo para versões atuais para que o servidor esteja mais seguro e não permita a exploração dessa CVE.
>**Detalhes Técnicos e Evidências**:
1. Simplesmente executamos o script dessa CVE. Os valores presentes nos args desse script não foram necessário pois reutilizei um outro comando que testei para um código que não estava funcionando.
![[Pasted image 20261105204010.png]]
2. Nisso não ganhamos shell ainda em si pois ao fechar o web socket esse shell fecha , mas vemos que o nohup é executado , assim precisamos só abrir um nc na porta que colocamos no exploit e rodar o exploit novamente.
![[Pasted image 20261105204157.png]]
Fazendo isso ganhamos shell como marimo.

>[!failure] Vulnerabilidade encontrada
>**Título**: Pack2TheRoot no servidor alvo (CVE-2026-4165)
>**Risk Rating**: 8.8
>**Resumo**: Esse CVE se resume a um pacote linux chamado PackageKit que possui versões vulneráveis a esse CVE e uma delas estava presente no servidor alvo. Assim permitindo que viremos root ao executar o código desse CVE.
>**Background(Contexto adicional)**: Pack2TheRoot é uma vulnerabilidade que afeta versões do PackageKit da `1.0.2` até a `1.3.4` em múltiplas distribuições Linux e nisso permite que usuários sem privilégio explorem race conditions para manipular operações de pacotes e obter acesso root ao sistema.
>**Impacto**: Sendo root podemos fazer qualquer exfiltração de dados e exploração no servidor alvo.
>**Conselhos e Remediações**: Esse pacote deve ser atualizado para versões mais recentes para que seja mitigado essa exploração de cve no servidor alvo.
>**Detalhes Técnicos e Evidências**:
1. Baixamos a poc em nosso kali e a transferimos para o servidor alvo , nisso é só preciso executar ela com python , havia o python3 no alvo então foi fácil.
![[Pasted image 20261105204651.png]]
2. Assim vemos que possuímos agora acesso root.
![[Pasted image 20261105204722.png]]

---

## 📚 Considerações Técnicas & Cheat Sheet

> [!info] Lições Aprendidas
> 1. 
> 2. 