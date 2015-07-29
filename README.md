# Oracle XE no Ubuntu utilizando o Docker

Este projeto possibilita a execução do [Oracle Database 11g Express Edition](http://www.oracle.com/technetwork/database/database-technologies/express-edition/overview/index.html) (Oracle XE) num Ubuntu 14.04 rodando em um contêiner Docker. Se, no lugar do Docker, você estiver procurando uma alternativa similar utilizando o Vagrant veja o projeto https://github.com/paulojeronimo/vagrant-ubuntu-oracle-xe.

## Instalação do Docker

No OS X, as instruções de instalação do Docker estão em https://docs.docker.com/installation/mac/.

Para utilizar o Docker no Fedora, leia https://docs.docker.com/installation/fedora/.

No Windows, siga os procedimentos descritos em http://docs.docker.com/windows/step_one/.

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
