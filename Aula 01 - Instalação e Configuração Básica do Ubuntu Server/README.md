# Relatório Técnico – Aula Prática 01: Virtualização e Instalação do Ubuntu Server 26.04

---

## 1. Identificação
* **Nome do Aluno:** João Vinícius Rocha Costa
* **Curso:** Bacharelado em Sistemas de Informação (BSI)
* **Disciplina:** Laboratório de Sistemas Operacionais e Redes
* **Data de Realização:** 05/08/2026
* **Título da Prática:** Introdução à Virtualização e Instalação do Ubuntu Server 26.04 LTS

---

## 2. Objetivo
Compreender os conceitos fundamentais de virtualização e gerência de recursos, configurar uma máquina virtual através do Oracle VM VirtualBox e realizar a instalação e pós-configuração do sistema operacional Ubuntu Server 26.04 LTS com particionamento flexível utilizando LVM.

---

## 3. Ambiente do Laboratório

### 3.1. Máquina Hospedeira (Host)
* **Sistema Operacional:** Windows
* **Diretório da ISO Original:** `C:\2026\BSI\VM\original\ubuntu-26.04-live-server-amd64.iso`
* **Diretório da VM:** `C:\2026\BSI\VM\2023007721\ubuntu_server`

### 3.2. Configuração da Máquina Virtual (Guest)
* **Hipervisor:** Oracle VM VirtualBox (com Extension Pack instalado)
* **Nome da VM:** `ubuntu_server`
* **Sistema Operacional:** Ubuntu (64-bit)
* **Memória RAM:** 512 MB
* **Processador:** 1 vCPU
* **Disco Rígido Virtual:** 32 GB (VDI, Dinamicamente Alocado)
* **Interface de Rede:** NAT (Interface `enp0s3`)

---


## 4. Procedimento de Instalação e Configuração

### 4.1. Preparação do Ambiente Host
1. Copiou-se a imagem ISO do Ubuntu Server 26.04 contida no servidor do laboratório (`\\172.20.22.179\labredes`) para o diretório local `C:\2026\BSI\VM\original\ubuntu-26.04-live-server-amd64.iso`.
2. Foi criada a estrutura de diretório de trabalho do aluno no host: `C:\2026\BSI\VM\2023007721\ubuntu_server`.

### 4.2. Criação e Montagem da VM no VirtualBox
1. Foi criada uma nova VM nomeada `ubuntu_server` vinculada à pasta criada no host.
2. Foram alocados 512 MB de memória RAM e 1 vCPU.
3. Foi criado um disco rígido virtual de 32 GB no formato VDI (Dinamicamente Alocado).
4. A imagem ISO do Ubuntu Server foi apontada na controladora IDE virtual da VM.

### 4.3. Instalação Interativa do Ubuntu Server
1. **Idioma e Teclado:** Selecionou-se o idioma `English` para o sistema operacional e o layout de teclado `Portuguese (Brazil)`.
2. **Rede:** A interface de rede `enp0s3` foi configurada automaticamente via DHCP.
3. **Proxy e Mirror:** As configurações padrão foram mantidas.
4. **Particionamento Personalizado (Custom Storage Layout - LVM):**
   * `/boot`: 1 GB (`ext4`) — Partição primária separada.
   * **Volume Group (`ubuntu-vg`):** Criado com o espaço livre do disco.
   * **Volume Lógico Root (`/`):** Nome `ubuntu-lv`, tamanho de 29 GB (`ext4`).
   * **Volume Lógico Swap:** Nome `swap-lv`, tamanho de ~2 GB (`swap`).
5. **Configuração de Perfil:**
   * **Name:** `Administrador`
   * **Server Name:** `ubuntu_server`
   * **Username:** `administrador`
   * **Password:** `adminifal`
6. **Serviços Adicionais:** Foi habilitada a instalação do **OpenSSH Server**.
7. **Conclusão:** Finalizou-se a instalação e realizou-se a reinicialização da máquina virtual com a remoção da mídia de instalação ISO.

---

## 5. Testes e Validação pós-instalação

A validação do ambiente foi executada via terminal do sistema recém-instalado após a reinicialização.

### 5.1. Primeiro Login
O primeiro acesso foi efetuado com sucesso utilizando o usuário `administrador` e a senha atribuída (`adminifal`).

### 5.2. Verificação da Interface de Rede
A validação de atribuição do endereço de IP foi realizada com o comando:
```bash
ip addr

Resultado: A interface `enp0s3` obteve com sucesso um endereço IPv4 no intervalo local (`10.0.2.15/24`) via DHCP do VirtualBox.

### 5.3. Verificação do Particionamento LVM

Para checar a correta aplicação do layout de partições LVM, executou-se:

```bash
df -h

```

* **Resultado:** Confirmou-se o ponto de montagem principal `/` alocado sobre o volume lógico `/dev/mapper/ubuntu--vg-ubuntu--lv` (29 GB) e a partição `/boot` alocada separadamente em `/dev/sda1` (1 GB).

### 5.4. Atualização dos Repositórios de Pacotes

A conectividade externa e o funcionamento do gerenciador de pacotes `apt` foram validados executando:

```bash
sudo apt-get update

```

* **Resultado:** As listas de pacotes dos repositórios oficiais do Ubuntu foram baixadas e atualizadas sem erros.

---

## 6. Problemas Enfrentados e Soluções

| Problema Encontrado | Causa do Problema | Solução Aplicada |
| --- | --- | --- |
| **Lentidão / Falha no Download de Pacotes** | Instabilidade pontual de rede durante a instalação do instalador interativo. | Mantidos os espelhos (mirrors) padrão e atualizada a lista de repositórios via `apt-get update` no pós-instalação. |
| **Dificuldade na digitação de caracteres especiais** | Layout do teclado desconfigurado no boot inicial. | Seleção explícita do layout `Portuguese (Brazil)` na primeira etapa de configuração do instalador. |

---

## 7. Conclusão

A prática permitiu compreender o fluxo completo de criação, alocação de recursos e instalação de um sistema operacional de rede em ambiente virtualizado. Foi possível analisar a aplicação prática do **LVM (Logical Volume Manager)** para flexibilização do armazenamento do servidor, bem como a importância da instalação do **OpenSSH Server** para permitir o gerenciamento remoto em ambientes de data center.