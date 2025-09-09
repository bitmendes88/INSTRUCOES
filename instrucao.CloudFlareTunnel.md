# Configuração Cloudflare Tunnel + Nginx + Aplicações em Containers

## Arquitetura Proposta
```
Internet → Cloudflare (SSL) → Cloudflare Tunnel → Nginx (HTTP 80) → Aplicações em Containers
Rede Local → Nginx (HTTP) → Aplicações em Containers
```

## Pré-requisitos
- Domínio cbi1.org configurado na Cloudflare
- Docker e Docker Compose instalados
- Acesso à conta Cloudflare

## Estrutura de Diretórios
```bash
mkdir -p ~/cbi1-infra/{config,nginx,sites,mail,app}
cd ~/cbi1-infra
```

## 1. Autenticação do Cloudflared

```bash
# Executar autenticação inicial
docker run -it --rm \
  -v ~/cbi1-infra/config:/home/nonroot/.cloudflared \
  cloudflare/cloudflared:latest tunnel login
```

## 2. Criar o Tunnel

```bash
# Criar tunnel
docker run -it --rm \
  -v ~/cbi1-infra/config:/home/nonroot/.cloudflared \
  cloudflare/cloudflared:latest tunnel create cbi1-tunnel
```

Anote o UUID retornado.

## 3. Configuração do Cloudflare Tunnel

Crie `~/cbi1-infra/config/config.yml`:

```yaml
tunnel: <UUID_DO_TUNNEL>
credentials-file: /home/nonroot/.cloudflared/<UUID_DO_TUNNEL>.json
loglevel: info

ingress:
  - hostname: app.cbi1.org
    service: http://nginx:80
    originRequest:
      connectTimeout: 30s
      httpHostHeader: app.cbi1.org

  - hostname: mail.cbi1.org
    service: http://nginx:80
    originRequest:
      connectTimeout: 30s
      httpHostHeader: mail.cbi1.org

  - service: http_status:404
```

## 4. Configuração do Nginx

Crie `~/cbi1-infra/nginx/nginx.conf`:

```nginx
worker_processes auto;
error_log /var/log/nginx/error.log notice;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $host $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    keepalive_timeout 65;

    # Configuração do app.cbi1.org
    server {
        listen 80;
        server_name app.cbi1.org;

        location / {
            proxy_pass http://app:3000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            proxy_connect_timeout 30s;
            proxy_send_timeout 30s;
            proxy_read_timeout 30s;
        }
    }

    # Configuração do mail.cbi1.org
    server {
        listen 80;
        server_name mail.cbi1.org;

        location / {
            proxy_pass http://mail:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            proxy_connect_timeout 30s;
            proxy_send_timeout 30s;
            proxy_read_timeout 30s;
        }
    }

    # Bloquear acesso a outros hosts
    server {
        listen 80 default_server;
        server_name _;
        return 404;
    }
}
```

## 5. Docker Compose

Crie `~/cbi1-infra/docker-compose.yml`:

```yaml
version: '3.8'

services:
  # Cloudflare Tunnel
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared-tunnel
    restart: unless-stopped
    command: tunnel --config /home/nonroot/.cloudflared/config.yml run
    volumes:
      - ./config:/home/nonroot/.cloudflared
    networks:
      - cbi1-network
    depends_on:
      - nginx
    environment:
      - TZ=America/Sao_Paulo

  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    container_name: nginx-proxy
    restart: unless-stopped
    ports:
      - "80:80"  # Expor para rede local
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    networks:
      - cbi1-network
    environment:
      - TZ=America/Sao_Paulo
    depends_on:
      - app
      - mail

  # Aplicação Principal
  app:
    image: sua-imagem-app:latest  # Substituir pela sua imagem
    container_name: app-container
    restart: unless-stopped
    volumes:
      - ./app:/app  # Ajustar conforme necessidade
    networks:
      - cbi1-network
    environment:
      - VIRTUAL_HOST=app.cbi1.org
      - NODE_ENV=production
    # expose:
    #   - "3000"

  # Serviço de Email
  mail:
    image: sua-imagem-mail:latest  # Substituir pela sua imagem
    container_name: mail-container
    restart: unless-stopped
    volumes:
      - ./mail:/app  # Ajustar conforme necessidade
    networks:
      - cbi1-network
    environment:
      - VIRTUAL_HOST=mail.cbi1.org
      - NODE_ENV=production
    # expose:
    #   - "8080"

networks:
  cbi1-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

## 6. Configurar DNS na Cloudflare

```bash
# Configurar DNS via cloudflared
docker run -it --rm \
  -v ~/cbi1-infra/config:/home/nonroot/.cloudflared \
  cloudflare/cloudflared:latest tunnel route dns cbi1-tunnel app.cbi1.org

