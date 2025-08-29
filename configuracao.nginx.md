# Guia de Configuração: Proxy Reverso Nginx para Mailcow e Supabase

## 📋 Visão Geral
Este guia explica como configurar um proxy reverso Nginx para os serviços Mailcow e Supabase, resolvendo conflitos de porta e proporcionando acesso via domínios próprios com SSL.

## 🎯 Objetivo
- Acessar Mailcow via `mail.cbi1.org` (sem porta 8443 na URL)
- Acessar Supabase via `studio.cbi1.org` (sem porta 8443 na URL)
- SSL centralizado no Nginx proxy
- Sem conflitos de porta

## 📝 Pré-requisitos
- Docker e Docker Compose instalados
- Mailcow e Supabase já configurados em containers
- Domínio `cbi1.org` configurado no DNS
- Certificados SSL para os subdomínios

## 🏗️ Estrutura de Arquivos
```
/opt/nginx-proxy/
├── docker-compose.yml
├── config/
│   ├── nginx.conf
│   └── sites/
│       ├── mail.conf
│       └── studio.conf
├── ssl-certs/
│   ├── mail.cbi1.org.crt
│   ├── mail.cbi1.org.key
│   ├── studio.cbi1.org.crt
│   └── studio.cbi1.org.key
└── logs/
    ├── access.log
    └── error.log
```

## 🔧 Configuração do Docker Compose

### `/opt/nginx-proxy/docker-compose.yml`
```yaml
version: '3.8'
services:
  nginx-proxy:
    image: nginx:alpine
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./config/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./config/sites:/etc/nginx/conf.d:ro
      - ./ssl-certs:/etc/ssl/certs:ro
      - ./logs:/var/log/nginx
    networks:
      - proxy-network
      - mailcow-network
      - supabase-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "nginx", "-t"]
      interval: 30s
      timeout: 10s
      retries: 3

networks:
  proxy-network:
    driver: bridge
  mailcow-network:
    external: true
  supabase-network:
    external: true
```

## 🛠️ Configuração do Nginx

### `/opt/nginx-proxy/config/nginx.conf`
```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log /var/log/nginx/access.log main;
    
    sendfile on;
    keepalive_timeout 65;
    
    # Configurações SSL padrão
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_session_timeout 10m;
    ssl_session_cache shared:SSL:10m;
    
    include /etc/nginx/conf.d/*.conf;
}
```

## 🌐 Configuração do Mailcow

### `/opt/nginx-proxy/config/sites/mail.conf`
```nginx
# Redirecionamento HTTP → HTTPS
server {
    listen 80;
    server_name mail.cbi1.org;
    return 301 https://$server_name$request_uri;
}

# Servidor HTTPS principal
server {
    listen 443 ssl http2;
    server_name mail.cbi1.org;
    
    # Certificados SSL
    ssl_certificate /etc/ssl/certs/mail.cbi1.org.crt;
    ssl_certificate_key /etc/ssl/certs/mail.cbi1.org.key;
    
    # 🔥 CONFIGURAÇÃO DE PROXY TRANSPARENTE
    # Reescreve URLs com :8443 para não ter porta
    proxy_redirect ~^(https?://[^:]+):8443(/.+)$ $1$2;
    
    location / {
        # Proxy para o container Mailcow
        proxy_pass https://mailcowdockerized-nginx-mailcow-1:8443;
        
        # Headers que "enganam" o Mailcow
        proxy_set_header Host $host:443;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port 443;
        proxy_set_header X-Forwarded-Host $host;
        
        # Configurações de timeout
        proxy_connect_timeout 300;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
        send_timeout 300;
        
        # Desabilitar verificação SSL para interno
        proxy_ssl_verify off;
        proxy_buffering off;
    }
    
    # Configuração específica para WebSocket
    location /api/ {
        proxy_pass https://mailcowdockerized-nginx-mailcow-1:8443/api/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 🗄️ Configuração do Supabase

### `/opt/nginx-proxy/config/sites/studio.conf`
```nginx
# Redirecionamento HTTP → HTTPS
server {
    listen 80;
    server_name studio.cbi1.org;
    return 301 https://$server_name$request_uri;
}

