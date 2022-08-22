
# Docker-Fundamentals

Comandos gerais para utilização com Docker, e projetos relevantes. Estaremos utilizando Linux - Ubuntu Server.



### Tópicos relevantes

- **Linux - Server:** https://ubuntu.com/download/server
- **Linux - Desktop:** https://ubuntu.com/download/desktop
- **DockerHub:** https://hub.docker.com/


### Instalação do Docker

#### Documentação da instalação: https://docs.docker.com/engine/install/ubuntu/

Passo a passo
```bash
  curl -fsSL https://get.docker.com -o get-docker.sh (baixando script de instalação do docker)
  sh get-docker.sh (executando o script)
```

### Imagens de container: 
#### Onde pesquisar: https://hub.docker.com/

As imagens disponibilizadas dos containers nos permitem inicializar algum tipo de ambiente específico, no desenvolvimento por exemplo, se preciso de um ambiente para linux, posso ir no DockerHub e pegar a imagem do ubuntu e rodar o container como se estivesse usando o sistema localmente de forma facilitada.
```bash
   docker pull ubuntu (se não especificar a versão ele baixa a ultima)
   docker images (lista as imagens baixadas)
   docker ps (lista os containers em execução)
   docker ps -a (lista todos os containers, até os que pararam)
   docker run ubuntu sleep 10 (roda o container do ubuntu por 10s)
```
#### Rodando o container do ubuntu e permitindo pseudo terminal e interação:
```bash
   docker run -it ubuntu (i - para modo interativo e t - para peseudo terminal)
```

PS: temos duas formas de utilizar os comandos docker, duas sintaxes diferentes "docker container" ao invés de apenas "docker", os dois funcionam.

- docker ps = docker container ls 
- docker run = docker container run 

#### Rodando o container especificando a versão da imagem com TAG
```bash
   docker pull debian:9 (: para indicar 9 que é TAG)
   docker run -dit debian:9 (lembrar de indicar a TAG ao iniciar ou ele vai baixar a versão final disponível)
```
#### Rodando o container com pseudo terminal e nomeando a imagem
```bash
   docker run -dit  --name image-ubuntu ubuntu (d - para rodar tambem em plano de fundo e não parar do nada i - para modo interativo e t - para peseudo terminal / "--name Nome" para nomear a imagem)
```
#### Acessando o bash do container ubuntu (podemos acessar outros aplicativos tbm)
```bash
   docker exec -it id/name /bin/bash (substitua o id pela numeracao ou apenas use o name, pode ser visto usando o docker ps)
```
Aqui você precisa atualizar os repositórios:
```bash
    apt update
    apt upgrade -y
    apt -y install nano (não tínhamos ele no container ubuntu)
    exit (sair do container)
```
#### Criando uma pasta dentro do container em execução
```bash
   docker exec id/name mkdir /destino (exec vai executar o mkdir substitua o id pela numeracao ou apenas use o name, pode ser visto usando o docker ps)
```
#### Copiando um arquivo local para o container
```bash
   docker cp meuarquivo.txt id/name:/destino (substitua o id pela numeracao ou apenas use o name, pode ser visto usando o docker ps)
```
#### Copiando vários arquivos locais para o container
```bash
    apt install zip -y
    zip meuzip.zip *.txt
    docker cp meuzip.zip id/name:/destino (substitua o id pela numeracao ou apenas use o name, pode ser visto usando o docker ps)
```
#### Parando o container e reiniciando
```bash
   docker stop id/name (substitua o id pela numeracao ou apenas use o name, pode ser visto usando o docker ps)
   docker start id/name (Aqui os dados não foram perdidos, mas caso você apague o container, para não perder dados deve mapear um lugar para salvar eles)
```
#### Excluindo o container
```bash
   docker rm id/name (substitua o id pela numeracao ou apenas use o name, pode ser visto usando o docker ps, se você colocar um -f ele força a exclusão e não precisa dar um stop antes)
```

#### Excluindo tudo?
 PS = Muito cuidado, não vá utilizar sem cautela, esses comandos realmente apagam tudo em desuso, vai perder dados se não for sábio.
```bash
   docker container prune (containers)
   docker volume prune (volumes)
```

