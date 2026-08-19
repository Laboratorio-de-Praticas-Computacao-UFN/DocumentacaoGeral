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


nginx.conf
```
server {
    listen 80;
    server_name localhost;

    # Limite máximo de upload (ex: imagens, PDFs)
    client_max_body_size 20M;

    # Roteamento de páginas dinâmicas para o Gunicorn
    location / {
        proxy_pass http://django_web_app:8082;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Arquivos estáticos servidos diretamente pelo Nginx
    location /static/ {
        alias /app/projeto/static/;
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    # Arquivos de mídia enviados por usuários
    location /media/ {
        alias /app/uploads/;
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    location ~ /\.git {
            deny all;
            access_log off;
            log_not_found off;
            return 404;
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
