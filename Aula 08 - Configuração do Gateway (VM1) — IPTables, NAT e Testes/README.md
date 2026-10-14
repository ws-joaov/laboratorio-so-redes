# Relatório Técnico – Aula Prática 08: Configuração do Gateway (VM1) — IPTables, NAT e Testes

---

## 1. Identificação

* **Nome do Aluno:** João Vinícius Rocha Costa
* **Curso:** Bacharelado em Sistemas de Informação (BSI)
* **Disciplina:** Laboratório de Sistemas Operacionais e Redes
* **Data de Realização:** 07/10/26
* **Título da Prática:** Configuração do Gateway (VM1) — IPTables, NAT e Testes de Conetividade

---

## 2. Objetivo
Configurar o mascaramento de rede (NAT) no servidor Gateway (VM1) através do `iptables`, habilitar o encaminhamento de pacotes IPv4 no kernel de forma permanente e garantir a persistência das regras do firewall. Além disso, atualizar o ficheiro do Netplan na máquina cliente (VM2) com a rota padrão e validar a conetividade de saída para a Internet através de testes de `ping`, resolução de nomes via DNS (`nslookup`/`dig`) e rastreamento de saltos (`traceroute`).

---

## 3. Ambiente do Laboratório

### 3.1. Nomes de Host e Perfis das VMs
* **VM1 (Gateway):** `vm1-gateway` (Configurado via `hostnamectl`)
* **VM2 (Cliente):** `vm2-cliente` (Configurado via `hostnamectl`)

### 3.2. Mapeamento de Redes e Endereçamento

| Máquina Virtual | Nome do Host | Interface | Modo de Rede | Endereço IP / Sub-rede | Rota Padrão (Gateway) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **VM1** | `vm1-gateway` | `enp0s3` | Placa em Ponte (*Bridge*) | `172.20.23.1/22` | `172.20.20.1` |
| **VM1** | `vm1-gateway` | `enp0s8` | Rede Interna (`intnet`) | `10.0.0.1/24` | *N/A (É o Gateway)* |
| **VM2** | `vm2-cliente` | `enp0s3` | Rede Interna (`intnet`) | `10.0.0.2/24` | `10.0.0.1` (VM1) |

---

## 4. Procedimento de Configuração

### 4.1. Definição dos Nomes de Host (`hostnamectl`)
1. **No servidor Gateway (VM1):**
   ```bash
   sudo hostnamectl set-hostname vm1-gateway
   hostname

2. **Na máquina cliente (VM2):**
```bash
sudo hostnamectl set-hostname vm2-cliente
hostname

```



> ![Print_1](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%208/Print1.png)

---

### 4.2. Habilitação do Encaminhamento de IP no Kernel (VM1)

Ativou-se o encaminhamento de pacotes no kernel e aplicou-se a alteração de forma persistente:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-ipforward.conf
sudo sysctl --system

```

A verificação do parâmetro ativo foi realizada com o comando `sudo sysctl net.ipv4.ip_forward`.

> ![Print_2](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%208/Print2.png)

---

### 4.3. Regras de IPTables para NAT / Mascaramento (VM1)

No Gateway (`vm1-gateway`), limparam-se as regras existentes e aplicaram-se as diretivas de tradução de endereços (NAT) e autorização de tráfego entre a rede interna e externa:

```bash
# Limpeza de regras prévias
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -X

# Regra de Mascaramento na interface WAN (enp0s3)
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

# Regras de autorização de tráfego no encaminhamento (FORWARD)
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

```

> ![Print_3](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%208/Print3.png)

---

### 4.4. Atualização da Rota Padrão na VM2 (Netplan)

Editou-se o ficheiro `/etc/netplan/00-installer-config.yaml` na **VM2** para declarar a rota padrão apontando para a interface LAN da VM1 (`10.0.0.1`):

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 10.0.0.2/24
      routes:
        - to: default
          via: 10.0.0.1
      nameservers:
        addresses:
          - 172.20.20.1
          - 1.1.1.1
          - 8.8.8.8

```

Aplicou-se a alteração com `sudo netplan apply`.

> ![Print_4](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%208/Print4.png)

---

### 4.5. Persistência das Regras do IPTables

Para impedir que as regras de NAT sejam perdidas ao reiniciar o servidor, instalou-se o pacote `iptables-persistent` e salvaram-se as regras ativas no ficheiro `/etc/iptables/rules.v4`:

```bash
sudo apt update
sudo apt install -y iptables-persistent
sudo netfilter-persistent save

```

---

## 5. Testes e Validação do Ambiente

Com o NAT ativo na VM1 e a rota padrão configurada na VM2, executou-se a bateria de validação da conetividade de saída a partir da **VM2 (`vm2-cliente`)**.

### 5.1. Teste de Conetividade ICMP (`ping`)

Executaram-se os testes de alcance sequencial a partir da VM2:

```bash
ping -c 4 10.0.0.1     # Gateway interno (VM1)
ping -c 4 172.20.20.1  # Gateway externo da rede física
ping -c 4 1.1.1.1      # IP público da Internet (Cloudflare DNS)

```

* **Resultado:** Todos os testes de `ping` obtiveram resposta com 0% de perda de pacotes, confirmando que a VM1 está a mascarar e a encaminhar o tráfego da rede interna para a rede pública com sucesso.



> ![Print_5](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%208/Print5.png)

---

### 5.2. Teste de Resolução de Nomes (DNS)

Testou-se a resolução de nomes de domínio externos a partir da VM2:

```bash
nslookup google.com
# ou
dig +short google.com

```

* **Resultado:** O servidor DNS configurado no Netplan respondeu com o endereço IP de destino correspondente ao domínio solicitado.



> ![Print_6](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%208/Print6.png)

---

### 5.3. Rastreamento de Rota de Saída (`traceroute`)

Verificou-se o caminho percorrido pelos pacotes que saem da VM2 em direção à Internet:

```bash
traceroute google.com

```

* **Resultado:** O primeiro salto (*hop 1*) registado foi obrigatoriamente o Gateway da rede interna (`10.0.0.1`), seguido pelo Gateway da rede física (`172.20.20.1`) e pelos roteadores da Internet, comprovando o encaminhamento via NAT.



> ![Print_7](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%208/Print7.png)

---

## 6. Conclusão

A atividade prática permitiu transformar com sucesso um servidor Linux Ubuntu Server num **Gateway NAT funcional e seguro**. A habilitação do `ip_forward` em conjunto com a regra de mascaramento no `iptables` demonstrou como é realizada a tradução de endereços IP em ambientes de rede corporativos, permitindo que máquinas de uma sub-rede privada (`10.0.0.0/24`) acedam a recursos da Internet pública através de uma única interface WAN. Por fim, a utilização do `iptables-persistent` garantiu a continuidade e fiabilidade das regras do firewall após reinicializações do sistema.
