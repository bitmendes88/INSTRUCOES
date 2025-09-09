Adaptações para exibição do QR Code na Evolution API

Sim, existem várias formas de adaptar quando o QR Code não é exibido. O problema geralmente está relacionado à configuração de URLs e redirecionamentos. Vou fornecer uma solução completa:

1. docker-compose.yml atualizado

```yaml
version: '3.8'

services:
  # Evolution API
  evolution-api:
    image: atendais/whatsapp-api:latest
    container_name: evolution-api
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "8081:8081"
    volumes:
      - evolution_data:/usr/src/app/instances
      - evolution_logs:/usr/src/app/logs
      - ./uploads:/usr/src/app/uploads  # Para acesso aos QR Codes
    networks:
      - evolution-network
    depends_on:
      - nginx

  # Nginx com SSL
  nginx:
    image: nginx:alpine
    container_name: evolution-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/ssl:/etc/nginx/ssl
      - ./nginx/logs:/var/log/nginx
      - ./uploads:/var/www/uploads  # Disponibilizar uploads via web
    networks:
      - evolution-network
    depends_on:
      - evolution-api

volumes:
  evolution_data:
    driver: local
  evolution_logs:
    driver: local

networks:
  evolution-network:
    driver: bridge
```

2. Arquivo .env atualizado

```env
# Configurações da Evolution API
PORT=8081
HOST=0.0.0.0

# Configurações do Redis (se necessário)
# REDIS_URI=redis://redis:6379
# REDIS_DB=0

# Configurações do Banco de Dados (se necessário)
# DB_CONNECTION=postgres
# DB_HOST=postgres
# DB_PORT=5432
# DB_DATABASE=evolution
# DB_USERNAME=evolution_user
# DB_PASSWORD=evolution_password

# Configurações de Segurança
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production
JWT_EXPIRES_IN=7d

# Configurações do WhatsApp - IMPORTANTE PARA QR CODE
WEBHOOK_GLOBAL_URL=https://seu-dominio-local.com/webhook
WEBHOOK_EVENTS=APPLICATION_STARTUP,QRCODE_UPDATED,MESSAGES_SET,MESSAGES_UPSERT,MESSAGES_UPDATE,MESSAGES_DELETE,SEND_MESSAGE,CONTACTS_SET,CONTACTS_UPDATED,CONTACTS_DELETE,PRESENCE_UPDATE,CHATS_SET,CHATS_UPDATED,CHATS_DELETE,GROUPS_UPSERT,GROUP_PARTICIPANTS_UPDATE,CONNECTION_UPDATE,CALL

# Configuração crítica para o QR Code
SERVER_URL=https://seu-dominio-local.com
INSTANCE_SERVER_HOST=https://seu-dominio-local.com
QRCODE_GENERATE_INTERVAL=5000
QRCODE_DISPLAY=true
QRCODE_LIMIT=30

# Configurações de Log
LOG_LEVEL=debug
LOG_ACTIVE=true

# Configurações de CORS
CORS_ORIGIN=*
CORS_METHODS=GET,HEAD,PUT,PATCH,POST,DELETE
CORS_CREDENTIALS=true

# Configurações de Cache
CACHE_ENABLED=true
CACHE_TIME=300

# Configurações de Rate Limit
RATE_LIMIT_ENABLED=true
RATE_LIMIT_MAX=100
RATE_LIMIT_WINDOW_MS=60000

# Configuração para salvar QR Codes como arquivo
SAVE_QRCODE=true
```

3. Configuração do Nginx atualizada (nginx/conf.d/default.conf)

```nginx
server {
    listen 80;
    server_name seu-dominio-local.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name seu-dominio-local.com;

    # SSL autoassinado
    ssl_certificate /etc/nginx/ssl/localhost.crt;
    ssl_certificate_key /etc/nginx/ssl/localhost.key;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
    ssl_prefer_server_ciphers off;

    # Servir arquivos estáticos (QR Codes)
    location /uploads/ {
        alias /var/www/uploads/;
        expires 1h;
        add_header Cache-Control "public";
    }

    # Configurações de proxy para a Evolution API
    location / {
        proxy_pass http://evolution-api:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts aumentados para QR Code
        proxy_connect_timeout 300s;
        proxy_send_timeout 300s;
        proxy_read_timeout 300s;
    }

    # Health check
    location /health {
        proxy_pass http://evolution-api:8081/health;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Logs
    access_log /var/log/nginx/evolution_access.log;
    error_log /var/log/nginx/evolution_error.log;
}
```

4. Script para verificar e solucionar problemas do QR Code

Crie um arquivo check-qrcode.sh:

```bash
#!/bin/bash

# Script para verificar e solucionar problemas com QR Code
echo "Verificando configuração do QR Code..."

# Verificar se as configurações estão corretas
echo "1. Verificando variáveis de ambiente..."
grep -E "SERVER_URL|INSTANCE_SERVER_HOST|QRCODE" .env

echo ""
echo "2. Verificando se o container está respondendo..."
docker-compose exec evolution-api curl -s http://localhost:8081/health | jq .

echo ""
echo "3. Verificando logs da API..."
docker-compose logs evolution-api --tail=50 | grep -i qrcode

echo ""
echo "4. Verificando se a pasta de uploads está acessível..."
docker-compose exec evolution-api ls -la /usr/src/app/uploads/

echo ""
echo "5. Verificando configuração do Nginx..."
docker-compose exec nginx nginx -t

echo ""
echo "Para acessar o QR Code diretamente, tente:"
echo "https://seu-dominio-local.com/instance/connect/:instanceName"
echo ""
echo "Ou verifique os arquivos em: ./uploads/"
```

5. Arquivo de configuração alternativa para desenvolvimento

Crie um arquivo docker-compose.override.yml para desenvolvimento:

```yaml
version: '3.8'

services:
  evolution-api:
    ports:
      - "8081:8081"  # Expor porta diretamente para testes
    environment:
      - NODE_ENV=development
      - DEBUG=evolution-api*
  
  # Servidor alternativo para servir QR Codes
  qrcode-server:
    image: nginx:alpine
    container_name: qrcode-server
    ports:
      - "8080:80"
    volumes:
      - ./uploads:/usr/share/nginx/html
    networks:
      - evolution-network
```

6. Como usar as adaptações

1. Execute o script de verificação:

```bash
chmod +x check-qrcode.sh
./check-qrcode.sh
```

1. Se o QR Code não aparecer na API, tente acessar diretamente:

```bash
# Acesse a API diretamente pela porta 8081
http://localhost:8081/instance/connect/:instanceName

# Ou verifique a pasta de uploads
ls -la ./uploads/
```

1. Para desenvolvimento, use a override:

```bash
docker-compose -f docker-compose.yml -f docker-compose.override.yml up -d
```

1. Acesse o servidor alternativo de QR Codes:

```
http://localhost:8080/  # Listará todos os QR Codes salvos
```

7. Soluções comuns para problemas de QR Code

1. Problema de CORS: Certifique-se de que CORS_ORIGIN=* está definido
2. URL incorreta: Verifique se SERVER_URL e INSTANCE_SERVER_HOST estão corretos
3. Problema de rede: Certifique-se de que os containers estão na mesma rede
4. Timeout: Aumente os timeouts no Nginx conforme configurado
5. Permissões: Verifique se a pasta uploads tem permissões de escrita

Essas adaptações devem resolver a maioria dos problemas relacionados à exibição do QR Code na Evolution API.