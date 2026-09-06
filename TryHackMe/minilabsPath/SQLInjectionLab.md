## Tags
#Hackthebox #tryhackme 

## Alvo
- Um site em uma máquina do Tryhackme que possuem sql injection vulnerabilidades

## Objetivo
- Baseado em união -> THM{SQL_INJECTION_3840}
- Bypass de autenticação -> THM{SQL_INJECTION_9581}
- Cego baseado em booleano -> THM{SQL_INJECTION_1093}
- Cego baseado no tempo -> THM{SQL_INJECTION_MASTER}

## Ferramentas e scripts utilizados
Sem script utilizado

## Resultados obtidos
Para o baseado em união utilizei da enumeração de colunas via UNION para descobrir quantas colunas havia na consulta original para o select do union funcionar , depois descobri que a coluna que retornava dado era a terceira e aí coloquei database() , peguei o nome do database e foi enumerando tabela e colunas pelo information_schema até chegar na tabela staff_users e criei essa sql aqui
```sql
0 UNION SELECT 1,2,group_concat(username,":",password) from staff_users;
```
que me devolveu o password do m4rtin.

--- 

O segundo sqli era de bypass de autenticação , nisso no username coloquei ' OR 1=1;-- assim retornou o primeiro usuário e com ele fiz login.

---

 A baseada em boolean demorou e é por tentativa e erro mas enumeramos desse jeito aqui

```sql
admin123' UNION SELECT 1,2,3 where database() like '%';--
```
```sql
admin123' UNION SELECT 1,2,3 where database() like 'a%';--
```
```sql
admin123' UNION SELECT 1,2,3 where database() like 's%';--
```
O a deu errado mas s retornou true , então sabemos a primeira letra , assim vamos adiante e descobrimos enumerando database , depois table_name , column_name , username e password.

Chegamos em admin:3845

---
Aqui usamos o SLEEP para fazer a mesma coisa do boolean mas agora usando o sleep para 3 segundos para a consulta sql demorar quando estiver certa.
```sql
admin123' UNION SELECT SLEEP(5);--
```
Aqui não demorou 
```sql
admin123' UNION SELECT SLEEP(5),2;--
```
Aqui demorou e acertamos a quantidade de colunas da consulta original, no fim fizemos uma força bruta e achamos admin 4961 
## Considerações
- ' UNION SELECT 1,... ;-- , essa deve ser a consulta sql inicial
- ' UNION SELECT 1,2 where database() LIKE "algo%" ;-- ao acertas as colunas devemos usar o where database() ao invés de from , ao acertar o nome aí podemos usar from information_schema.tables where table_schema = "nomedoschema".
- ' UNION SELECT 1, database() ao acertar a quantidade de colunas a consulta irá retornar o nome do database se colocado na posição certa , então poderia ser database(),2 ou 1,2,database().