docker run -it --rm \
  -v ~/cbi1-infra/config:/home/nonroot/.cloudflared \
  cloudflare/cloudflared:latest tunnel route dns cbi1-tunnel mail.cbi1.org
```

## 7. Inicialização do Sistema

```bash
# Iniciar todos os serviços
cd ~/cbi1-infra
docker-compose up -d

# Verificar status
docker-compose ps

# Verificar logs
docker-compose logs -f
```

## 8. Verificação da Configuração

```bash
# Testar se todos os containers estão comunicando
docker exec nginx-proxy nginx -t
docker exec nginx-proxy curl -I http://app:3000
docker exec nginx-proxy curl -I http://mail:8080

# Verificar tunnel
docker exec cloudflared-tunnel cloudflared tunnel list
```

## 9. Comandos de Gerenciamento

```bash
# Parar serviços
docker-compose down

# Reiniciar serviços
docker-compose restart

# Atualizar containers
docker-compose pull
docker-compose up -d

# Verificar logs específicos
docker-compose logs cloudflared
docker-compose logs nginx
docker-compose logs app
docker-compose logs mail

# Acessar container para troubleshooting
docker exec -it nginx-proxy sh
docker exec -it app-container sh
```

## 10. Configuração para Acesso Local

Para acesso local via HTTP (sem Cloudflare):

- **app.cbi1.org**: `http://localhost` (via Nginx na porta 80)
- **mail.cbi1.org**: `http://localhost` (via Nginx na porta 80)

O Nginx faz o routing baseado no header Host.

## 11. Variáveis de Ambiente das Aplicações

Crie arquivos de ambiente se necessário:

`~/cbi1-infra/app/.env`:
```env
NODE_ENV=production
PORT=3000
DATABASE_URL=...
```

`~/cbi1-infra/mail/.env`:
```env
NODE_ENV=production
PORT=8080
EMAIL_SERVER=...
```

Atualize o docker-compose.yml para incluir os volumes:
```yaml
volumes:
  - ./app/.env:/app/.env
  - ./mail/.env:/app/.env
```

## 12. Monitoramento e Logs

```bash
# Logs em tempo real
docker-compose logs -f --tail=50

# Estatísticas de uso
docker stats

# Verificar saúde dos containers
docker-compose ps
```

## Solução de Problemas

### Problema: Containers não se comunicam
```bash
# Verificar network
docker network inspect cbi1-network

# Testar conectividade
docker exec nginx-proxy ping app
docker exec nginx-proxy ping mail
```

### Problema: Tunnel não conecta
```bash
# Verificar autenticação
docker run -it --rm \
  -v ~/cbi1-infra/config:/home/nonroot/.cloudflared \
  cloudflare/cloudflared:latest tunnel token

# Verificar logs detalhados
docker-compose logs cloudflared
```

### Problema: Nginx não faz routing correto
```bash
# Testar configuração do Nginx
docker exec nginx-proxy nginx -t

# Verificar acesso às aplicações
docker exec nginx-proxy curl http://app:3000
docker exec nginx-proxy curl http://mail:8080
```

## Backup e Restauração

```bash
# Backup das configurações
tar -czf cbi1-infra-backup.tar.gz ~/cbi1-infra/

# Restaurar
tar -xzf cbi1-infra-backup.tar.gz -C ~/
cd ~/cbi1-infra
docker-compose up -d
```

Esta configuração fornece:
- ✅ Cloudflare Tunnel encaminhando para Nginx na porta 80
- ✅ Nginx servindo múltiplos apps com base no hostname
- ✅ SSL apenas na Cloudflare (HTTP local)
- ✅ Cada app em container separado com rede bridge
- ✅ Tudo executado via Docker Compose


Ah, entendi! Com o Nginx na frente, a configuração muda um pouco. Vou te ajudar a configurar corretamente:

1. Verifique a Arquitetura Atual

Seu fluxo provavelmente é: Cloudflare Tunnel → Nginx → Budibase

2. Configuração do Cloudflare Tunnel

Seu config.yml deve apontar para o Nginx:

