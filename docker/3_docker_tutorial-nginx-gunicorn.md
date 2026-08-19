# Configuração de Nginx e Gunicorn com Docker

## CONCEITOS
- **NGINX** = "Porteiro da aplicação". Gerencia conexões, serve arquivos estáticos, lida com criptografia SSL/HTTPS e protege a aplicação contra ataques de sobrecarga.
- **GUNICORN** = "Tradutor" e "executor" python. Traduz requisições web do Nginx em chamadas python e devolve respostas processadas.

```
Usuário/Navegador -> Nginx -> Gunicorn -> Django -> Banco de Dados
```

## GUNICORN
Configurado no Dockerfile (ou docker-compose, porém, não será nosso caso)

1. Atualizar linha CMD no DockerFile
```
# Comando padrão para iniciar o servidor de desenvolvimento
#CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

# Comando padrão para iniciar o Gunicorn
CMD ["gunicorn", "projeto.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "3"]
```
2. Adicionar gunicorn nos requirements
```
...
grpcio-status==1.80.0
gunicorn
h11==0.16.0
...
```
3. **REMOVER** linha command no docker-compose
```
web:
    command: >
      sh -c "python manage.py migrate && python manage.py runserver 0.0.0.0:8000"
```

## NGINX
Criar arquivo de configuração nginx.conf e colocar como serviço no docker-compose

1. Criar arquivo ``nginx.conf``
    (disponível ao final desse arquivo)
3. Adicionar bloco Nginx no docker-compose
```
nginx:
    image: nginx:alpine
    container_name: django_nginx
    restart: always
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - static_volume:/app/staticfiles
      - media_volume:/app/media
    depends_on:
      - web
    networks:
      - django_network
```

> OBS: Nesse ponto o site funciona no navegador sem nenhum CSS ou arquivo de mídia, apenas HTML puro, por isso são necessários os passos 3 e 4

3. Rodar no terminal
```
docker exec -it django_web_app python manage.py collectstatic --no-input 
```
4. Adicionar os volumes ``static_volume`` e ``media_volume`` no container web
```
volumes:
      - .:/app
      - static_volume:/app/projeto/static
      - media_volume:/app/uploads
```

## ARQUIVOS FINAIS

Dockerfile
```
# Usa uma imagem oficial do Python adaptada para Linux leve

FROM python:3.12-slim  

# Define o diretório de trabalho dentro do contêiner

WORKDIR /app

# Evita que o Python grave os arquivos .pyc no disco  

ENV PYTHONDONTWRITEBYTECODE 1

# Evita que o Python faça buffer das saídas stdout e stderr

ENV PYTHONUNBUFFERED 1

# Instala as dependências do sistema

# RUN apt-get update && apt-get install -y --no-install-recommends gcc libpq-dev && apt-get clean

# Instala as ferramentas que o mysqlclient exige para compilar

RUN apt-get update && apt-get install -y \
    gcc \
    default-libmysqlclient-dev \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Copia o arquivo de requisitos e instala as dependências do Python

COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt

# Copia todo o restante do código do computador para o contêiner

COPY . /app/

# Expõe a porta padrão que o Django/Gunicorn vai rodar  

EXPOSE 8000

# Comando padrão para iniciar o servidor de desenvolvimento
#CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

# Comando padrão para iniciar o Gunicorn
CMD ["gunicorn", "projeto.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "3"]
```

docker-compose.yml
```
services:
  db:
    image: mysql:8.0
    container_name: django_mysql_db
    restart: always
    environment:
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - django_network
        
  web:
    build: .
    container_name: django_web_app
    restart: always
    volumes:
      - .:/app
      - static_volume:/app/projeto/static
      - media_volume:/app/uploads
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      - db
    networks:
      - django_network

  nginx:
    image: nginx:alpine
    container_name: django_nginx
    restart: always
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - static_volume:/app/staticfiles
      - media_volume:/app/media
    depends_on:
      - web
    networks:
      - django_network

volumes:
  mysql_data:
  static_volume:
  media_volume:

networks:
  django_network:
    driver: bridge
```

nginx.conf
```
server {
    listen 80;
    server_name localhost;

    # Limite máximo de upload (ex: imagens, PDFs)
    client_max_body_size 20M;

    # Roteamento de páginas dinâmicas para o Gunicorn
    location / {
        proxy_pass http://django_web_app:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Arquivos estáticos servidos diretamente pelo Nginx
    location /static/ {
        alias /app/staticfiles/;
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    # Arquivos de mídia enviados por usuários
    location /media/ {
        alias /app/media/;
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }
}
```

> OBS: A IA deu duas opções de nginx.conf, uma mais completa e outra mais simples com apenas o necessário, utilizei a mais completa mas vou deixar a segunda versão aqui também

nginx.conf (simples)
```
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://django_web_app:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /static/ {
        alias /app/staticfiles/;
    }

    location /media/ {
        alias /app/media/;
    }
}
```
