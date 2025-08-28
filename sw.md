Entendido! Vou criar um sistema completo com Supabase local em containers Docker Swarm. Aqui está a implementação:

Estrutura Atualizada do Projeto

```
budibase-supabase-local/
├── docker-swarm.yml
├── nginx/
│   ├── nginx.conf
│   ├── ssl/
│   │   ├── cobom.app.crt
│   │   └── cobom.app.key
│   └── Dockerfile
├── supabase/
│   ├── config/
│   │   ├── config.toml
│   │   └── kong.yml
│   ├── scripts/
│   │   └── init-supabase.sql
│   └── Dockerfile
├── budibase/
│   └── Dockerfile
├── config/
│   └── budibase.env
└── scripts/
    ├── init-swarm.sh
    ├── deploy.sh
    └── generate-passwords.sh
```

1. Docker Swarm com Supabase Local

docker-swarm.yml:

```yaml
version: '3.8'

services:
  # Supabase Services
  supabase-db:
    image: supabase/postgres:15.1.0.89
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=${SUPABASE_DB_PASSWORD}
    volumes:
      - supabase_data:/var/lib/postgresql/data
      - ./supabase/scripts/init-supabase.sql:/docker-entrypoint-initdb.d/init-supabase.sql
    networks:
      - budibase-network
    deploy:
      replicas: 1
      placement:
        constraints: [node.role == worker]
      restart_policy:
        condition: on-failure

  supabase-studio:
    image: supabase/studio:20231219-5b9c4d3
    environment:
      - POSTGRES_PASSWORD=${SUPABASE_DB_PASSWORD}
      - STUDIO_PORT=3000
    depends_on:
      - supabase-db
    networks:
      - budibase-network
    deploy:
      replicas: 1
      placement:
        constraints: [node.role == worker]

  supabase-auth:
    image: supabase/gotrue:v2.103.1
    environment:
      - GOTRUE_DB_DRIVER=postgres
      - GOTRUE_DB_DATABASE_URL=postgresql://postgres:${SUPABASE_DB_PASSWORD}@supabase-db:5432/postgres
      - GOTRUE_SITE_URL=https://cobom.app
      - GOTRUE_JWT_SECRET=${SUPABASE_JWT_SECRET}
    depends_on:
      - supabase-db
    networks:
      - budibase-network
    deploy:
      replicas: 1
      placement:
        constraints: [node.role == worker]

  # Budibase Services
  budibase:
    image: budibase/budibase:latest
    environment:
      - JWT_SECRET=${JWT_SECRET}
      - MINIO_ACCESS_KEY=${MINIO_ACCESS_KEY}
      - MINIO_SECRET_KEY=${MINIO_SECRET_KEY}
      - REDIS_URL=${REDIS_URL}
      - REDIS_PASSWORD=${REDIS_PASSWORD}
      - COUCH_DB_URL=${COUCH_DB_URL}
      - COUCH_DB_USERNAME=${COUCH_DB_USERNAME}
      - COUCH_DB_PASSWORD=${COUCH_DB_PASSWORD}
      - POSTGRES_URL=postgresql://postgres:${SUPABASE_DB_PASSWORD}@supabase-db:5432/postgres
      - API_ENCRYPTION_PASSWORD=${API_ENCRYPTION_PASSWORD}
      - BB_ADMIN_EMAIL=${BB_ADMIN_EMAIL}
      - BB_ADMIN_PASSWORD=${BB_ADMIN_PASSWORD}
      - BB_URL=https://cobom.app
    depends_on:
      - supabase-db
    networks:
      - budibase-network
    deploy:
      replicas: 2
      placement:
        constraints: [node.role == worker]
      restart_policy:
        condition: on-failure

  # Redis interno para caching
  budibase_redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    environment:
      - REDIS_PASSWORD=${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    networks:
      - budibase-network
    deploy:
      replicas: 1
      placement:
        constraints: [node.role == worker]

  # CouchDB interno para metadados do Budibase
  budibase_couchdb:
    image: couchdb:3.3
    environment:
      - COUCHDB_USER=${COUCH_DB_USERNAME}
      - COUCHDB_PASSWORD=${COUCH_DB_PASSWORD}
    volumes:
      - couchdb_data:/opt/couchdb/data
    networks:
      - budibase-network
    deploy:
      replicas: 1
      placement:
        constraints: [node.role == worker]

  # Nginx com SSL
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
      - "3000:3000"  # Supabase Studio
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - budibase
      - supabase-studio
    networks:
      - budibase-network
    deploy:
      replicas: 1
      placement:
        constraints: [node.role == manager]
      restart_policy:
        condition: on-failure

networks:
  budibase-network:
    driver: overlay
    attachable: true

volumes:
  supabase_data:
    driver: local
  budibase_data:
    driver: local
  redis_data:
    driver: local
  couchdb_data:
    driver: local
```

2. Configuração do Nginx para Múltiplos Serviços

nginx/nginx.conf:

```nginx
upstream budibase_backend {
    server budibase:10000;
    keepalive 32;
}

upstream supabase_studio {
    server supabase-studio:3000;
    keepalive 32;
}

server {
    listen 80;
    server_name cobom.app studio.cobom.app;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name cobom.app;

    ssl_certificate /etc/nginx/ssl/cobom.app.crt;
    ssl_certificate_key /etc/nginx/ssl/cobom.app.key;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # Proxy para Budibase
    location / {
        proxy_pass http://budibase_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }

    location /health {
        proxy_pass http://budibase_backend/api/global/health;
        access_log off;
        add_header Content-Type application/json;
    }
}

server {
    listen 443 ssl http2;
    server_name studio.cobom.app;

    ssl_certificate /etc/nginx/ssl/cobom.app.crt;
    ssl_certificate_key /etc/nginx/ssl/cobom.app.key;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # Proxy para Supabase Studio
    location / {
        proxy_pass http://supabase_studio;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }
}
```

