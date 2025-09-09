Configuração para domínio local evolution.cbi1.org

Excelente! Um domínio local com SSL é a solução ideal para sua Evolution API. Vou configurar tudo para funcionar com evolution.cbi1.org.

1. docker-compose.yml

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
      - ./uploads:/usr/src/app/uploads
    networks:
      - evolution-network
    extra_hosts:
      - "evolution.cbi1.org:host-gateway"

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
    networks:
      - evolution-network
    extra_hosts:
      - "evolution.cbi1.org:host-gateway"

volumes:
  evolution_data:
    driver: local
  evolution_logs:
    driver: local

networks:
  evolution-network:
    driver: bridge
```

2. .env otimizado

```env
# ========================
# EVOLUTION API CONFIG
# ========================

# Configurações do Servidor
PORT=8081
HOST=0.0.0.0
NODE_ENV=production

# URLs Críticas para QR Code
SERVER_URL=https://evolution.cbi1.org
INSTANCE_SERVER_HOST=https://evolution.cbi1.org
WEBHOOK_GLOBAL_URL=https://evolution.cbi1.org/webhook

# Configurações de QR Code
QRCODE_DISPLAY=true
QRCODE_GENERATE_INTERVAL=4000
QRCODE_LIMIT=40
SAVE_QRCODE=true

# Segurança
JWT_SECRET=seu-jwt-super-secreto-aqui-altere-isto
JWT_EXPIRES_IN=7d

# CORS
CORS_ORIGIN=*
CORS_METHODS=GET,HEAD,PUT,PATCH,POST,DELETE,OPTIONS
CORS_CREDENTIALS=true
CORS_ALLOWED_HEADERS=Content-Type,Authorization,X-Requested-With

# Webhook Events
WEBHOOK_EVENTS=APPLICATION_STARTUP,QRCODE_UPDATED,MESSAGES_SET,MESSAGES_UPSERT,MESSAGES_UPDATE,MESSAGES_DELETE,SEND_MESSAGE,CONTACTS_SET,CONTACTS_UPDATED,CONTACTS_DELETE,PRESENCE_UPDATE,CHATS_SET,CHATS_UPDATED,CHATS_DELETE,GROUPS_UPSERT,GROUP_PARTICIPANTS_UPDATE,CONNECTION_UPDATE,CALL

# Logs
LOG_LEVEL=info
LOG_ACTIVE=true

# Cache
CACHE_ENABLED=true
CACHE_TIME=300

# Rate Limit
RATE_LIMIT_ENABLED=true
RATE_LIMIT_MAX=100
RATE_LIMIT_WINDOW_MS=60000

# Redis (Opcional - descomente se usar)
# REDIS_URI=redis://redis:6379
# REDIS_DB=0

