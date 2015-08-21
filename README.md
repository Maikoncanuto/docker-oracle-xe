# Oracle XE no Ubuntu utilizando o Docker

Este projeto possibilita a execução do [Oracle Database 11g Express Edition](http://www.oracle.com/technetwork/database/database-technologies/express-edition/overview/index.html) (Oracle XE) em uma box Vagrant ou num contêiner Docker. De qualquer forma, quando utilizado o Vagrant, o provisionamento da box será realizado através do Docker. Quando a opção for utilizar o Docker, no Windows ou no OS X, a máquina que executa o servidor Docker poderá ser construída (mais facilmente) através de uma das seguintes formas: utilizando o [Boot2docker](http://boot2docker.io/) ou, mais a recentemente, o [Docker Machine](boot2docker://docs.docker.com/machine/).

## Optando pelo Vagrant

O Vagrant pode provisionar uma VM utilizando o Docker. Talvez essa seja a solução mais simples e rápida para utilizar este projeto no caso de você estar utilizando um ambiente Windows ou OS X. Não será necessário instalar o Docker pois o Vagrant sabe como provisionar máquinas através dele mesmo sem o uso de seus binários.

Optando pelo uso do Vagrant, a execução do Oracle XE através deste projeto pode ser realizada com um único comando:
```
cp Vagrantfile.1 Vagrantfile
vagrant up
```

O provisionamento da máquina criada será realizado através do Docker seguindo as instruções do [Vagrantfile](Vagrantfile) e, após a finalização desse processo, você poderá utilizar acessar o Oracle XE.

### Contetando-se ao Oracle XE

Utilize as seguintes informações para estabelecer uma conexão:
* hostname: localhost
* port: 1521
* sid: xe
* username: system
* password: oracle

### Salvando a box

Salvar uma box é uma boa opção para poder restaurá-la rapidamente e não precisar perder tempo no seu provisionamento. Para isso, pare a box:
```
vagrant halt
```

Em seguida, empacote-a:
```
vagrant package --output oracle-xe.box
```

Agora, destrua a box (se tiver certeza que já foi gerado o arquivo ``package.box``):
```
vagrant destroy
```

Uma nota: No Cygwin, [ocorre um erro](https://groups.google.com/forum/#!topic/vagrant-up/ExFet5jMomU) na tentativa de execução do comando acima. A solução é acrescentar o parâmetro ``--force`` ao final desse comando.

### Restaurando e utilizando a box salva
```
vagrant box add --name oracle-xe oracle-xe.box
cp Vagrantfile.2 Vagrantfile
vagrant up
```

### Optando pelo Docker

Se você estiver utilizando o Linux em teu desktop, o uso do Vagrant perde o sentido. Nesse caso, é extremamente mais simples e rápida a utilização do Docker.

Em ambientes Windows ou OS X, se você não desejar utilizar o Vagrant, você precisará fazer a instalação do Boot2Docker ou do Docker Machine. Essas aplicações são necessárias para criar uma VM (Linux) que executará o servidor Docker e o contêiner oracle-xe criado neste projeto.

## Instalação do Docker

No OS X, as instruções de instalação do Docker estão em https://docs.docker.com/installation/mac/.

Para utilizar o Docker no Fedora, leia https://docs.docker.com/installation/fedora/.

No Windows, siga os procedimentos descritos em http://docs.docker.com/windows/step_one/. Alternativamente, utilize o [Chocolatey](http://chocolatey.org) para fazer a instalação dessas ferramentas. Instruções para isso podem ser encontradas em https://github.com/paulojeronimo/dicas-windows/blob/master/chocolatey.md#docker-e-docker-machine.

## Utilização do Docker Image

Um servidor Docker pode ser hospedado no ambiente Windows se for criada uma máquina virtual Linux através do Boot2Docker ou do Docker Image. O Boot2Docker é um projeto menos abrangente e mais simples que o Docker Image. Esse último é a solução mais atual.

Todos os procedimentos a seguir são realizados num shell (Bash) aberto pelo Cygwin.

### Criação da VM (utilizando um shell Cygwin)

```
docker-machine env --shell bash dm1
```

### Configuração das variáveis de ambiente utilizadas pelo Docker

```
eval $(docker-machine env --shell bash dm1)
```

### Determinação do IP do contêiner

```
docker-machine ip dm1
```

## Carregamento da imagem

Verifique se a imagem ``oracle-xe`` já é listada como na saída do comando a seguir:
```
$ docker images
REPOSITORY                          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
oracle-xe                           latest              923b5341870b        12 seconds ago      2.241 GB
...
```

Caso a tua execução do comando acima não apresente uma saída similar:
* Se você não possuir o arquivo da imagem (``oracle-xe.tar``), faça sua geração seguindo os passos em ["Construção e salvamento da imagem"](#construcao_da_imagem) para gerar a imagem e salvá-la nesse arquivo.
* Se você já possuir o arquivo da imagem não será necessária a sua construção. Sendo assim, vocẽ poderá simplesmente fazer o seu carregamento:
```bash
docker load -i oracle-xe.tar
```

## Execução (e criação) do contêiner

Execute o contêiner:
```
$ docker run --name oracle-xe -d -p 1522:22 -p 1521:1521 oracle-xe
acff4b050f8e983367945ad0677efd8f9d89f100dad27ad52bb1a063e69d2f59
```

Verifique se ele está mesmo em execução:
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED              STATUS              PORTS                                                    NAMES
acff4b050f8e        oracle-xe:latest    "/bin/sh -c '/usr/sb   About a minute ago   Up About a minute   0.0.0.0:1521->1521/tcp, 8080/tcp, 0.0.0.0:1522->22/tcp   oracle-xe 
```

Após inicializado o contêiner, você pode abrir o [Oracle SQL Developer](http://www.oracle.com/technetwork/developer-tools/sql-developer/overview/index-097090.html) e testar uma conexão ao Oracle XE. Os parâmetros para essa conexão são os seguintes:
```
hostname: localhost
port: 1521
sid: xe
username: system
password: oracle
```

Se desejar acessar o shell do contêiner do Oracle XE, execute:
```
ssh -p 1522 root@localhost
```

A senha do ``root`` é ``admin``.

## Finalização do contêiner

Quando não precisar mais do Oracle XE em execução, pare o contêiner:
```
docker stop oracle-xe
```

Você pode verificar se ele foi realmente parado com o comando ``docker ps``.

## Reinicialização do contêiner

Para colocar o contêiner e em execução novamente, execute:
```
docker start oracle-xe
```
Lembre-se que você pode, a qualquer momento, verificar o status de execução do contêiner através do comando ``docker ps``.

## Remoção do contêiner

Quando desejar remover o Oracle XE (isso destuirá todos os bancos), remova o contêiner:
```
docker rm oracle-xe
```

## <a name="construcao_da_imagem"></a>Construção e salvamento da imagem

Clone o projeto:
```
git clone https://github.com/paulojeronimo/docker-oracle-xe.git
```

Construa a imagem (esse procedimento deve demorar ... vá tomar um café enquanto isso):
```
docker build --rm -t oracle-xe .
```

Verifique se a imagem está sendo listada:
```
$ docker images
REPOSITORY                          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
oracle-xe                           latest              923b5341870b        12 seconds ago      2.241 GB
...
```

Salve a imagem (gerando o arquivo ``oracle-xe.tar``):
```
docker save -o oracle-xe.tar oracle-xe
```

## Remoção da imagem

Para remover a imagem (por exemplo, para ganhar espaço no disco) execute:
```
docker rmi oracle-xe
```
