# 🎯 Relatório de Invasão: Connected

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- ssh-keygen
- linpeas.sh
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap com -sC e -sV
> 

![[Pasted image 20261108230551.png]]

> [!info] Análise do Reconhecimento
>22 - ssh , se caso conseguirmos credenciais podemos logar pelo ssh
>80 e 443 - servidor web rodando com Apache e php em http e https, iremos enumerar e explorar por vulnerabilidades web.

---

## 🌐 2. Enumeração Web & VHOSTs

> [!quote] Dados e reconhecimento
>
### Exploração manual ![[Pasted image 20261108230755.png]]
Vemos aqui que a aplicação alvo está rodando esse freePBX , FreePBX 16.0.40.7, iremos olhar por CVE's em relação a essa versão.
www.exploit-db.com/exploits/52681 
Aqui encontramos uma cve em relação a essa versão do FreePBX , iremos usar esse exploit .

Com sucesso conseguimos shell no servidor alvo como user asterisk.

--- 
## 3. Exploração do servidor
Como asterisk rodamos o linpeas.sh .
![[Pasted image 20261109012555.png]]
Essa parte interessou bastante pois é visto que após ocorrer um evento de escrita em um arquivo dahdi_restarte ou criarmos ele , acontece do incron ser trigado e executar o dahdi.
Assim vemos com o find os arquivos que temos permissão de escrita e o /etc/dahdi/init.conf possui permissão de escrita, assim podemos colocar um reverse shell nele , dar touch em dahdi_restart e esperar o incron executar e nos devolver o shell como root.

---

## ⚡ 4. Exploração & Acesso Inicial (Foothold)


>[!failure] Vulnerabilidade encontrada
>**Título**: FreePBX 16.0.40.7 vulnerável a CVE 2025-57819 
>**Risk Rating**: 9.8
>**Resumo**: A aplicação alvo está utilizando esse serviço FreePBX na versão 16.0.40.7 , na qual ele possui algumas vulnerabilidades e uma delas é a CVE 2025-57819 que permite que executemos RCE no servidor alvo a partir de um exploit.
>**Background(Contexto adicional)**: Essa CVE é sobre uma injeção SQL desautenticada em um endpoint de gerenciamento de módulo do FreePBX , que permite que injetemos um sql injection no parâmetro brand no endpoint /admin/ajax.php , assim tendo o conteúdo de brand inserido em uma consulta SQL sem sanitização.
>**Impacto**: Podemos obter com essa CVE a execução de comandos no servidor alvo que podem nos resultar em enumeração , exfiltração de dados e um shell reverso por exemplo.
>**Conselhos e Remediações**: Subir uma versão nova do FreePBX para impedir a exploração dessa CVE's e outras , manter softwares de terceiro atualizados são boas práticas de segurança além de impedir exploração de CVE's públicos e conhecidos.
>**Detalhes Técnicos e Evidências**:
1. Acessamos o link www.exploit-db.com/exploits/52681 , correspondente ao exploit dessa CVE , baixamos o arquivo python e o executamos com o seguindo comando.
![[Pasted image 20261108232123.png]]
2. Em seguida irá gerar um shell dentro de alguns segundos.
![[Pasted image 20261108232158.png]]

>[!failure] Vulnerabilidade encontrada
>**Título**: Trigger que executa o incron em dahdi , executando comando arbitrário em init.conf
>**Risk Rating**: 8.4
>**Resumo**: O servidor alvo estava com gatilhos ativados onde se fizessemos um touch no arquivo /\*\*\/dahdi_restart o incron era ativado e executava o dahdi , incluíndo o init.conf. Assim podemos colocar um comando arbitrário em init.conf e depois dar touch no dahdi_restart , tendo o comando em init.conf executado como root posteriormente
>**Background(Contexto adicional)**:
>**Impacto**: Com isso podemos ganhar shell como root , enumerar dados , exfiltrar e fazer qualquer outra ação quebrando os princípios da CIA.
>**Conselhos e Remediações**: Garantir que o incron trabalhe somente em cima de ficheiros onde usuários comuns não possam ter suas ações interferendo o serviço do trigger sobre esse ficheiro, assim impedindo que um usuário modifique arquivos e o trigger do incron execute ele em conjunto com as regras do trigger sobre esse ficheiro.
>**Detalhes Técnicos e Evidências**:
1. Editamos o init.conf e colocamos um reverse shell
![[Pasted image 20261109013834.png]]
2. Iniciamos penelope na porta 6666.
3. Fazemos o touch 
![[Pasted image 20261109013926.png]]
4. Esperamos alguns segundo e conseguimos shell como root.
![[Pasted image 20261109014044.png]]

---