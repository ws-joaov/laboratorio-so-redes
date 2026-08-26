\# Relatório Técnico – Aula Prática 03: Estrutura de Diretórios (FHS) e Permissões Avançadas no Linux Server



\---



\## 1. Identificação

\* \*\*Nome do Aluno:\*\* João Vinícius Rocha Costa

\* \*\*Curso:\*\* Bacharelado em Sistemas de Informação (BSI)

\* \*\*Disciplina:\*\* Laboratório de Sistemas Operacionais e Redes

\* \*\*Data de Realização:\*\* 19/08/2026

\* \*\*Título da Prática:\*\* Estrutura de Diretórios (FHS) e Permissões Avançadas no Linux Server



\---



\## 2. Objetivo

Compreender a hierarquia padrão do sistema de arquivos do Linux (\*Filesystem Hierarchy Standard\* - FHS), criar estruturas de diretórios departamentais aninhadas utilizando o comando `mkdir -p`, aplicar políticas de segregação de acesso através de grupos e permissões octais (`770` e `660`) e validar o isolamento de ambientes corporativos via terminal.



\---



\## 3. Ambiente do Laboratório

\* \*\*Sistema Operacional Hospedeiro (Host):\*\* Windows

\* \*\*Hipervisor:\*\* Oracle VM VirtualBox

\* \*\*Sistema Operacional Convidado (Guest):\*\* Ubuntu Server 26.04 LTS (`ubuntu\_server`)

\* \*\*Usuário de Administração:\*\* `administrador` (com privilégios via `sudo`)



\---



\## 4. Procedimento de Configuração



\### 4.1. Inspeção da Estrutura FHS do Sistema

Inspecionou-se a organização padrão de diretórios do sistema:

\* Navegação até a pasta de configurações `/etc` (`cd /etc`) e verificação do conteúdo com `ls -F | head -n 15`.

\* Leitura das últimas linhas do log de autenticação localizado no diretório de dados variáveis com `sudo tail -n 10 /var/log/auth.log`.



\### 4.2. Criação Recursiva de Diretórios Departamentais

Sob o diretório de serviços locais `/srv`, criaram-se as pastas e subpastas das equipes de Tecnologia e Vendas em um único comando recursivo (`-p`):

```bash

cd /srv

sudo mkdir -p ti-dept/projetos vendas-dept/relatorios



```



\*A verificação da estrutura aninhada foi realizada executando o comando `ls -R` no diretório `/srv`.\*



\### 4.3. Criação de Grupos Departamentais e Associação de Usuários



1\. Criaram-se os grupos locais correspondentes aos departamentos:

```bash

sudo groupadd ti-group

sudo groupadd vendas-group



```





2\. Associaram-se os usuários criados nas práticas anteriores aos seus respectivos grupos:

```bash

sudo usermod -aG ti-group fulano

sudo usermod -aG vendas-group cicrano



```







\*(O usuário `novato` foi mantido sem grupo para servir como perfil externo de teste).\*



\### 4.4. Ajuste de Propriedades e Aplicação de Permissões Octais



1\. Alterou-se o dono e o grupo associado aos diretórios usando a sintaxe `usuario:grupo`:

```bash

sudo chown -R administrador:ti-group ti-dept/

sudo chown -R administrador:vendas-group vendas-dept/



```





2\. Aplicou-se a permissão de isolamento estrito `770` (`drwxrwx---`) nos diretórios:

```bash

sudo chmod 770 ti-dept

sudo chmod 770 vendas-dept



```







\### 4.5. Criação e Proteção de Arquivos Internos



Criou-se um arquivo confidencial no setor de Tecnologia e aplicou-se a permissão octal `660` (`-rw-rw----`):



```bash

sudo touch ti-dept/projetos/arquitetura\_rede\_vpn.txt

sudo chown administrador:ti-group ti-dept/projetos/arquitetura\_rede\_vpn.txt

sudo chmod 660 ti-dept/projetos/arquitetura\_rede\_vpn.txt



```



\---



\## 5. Testes e Validação do Ambiente



