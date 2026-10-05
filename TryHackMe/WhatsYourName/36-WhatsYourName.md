# 🎯 Relatório de Invasão: Whats Your Name?

> [!abstract] Resumo 
> **Vetor de Acesso Inicial:** XSS no formulário de registro de usuário onde pegamos o cookie de um moderador
> **Vetor de Elevação de Privilégios:** CSRF no chat Admin Bot como moderador permitindo fazer o admin trocar a senha para teste.
> **Cadeia de ataque completa**: O diretório public era indexável e permitiu que fosse visualizado a pasta js que continha os scripts com a lógica das funcionalidades principais como login e register. Foi visto que o register é feito e depois avaliado por um moderador em uma outra página , com isso foi usado um XSS para roubar o cookie desse moderador devido a falta de httponly. Como moderador foi visto que em login.worldwap.thm há a existência de duas funcionalidades importantes , sendo elas mudar senha e acessar o chat de mensagem. Para mudar a senha é visto que é preciso ser admin para executar isso , assim no chat de mensagem há um admin bot que abre links , permitindo que seja colocado um link malicioso contendo uma execução de csrf , com isso um servidor python arbitrário foi criado onde continha um form html que executava a troca de senha para o admin , quando o bot clicou no link a senha do admin foi trocada para uma senha definida de forma arbitrária . Assim o login como admin foi permitido através dessa senha.

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- ffuf
- burp
- fakepage.py
![[Pasted image 20261005135638.png]]
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap -sS

![[Pasted image 20261005110201.png]]

> [!example] Output do Nmap com -sC e -sV
> 

![[Pasted image 20261005110320.png]]

> [!info] Análise do Reconhecimento
>Vemos aqui a presença de duas aplicações web , uma rodando na porta 80 e outra na porta 8081, onde a na porta 80 não possui o httponly permitindo XSS. As duas aplicações estão rodando em cima de um servidor apache possivelmente indicando um LAMP.

---

## 🌐 2. Enumeração Web & VHOSTs

> [!quote] Dados e reconhecimento
>
### FFUF worldwap.thm:80
![[Pasted image 20261005111018.png]]

![[Pasted image 20261005111818.png]]



### FFUF worldwap.thm:8081
![[Pasted image 20261005111340.png]]

### Worldwap.thm:80
#### /public/html/*
![[Pasted image 20261005111436.png]]
![[Pasted image 20261005111455.png]]
![[Pasted image 20261005112414.png]]
![[Pasted image 20261005112430.png]]

![[Pasted image 20261005112753.png]]

![[Pasted image 20261005112802.png]]

>[!info] Atenção
>X-THM-API-Key no register, mostrando key da api.

#### /public/js/*
![[Pasted image 20261005111939.png]]
![[Pasted image 20261005111959.png]]
![[Pasted image 20261005112031.png]]
![[Pasted image 20261005112122.png]]
![[Pasted image 20261005112133.png]]

### login.worldwap.thm:8081/
#### login.php
![[Pasted image 20261005113137.png]]
#### profile.php
![[Pasted image 20261005132447.png]]

#### chat.php
![[Pasted image 20261005132459.png]]


---

## ⚡ 3. Exploração & Acesso Inicial (Foothold)

>[!failure] Vulnerabilidade encontrada
>**Título**: XSS armazenado
>**Risk Rating**: 10 DREAD
>**Resumo**: É possível capturar o cookie de moderator enviando um XSS armazenado para register.php , pois os registros são visualizados pelos moderadores para serem confirmados ou não. Com isso os campos são vulneráveis a XSS.
>**Background(Contexto adicional)**: XSS armazenado é a capacidade de colocarmos scripts JS ou elementos html em campos que o seu conteúdo é refletido na tela quando a página que carrega ele for buscada , com isso se não há higienização acontece de um payload que injeta um XSS poder ser executado para qualquer um que acesse a página onde ele está armazenado.
>**Impacto**: Podemos capturar o cookie de um moderado e em seguida acessar suas funcionalidades de revisar e confirmar registros.Além de poder acessar o profile.php em login.worldwarp.thm
>**Conselhos e Remediações**: Os campos devem ter htmlspecialchar para filtrar caracteres do tipo <,>,' e outros. Além de que httponly deve ser setado para que cookies não possam ser exfiltrados por js.
>**Detalhes Técnicos e Evidências**:
1. Primeiro acessamos a página de registro.

![[Pasted image 20261005133321.png]]
2. Depois usamos o script fakepage.py que criei para esse ctf, onde iremos receber os cookies roubados.
![[Pasted image 20261005133426.png]]
3. Injetamos o payload XSS em name e enviamos
![[Pasted image 20261005133534.png]]
4. Agora só pegamos o cookie que recebemos no fakepage.py e substituímos no navegador.
![[Pasted image 20261005133620.png]]![[Pasted image 20261005133630.png]]
![[Pasted image 20261005134202.png]]

>[!failure] Vulnerabilidade encontrada
>**Título**: CSRF 
>**Risk Rating**: 10 DREAD
>**Resumo**: Como moderador podemos ver em chat.php que há um admin bot que abre links que mandamos para ele , com isso podemos enviar um link de um servidor que controlamos e fazer ele cair em um CSRF para mudar sua senha.
>**Background(Contexto adicional)**: CSRF é uma vulnerabilidade criada para enganar o navegador da vítima fazendo ela realizar uma solicitação a um site de forma "legítima" para o site mas na verdade a vítima executou um script automático ou preencheu um formulário que parecia um formulário real , enviando seus dados para serem colocados em uma solicitação a um site que ela está logada no navegador dela , fazendo com que o navegador envie o cookie de autenticação dela na solicitação fazendo com que a request seja válida.
>**Impacto**: Podemos alterar a senha do admin e logar como admin logo em seguida podendo acessar suas funcionalidades.
>**Conselhos e Remediações**: Deve ser usado de Anti-CSRF Token que é um token gerado para a sessão do usuário e um token que fica oculto em formulários para que ao ser enviado o formulário seja verificado se o token e o cookie do formulário são iguais para verificar a veracidade da requisição.Além disso deve ser usar do SOP para definir strict ou lax para evitar de cookies sejam enviado a partir de requisições feitas por origem externa.
>**Detalhes Técnicos e Evidências**:
1. Vemos que o change password só funciona para admins.
![[Pasted image 20261005134829.png]]
![[Pasted image 20261005134834.png]]
2. Com isso iremos usar o fakepage.py para capturar a request do bot para que seja feito o csrf.Assim colocamos no chat do admin o seguinte link. http://ip:8000/csrf
![[Pasted image 20261005135023.png]]
3. Agora esperamos o bot acessar o link depois que enviamos.
![[Pasted image 20261005135117.png]]
4. Agora podemos logar como admin e senha teste.
![[Pasted image 20261005135500.png]]

---

## 📚 Considerações Técnicas & Cheat Sheet

> [!info] Lições Aprendidas
> 1. Em uma aplicação podemos usar render_template_string para renderizar o html que queremos , permitindo colocar um formulário falso que faz envio automático (nesse caso), para que conseguissemos trocar a senha da conta do admin.
