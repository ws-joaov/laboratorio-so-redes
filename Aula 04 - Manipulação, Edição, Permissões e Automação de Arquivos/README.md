# Relatório Técnico – Aula Prática 04: Manipulação, Edição, Permissões e Automação de Arquivos no Linux

---

## 1. Identificação
* **Nome do Aluno:** João Vinícius Rocha Costa
* **Curso:** Bacharelado em Sistemas de Informação (BSI)
* **Disciplina:** Laboratório de Sistemas Operacionais e Redes
* **Data de Realização:** 26/08/2026
* **Título da Prática:** Manipulação, Edição, Permissões e Automação de Arquivos no Linux

---

## 2. Objetivo
Compreender os métodos de criação, edição e manipulação de arquivos de texto via terminal utilizando os editores Nano e Vim, manipular diretórios e aplicar permissões de execução. Além disso, desenvolver e validar scripts em Shell (`Bash`) para automação do cadastramento de usuários e definição de senhas em lote utilizando estruturas de repetição (`for`) e utilitários administrativos (`useradd` e `chpasswd`).

---

## 3. Ambiente do Laboratório
* **Sistema Operacional Hospedeiro (Host):** Windows
* **Hipervisor:** Oracle VM VirtualBox
* **Sistema Operacional Convidado (Guest):** Ubuntu Server 26.04 LTS (`ubuntu\_server`)
* **Configuração da VM:** 512 MB de RAM, 1 vCPU, 32 GB de HD (VDI, LVM)
* **Usuário de Administração:** `administrador` (com acesso `sudo`)

---

## 4. Procedimento de Configuração

### 4.1. Manipulação Básica e Edição de Arquivos
1. Criou-se o arquivo `configuracao.conf` via `touch` e inseriram-se as diretivas `PORTA=8080` e `TIMEOUT=30` utilizando o editor Nano.
2. Criou-se o diretório `backups` (`mkdir backups`) e realizou-se a cópia do arquivo de configuração (`cp configuracao.conf backups/`).
3. Renomeou-se o arquivo original para `config\_antiga.conf` via `mv` e posteriormente realizou-se a remoção de testes com o comando `rm -i`.


---

### 4.2. Automação: Preparação do Ambiente de Scripts
1. **Criação da Lista de Usuários:** Criou-se o arquivo `usuarios.txt` contendo a lista dos 20 usuários (`aluno01` até `aluno20`), um por linha.
2. **Desenvolvimento do Script de Criação (`passo1\_criar.sh`):**
   ```bash
   #!/bin/bash
   # Script de criação de usuários em lote
   for usuario in $(cat usuarios.txt); do
       echo "Processando criação do usuário: $usuario"
       sudo useradd -m -s /bin/bash $usuario
   done
   echo "Processo de criação concluído!"





3. **Desenvolvimento do Script de Senhas (`passo2\_senhas.sh`):**

    ```bash

    #!/bin/bash
    # Script de definição de senhas em lote
    for usuario in $(cat usuarios.txt); do
        echo "Definindo senha padronizada para: $usuario"
        echo "$usuario:$usuario" | sudo chpasswd
    done
    echo "Todas as senhas foram atualizadas com sucesso!"


    ```

---

### 4.3. Concessão de Permissões e Execução

Concedeu-se a permissão de execução aos arquivos de script através da notação simbólica `chmod +x`:

```bash

chmod +x passo1\_criar.sh

chmod +x passo2\_senhas.sh



```

Em seguida, executaram-se os scripts em ordem sequencial:

```bash

./passo1\_criar.sh

./passo2\_senhas.sh



```


---

## 5. Testes e Validação do Ambiente

### 5.1. Validação do Cadastramento em Lote (`getent`)

![Print ATV 4_Users_Pass](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%204/Print%20ATV%204_Users_Pass.png)
---

### 5.2. Teste de Acesso e Login Shell (`su -`)

![Print ATV 4_Validacao](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%204/Print%20ATV%204_Validacao.png)


![Print ATV 4_Login](https://github.com/ws-joaov/laboratorio-so-redes/blob/main/Capturas%20de%20Tela/ATV%204/Print%20ATV%204_Login.png)

---

## 6. Problemas Enfrentados e Soluções

| Problema Encontrado | Causa do Problema | Solução Aplicada |
| --- | --- | --- |
| **Erro `Permission denied` ao tentar rodar `./passo1\_criar.sh\*\*` | Os arquivos de script recém-criados no Linux não possuem permissão de execução por padrão. | Aplicação do comando `chmod +x passo1\_criar.sh passo2\_senhas.sh` antes da execução. |
| **Falha ao criar os usuários durante o laço `for\*\*` | O comando `useradd` exige privilégios de superusuário para alterar os arquivos do sistema. | Inclusão da instrução `sudo` antes das chamadas `useradd` e `chpasswd` dentro do script. |

---

## 7. Conclusão

A prática permitiu compreender o papel crucial da edição e manipulação de arquivos via linha de comando para administradores de sistemas. Além disso, demonstrou o poder e a eficiência do **Shell Scripting** na automação de tarefas repetitivas, como a criação massiva de contas e atualização de credenciais com `useradd` e `chpasswd`. Esse tipo de automação reduz drasticamente a margem de erro humano e o tempo de execução no gerenciamento de infraestruturas de rede.