Para garantir a simulação idêntica a um novo login no servidor, utilizou-se o comando `su - <usuario>` (com hífen), forçando o carregamento do diretório pessoal `/home/<usuario>` e das variáveis de ambiente específicas do perfil.



\### 5.1. Teste A: Acesso Autorizado do Membro do Setor (`fulano`)



1\. Autenticação na sessão do usuário `fulano`:

```bash

su - fulano



```





2\. Tentativa de navegação e leitura dos arquivos do departamento de TI:

```bash

cd /srv/ti-dept

ls -l projetos/



```







\* \*\*Resultado:\*\* O acesso foi concedido com sucesso. O arquivo `arquitetura\_rede\_vpn.txt` foi exibido com as permissões `-rw-rw----`, pois o usuário `fulano` faz parte do `ti-group`.



3\. Encerramento da sessão com `exit`.



\### 5.2. Teste B: Bloqueio de Acesso de Usuário Externo (`cicrano`)



1\. Autenticação na sessão do usuário `cicrano` (membro da equipe de Vendas):

```bash

su - cicrano



```





2\. Tentativa de entrada na pasta confidencial do setor de TI:

```bash

cd /srv/ti-dept



```







\* \*\*Resultado:\*\* O sistema operacional bloqueou o acesso imediatamente com a mensagem `-bash: cd: /srv/ti-dept: Permission denied`, confirmando a eficácia da máscara `770`.



3\. Encerramento da sessão com `exit`.



\---



\### 5.3. Resolução do Desafio Prático de Laboratório



Atendendo ao desafio proposto para consolidação da prática:



1\. \*\*Criação da pasta e do grupo de Diretoria:\*\*

```bash

sudo mkdir -p /srv/diretoria-dept

sudo groupadd diretoria-group



```





2\. \*\*Atribuição do usuário `beltrano` ao novo grupo:\*\*

```bash

sudo usermod -aG diretoria-group beltrano



```





3\. \*\*Ajuste de propriedades e isolamento estrito (`770`):\*\*

```bash

sudo chown administrador:diretoria-group /srv/diretoria-dept

sudo chmod 770 /srv/diretoria-dept



```





4\. \*\*Criação e proteção do arquivo `orcamento\_ti.txt` (`660`):\*\*

```bash

sudo touch /srv/diretoria-dept/orcamento\_ti.txt

sudo chown administrador:diretoria-group /srv/diretoria-dept/orcamento\_ti.txt

sudo chmod 660 /srv/diretoria-dept/orcamento\_ti.txt



```





5\. \*\*Testes do Desafio:\*\*

\* \*\*Usuário `beltrano` (Diretoria):\*\* Executou `su - beltrano`, acessou `/srv/diretoria-dept` e visualizou/editou o arquivo `orcamento\_ti.txt` com sucesso.

\* \*\*Usuário `fulano` (TI):\*\* Executou `su - fulano` e tentou o comando `cd /srv/diretoria-dept`. O sistema negou o acesso exibindo `Permission denied`.







\---



\## 6. Problemas Enfrentados e Soluções



| Problema Encontrado | Causa do Problema | Solução Aplicada |

| --- | --- | --- |

| \*\*Erro de permissão ao tentar navegar em pastas criadas com `mkdir -p\*\*` | As subpastas herdaram permissões do `root` sem repassar o grupo correto durante a criação. | Aplicação da flag de recursividade `-R` no comando de alteração de posse (`sudo chown -R administrador:ti-group ti-dept/`). |

| \*\*Diferença de comportamento em testes entre usuários\*\* | Uso do comando `su usuario` em vez de `su - usuario`, mantendo o diretório de trabalho do administrador. | Adoção padronizada da sintaxe `su - usuario` para garantir uma \*Login Shell\* isolada. |



\---



\## 7. Conclusão



A prática permitiu compreender o papel central da árvore de diretórios do Linux (FHS) e como organizar serviços corporativos dentro da pasta `/srv`. Ficou comprovado que a combinação de diretórios criados de forma recursiva (`mkdir -p`), a atribuição precisa de grupos departamentais e o uso da permissão octal `770` garantem a segregação absoluta de informações confidenciais em servidores de produção compartilhados por múltiplos setores.



```



\---

