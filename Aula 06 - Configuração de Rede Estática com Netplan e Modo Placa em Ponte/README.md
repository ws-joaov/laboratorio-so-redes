# Relatório Técnico – Aula Prática 06: Configuração de Rede Estática com Netplan e Modo Placa em Ponte (Bridge Adapter) no VirtualBox

---

## 1. Identificação
* **Nome do Aluno:** João Vinícius Rocha Costa
* **Curso:** Bacharelado em Sistemas de Informação (BSI)
* **Disciplina:** Laboratório de Sistemas Operacionais e Redes
* **Data de Realização:** 23/09/26
* **Título da Prática:** Configuração de Rede Estática com Netplan e Modo Placa em Ponte (Bridge Adapter) no VirtualBox

---

## 2. Objetivo
Reconfigurar a interface de rede da máquina virtual Ubuntu Server no VirtualBox do modo NAT para o modo Placa em Ponte (*Bridge Adapter*), integrando a VM diretamente à sub-rede física do laboratório (`192.168.18.1/24`). Além disso, realizar a busca por um endereço IP livre via Host, configurar o endereçamento estático utilizando a sintaxe YAML do Netplan (`/etc/netplan/00-installer-config.yaml`) e validar a conectividade bidirecional e o roteamento externo (`traceroute`).

---

## 3. Ambiente do Laboratório
* **Sistema Operacional Hospedeiro (Host):** Windows (PowerShell/CMD)
* **Hipervisor:** Oracle VM VirtualBox
* **Sistema Operacional Convidado (Guest):** Ubuntu Server 26.04 LTS (`ubuntu_server`)
* **Configuração de Rede:** Placa em Ponte (*Bridge Adapter*) na sub-rede `192.168.18.1/24`
* **Usuário de Administração:** `administrador` (com privilégios via `sudo`)

---

## 4. Procedimento de Configuração

### 4.1. Verificação de IP Livre na Rede do Laboratório
No computador hospedeiro (Windows), executou-se o teste de disponibilidade no bloco `192.168.18.X` para encontrar um IP sem uso:

```powershell
ping 192.168.18.22

```

* **Resultado:** O comando retornou *"Esgotado o tempo limite do pedido"*, confirmando que o IP `192.168.18.22` estava **livre** para uso na VM.




> ![Print_1](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%206/Print_1.png)

---

### 4.2. Alteração do Modo de Rede no VirtualBox

Nas configurações da máquina virtual `ubuntu_server` no VirtualBox, alteraram-se as propriedades do Adaptador 1:

* **Conectado a:** Placa em Ponte (*Bridge Adapter*)


* **Nome:** Placa de rede física do Host conectada ao switch do laboratório


* **Cabo Conectado:** Marcado



> ![Print_2](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%206/Print_2.png)

---

### 4.3. Configuração do Endereço Estático via Netplan

No console da VM, editou-se o arquivo `/etc/netplan/00-installer-config.yaml` com o editor `nano` aplicando a seguinte estrutura:

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

```

Após salvar o arquivo, executou-se o comando para aplicar as alterações:

```bash
sudo netplan apply

```

> ![Print_3](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%206/Print_3.png)

---

## 5. Testes e Validação do Ambiente

### 5.1. Validação do IP Atribuído (`ip addr` / `netplan status`)

Verificou-se a atribuição correta do IP estático à interface `enp0s3`:

```bash
ip addr show enp0s3

```

* **Resultado:** A interface exibiu o endereço `192.168.18.22/24` ativo.



![Print_4](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%206/Print_4.png)

---

### 5.2. Testes de Conectividade Bidirecional (Ping)

1. **Host Windows para a VM Guest:**
```powershell
ping 192.168.18.22

```


* **Resultado:** Respostas recebidas sem perdas de pacotes (`0% de perda`), confirmando a visibilidade da VM na rede em ponte.




2. **VM Guest para o Host Windows:**
```bash
ping -c 4 192.168.18.8

```


* **Resultado:** 4 pacotes ICMP transmitidos e recebidos com sucesso.





> ![Print_5](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%206/Print_5.png)

---

### 5.3. Rastreamento de Rotas Externas (`traceroute`)

1. **Rastreamento para `google.com`:**
```bash
traceroute google.com

```


* **Resultado:** Primeiro salto (*hop 1*) confirmado no gateway padrão do laboratório (`192.168.18.1`), seguido pelo roteamento externo.




2. **Rastreamento para `one.one.one.one` (Cloudflare DNS):**
```bash
traceroute one.one.one.one

```


* **Resultado:** Resolução DNS concluída para `1.1.1.1` e saltos registrados com sucesso até o destino.





> ![Print_6](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%206/Print_6.png)

---

## 6. Problemas Enfrentados e Soluções

| Problema Encontrado | Causa do Problema | Solução Aplicada |
| --- | --- | --- |
| **Erro ao executar `netplan apply**` | Utilização inadvertida da tecla `Tab` para alinhamento das linhas ou indentação incorreta no arquivo YAML.| Correção do arquivo utilizando exclusivamente espaços simples para indentação e verificação do alinhamento hierárquico.
| **Conflito de IP na rede do laboratório** | Tentativa de usar um IP já alocado a outro terminal da rede. | Teste prévio de disponibilidade utilizando o `ping` via Host Windows antes de fixar o endereço no Netplan.

---

## 7. Conclusão

A prática permitiu compreender a diferença operacional entre as modalidades de rede no VirtualBox. O modo **Placa em Ponte (*Bridge Adapter*)** provou ser a solução ideal para servidores de produção que necessitam ser acessados diretamente por outros dispositivos da rede local sem intermediação de regras de NAT. Além disso, o uso do **Netplan** consolidou a aplicação de configurações estáticas declarativas no Ubuntu Server, garantindo que o servidor mantenha seu endereço IP fixo e rotas ativas após reinicializações do sistema.

