### 1) criar o Stack
- a) container com a aplicação django_web_app
- b) container com o mysql django_mysql_db
- c) configurações dos arquivos
	- dockerfile: 
	- docker-compose.yml
	- .env
	- settings.py
	- configuração do gunicorn
	- configuração do nginx

### 2) levar para o portainer
- a) fazer backup do stack antigo (wordpress e mysql)
- b) colocar em funcionamento no servidor ufn



# arquivo dockerfile
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
	
	# CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
	CMD ["gunicorn", "projeto.wsgi:application", "--bind", "0.0.0.0:8082", "--workers", "3"]
```

# arquivo docker-compose.yml
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
	    # command: >
	    #   sh -c "python manage.py migrate && python manage.py runserver 0.0.0.0:8000"
	    
	    ports:
	      - "8082:8082"
	    env_file:
	      - .env
	    depends_on:
	      - db
	    networks:
	      - django_network
	    volumes:
	      - .:/app
	      - static_volume:/app/projeto/static
	      - ./uploads:/app/uploads 
	  nginx:
	    image: nginx:alpine
	    container_name: django_nginx
	    restart: always
	    ports:
	      - "80:80"
	    volumes:
	      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
	      - static_volume:/app/projeto/static:ro
	      - ./uploads:/app/uploads:ro 
	    depends_on:
	      - web
	    networks:
	      - django_network
	
	volumes:
	  mysql_data:
	  static_volume:  # Lembre-se de declarar estes para evitar erros!
	  media_volume:
	
	networks:
	  django_network:
	    driver: bridge
```
# arquivo .env
```
	SECRET_KEY='69bi!vl=ayz6f(m7)gmx=)vt5b2x&z-ff87ib3dh!y(%(g5_f8'
	DEBUG=True
	STATIC_URL=/static/
	
	# Dados do Banco de Dados
	DB_NAME=rnmn_db
	DB_USER=rnmn
	DB_PASSWORD=@@@@@@
	DB_ROOT_PASSWORD=@@@@@@@
	
	# URL montada dinamicamente
	DATABASE_URL=mysql://${DB_USER}:${DB_PASSWORD}@db:3306/${DB_NAME}
	
	DOMINIO_URL='http://localhost:8082'
	
	EMAIL_ADMINISTRACAO=''
	EMAIL_BACKEND=django.core.mail.backends.smtp.EmailBackend
	EMAIL_HOST = 'smtp.gmail.com'
	EMAIL_HOST_USER = ''
	EMAIL_HOST_PASSWORD = ''
	
	EMAIL_PORT = 587
	DEFAULT_FROM_EMAIL="sender@ersistemas.info"
	
	EMAIL_USE_TLS = True
	EMAIL_USE_SSL = False
	EMAIL_USE_STARTTLS = False
```


=============================================
arquivo settings.py

DATABASES = {
        'default': config(
        'DATABASE_URL',
        default='sqlite:///' + os.path.join(BASE_DIR, 'db.sqlite3'),
        cast=db_url
    )
}