```yaml
tunnel: <SEU_TUNNEL_ID>
credentials-file: /root/.cloudflared/<SEU_TUNNEL_ID>.json

ingress:
  - hostname: seu-dominio.com
    service: http://localhost:80    # Porta do Nginx
    originRequest:
      noTLSVerify: false           # Nginx geralmente tem SSL
      httpHostHeader: seu-dominio.com

  - service: http_status:404
```

3. Configuração do Nginx para Budibase

nginx.conf ou seu virtual host:

```nginx
server {
    listen 80;
    server_name seu-dominio.com;

    # Proxy para o Budibase
    location / {
        proxy_pass http://budibase:10000;
        
        # Headers importantes para Cloudflare + Budibase
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $host;
        
        # Timeouts
        proxy_connect_timeout 300;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
        send_timeout 300;
    }

    # Health check
    location /health {
        proxy_pass http://budibase:10000/health;
        access_log off;
    }
}
```

4. Docker Compose com Nginx

docker-compose.yml:

```yaml
version: '3.8'
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - ./ssl:/etc/ssl/certs
    depends_on:
      - budibase
    networks:
      - budibase-network

  budibase:
    image: budibase/budibase:latest
    environment:
      - BUDIBASE_PUBLIC_URL=https://seu-dominio.com
      - JWT_SECRET=sua-chave-secreta-aqui
      - REDIS_URL=redis://redis:6379
      - MINIO_URL=http://budibase:10000
      - PORT=10000
      - BUDIBASE_SERVER_URL=https://seu-dominio.com
    depends_on:
      - redis
      - minio
    networks:
      - budibase-network

  redis:
    image: redis:7-alpine
    networks:
      - budibase-network

  minio:
    image: minio/minio
    environment:
      - MINIO_ROOT_USER=usuario
      - MINIO_ROOT_PASSWORD=senha
    networks:
      - budibase-network

networks:
  budibase-network:
    driver: bridge
```

5. Variáveis de Ambiente Cruciais

No Budibase, estas variáveis são ESSENCIAIS:

```yaml
environment:
  - BUDIBASE_PUBLIC_URL=https://seu-dominio.com
  - BUDIBASE_SERVER_URL=https://seu-dominio.com
  - BUDIBASE_CORS_ALLOWED_ORIGINS=https://seu-dominio.com
  - JWT_SECRET=uma-chave-secreta-forte-aqui
```

6. Comandos para Testar

```bash
# Testar se Nginx está respondendo
curl -v http://localhost:80

# Testar se Budibase está respondendo internamente
curl -v http://budibase:10000/health

# Ver logs do Nginx
docker logs nginx-container

# Ver logs do Budibase
docker logs budibase-container

# Ver logs do Cloudflare Tunnel
cloudflared tunnel tail seu-tunnel
```

7. Problemas Comuns e Soluções

A. Erro 502 Bad Gateway

```bash
# Verificar se containers estão comunicando
docker exec nginx-container ping budibase
```

B. Erro de CORS

Adicione no Nginx:

```nginx
add_header 'Access-Control-Allow-Origin' 'https://seu-dominio.com';
add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
```

C. Timeouts

Aumente timeouts no Nginx:

```nginx
proxy_connect_timeout 600;
proxy_send_timeout 600;
proxy_read_timeout 600;
```

8. Verificação Final

1. ✅ Nginx responde localmente na porta 80
2. ✅ Budibase responde internamente na porta 10000
3. ✅ Tunnel Cloudflare apontando para porta 80
4. ✅ Variáveis de ambiente corretas no Budibase
5. ✅ DNS apontando para o Tunnel

Qual erro específico você está vendo? Pode compartilhar os logs do Nginx ou do Cloudflare Tunnel?


Entendi! Seu Nginx também é um container. Nesse caso, a melhor opção é configurar o Cloudflare Tunnel em um container Docker na mesma rede que seus outros containers.

🐳 Configuração Recomendada com Docker

1. Estrutura da Sua Rede

```
Cloudflare Tunnel (container) → Nginx (container) → Budibase (container)
```

2. Docker Compose Completo

Arquivo: docker-compose.yml

