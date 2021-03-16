Roteiro Tutorial TaskQ

0. O que é o TaskQ?
O TaskQ é uma fila de tarefas, permitindo o uso de múltiplos usuários concorrentes
em um único servidor. O funcionamento é semelhante ao de ferramentas como o Slurm,
no entanto o TaskQ é direcionado a servidores *single machine*.

0.1. Como funciona o TaskQ?
O objetivo do TaskQ é permitir que usuários concorrentes façam uso de um mesmo
servidor, em um contexto em que os usários do servidor possuem limitações de privilégios.
Para que isso seja possível, o TaskQ deve ser instalado e iniciado por um usuário
que possua privilégios de *root*. Esse usuário, chamado **Host User**, será responsável
por executar as tarefas inseridas na fila pelos usuários do servidor, ou *Guest User*.
Nesse caso, cada *Guest User* insere suas respectivas tarefas na fila, porém quem
as executa é o usuário dono da fila, ou *Host User*. Por esse motivo é necessário
que cada usuário utilize uma pasta com configurações de privilégios específicos
que permita o acesso do *Host User*.


1. Instalação

1.1. Instalando o Screen
Para que o TaskQ funcione corretamente, é necessário instalar o Screen:

```
sudo apt-get update
sudo apt-get install screen
```

1.1. Instalando o TaskQ
Inicialmente, o TaskQ deve ser instalado pelo *Host User* utilizando privilégios
de *root*.

```
sudo pip install task-q
```

Em seguida, é necessário fazer a o setup inicial do TaskQ:

```
sudo taskq install $HOME $(id -u $USER)
```

1.2. Iniciando a fila de tarefas
Finalmente, é necessário inciar a fila de tarefas. Apenas o *Host User* possui
privilégios para tal.

```
taskq start
```

Da mesma maneira, é possível parar a fila com o comando:

```
taskq stop
```


2. Uso

2.1. Criando um Workspace
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

2.2. Adicionando uma task na fila
Para adicionar uma tarefa na fila, por exemplo o comando ```sleep 10```, rode o
seguinte comando:

```
taskq add 'sleep 10'
```

2.3. Verificando a fila
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

2.4. Abortando uma task
Para abortar uma tarefa, é necessário o ID da tarefa, que pode ser obtido com o
comando ```taskq show-queue```.

```
taskq abort <task_id>
```

3. Utilizando o Docker
As tarefas no TaskQ nada mais são que comandos. Nesse caso, para utilizar o Docker,
os comandos necessários para fazer o *build* de uma imagem, assim como para rodar
um container devem ser inseridos na fila.

3.1. Build de imagem
Para construir uma imagem, rode o seguinte comando:

```
docker build -t <image_name> /path/to/dockerfile_context/
```

Nesse comando, ```/path/to/dockerfile_context/``` se refere à pasta onde se
encontra o Dockerfile, ou seja, o contexto do Dockerfile.


3.2. Executando um experimento
Após construir a imagem, ela pode ser utilziada para rodar um experimento em um
container.

```
docker run --name <container_name> -v /host/dir:/container/dir <image_name> <command>
```

É importante lembrar que o volume utilizado pelo container deve estar dentro do
workspace configurado anteriormente.
