## O Ansible é um orquestrador de servidores, pode ser utilizado para automatizar processos em escala, economizando tempo de forma simples e eficaz. 


## Instalação:
* ` sudo apt-get install software-properties-common `
* ` sudo apt-add-repository ppa:ansible/ansible `
* ` sudo apt-get update ` 
* ` sudo apt-get install ansible `

## Configuração
* Verifique se o pacote `openssh-client` e `openssh-server` estão instalados e funcionais na maquina que irá orquestrar as outras.
* Abra o arquivo ` /etc/ansible/hosts ` com um editor de texto e adicione novo grupo com os ips das maquinas que serão orquestradas, abaixo consta um exemplo(substitua os xx pelos digitos do ip desejado):

``` yaml

[swarm-dev]

swarm@xx.xx.xx.7 #maquina 1 
swarm@xx.xx.xx.8 #maquina 2 
swarm@xx.xx.xx.9 #maquina 3 
swarm@xx.xx.xx.10 #maquina 4 
swarm@xx.xx.xx.11 #maquina 5 

```
* Faça conexão ssh com cada uma das maquinas do grupo pelo menos uma vez e depois encerre essa conexão: `ssh swarm@200.239.93.X` e logo após `exit`
* Verifique se o python2 está instalado em todas as maquinas que serão orquestradas:
` python2 --version ` , se não tiver instalado o próprio terminal irá indicar o comando para realizar a instalação
* Feito os passos acima faça um teste de conexão do ansible com o grupo :  ` ansible swarm-dev -m ping -k `, caso esteja tudo ok irá aparecer uma mensagem de `SUCCESS` 

`OBS`: a opção `-k` é para passar o password de acesso ssh, caso já tenha uma chave configurada em todas as maquinas para o usuário `swarm` essa opção pode ser retirada.

## Utilizando playbooks:
Os playbooks do ansible são como scripts que são executados em paralelo nas maquinas orquestradas, é escrito em `yaml`. O playbook é dividido em tarefas(`tasks`) assim possibilitando uma visão melhor da execução em cada máquina. 

* Crie um playbook com o nome de test.yaml como o exemplo abaixo(todos os campos estão descritos nos comentários do arquivo): 
``` yaml
--- 
- name: Teste # Nome do playbook
  hosts: swarm-dev # Nome do grupo de maquinas que será orquestrada
  become: yes # necessário para autenticar como sudo
  serial: 5 #Numero de execuções em paralelo 
  become_user: root #ao entrar na maquina, irá autenticar como sudo 
  become_method: sudo # Método para entrar como sudo 
  tasks:

     - name: Indicando o nome da maquina e dando oi # Nome da tarefa
       command: bash -lc "hostname && echo oi"  # Módulo utilizado para mandar comandos 
       register: result #Variável que guarda o resultado da execução do comando
     - debug: # Utilizado para mostrar uma mensagem
         msg: "{{ result.stdout_lines  }}" # Mensagem com o resultado da execução
```
* Para utilizar o playbook : `ansible-playbook test.yaml -k -K`, `OBS`: o parâmetro `-K`é utilizado para passar a chave de autenticação como sudo. 


## Playbook que executa um script em escala:
O ansible tem um modulo `script` que é utilizado para executar um script local nas maquinas do grupo selecionado.

* Crie um script de teste chamado `script.sh`  como o abaixo : 

``` bash
#!/bin/bash
echo oi 
echo Executando script na maquina $(hostname)

```
* Crie um playbook com o nome script.yaml como o abaixo : 
``` yaml
--- 
- name: Playbook de execução de scripts # Nome do playbook
  hosts: swarm-dev # Nome do grupo de maquinas que será orquestrada
  become: yes # necessário para autenticar como sudo
  serial: 5 #Numero de execuções em paralelo 
  become_user: root #ao entrar na maquina, irá autenticar como sudo 
  become_method: sudo # Método para entrar como sudo 
  tasks:

     - name: Executando script em escala # Nome da tarefa
       script: /home/rodrigo/scripts/script.sh #Local do script
       register: result #Variável que guarda o resultado da execução do comando
     - debug: # Utilizado para mostrar uma mensagem
         msg: "{{ result.stdout_lines  }}" # Mensagem com o resultado da execução
```
* Verifique o caminho do script.sh no campo `script`do playbook, no exemplo o caminho é `/home/rodrigo/scripts/script.sh `  
* Execute o playbook: `ansible-playbook script.yaml -k -K `
