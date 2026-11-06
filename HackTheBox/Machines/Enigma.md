# 🎯 Relatório de Invasão:  Enigma

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- nfs
- ffuf
- rpcclient
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap com -sC e -sV
> 

![[Pasted image 20261106115417.png]]
Havia olhado só as portas comuns , com isso fiz uma varredura completa e achei mais portas .
![[Pasted image 20261106130743.png]]
> [!info] Análise do Reconhecimento
>22 - ssh , bom se tivermos credenciais
>111- rpc , podemos tentar enumerar usuários e demais coisas.
>80 - servidor web , enumerar e explorar por vulnerabilidades
>110,143,993 e 995 - Serviços de email , explorar por emails que contenham credenciais ou informações expostas
>2049 - nfs , serviço de compartilhamento de arquivos na internet

### pop3(110) e pop3s(995)
Tentei fazer login como anônimo nos dois serviços e não foi possível.
![[Pasted image 20261106124530.png]]
Usando os scripts de pop3 não foi possível achar informações e nem contas válidas.
### Imap(143) e Imaps(993)
Tentei fazer login como anônimo nos dois serviços e não foi possível.
![[Pasted image 20261106123133.png]]
Tentei passar todos os scripts específicos do Imap , foi feito brute force e demais enumerações mas não foi encontrado nada.
### Rpc(111)
Não aceita sessão nula.

### nfs(2049)
![[Pasted image 20261106130842.png]]
Listei os diretórios disponíveis no nfs e apareceu esse aí , onde eu montei e ganhei acesso a 1 pdf.
![[Pasted image 20261106130933.png]]
![[Pasted image 20261106130947.png]]
Com isso foi possível logar nesse domínio com o usuário kevin e senha dele , pois ele não havia trocado ela. Além disso no pdf havia mais um email it@enigma.htb.
![[Pasted image 20261106131040.png]]

---

## 🌐 2. Enumeração Web & VHOSTs

> [!quote] Dados e reconhecimento
>

### Ffuf
#### Diretórios e arquivos
![[Pasted image 20261106122632.png]]
Enumeração por arquivo e diretórios com ffuf não gerou resultados.Com isso ireis procurar por vhosts.
#### Vhosts
![[Pasted image 20261106125420.png]]
Não foi achado nenhum vhost
### Exploração manual
#### enigma.htb
![[Pasted image 20261106121434.png]]
Nessa página é possível identificar um email (support@enigma.htb) , com isso temos ideia de um usuário que podemos usar para tentar fazer login nos serviços de email.

#### mail001.enigma.htb
![[Pasted image 20261106131120.png]]

![[Pasted image 20261106131144.png]]
Assim descobrimos um novo usuário além do kevin , que é a sarah.
Usuário enumerado até agora:
- support
- kevin
- sarah
- it
Com isso eu peguei a mesma senha do kevin e retestei para a sarah e consegui login.
![[Pasted image 20261106133724.png]]
![[Pasted image 20261106133733.png]]
Como sarah vemos que a gente conseguiu credenciais de admin e senha para esse novo vhost.

### support_001.enigma.htb
![[Pasted image 20261106133917.png]]
![[Pasted image 20261106133934.png]]
A versão desse software é vulnerável a CVE-2025-69212 , na qual permitiu que conseguissemos um reverse shell após explorar essa vulnerabilidade.

---
## 3. Enumerando o servidor
### www-data
![[Pasted image 20261106161253.png]]
![[Pasted image 20261106161423.png]]
![[Pasted image 20261106161448.png]]
![[Pasted image 20261106161746.png]]

### haris
Como haris vemos que tem uma porta 1337 rodando um serviço como root , mas é acessada somente via local , assim transportei uma chave ssh publica para o haris no servidor e assim abri um tunel para fazer port forwading.
![[Pasted image 20261106165632.png]]
Explorando a versão é visto que há uma cve para esse serviço e assim conseguimos virar root.


---
## ⚡ 4. Exploração & Acesso Inicial (Foothold)