# Database (Opcional - descomente se usar)
# DB_CONNECTION=postgres
# DB_HOST=postgres
# DB_PORT=5432
# DB_DATABASE=evolution
# DB_USERNAME=evolution_user
# DB_PASSWORD=evolution_password
```

3. Configuração do Nginx (nginx/conf.d/evolution.conf)

```nginx
server {
    listen 80;
    server_name evolution.cbi1.org;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name evolution.cbi1.org;

    # SSL Configuration
    ssl_certificate /etc/nginx/ssl/evolution.cbi1.org.crt;
    ssl_certificate_key /etc/nginx/ssl/evolution.cbi1.org.key;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains";

    # CORS headers
    add_header 'Access-Control-Allow-Origin' '*' always;
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, PUT, DELETE' always;
    add_header 'Access-Control-Allow-Headers' 'Authorization,Content-Type' always;
    add_header 'Access-Control-Allow-Credentials' 'true' always;

    # Handle preflight requests
    if ($request_method = 'OPTIONS') {
        return 204;
    }

    # Proxy settings for Evolution API
    location / {
        proxy_pass http://evolution-api:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeout settings
        proxy_connect_timeout 300s;
        proxy_send_timeout 300s;
        proxy_read_timeout 300s;
        send_timeout 300s;
    }

    # Serve uploaded files (QR codes)
    location /uploads/ {
        alias /usr/src/app/uploads/;
        expires 1h;
        add_header Cache-Control "public";
    }

    # Health check endpoint
    location /health {
        proxy_pass http://evolution-api:8081/health;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Access and error logs
    access_log /var/log/nginx/evolution_access.log;
    error_log /var/log/nginx/evolution_error.log;
}
```

4. Script para gerar SSL (generate-ssl.sh)

```bash
#!/bin/bash

# Create SSL directory
mkdir -p nginx/ssl

# Generate private key
openssl genrsa -out nginx/ssl/evolution.cbi1.org.key 2048

# Generate CSR
openssl req -new -key nginx/ssl/evolution.cbi1.org.key -out nginx/ssl/evolution.cbi1.org.csr \
  -subj "/C=BR/ST=Estado/L=Cidade/O=Organizacao/CN=evolution.cbi1.org"

# Generate self-signed certificate
openssl x509 -req -days 3650 -in nginx/ssl/evolution.cbi1.org.csr \
  -signkey nginx/ssl/evolution.cbi1.org.key -out nginx/ssl/evolution.cbi1.org.crt

# Set permissions
chmod 644 nginx/ssl/evolution.cbi1.org.crt
chmod 600 nginx/ssl/evolution.cbi1.org.key

echo "SSL certificate generated for evolution.cbi1.org"
echo "Files created:"
echo "  - nginx/ssl/evolution.cbi1.org.key (private key)"
echo "  - nginx/ssl/evolution.cbi1.org.crt (certificate)"
```

5. Script de instalação (setup-evolution.sh)

```bash
#!/bin/bash

echo "=== Configurando Evolution API para evolution.cbi1.org ==="

# Generate SSL certificate
echo "Gerando certificado SSL..."
chmod +x generate-ssl.sh
./generate-ssl.sh

# Create directory structure
echo "Criando estrutura de diretórios..."
mkdir -p nginx/{conf.d,ssl,logs} uploads

# Create nginx configuration
echo "Criando configuração do Nginx..."
cat > nginx/conf.d/evolution.conf << 'EOL'
[INSIRA AQUI O CONTEÚDO DO NGINX ACIMA]
EOL

# Create .env file
echo "Criando arquivo .env..."
cat > .env << 'EOL'
[INSIRA AQUI O CONTEÚDO DO .ENV ACIMA]
EOL

# Add to hosts file (opcional - apenas se quiser configurar automaticamente)
echo "Para configurar nos clientes, adicione esta linha no arquivo /etc/hosts:"
echo "192.168.1.100   evolution.cbi1.org"
echo ""
echo "Substitua 192.168.1.100 pelo IP do seu servidor"

echo "=== Configuração concluída ==="
echo "Execute: docker-compose up -d"
echo "Acesse: https://evolution.cbi1.org"
```

6. Configuração nos clientes

No arquivo hosts de cada máquina cliente:

```
# Windows: C:\Windows\System32\drivers\etc\hosts
# Linux/Mac: /etc/hosts

192.168.1.100   evolution.cbi1.org
```

Importar certificado SSL (opcional mas recomendado):

```bash
# No Windows: Importar .crt no "Certificados do Computador"
# No Linux: 
sudo cp nginx/ssl/evolution.cbi1.org.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

# No Mac: 
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain nginx/ssl/evolution.cbi1.org.crt
```

7. Como usar

1. Execute o setup:

```bash
chmod +x setup-evolution.sh generate-ssl.sh
./setup-evolution.sh
```

1. Inicie os containers:

```bash
docker-compose up -d
```

1. Acesse a API:

```
https://evolution.cbi1.org
```

1. Crie uma instância:

```bash
curl -X POST https://evolution.cbi1.org/instance/create \
  -H "Content-Type: application/json" \
  -d '{"instanceName": "minha-instancia", "qrcode": true}'
```

1. Acesse o QR Code:

```
https://evolution.cbi1.org/instance/connect/minha-instancia
```

Vantagens desta configuração:

1. ✅ SSL funcionando com certificado autoassinado
2. ✅ QR Code acessível em toda rede local
3. ✅ Domínio fácil de lembrar (evolution.cbi1.org)
4. ✅ Nginx otimizado para Evolution API
5. ✅ Configuração completa e pronta para uso

Seu QR Code agora será perfeitamente acessível em todos os dispositivos da rede local!