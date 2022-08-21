
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
   docker start id/name (Aqui os dados não foram perdidos, mas caso você apaga o container, para não correr risco de perder dados por precaução deve mapear um lugar para salvar eles)
```
#### Excluindo o container
```bash
   docker rmi id/name (substitua o id pela numeracao ou apenas use o name, pode ser visto usando o docker ps)
```

### Dockerfile

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
##### Docker image MYSQL
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
   apt -y install mysql-client (com ele não precisamos abrir o bash do container só pra executar o comando do mysql -u root -p --protocol=tcp)

```