```yaml
version: '3.8'

services:
  # Cloudflare Tunnel
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared-tunnel
    restart: unless-stopped
    command: tunnel --config /etc/cloudflared/config.yml run
    volumes:
      - ./cloudflared/config.yml:/etc/cloudflared/config.yml
      - ./cloudflared/credentials.json:/etc/cloudflared/credentials.json
    networks:
      - app-network
    depends_on:
      - nginx

  # Nginx como Proxy Reverso
  nginx:
    image: nginx:alpine
    container_name: nginx-proxy
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/sites-available:/etc/nginx/sites-available
    networks:
      - app-network
    depends_on:
      - budibase

  # Budibase
  budibase:
    image: budibase/budibase:latest
    container_name: budibase-app
    restart: unless-stopped
    environment:
      - BUDIBASE_PUBLIC_URL=https://app.cbi1.org
      - BUDIBASE_SERVER_URL=https://app.cbi1.org
      - BUDIBASE_CORS_ALLOWED_ORIGINS=https://app.cbi1.org
      - JWT_SECRET=sua-chave-secreta-muito-forte
      - REDIS_URL=redis://redis:6379
      - MINIO_URL=http://minio:9000
      - PORT=10000
    networks:
      - app-network
    depends_on:
      - redis
      - minio

  # Redis
  redis:
    image: redis:7-alpine
    container_name: budibase-redis
    restart: unless-stopped
    networks:
      - app-network

  # MinIO
  minio:
    image: minio/minio
    container_name: budibase-minio
    restart: unless-stopped
    environment:
      - MINIO_ROOT_USER=minioadmin
      - MINIO_ROOT_PASSWORD=minioadmin
    command: server /data --console-address ":9001"
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
    ipam:
      config:
        - subnet: 10.44.133.0/24
```

3. Configuração do Cloudflare Tunnel

Arquivo: cloudflared/config.yml

```yaml
tunnel: SEU_TUNNEL_ID
credentials-file: /etc/cloudflared/credentials.json
logfile: /var/log/cloudflared.log
loglevel: info

ingress:
  - hostname: app.cbi1.org
    service: http://nginx:80  # ⚡ Nome do serviço no Docker
    originRequest:
      connectTimeout: 30s
      httpHostHeader: app.cbi1.org
      noTLSVerify: true

  - service: http_status:404
```

4. Configuração do Nginx

Arquivo: nginx/nginx.conf

```nginx
events {
    worker_connections 1024;
}

http {
    upstream budibase {
        server budibase:10000;  # ⚡ Nome do serviço Docker
    }

    server {
        listen 80;
        server_name app.cbi1.org;

        location / {
            proxy_pass http://budibase;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            proxy_connect_timeout 300;
            proxy_send_timeout 300;
            proxy_read_timeout 300;
        }

        location /health {
            proxy_pass http://budibase/health;
            access_log off;
        }
    }
}
```

5. Script de Inicialização

Arquivo: start-app.sh

```bash
#!/bin/bash

# Criar diretórios de configuração
mkdir -p cloudflared nginx

# Iniciar todos os serviços
docker-compose down
docker-compose up -d

# Verificar status
echo "Aguardando inicialização..."
sleep 10

echo "Status dos containers:"
docker ps

echo "Testando conectividade:"
docker exec nginx-proxy curl -s http://budibase:10000/health || echo "Erro ao conectar Budibase"
```

6. Vantagens Desta Abordagem

· ✅ Comunicação por nome de serviço: nginx:80 e budibase:10000
· ✅ Rede isolada: Todos containers na mesma rede Docker
· ✅ Gerenciamento único: Um único docker-compose para tudo
· ✅ Fácil manutenção: Todos serviços gerenciados juntos

7. Comandos de Verificação

```bash
# Verificar se todos containers estão na mesma rede
docker network inspect app-network

# Testar comunicação entre containers
docker exec cloudflared-tunnel ping nginx
docker exec nginx-proxy ping budibase

# Ver logs do Cloudflare Tunnel
docker logs cloudflared-tunnel

# Ver logs do Nginx
docker logs nginx-proxy

# Ver logs do Budibase
docker logs budibase-app
```

8. Troubleshooting

Se houver problemas de conexão:

```bash
# Verificar DNS interno do Docker
docker exec nginx-proxy cat /etc/hosts

# Testar conectividade manualmente
docker exec nginx-proxy curl -v http://budibase:10000/health

# Reiniciar serviços específicos
docker-compose restart cloudflared nginx
```

Esta configuração é ideal para seu caso porque:

1. Todos serviços ficam na mesma rede Docker
2. Comunicação por nomes de serviço (não precisa de IPs fixos)
3. Fácil gerenciamento e escalabilidade
4. Isolamento completo do ambiente

Agora o tunnel se comunica com o Nginx usando o nome do serviço Docker (nginx:80) e o Nginx se comunica com o Budibase também pelo nome do serviço (budibase:10000).