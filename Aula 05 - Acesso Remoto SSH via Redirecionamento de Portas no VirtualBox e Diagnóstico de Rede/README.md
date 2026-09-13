# Relatório Técnico – Aula Prática 05: Acesso Remoto SSH via Redirecionamento de Portas no VirtualBox e Diagnóstico de Rede

---

## 1. Identificação
* **Nome do Aluno:** João Vinícius Rocha Costa
* **Curso:** Bacharelado em Sistemas de Informação (BSI)
* **Disciplina:** Laboratório de Sistemas Operacionais e Redes
* **Data de Realização:** 10/09/2026
* **Título da Prática:** Acesso Remoto SSH via Redirecionamento de Portas no VirtualBox e Diagnóstico de Rede

---

## 2. Objetivo
Configurar o acesso remoto seguro via SSH (Secure Shell) em um ambiente com interface em modo NAT utilizando o mecanismo de Redirecionamento de Portas (*Port Forwarding*) no VirtualBox. Além disso, realizar a auditoria e inspeção diagnóstica de rede em ambos os sistemas (Host e Guest) através de comandos de leitura de interface, rotas padrão, rastreamento de saltos, gerenciamento de sessões ativas e checagem do estado de portas TCP (`LISTENING` e `ESTABLISHED`).

---

## 3. Ambiente do Laboratório
* **Sistema Operacional Hospedeiro (Host):** Windows (PowerShell)
* **Hipervisor:** Oracle VM VirtualBox
* **Sistema Operacional Convidado (Guest):** Ubuntu Server 26.04 LTS (`ubuntu_server`)
* **Configuração de Rede Guest:** Modo NAT (IP padrão `10.0.2.15/24`)
* **Usuário de Administração:** `administrador` (com privilégios via `sudo`)

---

## 4. Procedimento de Configuração

###  Preparação e Diagnóstico Inicial no Ubuntu Server
1. **Instalação de Utilitários Diagnósticos:**
   Atualizou-se a lista de repositórios e instalaram-se as ferramentas `net-tools` e `traceroute`:
   ```bash
   sudo apt update
   sudo apt install -y net-tools traceroute

---

## 5. Testes e Evidências
### 5.1. Inspeção de Interfaces, Rotas e saltos da rede:
* Executou-se `ifconfig` para obter o IP `10.0.2.15` e a máscara `255.255.255.0` da placa `enp0s3`.


* Executou-se `route -n` para identificar o Gateway Padrão `10.0.2.2`.


* Testou-se o rastreamento de saltos até a internet com `traceroute 8.8.8.8`.

> `![Print1](https://github.com/ws-joaov/laboratorio-so-redes/raw/main/Capturas%20de%20Tela/ATV%205/Print1.png)`

> `![Print2](https://github.com/ws-joaov/laboratorio-so-redes/raw/main/Capturas%20de%20Tela/ATV%205/Print2.png)`

### 5.2. Diagnóstico no Host Windows
#### Executando netstat -an | findstr 5222 ANTES de ativar o redirecionamento (vazio/sem resposta).

1. No PowerShell do Windows, verificou-se o endereço IP da máquina física com `ipconfig /all`.

2. Verificou-se que a porta `2222` **não** estava em escuta no Windows antes da criação da regra no VirtualBox:

```powershell
netstat -an | findstr 2222

```

*(A consulta não retornou nenhuma linha, confirmando que a porta estava livre).*

3. Validação de Porta em Escuta (`LISTENING`) no Windows
#### Netstat -an | findstr 5222 no Windows DEPOIS de ativar a regra, mostrando a porta em estado LISTENING

Com a regra aplicada no VirtualBox, executou-se novamente a checagem no PowerShell:

```powershell
netstat -an | findstr 2222

```

4. Validação de Conexão Ativa (`ESTABLISHED`) no Windows
#### Netstat -an | findstr 5222 DURANTE a sessão SSH ativa, demonstrando o estado ESTABLISHED

Em uma segunda janela do PowerShell, com a sessão SSH mantida aberta, executou-se o diagnóstico de socket ativo:

```powershell
netstat -an | findstr 2222

```

> `![Print3](https://github.com/ws-joaov/laboratorio-so-redes/raw/main/Capturas%20de%20Tela/ATV%205/Print3.png)`

---

5. Configuração da Regra de Redirecionamento no VirtualBox
#### Regra SSH criada no VirtualBox (Porta Host 5222 -> Porta Guest 22)

Acessou-se **Configurações da VM** -> **Rede** -> **Adaptador 1 (NAT)** -> **Avançado** -> **Redirecionamento de Portas** e cadastrou-se a seguinte regra:

> `![Print4](https://github.com/ws-joaov/laboratorio-so-redes/raw/main/Capturas%20de%20Tela/ATV%205/Print4.png)`

---

6. Conexão SSH Remota a partir do Host
#### Terminal remoto no PowerShell logado via ssh -p 5222 administrador@127.0.0.1

No CMD do Windows, iniciou-se a sessão remota direcionada ao endereço local na porta `2222`:

```powershell
ssh administrador@127.0.0.1 -p 2222

```

* Efetuou-se a aceitação da chave de autenticidade (*fingerprint*) e inseriu-se a senha `adminifal`.

7. Auditoria de Sessões com `w` no Terminal Linux Remoto
#### Comando w dentro da sessão SSH, destacando a presença do terminal pts/0

Dentro do terminal SSH ativo no PowerShell, executou-se o comando `w` para inspecionar os logins ativos no servidor:

```bash
w

```

* **Análise da saída:** Identificou-se a sessão `tty1` (console local do VirtualBox) e a nova sessão `pts/0` (pseudo-terminal do acesso remoto SSH vindo de `10.0.2.2`).

> `![Print5](https://github.com/ws-joaov/laboratorio-so-redes/raw/main/Capturas%20de%20Tela/ATV%205/Print5.png)`

---

## 6. Problemas Enfrentados e Soluções

| Problema Encontrado | Causa do Problema | Solução Aplicada |
| --- | --- | --- |
| **Comando `ifconfig` não encontrado** | O pacote `net-tools` não vem instalado por padrão no Ubuntu Server recente.| Instalação prévia via `sudo apt install net-tools`.
| **Conexão recusada ao tentar `ssh -p 2222`** | IP do Guest cadastrado incorretamente na regra NAT ou serviço SSH inativo.| Confirmação do IP `10.0.2.15` no `ifconfig` e validação do pacote `openssh-server`.
| **Bloqueio no primeiro acesso SSH** | Ausência da aceitação inicial da chave pública do servidor. | Confirmação informando `yes` no prompt do PowerShell.

---

## 7. Conclusão

A atividade prática possibilitou compreender a mecânica de **Port Forwarding (Redirecionamento de Portas)** sob o modo de rede NAT no VirtualBox. 
Provou-se como o hipervisor pode atuar como um roteador virtual interceptando conexões na porta `2222` do Host e repassando-as para a porta padrão `22` do Guest. 
Além disso, os diagnósticos com `netstat`, `route` e `w` reforçaram a importância da auditoria de portas e monitoramento de sessões ativas em servidores de infraestrutura.
