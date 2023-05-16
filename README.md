# docker-sheet-cheat

comando run
exemplo:

docker container run debian bash --version

esse comando tem 4 comandos implicitos nele, que são os seguintes comandos:

fará o download da imagem se a imagem não existir
# docker image pull

fará a criação do container
#docker container create

fará a inicialização do container
#docker container start

fará a execução do container no modo interativo
#docker container exec
é importante destacar que o comando run sempre executará um novo container ao usa-lo

#docker container ps || #docker ps || #docker container ls || #docker container list 
lista quais os containers de status ativo


#docker ps -a
lista todos os containers que foram executados, independente do status


#docker run --rm

exemplo de um container nginx rodando na porta 80 do container

#docker run -p 8080:80 nginx

a porta 8080 seria a porta externa para acessar o nginx e a porta 80 seria a interna do nginx

Exemplo de montagem de volume(diretório), container apontando para um diretório fora dele, no exemplo esta apontando para o diretório corrente $(pwd) no subdiretório /html, após os dois pontos ":" é o diretório padrão do nginx onde ele olha para encontrar as paginas html
  
#docker run -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html nginx

Esse exemplo é interessante, pois quando estamos desenvolvendo numa máquina nosso código, ele será executado no container que tem todas as dependencias necessárias que a aplicação que estamos desenvolvendo precise

#docker run -d

esse comando executa um container no modo daemon(programa que é executado em plano de fundo ou background) 

#docker run -d --name ex-daemon-basic -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html nginx



#docker stop [nome-do-container] ou #docker stop [container-id]
comando para parar a execução de um container, é interessante para parar um container executando no modo daemon (background), lembrando que para vermos o CONTAINER ID basta executar o comando #docker ps

#docker start [nome-do-container] ou #docker start [container-id]
comando para iniciar a execução de um container

#docker restart [nome-do-container] ou #docker restart [container-id]
comando para reiniciar um container

#docker logs [nome-do-container] ou #docker logs [container-id]
mostra logs referente ao container

#docker inspect [nome-do-container] ou #docker inspect [container-id]
mostra em formato json varias informações sobre o container como: que tipo de imagem ele se baseia, informações de rede, volume, onde está o diretório de log, entre todas as outras informações que compõe o container.

IMAGE

#docker image pull [nome-imagem]
Baixa imagem de um regitry(repositório onde contém varias imagens de docker, exemplo: hub.docker.com)

#docker image pull redis:latest  
esse exemplo baixa uma imagem di redis após o dois pontos":" a flag latest indica a ultima versão da imagem, se quisermos indicar outra versão da imagem, basta especificar nessa flag.

#docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
cria uma outra tag para uma imagem, é a mesma imagem só que com tag diferente, se reparamos no hash da imagem é o mesmo no IMAGE ID, abaixo um exemplo

#docker image rm code-regis:latest redis:latest

ao executar um #docker image ls temos o resultado seguinte:
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              bc9a0695f571        6 days ago          133MB
code-regis          latest              74d107221092        13 days ago         104MB
redis               latest              74d107221092        13 days ago         104MB
debian              latest              ef05c61d5112        13 days ago         114MB
hello-world         latest              bf756fb1ae65        11 months ago       13.3kB

onde o hash das imagens são o mesmo e o nome da imagem (tag) é diferente

Criando um descritor para a imagem Dockerfile

1 - criar um arquivo sem extensão com o nome Dockerfile em um diretório desejado.(Atenção nessa hora pois nome do arquivo precisa ser exatamente esse: "Dockerfile")

2 - abrir o aquivo Dockerfile com o editor desejado e inserir as seguintes linhas e salvar:

FROM nginx:latest
RUN echo '<h1>Hello World !</h1>' > /usr/share/nginx/html/index.html



o FROM indica qual a imagem será executada
o RUN executa tudo que esta nessa linha

3 - Acessar o diretório onde esta o arquivo Dockerfile e executar o comando abaixo:

#docker image build -t ex-simple-build . 

docker image build -> "compila"/"constrói" a imagem
-t [ex-simple-build]-> inserir a tag/nome da imagem 
. -> é para executar no diretório corrente     


para verificar se a imagem foi criada basta executar o comando #docker image ls, trazendo o resultado similar ao abaixo:

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ex-simple-build     latest              a91c1c45a069        7 seconds ago       133MB

4 - agora que temos a imagem pronta, podemos criar o container

#docker run -p 80:80 ex-simple-build 

se acessar no navegador o endereço localhost:80 veremos o resultado de uma pagina html com a mensagem Hello World !

NETWORK

#docker network ls 

Lista os tipos de redes do container exceto o Overlay Network que é do Swarm (Ferramenta de Cluster do Docker), exemplo abaixo é da saida do comando #docker network ls:

NETWORK ID          NAME                DRIVER              SCOPE
42a8fb1aa27f        bridge              bridge              local
43f21dc662a7        host                host                local
2637b79624f1        none                null                local

#docker network create --driver bridge rede_nova
Comando cria uma nova rede com draiver bridge e com o nome rede_nova, se listarmos com o comando #docker network ls iremos ver que a rede foi criada, abaixo a rede_nova na lista:

NETWORK ID          NAME                DRIVER              SCOPE
42a8fb1aa27f        bridge              bridge              local
43f21dc662a7        host                host                local
2637b79624f1        none                null                local
39d8c350e101        rede_nova           bridge              local


 

#docker network inspect [driver-da-rede] [nome-da-rede] 
Mostra arquivo com as configurações da rede

#docker run -d --name container3 --net rede_nova alpine sleep 1000
esse exemplo estamos iniciando um container no modo daemon(-d) (background) dando o nome(--name) de container3 e dizendo que ele fara parte da rede(--net) com o nome de rede_nova e sera usada a imagem do alpine

#docker network connect bridge container3
esse comando conecta(connect) na rede de nome bridge o conteiner3, o uso disso é util para que o container consiga se comunicar com os conteiners que estão na rede bridge, pois são redes de faixas diferentes. Basicamente esse comando adiciona uma outra interface de rede no container com a faixa de rede da rede  que adicionamos


#docker network disconnect bridge container3
remove a conexão com a rede de nome bridge

DOCKER COMPOSE

Os comandos e explicações que virão logo abaixo se baseiam na criação de um diretório contendo as partes de uma aplicação web o backend será em js e usará o node.js, servidor http nginx e o Banco de Dados MongoDB

o Diretório Raiz do projeto é o node-mongo-compose, abaixo dele teremos os diretórios backend e frontend e o arquivo docker-compose.yml.

Dentro do diretório frontend tem um index.html com um conteudo simples

Dentro do diretório backend temos o arquivo app.js que é o arquivo com o código do backend que esta fazendo a conexao com o BD e tem a configuração de porta que o BD ficara escutando, também tem o arquivo package.json que são as configurações do node e as dependencias que ele usará.

Acima é descrito superficialmente as partes de uma aplicação web muito simples para que seja configurado o arquivo docker-compose.yml onde está contido a versão do docker compose que será usada, serviços que serão implantados por imagens para funcionar o frontend, backend e o BD. 



curl -L "https://github.com/docker/compose/releases/download/1.27.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose













