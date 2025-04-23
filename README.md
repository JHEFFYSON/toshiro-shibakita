Implementação de Microsserviços com Docker Swarm, MySQL e Load Balancer com proxy Nginx
Criação de uma infraestrutura de microsserviços utilizando Docker Swarm, MySQL e um balanceador de carga com proxy Nginx.
Aqui estão os principais passos:

-Provisionamento de VMs
-Instalação do Docker
-Configuração do MySQL: Volumes são criados e um container MySQL é executado, configurado com um banco de dados específico.
-Conexão ao Banco de Dados: Ferramentas como MySQL Workbench são usadas para conectar e gerenciar o banco de dados.
-Criação de Tabelas: Uma tabela chamada dados é criada no banco de dados.
-Instalação do Webserver: Um container Apache com PHP está configurado para servir a aplicação web.
-Teste da Aplicação: A aplicação é acessada via navegador para verificar seu funcionamento.
-Estresse do Container: Ferramentas como Loader.io são usadas para testar a capacidade de resposta do container.
-Criação do Docker Swarm: Um cluster Docker Swarm é inicializado, com um nó mestre e nós trabalhadores.
-Replicação do Webserver: Réplicas do webserver são criadas para garantir alta disponibilidade.
-Montagem de Volumes: Volumes são montados no cluster para compartilhamento de dados entre containers.
-Estresse do Container: Novamente, Loader.io é utilizado para testar a escalabilidade da aplicação.
-A seguir o passo a passo detalhado.

01- PROVISIONAR 3 VMs AWS EC2 free tier (t2.micro/ubuntu)
Três instâncias EC2 na AWS são configuradas com Ubuntu, liberando portas essenciais no Security Group.


Para que todo o processo funcione é preciso liberar as seguintes portas no Security Group por meio de suas regras de entrada:

3306
80
22
2377
4500
Todas as portas para o Grupo de Segurança



02- INSTALAR DOCKER NAS 3 VMs EC2
Docker é instalado em três VMs para permitir a execução de contêineres.

curl -fsSL https://get.docker.com -o get-docker.sh  
sudo sh ./get-docker.sh  
chmod 666 /var/run/docker.sock  


03- CRIE VOLUMES APP e DADOS, E EXECUTAR CONTAINER MYSQL
Os volumes são criados e um contêiner MySQL é executado, configurado com um banco de dados específico.

docker volume create app  
volume create data  
docker run -e MYSQL_ROOT_PASSWORD=Senha123 -e MYSQL_DATABASE=meubanco --name mysql-A -d -p 3306:3306 --mount type=volume,src=data,dst=/var/lib/mysql/ mysql:5.7  



04- CONECTAR AO BD meubanco
Ferramentas como MySQL Workbench são usadas para conectar e gerenciar banco de dados.




05- CRIAR TABELA dados
Uma tabela chamada dados é criada no banco de dados.

CREATE TABLE dados (  
    AlunoID int,  
    Nome varchar(50),  
    Sobrenome varchar(50),  
    Endereco varchar(150),  
    Cidade varchar(50),  
    Host varchar(50)  
);  


06- INSTALAR CONTAINER WEBSERVER APACHE COM PHP
Um contêiner Apache com PHP está configurado para servir a aplicação web.

docker run --name web-server -dt -p 80:80 --mount type=volume,src=app,dst=/app/ webdevops/php-apache:alpine-php7



07- TESTAR A APLICAÇÃO
A aplicação é acessada via navegador para verificar seu funcionamento.
Acesse o navegador com o endereço:http://<PUBLIC IP>


08- EXTRESSAR O CONTAINER
Ferramentas como loader.io são usadas para testar a capacidade de resposta do container.
8.1- Acesse https://loader.io
8.2- Defina o host do Target
8.3- Faça a verificação do Target
8.4- Copie o token de verificação e use-o para criar o arquivo no diretório app


09- ENXAME CRIAR DOCKER
Um cluster Docker Swarm é inicializado, com um nó mestre e nós trabalhadores.
9.1- SEM MESTRE:docker swarm init

9.2- NOS TRABALHADORES: comando docker joinfornecido pelodocker swarm init


10- CRIE RÉPLICAS DO WEBSERVER APACHE COM PHP
Réplicas do servidor web são criadas para garantir alta disponibilidade.
NÃO MESTRE:

docker service create --name web-server --replicas 10 -dt -p 80:80 --mount type=volume,src=app,dst=/app/ webdevops/php-apache:alpine-php7



11- MONTAR O VOLUME NO CLUSTER
Os volumes são montados no cluster para compartilhamento de dados entre contêineres.
11.1- SEM MESTRE: apt-get install nfs-server

11.2- TRABALHADORES DA NOS:apt-get install nfs-common

11.3- SEM MESTRE: nano /etc/exports /var/lib/docker/volumes/app/ *(rw,sync,subtree_check)

11.4- SEM MESTRE:

exportfs -ar
showmount -e
11.5- TRABALHADORES NOS:mount -o v3 <IP LEADER>:/var/lib/docker/volumes/app/_data /var/lib/docker/volumes/app/_data


12- EXTRESSAR O CONTAINER
Novamente, Loader.io é utilizado para testar a escalabilidade da aplicação.

12.1- Acesse https://loader.io 12.2- Defina o host do Target 12.3- Faça a verificação do Target imagem 12.4- Copie o token de verificação e use-o para criar o arquivo no diretório app imagem 12.5- Defina o teste
