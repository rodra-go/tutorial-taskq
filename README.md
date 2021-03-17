# Tutorial TaskQ

## Conteúdo

- [1. Introdução](#1-introdução)
  * [1.1. O que é o TaskQ?](#11-o-que-é-o-taskq)
  * [1.2. Como funciona o TaskQ?](#12-como-funciona-o-taskq)
- [2. Instalação](#2-instalação)
  * [2.1. Instalando o Screen](#21-instalando-o-screen)
  * [2.2. Instalando o TaskQ](#22-instalando-o-taskq)
  * [2.3. Iniciando a fila de tarefas](#23-iniciando-a-fila-de-tarefas)
- [3. Uso](#3-uso)
  * [3.1. Criando um Workspace](#31-criando-um-workspace)
  * [3.2. Adicionando uma task na fila](#32-adicionando-uma-task-na-fila)
  * [3.3. Verificando a fila](#33-verificando-a-fila)
  * [3.4. Abortando uma task](#34-abortando-uma-task)
- [4. Utilizando o Docker](#4-utilizando-o-docker)
  * [4.1. Build da imagem](#41-build-da-imagem)
  * [4.2. Executando um experimento](#42-executando-um-experimento)


## 1. Introdução

### 1.1. O que é o TaskQ?
O TaskQ é uma fila de tarefas, permitindo o uso de múltiplos usuários concorrentes
em um único servidor. O funcionamento é semelhante ao de ferramentas como o Slurm,
no entanto o TaskQ é direcionado a servidores *single machine*.

### 1.2. Como funciona o TaskQ?
O objetivo do TaskQ é permitir que usuários concorrentes façam uso de um mesmo
servidor, em um contexto em que os usários do servidor possuem limitações de privilégios.
Para que isso seja possível, o TaskQ deve ser instalado e iniciado por um usuário
que possua privilégios de *root*. Esse usuário, chamado **Host User**, será responsável
por executar as tarefas inseridas na fila pelos usuários do servidor, ou *Guest User*.
Nesse caso, cada *Guest User* insere suas respectivas tarefas na fila, porém quem
as executa é o usuário dono da fila, ou *Host User*. Por esse motivo é necessário
que cada usuário utilize uma pasta com configurações de privilégios específicos
que permita o acesso do *Host User*.


## 2. Instalação

### 2.1. Instalando o Screen
Para que o TaskQ funcione corretamente, é necessário instalar o Screen:

```
sudo apt-get update
sudo apt-get install screen
```

### 2.2. Instalando o TaskQ
Inicialmente, o TaskQ deve ser instalado pelo *Host User* utilizando privilégios
de *root*.

```
sudo pip install task-q
```

Em seguida, é necessário fazer a o setup inicial do TaskQ:

```
sudo taskq install $HOME $(id -u $USER)
```

### 2.3. Iniciando a fila de tarefas
Finalmente, é necessário inciar a fila de tarefas. Apenas o *Host User* possui
privilégios para tal.

```
taskq start
```

Da mesma maneira, é possível parar a fila com o comando:

```
taskq stop
```


## 3. Uso

### 3.1. Criando um Workspace
É necessário que a pasta de trabalho utilizada pelo *Guest User* possua privilégios
específicos. Para configurar a pasta adequadamente, rode os seguintes comandos
na pasta escolhida para trabalhar:

```
sudo chown -R :100 /path/to/workspace
sudo chmod -R g+rwxs /path/to/workspace
sudo setfacl -d -m g::rwx /path/to/workspace
```

Os comandos acima configuram a pasta de modo que todos os arquivos criados nessa
pasta sejam acessíveis pelos usuários pertencentes ao grupo 100, ou grupo **users**.
Portanto, é necessário que o *Guest User* esteja incluído nesse grupo. Para incluir
um usuário no grupo 100, rode o seguinte comando:

```
sudo usermod -aG 100 <username>
```

### 3.2. Adicionando uma task na fila
Para adicionar uma tarefa na fila, por exemplo o comando ```sleep 10```, rode o
seguinte comando:

```
taskq add 'sleep 10'
```

### 3.3. Verificando a fila
Para verificar as tarefas aguardando na fila, utilize o comando *show-queue*:

```
taskq show-queue
```

O comando *show-queue* possui algumas opções de funcionamento:
- **all**: mostra todas as tarefas, incluindo as finalizadas e em processamento
- **running**: mostra apenas as tarefas em processamento
- **mine**: mostra apenas as tarefas criadas pelo usuário
- **done**: mostra apenas as tarefas finalizadas

Para utilizar a opção **running**, por exemplo, faça:

```
taskq show-queue --running
```

### 3.4. Abortando uma task
Para abortar uma tarefa, é necessário o ID da tarefa, que pode ser obtido com o
comando ```taskq show-queue```.

```
taskq abort <task_id>
```

## 4. Utilizando o Docker
As tarefas no TaskQ nada mais são que comandos. Nesse caso, para utilizar o Docker,
os comandos necessários para fazer o *build* de uma imagem, assim como para rodar
um container devem ser inseridos na fila. Para demonstrar a utilização, utilizaremos
um projeto exemplo construído com o Kedro, o qual está contido nesse repositório.

### 4.1. Build da imagem
Para construir uma imagem, rode o seguinte comando:

```
taskq add 'docker build -t tutorial-taskq /path/to/dockerfile_context/'
```

Nesse comando, ```/path/to/dockerfile_context/``` se refere à pasta onde se
encontra o Dockerfile, ou seja, o contexto do Dockerfile.


### 4.2. Executando um experimento
Após construir a imagem, ela pode ser utilziada para rodar um experimento em um
container.

```
taskq add 'docker run --rm --name tutorial_taskq -v /host/dir:/usr/src/code tutorial-taskq kedro run'
```

Para descobrir qual o endereço representado por ```/host/dir```, basta rodar o
comando ```pwd``` dentro da pasta do repositório.
É importante lembrar que o volume utilizado pelo container deve estar dentro do
workspace configurado anteriormente.
