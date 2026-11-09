# 🎯 Relatório de Invasão: TwoMillion

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- scp
- caido
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap com -sC e -sV
> 

![[Pasted image 20261109104615.png]]

> [!info] Análise do Reconhecimento
>22 - ssh , caso tenhamos credenciais podemos ganhar shell no servidor de destino
>80 - http, servidor web rodando em nginx com domínio de nome 2million.htb
### Exploração do servidor
![[Pasted image 20261109121729.png]]
Como admin achei esse email que foi enviado para o user admin e nele diz sobre CVE no kernel do linux especialmente no OverlayFS , com isso pesquisei as CVE's de 2023 para OverlayFS e achei essa CVE-2023-0386, com ela conseguimos shell como root.

---

## 🌐 2. Enumeração Web & VHOSTs

> [!quote] Dados e reconhecimento
>
### Exploração manual
![[Pasted image 20261109104828.png]]
![[Pasted image 20261109111354.png]]
Essa tela nos diz para sermos livres para tentar hackear , com isso vemos no page source e vemos no fim dela um arquivo js sendo incluido e também um script .
![[Pasted image 20261109111729.png]]
Com isso olhamos o código ofuscado no arquivo js e desofuscamos e desempacotamos.
``` 
function verifyInviteCode(code) { var formData = { "code": code }; $.ajax({ type: "POST", dataType: "json", data: formData, url: '/api/v1/invite/verify', success: function(response) { console.log(response); }, error: function(response) { console.log(response); } }); } function makeInviteCode() { $.ajax({ type: "POST", dataType: "json", url: '/api/v1/invite/how/to/generate', success: function(response) { console.log(response); }, error: function(response) { console.log(response); } }); }
```
Com isso fizemos o post para /api/v1/invite/how/to/generate que nos deu um texto cifrado em ROT13 que decifrado ficou assim "In order to generate the invite code, make a POST request to /api/v1/invite/generate" , com isso mandamos POST para essa rota e conseguimos o codigo , ele estava em base64 mas decodificamos e deu certo.
Registrei a conta e loguei.
![[Pasted image 20261109112724.png]]
Como usuário registrado comecei a explorar e percebi que a /api era indexada e acessei /api/v1.
![[Pasted image 20261109121135.png]]
Nesse endpoint via a rota que diz mudar as informações de usuário para admin , com isso comecei a testar ela e vi que eu não precisava ser admin. Assim simplesmente coloquei um json no body e apareceu erro de falta de email , username e is_admin , com isso consegui criar o payload todo e enviei , consegui ser admin.
![[Pasted image 20261109121430.png]]
Como admin tem como usar agora a rota /api/v1/admin/vpn/generate , onde é visto uma vulnerabilidade de RCE na qual usei ls -alh e vi a existência de .env na aplicação e ao ler consegui credenciais do admin.
![[Pasted image 20261109121559.png]]
Assim logamos via ssh.

---

## ⚡ 3. Exploração & Acesso Inicial (Foothold)

>[!failure] Vulnerabilidade encontrada
>**Título**: Recuperação de credenciais do admin via RCE em rota web
>**Risk Rating**: 9.1
>**Resumo**: A partir de uma cadeia de ataques é possível usar um endpoint como admin que supostamente iria gerar o arquivo de vpn para ele mas assim podemos injetar um comando e ter ele executado no servidor de destino
>**Background(Contexto adicional)**: RCE é a capacidade de podermos executar comandos no servidor de destino a partir de uma entrada vulnerável que não é higienizada.
>**Impacto**: Podemos ler e exfiltrar dados de acordo com as permissões do usuário na qual estamos executando os comandos no servidor.
>**Conselhos e Remediações**: O endpoint de mudança de role para o usuário deve pode ser usado somente por admins , assim colocando essa verificação . Além disso a rota de gerar vpn deve ser higienizada para que a entrada do usuário seja usada como valor de forma que não escape o comando e possa executar um comando arbitrário.
>**Detalhes Técnicos e Evidências**:
1. Logados como usuário podemos usar a rota /api/v1/admin/settings/update para fazer update de nossas informações.
![[Pasted image 20261109122547.png]]
2. Após virarmos admin podemos agora executar a RCE na rota /api/v1/admin/vpn/generate.
![[Pasted image 20261109122618.png]]

>[!failure] Vulnerabilidade encontrada
>**Título**: Servidor alvo vulnerável a CVE-2023-0386 em cima do sub sistema overlayfs no kernel linux
>**Risk Rating**: 7.8
>**Resumo**: Essa vulnerabilidade está presente no servidor alvo e permite que ganhemos privilégio root após abusar do OverlayFS para copiar um setuid arquivo de um mount nosuid para um outro mount , permitindo que possamos executar códigos arbitrários com privilégio de root.
>**Background(Contexto adicional)**:
>**Impacto**: Após virarmos root podemos fazer qualquer enumeração , exfiltração e modificação no servidor alvo.
>**Conselhos e Remediações**: O kernel do linux deve ser atualizado , além de atualizar qualquer pacote que esteja instalado nele para evitar CVE's de versões antigas.
>**Detalhes Técnicos e Evidências**:
1. Damos git clone https://github.com/xkaneiki/CVE-2023-0386 em nossa máquina e assim usamos scp para enviar para o servidor 
![[Pasted image 20261109123057.png]]
2. Vamos para /tmp , executamos unzip e em seguida make all.
![[Pasted image 20261109123159.png]]
3. Executamos esses dois comandos abaixo.
![[Pasted image 20261109123226.png]]
```
./fuse ./ovlcap/lower ./gc & -> background
./exp -> foreground
``` 
4. Com isso conseguimos acesso a shell root.
![[Pasted image 20261109123331.png]]
---

## 🔑 Tabela de Credenciais Capturadas

| Serviço / Local | Usuário | Senha / Hash      | Origem do Achado                           |
| :-------------- | :------ | :---------------- | :----------------------------------------- |
| SSH             | admin   | SuperDuperPass123 | Arquivo .env no web root , dei cat via RCE |

---

## 📚 Considerações Técnicas & Cheat Sheet

> [!info] Lições Aprendidas
> 1. Explorar /api sempre para ver se é indexada.
> 2. Olhar sempre os arquivos .js na aplicação para caso tenha algum arquivo ofuscado que seja importante e nos dê informações sobre rotas , credenciais e demais coisas.