>[!failure] Vulnerabilidade encontrada
>**Título**: NFS contendo um arquivo pdf com credenciais expostas nele
>**Risk Rating**: 7.5
>**Resumo**: Um nfs disponível no servidor alvo continha credenciais expostas em texto claro em um pdf, permitindo que acessemos a conta do usuário no serviço a qual ele era cadastrado.
>**Background(Contexto adicional)**: 
>**Impacto**: Ao ter essas credenciais podemos logar com o usuário e senha , assim permitindo que executemos ações em nome desse usuário.
>**Conselhos e Remediações**: Esse tipo de informação não deve ser posta em arquivos dentro de nfs público , assim essa informação deveria ser passada por um canal mais seguro. Além de que deve ser reforçado que as senhas padrões de uma conta devem ser trocadas após o login , que no caso o usuário dono dessas credenciais não fez isso mas deveria.
>**Detalhes Técnicos e Evidências**:
1. Nfs /srv/nfs/onboarding
![[Pasted image 20261106145852.png]]
2. Após montar ele temos dentro do diretório montado o pdf.
![[Pasted image 20261106145935.png]]
3. A seguir temos o pdf em si.
![[Pasted image 20261106150012.png]]
4. Login com as credenciais do kevin.
![[Pasted image 20261106150126.png]]
![[Pasted image 20261106150137.png]]

>[!failure] Vulnerabilidade encontrada
>**Título**: Injeção de comando no SO via processamento de arquivo P7M no software  OpenSTAManager (CVE-2025-69212)
>**Risk Rating**: 9.4
>**Resumo**: O servidor alvo está rodando uma instância do software web OpenSTAManager na versão 2.9.8 que é vulnerável a esse CVE , que permite um RCE no SO.
>**Background(Contexto adicional)**: Esse CVE explora a vulnerabilidade vinda do mal processamento que a aplicação faz em cima do nome dos arquivos dentro do método decodeP7M() , na qual esse nome é usando em uma função posterior que executa um comando no sistema operacional . Assim pela falta de higienização do nome e como o nome do arquivo é inserido na função acaba que pode se injetar comandos arbitrários.
>**Impacto**: Podemos criar um webshell e posteriormente receber um reverse shell como www-data , permitindo que adentremos dentro do servidor alvo. Permitindo que enumeremos , exploremos e exfiltremos dados do alvo.
>**Conselhos e Remediações**: Deve se atualizar a versão do software para versões mais atuais para evitar CVE's relacionadas a essa e outras versões vulneráveis.
>**Detalhes Técnicos e Evidências**:
1. Primeiro criamos o zip.
![[Pasted image 20261106153413.png]]
2. Depois enviamos ele nessa seguinte rota.
![[Pasted image 20261106153435.png]]
![[Pasted image 20261106153510.png]]
3. Por fim após enviar podemos agora interagir com o webshell que enviamos.
![[Pasted image 20261106153602.png]]

>[!failure] Vulnerabilidade encontrada
>**Título**: OliveTin injeção de comando no sistema operacional (CVE-2026-27626)
>**Risk Rating**: 9.9-10
>**Resumo**: Essa vulnerabilidade é em cima do serviço OliveTin que é uma aplicação web que permite execução de alguns comandos pela UI , porém essa CVE permite que injetemos comandos no contexto do SO e devido ao fato desse serviço estar rodando como root então os comandos executados são com privilégio de root.
>**Background(Contexto adicional)**:
>**Impacto**: Podemos copiar o binário bash para outro lugar , colocar permissões de execução e executar ele depois como qualquer usuário e assim virar root devido ao fato de que todo o processo foi feito com permissão de root.
>**Conselhos e Remediações**: Esse software não deveria estar rodando como root caso não seja estritamente necessário , além de que a sua versão deve ser atualizada para uma versão mais atual que seja mais protegida.
>**Detalhes Técnicos e Evidências**:
1. Após fazer post forwading como haris , podemos simplesmente executar a poc.
![[Pasted image 20261106170207.png]]
2. Agora podemos logar como haris e usar esse bash.
![[Pasted image 20261106170245.png]]


---

## 🔑 Tabela de Credenciais Capturadas

| Serviço / Local | Usuário   | Senha / Hash   | Origem do Achado                                                                             |
| :-------------- | :-------- | :------------- | :------------------------------------------------------------------------------------------- |
| OpenSTAManager  | admin     | Ne3s4rtars78s  | Email no inbox da sarah                                                                      |
| Roundcube       | kevin     | Enigma2024!    | Pdf em nfs público                                                                           |
| Roundcube       | sarah     | Enigma2024!    | Reutilização da senha do kevin                                                               |
| Mysql           | roundcube | Yo270x26!gTx02 | /var/www/html/roundcube/config/config.inc.php                                                |
| Mysql           | brollin   | Fri3nds@9099   | Arquivo de config do banco usado pela OpenSTAManager                                         |
|                 | harris    | bestfriends    | Tabela zz_users no banco openstamanager de forma criptograda . Descriptografada via hashcat. |

---

## 📚 Considerações Técnicas & Cheat Sheet

> [!info] Lições Aprendidas
> 1. Não esquecer de usar as credenciais de mysql , usar todas que achar.