3. Script de Inicialização do Supabase

supabase/scripts/init-supabase.sql:

```sql
-- Criar database específico para Budibase
CREATE DATABASE budibase;

-- Criar usuário específico para Budibase (opcional)
CREATE USER budibase_user WITH PASSWORD 'budibase_password';
GRANT ALL PRIVILEGES ON DATABASE budibase TO budibase_user;

-- Extensões necessárias
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- Configurações adicionais
ALTER DATABASE postgres SET timezone TO 'UTC';
ALTER DATABASE budibase SET timezone TO 'UTC';
```

4. Variáveis de Ambiente Atualizadas

config/budibase.env:

```env
# Supabase Database
SUPABASE_DB_PASSWORD=supabase_password_forte_32_caracteres
SUPABASE_JWT_SECRET=supabase_jwt_secret_32_caracteres

# Budibase Security
JWT_SECRET=budibase_jwt_secret_32_caracteres
API_ENCRYPTION_PASSWORD=budibase_encryption_32_caracteres

# MinIO (armazenamento)
MINIO_ACCESS_KEY=minio_access_key_aleatorio
MINIO_SECRET_KEY=minio_secret_key_aleatorio_32_caracteres

# Redis (interno)
REDIS_URL=redis://budibase_redis:6379
REDIS_PASSWORD=redis_password_forte_32_caracteres

# CouchDB (para metadados do Budibase - interno)
COUCH_DB_URL=http://admin:senha_couchdb@budibase_couchdb:5984
COUCH_DB_USERNAME=admin
COUCH_DB_PASSWORD=couchdb_password_forte_32_caracteres

# Configurações adicionais do Budibase
BB_ADMIN_EMAIL=admin@cobom.app
BB_ADMIN_PASSWORD=admin_password_forte
BB_URL=https://cobom.app
```

5. Script para Gerar Senhas Seguras

scripts/generate-passwords.sh:

```bash
#!/bin/bash

echo "Gerando senhas seguras para o sistema..."

# Gerar todas as senhas necessárias
generate_password() {
    openssl rand -base64 32 | tr -d '/+=' | head -c 32
}

echo "SUPABASE_DB_PASSWORD=$(generate_password)" >> ../config/budibase.env
echo "SUPABASE_JWT_SECRET=$(generate_password)" >> ../config/budibase.env
echo "JWT_SECRET=$(generate_password)" >> ../config/budibase.env
echo "API_ENCRYPTION_PASSWORD=$(generate_password)" >> ../config/budibase.env
echo "REDIS_PASSWORD=$(generate_password)" >> ../config/budibase.env
echo "COUCH_DB_PASSWORD=$(generate_password)" >> ../config/budibase.env
echo "MINIO_ACCESS_KEY=$(openssl rand -hex 16)" >> ../config/budibase.env
echo "MINIO_SECRET_KEY=$(generate_password)" >> ../config/budibase.env

echo "Senhas geradas e salvas em config/budibase.env"
```

6. Script de Deploy Atualizado

scripts/deploy.sh:

```bash
#!/bin/bash

# Verificar se as senhas foram geradas
if ! grep -q "SUPABASE_DB_PASSWORD" ../config/budibase.env; then
    echo "Gerando senhas automaticamente..."
    ./generate-passwords.sh
fi

# Carregar variáveis de ambiente
set -a
source ../config/budibase.env
set +a

# Implantar stack no Docker Swarm
echo "Implantando stack Budibase + Supabase..."
docker stack deploy -c ../docker-swarm.yml budibase-supabase

# Verificar status
echo "Verificando status dos serviços..."
docker service ls

echo "Implantação concluída!"
echo "Acesse:"
echo " - Budibase: https://cobom.app"
echo " - Supabase Studio: https://studio.cobom.app"
echo " - Banco de dados: supabase-db:5432"
```

7. Configuração do Hosts

Adicione estas linhas ao arquivo /etc/hosts em ambas as máquinas:

```
127.0.0.1   cobom.app
127.0.0.1   studio.cobom.app
# Ou os IPs dos seus nodes se for acessar de outra máquina
```

Como Executar

1. Preparar as máquinas:
   ```bash
   # Instalar Docker em ambas as máquinas
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   ```
2. Inicializar o Swarm:
   ```bash
   chmod +x scripts/init-swarm.sh
   ./scripts/init-swarm.sh
   ```
3. Gerar certificados SSL:
   ```bash
   mkdir -p nginx/ssl
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
     -keyout nginx/ssl/cobom.app.key \
     -out nginx/ssl/cobom.app.crt \
     -subj "/CN=cobom.app"
   ```
4. Implantar o sistema:
   ```bash
   chmod +x scripts/deploy.sh
   ./scripts/deploy.sh
   ```
5. Verificar o status:
   ```bash
   docker service ls
   docker service logs budibase-supabase_budibase
   docker service logs budibase-supabase_supabase-db
   ```

Agora você terá um sistema completo com:

· ✅ Supabase PostgreSQL local
· ✅ Supabase Studio para gerenciamento
· ✅ Supabase Auth para autenticação
· ✅ Budibase conectado ao Supabase
· ✅ Nginx com SSL e múltiplos domínios
· ✅ Docker Swarm distribuído em duas máquinas