### Docker image MYSQL - Exemplo 
```bash
   docker pull mysql (Ele vai buscar a imagem do mysql no dockerhub, ultima versão porque não especifiquei nenhuma TAG)
   docker run -e MYSQL_ROOT_PASSWORD=root --name mysql-container -d -p 3306:3306 mysql (-d executa em segundo plano, -p para publicar as portas do container para acessar via host, -e para setar variaveis de ambiente, o ultimo comando é o nome da imagem que foi baixada)
   
   docker exec -it mysql-container bash (estou chamando o bash do container)
   mysql -u root -p --protocol=tcp (aqui estamos passando o user com -u e o protocolo usado, após confirmar com a senha definida no run veremos a identificação "mysql" e aí sim poderemos por exemplo, criar um database)
   
   CREATE DATABASE aulateste;
   SHOW DATABASES;
   exit (para sair do mysql)
   exit (para sair do container)
   
   ip a (procure a interface de rede com nome de docker, cada container recebe um ip)
   docker inspect nomeContainer (retorna informações do seu container)
   apt -y install mysql-client (com ele não precisamos abrir o bash do container só para entrar no ambiente "mysql", só executar o comando do mysql -u root -p --protocol=tcp)

```

#### Excluindo o container MYSQL - CUIDADOS
Ao excluir o container os dados serão apagados, para envitar isso mapeamos um local para guardar os dados do volume, um local que não seja dentro do container, isso é feito na execução do container:

```bash
   docker inspect mysql-container (usaremos para procurar a origem dos dados do container mysql, procure por volume para encontrar o caminho - procurando achamos /var/lib/mysql")
   
   docker run -e MYSQL_ROOT_PASSWORD=root --name mysql-container -d -p 3306:3306 mysql --volume=/data/mysql-container:/var/lib/mysql (aqui nós criamos uma pasta chamada data e dentro dela uma pasta com o nome do container, direto da raiz do diretório, assim os dados serão salvos ali - indicamos então DESTINO:ORIGEM dos dados do container usando o -v ou --volume=)
```
#### Essa forma de persistir dados usada acima é chamada de mount do tipo BIND - que é basicamente vincular um determinado diretório ou arquivo
#### PS: Ao indicar o volume na hora de subir o container, os dados do mysql serão salvos no Host, o que me permite excluir o container e manter os dados tranquilamente, quando for subir um novo container mysql é só você indicar o mesmo volume e ele irá receber os dados que estavam salvos na pasta destino.


#### Persistência de dados com mount
##### Bind
  Vincular um arquivo ou diretório do host para o container, assim permitindo que os dados fiquem armazenados e sejam reutilizados em próximas instâncias de container.
  Exemplo:
  
###### docker run -dti --mount type=bind,src=/datadir/imagedir,dst=/data debian
  (aqui --mount permite escolher o tipo de mount, que no caso é bind, src é nossa pasta local, dst indica uma pasta que deve ser criada no container, estamos usando a imagem do debian, se eu colocar um ",ro" antes de indicar a imagem, não poderão mexer nos arquivos lá pois serão de apenas leitura)
  
##### Named
  ```bash
    docker volume ls (lista os volumes)
    docker volume create data-debian (criando volume com nome data-debian, é criado na pasta padrão do docker /var/lib/docker)
  ```
  Você cria um volume nomeado e ele já é salvo em um diretório sob controle do docker (padrão), são referenciados pelo seu nome, o que facilita a organização, agora só referenciar na hora de subir o container:
  ```bash
  docker run -dti --name debian-container --mount type=volume,src=data-debian,dst=/data
  (criamos um mount do tipo volume, no src é só indicar o nome do volume criado, o resto se mantém, dti -> segundo plano, pseudo terminal e interação, dst=/data cria a pasta na raiz do debian)
```
 E para apagar o volume: 
  ```bash
   docker volume rm nomeVolume
   docker volume prune (apaga todos os volumes que não estão em uso, então cuidado)
```

##### Dockerfile volume
  Basicamente usamos a instrução VOLUME no arquivo dockerfile e ele será salvo de maneira similar ao named.

#### PS = Sempre verificar que tipo de mount você está usando com o docker inspect nomeContainer


### Docker image APACHE - Exemplo 
#### Apache Container

Baixando a imagem oficial do apache:

