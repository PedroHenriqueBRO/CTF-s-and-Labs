# Tópico

# Tags

#Hackthebox 

# Resumo
Foi dado duas perguntas sobre NFS para o servidor de destino e nisso de primeira utilizei o nmap para visualizar todas as portas rodando.
``` 
sudo nmap -sS 10.129.174.194
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-25 17:19 -0400
Nmap scan report for 10.129.174.194
Host is up (0.42s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2049/tcp open  nfs

Nmap done: 1 IP address (1 host up) scanned in 6.44 seconds

```
Assim vim que as portas 111 e 2049 estavam abertas e são relativamente as portas de utilização do NFS.
Rodei depois com -sC para ver os scripts achariam informações relevantes
``` 
sudo nmap -sC 10.129.174.194 -p 111,2049
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-25 17:20 -0400
Nmap scan report for 10.129.174.194
Host is up (0.27s latency).

PORT     STATE SERVICE
111/tcp  open  rpcbind
| rpcinfo: 
|   program version    port/proto  service
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      40567/tcp6  mountd
|   100005  1,2,3      47509/tcp   mountd
|   100005  1,2,3      50191/udp   mountd
|   100005  1,2,3      58949/udp6  mountd
|   100021  1,3,4      32944/udp6  nlockmgr
|   100021  1,3,4      41883/tcp   nlockmgr
|   100021  1,3,4      44031/tcp6  nlockmgr
|   100021  1,3,4      59590/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp open  nfs_acl

Nmap done: 1 IP address (1 host up) scanned in 7.14 seconds
```
Nisso ainda não tivemos , somente vemos as versões e quais portas/protocolos elas rodam os serviços em cima do rpc.
Com essa varredura não tendo dado muito resultado partir para nfs* e assim executei todos os scripts nfs* nessas portas
``` 
sudo nmap -sS --script nfs* 10.129.174.194 -p 111,2049
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-25 17:22 -0400
Nmap scan report for 10.129.174.194
Host is up (0.43s latency).

PORT     STATE SERVICE
111/tcp  open  rpcbind
| nfs-showmount: 
|   /var/nfs 10.0.0.0/8
|_  /mnt/nfsshare 10.0.0.0/8
| nfs-statfs: 
|   Filesystem     1K-blocks  Used       Available  Use%  Maxfilesize  Maxlink
|   /var/nfs       4062912.0  3422544.0  414272.0   90%   16.0T        32000
|_  /mnt/nfsshare  4062912.0  3422544.0  414272.0   90%   16.0T        32000
| nfs-ls: Volume /var/nfs
|   access: Read Lookup Modify Extend Delete NoExecute
| PERMISSION  UID    GID    SIZE  TIME                 FILENAME
| rwxr-xr-x   65534  65534  4096  2021-11-08T15:08:27  .
| ??????????  ?      ?      ?     ?                    ..
| rw-r--r--   65534  65534  39    2021-11-08T15:08:27  flag.txt
| 
| 
| Volume /mnt/nfsshare
|   access: Read Lookup Modify Extend Delete NoExecute
| PERMISSION  UID    GID    SIZE  TIME                 FILENAME
| rwxr-xr-x   65534  65534  4096  2021-11-08T14:06:40  .
| ??????????  ?      ?      ?     ?                    ..
| rw-r--r--   65534  65534  59    2021-11-08T14:06:40  flag.txt
|_
2049/tcp open  nfs

Nmap done: 1 IP address (1 host up) scanned in 18.87 seconds
```
Aqui vemos dois volumes de montagem disponíveis no servidor nfs e são os dois que precisamos montar para acessar a flag de cada um e responder as duas perguntas.
Nisso agora eu fiz a montagem desses dois volumes localmente via mount.
``` 
sudo mount -t nfs 10.129.174.194:/ . 
```
Assim eu trouxe os dois volumes de uma vez localmente na pasta que fiz.
```
┌──(kali㉿kali)-[~/Desktop/NFSmounts]
└─$ ls
mnt  var
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts]
└─$ cd mnt/     
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts/mnt]
└─$ ls
nfsshare
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts/mnt]
└─$ cd nfsshare 
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts/mnt/nfsshare]
└─$ ls
flag.txt
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts/mnt/nfsshare]
└─$ ls -alh                 
total 12K
drwxr-xr-x 2 nobody nogroup 4.0K Nov  8  2021 .
drwxr-xr-x 3 root   root    4.0K Nov  8  2021 ..
-rw-r--r-- 1 nobody nogroup   59 Nov  8  2021 flag.txt
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts/mnt/nfsshare]
└─$ cat flag.txt 
HTB{8o7435zhtuih7fztdrzuhdhkfjcn7ghi4357ndcthzuc7rtfghu34}
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts/mnt/nfsshare]
└─$ cd ..      
c                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts/mnt]
└─$ cd ..
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts]
└─$ ls     
mnt  var
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts]
└─$ cd var 
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts/var]
└─$ ls
nfs
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts/var]
└─$ cd nfs 
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts/var/nfs]
└─$ ls -alh
total 12K
drwxr-xr-x  2 nobody nogroup 4.0K Nov  8  2021 .
drwxr-xr-x 14 root   root    4.0K Nov  8  2021 ..
-rw-r--r--  1 nobody nogroup   39 Nov  8  2021 flag.txt
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/NFSmounts/var/nfs]
└─$ cat flag.txt 
HTB{hjglmvtkjhlkfuhgi734zthrie7rjmdze}
```
Com as duas flags eu respondi as duas questões e terminei a seção.



