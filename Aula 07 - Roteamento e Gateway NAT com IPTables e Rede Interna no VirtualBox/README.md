# Relatório Técnico – Aula Prática 07: Encaminhamento e Gateway NAT com IPTables e Rede Interna no VirtualBox

---

## 1. Identificação

* **Nome do Aluno:** João Vinícius Rocha Costa
* **Curso:** Bacharelado em Sistemas 	de Informação (BSI)
* **Disciplina:** Laboratório de Sistemas Operacionais e Redes
* **Data de Realização:** 30/09/26
* **Título da Prática:** Encaminhamento e Gateway NAT com IPTables e Rede Interna no VirtualBox



---

## 2. Objetivo

Configurar um servidor Linux Ubuntu Server como **Gateway de Rede** interligando duas redes distintas: a rede externa do laboratório (WAN via *Bridge Adapter*) e uma rede privada isolada (LAN via *Internal Network*). A prática visa aplicar endereçamento IP estático duplo na máquina Gateway (VM1), configurar a máquina cliente (VM2) na rede interna apontando a VM1 como o seu Gateway Padrão e realizar os testes iniciais de conetividade local e verificação da necessidade do mascaramento NAT.

---

## 3. Ambiente do Laboratório

### 3.1. Arquitetura da Topologia

* **VM1 (Gateway / Routers):** 2 Placas de rede ativas (WAN em *Bridge Adapter* e LAN em *Internal Network*).


* **VM2 (Cliente da Rede Interna):** 1 Placa de rede ativa (LAN em *Internal Network*).


## 4. Procedimento de Configuração

### 4.1. Configuração dos Adaptadores no VirtualBox

1. **Configuração da VM1 (Gateway):**
* **Adaptador 1 (`enp0s3`):** Habilitado, conectado em **Placa em Ponte (*Bridge Adapter*)** apontando para a placa física do computador.


* **Adaptador 2 (`enp0s8`):** Habilitado, conectado em **Rede Interna (*Internal Network*)** com o nome `intnet`.




2. **Configuração da VM2 (Cliente):**
* **Adaptador 1 (`enp0s3`):** Habilitado, conectado em **Rede Interna (*Internal Network*)** com o nome idêntico `intnet`.





> ![Print_1](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%207/Print1.png)
> ![Print_2](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%207/Print2.png)
> ![Print_3](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%207/Print3.png)

---

### 4.2. Configuração do Netplan na VM1 (Gateway)

Acedeu-se ao ficheiro `/etc/netplan/00-installer-config.yaml` na VM1 e definiu-se a atribuição de IPs para a interface WAN (`enp0s3`) e LAN (`enp0s8`):

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 192.168.18.22/24
      routes:
        - to: default
          via: 192.168.18.1
      nameservers:
        addresses:
          - 192.168.18.1
          - 1.1.1.1
          - 8.8.8.8
    enp0s8:
      dhcp4: false
      addresses:
        - 10.0.0.1/24

```

Aplicaram-se as alterações com o comando `sudo netplan apply`.

> ![VM1](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%207/VM1.png)

---

### 4.3. Configuração do Netplan na VM2 (Cliente)

Acedeu-se ao ficheiro de rede no cliente VM2 e configurou-se o IP local `10.0.0.2/24`:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 10.0.0.2/24
      nameservers:
        addresses:
          - 192.168.18.22
          - 1.1.1.1
          - 8.8.8.8

```

Aplicaram-se as definições através de `sudo netplan apply`.

> ![VM2](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%207/VM2.png)

---

## 5. Testes e Validação do Ambiente

### 5.1. Teste de Conetividade Local (LAN Interna)

Realizou-se a verificação de comunicação direta entre o cliente (VM2) e a interface interna do Gateway (VM1):

1. **Ping da VM1 para a VM2 (`ping 10.0.0.2`):**
* **Resultado:** Comunicação estabelecida com sucesso na rede privada local.




2. **Ping da VM2 para a VM1 (`ping 10.0.0.1`):**
* **Resultado:** Pacotes ICMP entregues e respondidos com 0% de perda.





> ![PING1](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%207/Ping1.png)

---

### 5.2. Teste de Conetividade Externa e Diagnóstico de Ausência de NAT

Executaram-se os testes de saída a partir da **VM2** em direção à rede externa e à Internet para avaliar o comportamento do encaminhamento:

1. **Ping da VM2 para o Gateway Externo (`ping 172.20.20.1`):**
* **Resultado:** Sem resposta (tempo limite esgotado).




2. **Ping da VM2 para a Internet Pública (`ping 1.1.1.1`):**
* **Resultado:** Sem resposta / inacessível.




3. **Análise Técnica:** A falha no acesso externo a partir da VM2 nesta fase é o **comportamento esperado**, visto que o servidor VM1 ainda não possui as regras de tradução de endereços NAT (*MASQUERADE*) e o reencaminhamento de pacotes (`ip_forward`) ativado no kernel do Linux.



> ![PING2](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%207/Ping2.png)

---

### 5.3. Rastreamento de Rotas (`traceroute`)

Executou-se o rastreamento de rota a partir da VM1 e da VM2:

1. **Traceroute na VM1 (`traceroute 1.1.1.1`):**
* **Resultado:** O tráfego sai diretamente pela interface WAN `enp0s3` alcançando o gateway `172.20.20.1` e os saltos da Internet.


2. **Traceroute na VM2 (`traceroute 1.1.1.1`):**
* **Resultado:** O rastreamento interrompe-se no primeiro salto ou não avança além do Gateway, confirmando que os pacotes chegam à VM1 mas não são traduzidos nem reencaminhados para a interface WAN.





> ![PING3](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%207/Ping3.png)

---

## 6.  Conclusão

A prática permitiu estruturar uma topologia de rede corporativa dividida em dois segmentos distintos (WAN e LAN) utilizando virtualização no VirtualBox. A configuração da VM1 como máquina de dupla interface provou ser eficaz para delimitar a fronteira da rede privada, enquanto a VM2 foi devidamente integrada à LAN através da atribuição do Gateway Padrão `10.0.0.1`. A verificação empírica da ausência de acesso externo na VM2 evidenciou a importância essencial da implementação futura de serviços de tradução de endereços (NAT) e regras de filtragem via IPTables para viabilizar o roteamento seguro de dados.