# 🚩 Offensive Security Journey & Practical Labs

[![GitHub last commit](https://img.shields.io/github/last-commit/PedroHenriqueBRO/CTF-s-and-Labs?style=flat-square&color=crimson)](https://github.com/PedroHenriqueBRO/CTF-s-and-Labs)
[![Hack The Box](https://img.shields.io/badge/HackTheBox-Certified%20Junior%20Cybersecurity%20Analyst-brightgreen?style=flat-square&logo=hackthebox)](https://academy.hackthebox.com/)
[![TryHackMe](https://img.shields.io/badge/TryHackMe-Pathways%20Completed-red?style=flat-square&logo=tryhackme)](https://tryhackme.com/)
[![Study Time](https://img.shields.io/badge/Dedication-4h%2Fday-blueviolet?style=flat-square)]()

Repositório dedicado à documentação prática da minha transição de **Software Developer** para a área de **Cybersecurity / Offensive Security**. 

Aqui registro writeups detalhados, anotações de exploração, resolução de desafios CTF e os mini-labs práticos realizados ao longo das trilhas de certificação.

---

## 🎯 Rotina e Foco de Estudos

* **Carga Horária:** Pelo menos **4 horas diárias** dedicadas a conceitos teóricos, laboratórios práticos e resolução de máquinas.
* **Objetivo Atual:** Conclusão do path e obtenção da certificação **CJCA (Certified Junior Cybersecurity Analyst)** no Hack The Box Academy.
* **Metodologia:** Estudo aprofundado de protocolos, análise de tráfego, enumeração manual e resolução dos laboratórios/mini-labs práticos de cada módulo.

---

## 🗺️ Trilhas de Aprendizado

### 🟢 Hack The Box Academy (Foco Principal — Trilha CJCA)
* **Network Enumeration with Nmap:** Varreduras ativas, NSE scripts, análise de portas filtradas e técnicas de evasão de firewall/IDS (source-port 53, pacotes customizados).
* **Footprinting & Enumeration:** Enumeração de serviços de rede (SMB/Samba, RPC/RID Cycling, FTP, SSH e Web).
* **Mini-Labs & Labs de Seção:** Validação prática contínua de cada módulo da trilha.

### 🔴 TryHackMe (Base Prática Consolidada)
* **Junior Cybersecurity Analyst:** Fundamentos de redes, logs e segurança defensiva.
* **Web Penetration Testing:** Exploração de falhas como LFI, RCE via Log Poisoning, File Upload Bypasses e IDOR.
* **Linux Privilege Escalation:** Vetores de elevação de privilégios como binários SUID, Cronjobs e Path Hijacking.

---

## 📂 Estrutura do Repositório

```text
.
├── HackTheBox/
│   └── Labs/
│       ├── Lab NMAP Fácil.md       # Fundamentos de scanning e enumeração básica
│       ├── Lab NMAP Médio.md       # Varreduras avançadas e enumeração de serviços
│       ├── Lab NMAP Difícil.md     # Firewall & IDS/IPS Evasion (source-port bypass)
│       └── MiniLabSmb.md           # Enumeração de SMB/Samba, RPC e Null Sessions
│
└── TryHackMe/
    ├── Archangel/
    │   └── Archangel.md            # LFI to RCE via Apache Log Poisoning & Path Hijacking
    └── Opacity/
        └── Opacity.md              # File Upload Bypass, KeePass Cracking & John the Ripper
