# Usando DOCKER

## O que precisamos aprender:
    
### como rodar comandos python manage.py ou pip install enquanto docker executa
    
- exemplo 1
```
docker exec -it django_web_app pip install django-extensions pygraphviz

docker exec -it django_web_app pip freeze > requirements.txt
```

- exemplo 2
```
docker exec -it django_web_app python manage.py makemigrations
docker exec -it django_web_app python manage.py migrate
```

### lista de comandos docker-compose
   
#### Gerenciamento do Ciclo de Vida

* docker compose up: Cria e inicia os containers.
* docker compose up -d: Inicia os containers em segundo plano (modo detached).
* docker compose down: Para e remove os containers, redes e volumes criados pelo up.
* docker compose stop: Apenas para os containers, sem removê-los.
* docker compose start: Inicia os containers que já foram criados e estão parados.
* docker compose restart: Reinicia os serviços.

#### Monitoramento e Status

* docker compose ps: Lista os containers do projeto e seus status atuais.
* docker compose logs: Exibe as saídas de texto (logs) de todos os serviços.
* docker compose logs -f [servico]: Acompanha os logs em tempo real (substitua [servico] pelo nome do container se quiser filtrar).
* docker compose top: Exibe os processos em execução dentro dos containers.

#### Modificações e Construção

* docker compose build: Cria ou atualiza as imagens com base no arquivo Dockerfile.
* docker compose up --build: Força a reconstrução das imagens antes de iniciar os containers.
* docker compose pull: Baixa as imagens mais recentes declaradas no arquivo.

#### Execução de Comandos

* docker compose exec [servico] [comando]: Executa um comando dentro de um container ativo (ex: docker compose exec web bash).
* docker compose run [servico] [comando]: Cria um container temporário para rodar um comando isolado.

#### Limpeza e Validação

* docker compose down -v: Remove os containers e deleta os volumes de dados anexados.
* docker compose config: Valida a sintaxe e exibe a configuração final do arquivo docker-compose.yml.

### Como configurar gunicorn e nginx no docker para poder usar depois do servidor 

?????



## Importante


###  O que é Docker?
O Docker é uma tecnologia de virtualização a nível de sistema operacional. Ao contrário de uma Máquina Virtual (VM) tradicional, que precisa de um sistema operacional inteiro para cada instância, o Docker compartilha o núcleo (kernel)
- do sistema operacional hospedeiro. Ele empacota o código do seu aplicativo junto com todas as dependências (bibliotecas, variáveis de ambiente) em um único arquivo chamado Imagem. Quando essa imagem é executada, ela se torna um Contêiner isolado e leve. 

###  Para que usar o Docker?

* Fim do "Na minha máquina funciona": Garante que o sistema rode exatamente igual no Windows de desenvolvimento e no Ubuntu de produção.
* Isolamento total: Permite rodar diferentes versões de Python ou bancos de dados na mesma máquina sem conflitos.
* Portabilidade ultra-rápida: Facilita migrar o aplicativo entre computadores ou servidores em poucos minutos.
* Instalação simplificada: Você não precisa instalar Python, PostgreSQL ou Redis diretamente no seu Windows.


## Imagem (Image)
A imagem é um arquivo estático que contém tudo o que é necessário para executar um aplicativo. 

* É o "projeto", "molde" ou "receita" do  sistema.
* Inclui o código, o sistema operacional básico, as dependências e as configurações.
* Ela é imutável, o que significa que nunca muda após ser criada. 

##  Container
O container é a imagem em execução.

* É a instância viva daquela "receita" (imagem).
* Funciona como uma máquina virtual extremamente leve, isolada do resto do computador.
* Se você deletar o container, tudo o que foi alterado dentro dele enquanto ele rodava é perdido por padrão.

## Volume
O volume é o mecanismo de armazenamento de dados persistentes do Docker.

* Serve para salvar arquivos para que eles não sumam quando o container for destruído.
* Funciona como um "HD externo" virtual que você conecta ao container.
* Permite que bancos de dados salvem suas informações com segurança fora do container.

## Stack
A stack (pilha) é um conjunto de containers interconectados que formam uma aplicação completa. 

* É usada para gerenciar múltiplos serviços que trabalham juntos (ex: um container com o site, outro com o banco de dados e outro com o sistema de login).
* Geralmente é definida através de um arquivo chamado docker-compose.yml.
* Permite subir ou derrubar toda a sua infraestrutura de uma só vez.  

### Para usar docker:
    1) instalar e configurar o sistema docker instalado na máquina (local ou servidor) - windows (docker desktop e o WSL)
    2) escolher o sistema a ser aplicado o docker
    3) configurar três arquivos:
        - dockerfile
        - docker-compose.yml
        - .dockerignore
    4) criar um stack docker
        - container_name: django_mysql_db (db)
        - container_name: django_web_app (web)
    5) configurar .env
        # Dados do Banco de Dados
        DB_NAME=rnmn_db
        DB_USER=rnmn
        DB_PASSWORD=rnmn12345!
        DB_ROOT_PASSWORD=senharoot12345!

        # URL montada dinamicamente
        DATABASE_URL=mysql://${DB_USER}:${DB_PASSWORD}@db:3306/${DB_NAME}