# Servidor HTTPS principal
server {
    listen 443 ssl http2;
    server_name studio.cbi1.org;
    
    # Certificados SSL
    ssl_certificate /etc/ssl/certs/studio.cbi1.org.crt;
    ssl_certificate_key /etc/ssl/certs/studio.cbi1.org.key;
    
    location / {
        # Proxy para o Supabase Studio
        proxy_pass http://supabase-studio:3000;
        
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    location /auth/ {
        # Proxy para o serviço de autenticação
        proxy_pass http://supabase-auth:9999;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    location /rest/ {
        # Proxy para a API REST
        proxy_pass http://supabase-rest:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 🚀 Implementação Passo a Passo

### 1. **Preparar Ambiente**
```bash
sudo mkdir -p /opt/nginx-proxy/{config/sites,ssl-certs,logs}
cd /opt/nginx-proxy
```

### 2. **Configurar Certificados SSL**
```bash
# Copiar certificados para a pasta ssl-certs
sudo cp /caminho/para/certificados/* /opt/nginx-proxy/ssl-certs/
```

### 3. **Criar Redes Docker**
```bash
# Criar rede para o proxy
docker network create proxy-network

# Conectar containers existentes às redes
docker network connect proxy-network mailcowdockerized-nginx-mailcow-1
docker network connect proxy-network supabase-kong
```

### 4. **Ajustar Configurações dos Serviços**
**No Mailcow:** Voltar para porta 8443 interna
**No Supabase:** Usar apenas `expose:` no docker-compose

### 5. **Iniciar Proxy Reverso**
```bash
cd /opt/nginx-proxy
docker-compose up -d
```

### 6. **Verificar Configuração**
```bash
# Testar configuração Nginx
docker exec nginx-proxy nginx -t

# Verificar logs
docker logs nginx-proxy

# Testar acesso
curl -I https://mail.cbi1.org
curl -I https://studio.cbi1.org
```

## 🧪 Testes e Validação

### Testar Proxy Reverso
```bash
# Testar redirecionamento HTTP → HTTPS
curl -I http://mail.cbi1.org
curl -I http://studio.cbi1.org

# Testar acesso HTTPS
curl -k -I https://mail.cbi1.org
curl -k -I https://studio.cbi1.org

# Verificar se URLs não contêm porta 8443
curl -k https://mail.cbi1.org | grep -o "https://[^\"]*" | head -5
```

### Verificar Conexões de Rede
```bash
# Verificar se containers estão conectados às redes
docker network inspect proxy-network
docker network inspect mailcow-network
docker network inspect supabase-network
```

## 🛠️ Solução de Problemas

### Problema: Links quebrados no Mailcow
**Solução:** Verificar configuração do `proxy_redirect` e headers

### Problema: SSL não funciona
**Solução:** Verificar caminhos dos certificados e permissões

### Problema: Containers não se comunicam
**Solução:** Verificar conexões de rede com `docker network inspect`

## 📊 Monitoramento
```bash
# Verificar logs em tempo real
docker logs -f nginx-proxy

# Monitorar acesso
tail -f /opt/nginx-proxy/logs/access.log

# Verificar saúde do container
docker ps | grep nginx-proxy
```

## 🔒 Considerações de Segurança

1. **Isolamento de Rede**: Cada serviço em sua própria rede
2. **SSL Terminado no Proxy**: Criptografia até o proxy
3. **Headers de Segurança**: Adicionar headers security headers
4. **Logs**: Monitoramento de acesso centralizado

## ✅ Benefícios desta Configuração

- ✅ **Sem conflitos de porta**
- ✅ **URLs limpas** (sem :8443)
- ✅ **SSL centralizado**
- ✅ **Fácil manutenção**
- ✅ **Escalável** para novos serviços

Esta configuração proporciona uma solução profissional e production-ready para seus serviços! 🎉
