# Relatório Técnico – Aula Prática 02: Administração de Usuários, Grupos e Permissões no Linux



---



## 1. Identificação
* **Nome do Aluno:** João Vinícius Rocha Costa
* **Curso:** Bacharelado em Sistemas de Informação (BSI)
* **Disciplina:** Laboratório de Sistemas Operacionais e Redes
* **Data de Realização:** 12/08/2026
* **Título da Prática:** Administração de Usuários, Grupos e Permissões no Ubuntu Server 26.04 LTS



---




## 2. Objetivo

Capacitar-se na administração do controle de acesso no Ubuntu Server 26.04 LTS, englobando a criação e gerenciamento de contas de usuário e grupos de trabalho, o manuseio de propriedades de arquivos/diretórios via `chown` e `chgrp`, a aplicação de permissões em modo octal (`chmod`) e a validação do isolamento de acessos via terminal.



---



## 3. Ambiente do Laboratório

* **Sistema Operacional Hospedeiro (Host):** Windows

* **Hipervisor:** Oracle VM VirtualBox

* **Sistema Operacional Convidado (Guest):** Ubuntu Server 26.04 LTS (`ubuntu\_server`)

* **Usuário de Administração:** `administrador` (com acesso `sudo`)



---



## 4. Procedimento de Configuração



### 4.1. Criação dos Usuários do Sistema

Com o usuário `administrador`, foram criadas quatro contas de teste no sistema através do utilitário `adduser`:

```bash

sudo adduser fulano

sudo adduser cicrano

sudo adduser beltrano

sudo adduser novato



```



*A validação das contas foi confirmada inspecionando as últimas linhas do arquivo `/etc/passwd` com `tail -n 4 /etc/passwd`.*



### 4.2. Criação de Grupos e Associação de Usuários



1\. Criou-se o grupo de desenvolvimento `devs`:

```bash

sudo groupadd devs



```





2\. Adicionaram-se os usuários `fulano`, `cicrano` e `beltrano` ao grupo `devs` (o usuário `novato` permaneceu fora do grupo):

```bash

sudo usermod -aG devs fulano

sudo usermod -aG devs cicrano

sudo usermod -aG devs beltrano



```







*A verificação do grupo foi realizada consultando o arquivo `/etc/group` via `grep "devs" /etc/group`.*



### 4.3. Criação do Diretório Compartilhado e Posse



1\. Foi criada a pasta compartilhada `/srv/projeto`:

```bash

sudo mkdir -p /srv/projeto



```





2\. A posse do diretório foi transferida para o usuário `administrador` e o grupo associado para `devs`:

```bash

sudo chown administrador /srv/projeto

sudo chgrp devs /srv/projeto



```







### 4.4. Aplicação de Permissões (Modo Octal)



1\. Aplicou-se a permissão octal `770` ao diretório `/srv/projeto` (Dono e Grupo com `rwx`; Outros sem acesso):

```bash

sudo chmod 770 /srv/projeto



```





2\. Criou-se o arquivo `/srv/projeto/config\_redes.txt` e configurou-se a permissão `660` (Dono e Grupo com `rw-`; Outros sem acesso):

```bash

echo "Especificacao tecnica do roteador de borda" | sudo tee /srv/projeto/config\_redes.txt

sudo chmod 660 /srv/projeto/config\_redes.txt



```







---



## 5. Testes e Validação do Ambiente



A alternância de sessões para os testes foi realizada utilizando o comando `su - <usuario>` para garantir o carregamento de uma *Login Shell* limpa com as variáveis de ambiente e diretório base do perfil testado.



### 5.1. Teste A: Acesso com Usuário do Grupo (`fulano`)



1\. Alternou-se para o usuário `fulano`:

```bash

su - fulano



```





2\. Navegação e edição do arquivo compartilhado:

```bash

cd /srv/projeto

echo "Revisado por Fulano" >> config\_redes.txt

cat config\_redes.txt



```







* **Resultado:** O usuário `fulano` conseguiu navegar no diretório e acrescentar conteúdo ao arquivo sem restrições, pois pertence ao grupo `devs`.



3\. Encerrou-se a sessão com `exit`.



### 5.2. Teste B: Bloqueio com Usuário Externo (`novato`)



1\. Alternou-se para o usuário `novato`:

```bash

su - novato



```





2\. Tentativa de navegação no diretório protegido:

```bash

cd /srv/projeto



```







* **Resultado:** O sistema retornou `-bash: cd: /srv/projeto: Permission denied`.



3\. Tentativa de listagem do conteúdo:

```bash

ls -l /srv/projeto



```







* **Resultado:** O sistema retornou `ls: cannot open directory '/srv/projeto': Permission denied`, validando o isolamento da permissão octal `770`.



4\. Encerrou-se a sessão com `exit`.



---



### 5.3. Execução do Exercício Prático de Fixação



Atendendo ao desafio proposto na atividade:



1\. **Criação do grupo `financeiro`:**

```bash

sudo groupadd financeiro



```





2\. **Associação dos usuários `cicrano` e `beltrano` ao novo grupo:**

```bash

sudo usermod -aG financeiro cicrano

sudo usermod -aG financeiro beltrano



```





3\. **Criação da pasta e ajuste de propriedades:**

```bash

sudo mkdir -p /srv/financeiro

sudo chown administrador /srv/financeiro

sudo chgrp financeiro /srv/financeiro



```





4\. **Aplicação das restrições de permissão (`770`):**

```bash

sudo chmod 770 /srv/financeiro



```





5\. **Validação dos privilégios do setor Financeiro:**

* **Usuário `cicrano` (Pertence ao Financeiro):**

Ao executar `su - cicrano`, conseguiu navegar até `/srv/financeiro`, criar o arquivo `relatorio.txt` e salvar alterações.

* **Usuários `fulano` e `novato` (Não pertencem ao Financeiro):**

Ao executar `su - fulano` ou `su - novato` e tentar o comando `cd /srv/financeiro`, ambos receberam `Permission denied`.







---



## 6. Problemas Enfrentados e Soluções



| Problema Encontrado | Causa do Problema | Solução Aplicada |



| Comportamento inesperado ao testar permissões com `su` | Utilização do comando `su usuario` sem o hífen `-`, mantendo o diretório e contexto da sessão anterior. | Utilização estrita da sintaxe `su - usuario` para forçar o carregamento de uma *Login Shell* completa. |

| Erro ao tentar editar arquivo em `/srv/projeto` | Arquivo criado incialmente pelo `root` sem a permissão de escrita para o grupo `devs`. | Aplicação do comando `chmod 660 /srv/projeto/config\_redes.txt` definindo escrita para o grupo. |



---



## 7. Conclusão



A prática demonstrou a eficiência do modelo de controle de acesso nativo do Linux baseado na trinca Dono/Grupo/Outros. A utilização combinada de grupos de trabalho (`groupadd`/`usermod`) e diretórios com permissões estritas em modo octal (`chmod 770`) provou ser uma solução simples e robusta para isolar dados sensíveis de diferentes departamentos em um servidor corporativo.

