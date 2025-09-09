Aqui está um docker-compose completo com Redis, PostgreSQL, Chatwoot e Nginx:

🐳 docker-compose.yml

```yaml
version: '3.8'

services:
  # Evolution API
  evolution-api:
    image: evolutionapi/evolution-api:latest
    container_name: evolution-api
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      - PORT=8080
      - LOG_LEVEL=info
      - ENABLE_INSECURE_AUTH=true
      - API_KEY=${EVOLUTION_API_KEY}
      - INSTANCE_KEYS=${INSTANCE_KEYS}
      - REDIS_ENABLED=true
      - REDIS_URI=redis://redis:6379
      - DATABASE_ENABLED=true
      - DATABASE_CONNECTION_URI=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}
      - CORS_ENABLED=true
      - CORS_ORIGIN=*
      - STORE_MESSAGE=true
      - STORE_MESSAGE_MEDIA=true
      - STORE_CONTACTS=true
      - STORE_CHATS=true
    volumes:
      - evolution-instances:/evolution/instances
      - evolution-logs:/evolution/logs
    depends_on:
      - redis
      - postgres
    networks:
      - app-network

  # Redis
  redis:
    image: redis:alpine
    container_name: redis
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - app-network

  # PostgreSQL
  postgres:
    image: postgres:15-alpine
    container_name: postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_HOST_AUTH_METHOD=trust
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    networks:
      - app-network

  # Chatwoot
  chatwoot:
    image: chatwoot/chatwoot:latest
    container_name: chatwoot
    restart: unless-stopped
    environment:
      - RAILS_ENV=production
      - NODE_ENV=production
      - FRONTEND_URL=${CHATWOOT_URL}
      - POSTGRES_HOST=postgres
      - POSTGRES_USERNAME=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DATABASE=${POSTGRES_DB}
      - REDIS_URL=redis://redis:6379
      - SECRET_KEY_BASE=${CHATWOOT_SECRET_KEY}
      - RAILS_MAX_THREADS=5
      - ENABLE_ACCOUNT_SIGNUP=true
      - FORCE_SSL=false
    volumes:
      - chatwoot-data:/app/storage
    depends_on:
      - redis
      - postgres
    ports:
      - "3000:3000"
    networks:
      - app-network

  # Nginx
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
      - ./nginx/logs:/var/log/nginx
    depends_on:
      - evolution-api
      - chatwoot
    networks:
      - app-network

volumes:
  redis-data:
  postgres-data:
  evolution-instances:
  evolution-logs:
  chatwoot-data:

networks:
  app-network:
    driver: bridge
```

📝 Arquivo .env

```env
# ========================
# EVOLUTION API
# ========================
EVOLUTION_API_KEY=seu-jwt-token-super-secreto-evolution-123456
INSTANCE_KEYS=whatsapp1:key-instance-123,whatsapp2:key-instance-456

# ========================
# POSTGRESQL
# ========================
POSTGRES_DB=evolution_chatwoot
POSTGRES_USER=admin
POSTGRES_PASSWORD=senha-super-segura-postgres-789

# ========================
# CHATWOOT
# ========================
CHATWOOT_URL=https://seu-dominio.com
CHATWOOT_SECRET_KEY=chatwoot-secret-key-base-muito-longa-aqui-1234567890

# ========================
# DOMÍNIOS E URLS
# ========================
DOMAIN=seu-dominio.com
EVOLUTION_SUBDOMAIN=api
CHATWOOT_SUBDOMAIN=chat
```

🛠️ Nginx Configuration

Crie o arquivo nginx/nginx.conf:

```nginx
events {
    worker_connections 1024;
}

http {
    upstream evolution-api {
        server evolution-api:8080;
    }

    upstream chatwoot {
        server chatwoot:3000;
    }

    server {
        listen 80;
        server_name api.seu-dominio.com chat.seu-dominio.com;

        # Redirect HTTP to HTTPS
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name api.seu-dominio.com;

        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;

        location / {
            proxy_pass http://evolution-api;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # WebSocket support
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
    }

    server {
        listen 443 ssl http2;
        server_name chat.seu-dominio.com;

        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;

        location / {
            proxy_pass http://chatwoot;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # Chatwoot specific headers
            proxy_set_header X-Forwarded-Host $host;
        }

        # WebSocket for Action Cable
        location /cable {
            proxy_pass http://chatwoot;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
            proxy_set_header Host $host;
        }
    }
}
```

📋 Arquivo de Inicialização do PostgreSQL

Crie postgres/init.sql:

```sql
-- Criar database para Evolution API
CREATE DATABASE evolution_db;

-- Criar database para Chatwoot (já é criado automaticamente, mas podemos configurar)
CREATE DATABASE chatwoot_production;

-- Criar usuário específico se necessário
CREATE USER evolution_user WITH PASSWORD 'evolution_password';
GRANT ALL PRIVILEGES ON DATABASE evolution_db TO evolution_user;

CREATE USER chatwoot_user WITH PASSWORD 'chatwoot_password';
GRANT ALL PRIVILEGES ON DATABASE chatwoot_production TO chatwoot_user;
```

🚀 Script de Deploy

Crie deploy.sh:

```bash
#!/bin/bash

# Criar diretórios necessários
mkdir -p nginx/ssl nginx/logs postgres

# Gerar SSL self-signed (para desenvolvimento)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/key.pem \
  -out nginx/ssl/cert.pem \
  -subj "/CN=localhost"

# Substituir domínios no nginx.conf
sed -i "s/seu-dominio.com/${DOMAIN:-localhost}/g" nginx/nginx.conf
sed -i "s/api.seu-dominio.com/${EVOLUTION_SUBDOMAIN:-api}.${DOMAIN:-localhost}/g" nginx/nginx.conf
sed -i "s/chat.seu-dominio.com/${CHATWOOT_SUBDOMAIN:-chat}.${DOMAIN:-localhost}/g" nginx/nginx.conf

# Iniciar os containers
docker-compose up -d

echo "✅ Deploy realizado com sucesso!"
echo "🌐 Evolution API: https://${EVOLUTION_SUBDOMAIN:-api}.${DOMAIN:-localhost}"
echo "💬 Chatwoot: https://${CHATWOOT_SUBDOMAIN:-chat}.${DOMAIN:-localhost}"
echo "📊 PostgreSQL: localhost:5432"
```

🔧 Comandos Úteis

```bash
# Dar permissão ao script
chmod +x deploy.sh

# Executar deploy
./deploy.sh

# Ver logs
docker-compose logs -f

# Parar tudo
docker-compose down

# Backup dos dados
docker-compose exec postgres pg_dump -U admin evolution_chatwoot > backup.sql
```

⚙️ Configuração Pós-Deploy

1. Configurar Chatwoot

Acesse https://chat.seu-dominio.com e complete o setup inicial.

2. Integrar Evolution API com Chatwoot

Use a API do Chatwoot para criar canais do WhatsApp.

3. Configurar SSL Real (Produção)

Substitua os certificados self-signed por certificados Let's Encrypt ou de sua CA.

Este setup fornece uma stack completa para WhatsApp Business com interface de atendimento via Chatwoot e proxy reverso com Nginx!