```bash
docker pull httpd
```
Criando o diretório local onde ficará a aplicação:
```bash
mkdir /data/apache-image
nano /data/apache-image/index.html
```
Digite o seguinte conteúdo no index:
```bash
<html>
<head></head>
<body>
<h1>Apache OK</h1>
</body>
</html>
```
Agora vamos subir o container e "linkar" os diretórios local e do container:

```bash
docker run --name apache-image -d -p 80:80 --volume=/data/apache-image:/usr/local/apache2/htdocs httpd
```

Após isso nós pegamos o ip da máquina para testar no navegador.

### Docker image APACHE com PHP - Exemplo 
Vamos baixar a imagem do PHP com APACHE:
```bash
docker pull php:7.4-apache
```
Criaremos um diretório para o arquivo
```bash
   mkdir /data/php-image
   nano /data/php-image/index.php
```
Digite o seguinte no arquivo php:
```bash
<html>
<head></head>
<body>
<h1>Apache OK</h1>

<?php phpinfo(); ?>
</body>
</html>
```
Agora vamos subir o container e "linkar" os diretórios local e do container:

```bash
docker run --name php-image -d -p 8080:80 --volume=/data/php-image:/var/www/html php:7.4-apache
```

Após isso nós pegamos o ip da máquina para testar no navegador.

#### Delimitando recursos para os containers

Visualizando uso de recursos:

```bash
docker stats nomeContainer
```
containers já criados podemos atualizar e impor limites com o update, caso contrário os parâmetros podem ser passados na hora de escrever o comando run:

```bash
docker update nomeContainer -m 128M --cpus 0.2 (-m para limitar uso de memória e --cpus 0.2 para definir o uso de cpu como 20% no máximo)
```
Após definido o limite, experimente usar o stats (comando anterior, novamente)

#### Estressando o container
```bash
docker exec -ti nomeContainer bash (abrir o bash do container)
apt update (atualizar)
apt -y install stress
stress --cpu 1 --vm-bytes 50m --vm 1 --vm-bytes 50m (--vm-bytes define o volume de dados, o numero do lado indica o tamanho em bytes, o --vm 1 indica que um pacote será enviado para estressar a memória, um de também 50 bytes )
```
#### Logs e processos

Visualizando uso de recursos:

```bash
docker info (dá informações sobre o servidor)
docker top nomeContainer (mostra processos)
docker logs nomeContainer (logs do container)
```

#### Isolando conjunto de containers com uma rede

Reunindo informações sobre a rede:
```bash
docker network inspect
```
Aqui nós iremos usar o bash do container ubuntu para instalar o ping e testar a conectividade
```bash
docker exec -ti nomeContainer bash
apt update
apt install -y iputils-ping
```
Hora de criar a rede e colocar os containers nela
```bash
docker network create minha-rede
docker run -dti --name nomeContainer --network minha-rede ubuntu
docker run -dti --name nomeContainer2 --network minha-rede ubuntu
```
Hora de testar a conectividade com o ping instalado no nomeContainer
```bash 
docker exec -ti nomeContainer bash
ping ipContainer2 (aqui vamos pingar os containers vistos no primeiro comando)
docker network inspect minha-rede
``` 


## Dockerfile
#### Introduzindo Dockerfile

```bash
   nano app.py (criaremos uma aplicação python simples que imprime o nome da pessoa)
```
No arquivo digite:
```bash
   nome = input("Qual seu nome?")
   print(nome)
```
##### Criando o Dockerfile
```bash
   nano dockerfile (criaremos o arquivo e editaremos com o nano)
```
No arquivo digite:
```bash
   FROM ubuntu (indica a base do container)
   RUN apt update && apt install -y python3 && apt clean (o & comercial está adicionando os comandos que seriam individuais, por ultimo o clean limpa os arquivos baixados)
   COPY app.py /opt/app.py (copiando para dentro do container)
   CMD python3 /opt/app.py (executando o arquivo utilizando python3)
```
Após salvar só fazer a build
```bash
   docker build . -t ubuntu-python (. indica que o arquivo está no mesmo diretório e o -t permite indicar o nome da imagem)
   docker run -it --name meuapp ubuntu-python (não usamos -dti porque o d indicaria que seria executado em segundo plano e no arquivo python precisamos digitar nosso nome pro script ter utilidade)
```
#### Dockerfile APACHE - movimentando o site
 comando para baixar o arquivo do site usado no exemplo: wget http://site1368633667.hospedagemdesites.ws/site1.zip

```bash
   nano dockerfile (criaremos o arquivo e editaremos com o nano)
```
No arquivo digite:
```bash
   FROM debian (indica a base do container)
   RUN apt-get update && apt-get install -y apache2 && apt-get clean (o & comercial está adicionando os comandos que seriam individuais, por ultimo o clean limpa os arquivos baixados)
   ADD site.tar /var/www/html (O add vai copiar o arquivo para o local indicado e descompactar o TAR)
   LABEL description="Apache Webserver 1.0" (Descrição)
   VOLUME /var/www/html/ (Criamos o volume onde os arquivos devem ser salvos)
   EXPOSE 80 (Porta que queremos expor)

   ENTRYPOINT ["/usr/sbin/apachectl"] (localização do executavel do apache)
   CMD ["-D", "FOREGROUND"] (Indicando que deve rodar em primeiro plano)
```
Após salvar só fazer a build
```bash
   docker image build -t debian-apache:1.0 .
   docker run  -dti -p 80:80 --name meu-apache debian-apache:1.0
```
#### Criando o Dockerfile a partir de uma linguagem de programação
Arquivo python:

nome = input("Qual é o seu nome? ")
	print (nome)


=====================================

Dockerfile:
```bash
FROM python

WORKDIR /usr/src/app

COPY app.py /usr/src/app

CMD [ "python", "./app.py" ]
```
Hora de executar:
```bash
   docker image build -t app-python:1.0 . (o . no final indica que o dockerfile está no diretório atual)
   docker run -ti app-python:1.0 (executando a imagem gerada no comando anterior)
```
#### Criando um servidor de imagens
```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2 (ao dar run no registry sem pull antes, ele identifica que você não tem e já baixa sozinho)
docker logout (saindo do dockerhub por precaução)
docker image tag [id] localhost:5000/meu-apache:1.0 (mudando a tag da image)
curl localhost:5000/v2/_catalog 
docker push  localhost:5000/my-go-app:1.0 (enviando para o servidor)
nano /etc/docker/daemon.json (editando o arquivo com o conteúdo abaixo, para que permita o envio mesmo pelo metodo http, sem exigir o https)

	{ "insecure-registries":["10.0.0.189:5000"] }
	
systemctl restart docker (restart para funcionar)

docker push  localhost:5000/my-go-app:1.0 (enviando novamente)
```
## Docker Compose
O docker-compose é uma forma de através de um arquivo de configuração YAML definir diferentes características para um mesmo sistema através da definição de dois ou mais containers, subindo todas as dependências para seu projeto funcionar, tudo isso com um único comando.

```bash
apt-get install -y docker-compose (instalando)
```
Criando um diretório para usar como mount BIND.
```bash
mkdir /data
mkdir /data/mysql-A
mkdir /compose
mkdir /compose/primeiro (diretório para deixar os arquivos docker-compose)
```
#### Criando o arquivo YAML ou YML
```bash
   nano /compose/primeiro/docker-compose.yml
```
#### Agora vamos ao conteúdo:
```bash
version: '3.7'

services:
  mysqlsv:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: "root"
      MYSQL_DATABASE: "testedb"
    ports:
      - "3306:3306"
    volumes:
      - /data/mysql-C:/var/lib/mysql
    networks:
      - minha-rede

  adminer:
    image: adminer
    ports:
      - 8080:8080
    networks:
      - minha-rede

networks: 
  minha-rede:
    driver: bridge
```

#### Hora de rodar:
```bash
   docker-compose up -d
```

### Links úteis
#### O que são containers? 

https://www.ibm.com/br-pt/cloud/learn/containers

https://www.redhat.com/pt-br/topics/containers

 

#### Diferenças entre containers e máquinas virtuais (VMs)

https://www.redhat.com/pt-br/topics/containers/containers-vs-vms

  

#### O que é Docker? 

https://www.redhat.com/pt-br/topics/containers/what-is-docker

https://docs.microsoft.com/pt-br/dotnet/architecture/microservices/container-docker-introduction/docker-defined

  

#### Comandos essenciais 

https://medium.com/xp-inc/principais-comandos-docker-f9b02e6944cd

https://docs.docker.com/engine/reference/commandline/docker/

https://dev.to/soutoigor/docker-imagens-containers-e-seus-principais-comandos-23p6 
