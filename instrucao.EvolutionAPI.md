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


O que é CORS?

Definição Simplificada

CORS (Cross-Origin Resource Sharing) é um mecanismo de segurança dos navegadores que controla como recursos de um site (como APIs, fontes, imagens) podem ser acessados por outro domínio diferente do domínio de origem.

Analogia Prática

Imagine que CORS é como um segurança de boate:

· A boate é seu servidor (evolution-api)
· Os clientes são os navegadores (Chrome, Firefox, etc.)
· O segurança (CORS) verifica se cada cliente tem permissão para entrar

Se você tentar acessar a API de um domínio diferente (por exemplo, de http://localhost:3000 para https://seu-dominio-local.com), o "segurança" (navegador) pergunta: "Esse domínio tem permissão para acessar os recursos?"

Por que o CORS é importante para sua Evolution API?

No seu caso, o CORS é crucial porque:

1. Seu frontend (que mostra o QR Code) pode estar em um domínio
2. Sua API está em outro domínio (ou porta diferente)
3. Sem CORS configurado corretamente, o navegador bloqueia as requisições

Como funciona na prática

Quando seu navegador faz uma requisição para outra origem, ele primeiro envia uma requisição prévia (preflight) OPTIONS para verificar se tem permissão:

```
Navegador: OPTIONS https://seu-dominio-local.com/instance/connect
Servidor: Access-Control-Allow-Origin: https://meu-frontend.com
Navegador: OK, então posso fazer a requisição REAL
```

Configuração no seu .env

No seu arquivo .env, estas configurações controlam o CORS:

```env
# Permite qualquer domínio (*) ou domínios específicos
CORS_ORIGIN=*

# Quais métodos HTTP são permitidos
CORS_METHODS=GET,HEAD,PUT,PATCH,POST,DELETE

# Se permite envio de cookies/credenciais
CORS_CREDENTIALS=true

# Quais cabeçalhos são permitidos
CORS_ALLOWED_HEADERS=Content-Type,Authorization
```

Problemas comuns de CORS com QR Code

Se o QR Code não aparece, pode ser devido a:

1. CORS não configurado: A API não envia os cabeçalhos corretos
2. Domínio não permitido: Seu frontend não está na lista de origens permitidas
3. Credenciais bloqueadas: O navegador bloqueia cookies necessários

Solução no Nginx

Além da configuração na API, você pode configurar CORS diretamente no Nginx:

```nginx
server {
    listen 443 ssl http2;
    server_name seu-dominio-local.com;

    # Configurações CORS no Nginx
    add_header 'Access-Control-Allow-Origin' 'https://seu-frontend.com' always;
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, PUT, DELETE' always;
    add_header 'Access-Control-Allow-Headers' 'Authorization,Content-Type' always;
    add_header 'Access-Control-Allow-Credentials' 'true' always;

    # Para requisições OPTIONS (preflight)
    if ($request_method = 'OPTIONS') {
        return 204;
    }

    # Resto da configuração...
}
```

Modo de Desenvolvimento vs Produção

· Desenvolvimento: CORS_ORIGIN=* (permite tudo)
· Produção: CORS_ORIGIN=https://seu-dominio-real.com (apenas domínio específico)

Testando problemas de CORS

1. Abra o DevTools do navegador (F12)
2. Vá na aba "Network"
3. Tente carregar o QR Code
4. Verifique se há erros de CORS nas requisições

Resumo

CORS é um mecanismo de segurança essencial que:

· ✅ Controla acesso entre diferentes domínios
· ✅ Protege contra certos tipos de ataques
· ✅ É configurado tanto no servidor quanto possível no proxy (Nginx)
· ✅ Pode bloquear o QR Code se mal configurado

No seu caso, com CORS_ORIGIN=*, você está permitindo que qualquer domínio acesse sua API, o que resolve o problema do QR Code não aparecer, mas deve ser ajustado para produção.


Explicação das Variáveis: SERVER_URL, INSTANCE_SERVER_HOST e QRCODE

SERVER_URL

O que é: URL base completa onde sua Evolution API está hospedada.

Para que serve:

· Gerar URLs completas para webhooks
· Construir links absolutos para recursos (como QR Codes)
· Definir a origem correta para redirecionamentos

Exemplo:

```env
SERVER_URL=https://seu-dominio-local.com
```

Por que é importante para o QR Code: O WhatsApp exige que a URL do QR Code seja acessível publicamente.Se o SERVER_URL estiver incorreto (ex: http://localhost:8081), o QR Code não funcionará porque o WhatsApp não consegue acessar servidores locais.

---

INSTANCE_SERVER_HOST

O que é: URL específica onde as instâncias do WhatsApp estão hospedadas.

Para que serve:

· Comunicar-se com o serviço de WhatsApp Web
· Estabelecer a conexão WebSocket para cada instância
· Gerenciar múltiplas instâncias simultaneamente

Exemplo:

```env
INSTANCE_SERVER_HOST=https://seu-dominio-local.com
```

Diferença para SERVER_URL: Enquanto oSERVER_URL é para a API geral, o INSTANCE_SERVER_HOST é especificamente para as conexões das instâncias do WhatsApp.

---

Configurações de QRCODE

QRCODE_GENERATE_INTERVAL

O que é: Intervalo em milissegundos entre as gerações de QR Code.

Para que serve:

· Atualizar automaticamente o QR Code se expirar
· Manter a sessão ativa enquanto aguarda escaneamento

Exemplo:

```env
QRCODE_GENERATE_INTERVAL=5000  # 5 segundos
```

QRCODE_DISPLAY

O que é: Define se o QR Code deve ser exibido/logado no console.

Para que serve:

· Debug e desenvolvimento
· Visualização direta no terminal

Exemplo:

```env
QRCODE_DISPLAY=true
```

QRCODE_LIMIT

O que é: Número máximo de tentativas de geração de QR Code.

Para que serve:

· Prevenir loops infinitos de geração
· Gerenciar recursos do servidor

Exemplo:

```env
QRCODE_LIMIT=30  # 30 tentativas máximo
```

SAVE_QRCODE

O que é: Se deve salvar o QR Code como arquivo de imagem.

Para que serve:

· Acesso alternativo ao QR Code
· Backup das imagens geradas

Exemplo:

```env
SAVE_QRCODE=true
```

---

COMO FUNCIONA O FLUXO DO QR CODE

1. Inicialização:
   ```bash
   API → WhatsApp: "Quero conectar"
   WhatsApp → API: "Aqui está o QR Code"
   ```
2. Geração:
   ```bash
   API usa SERVER_URL para criar link acessível
   QR Code é gerado com URL contendo token único
   ```
3. Exibição:
   ```bash
   Frontend acessa: https://seu-dominio-local.com/qr-code?token=XYZ
   Ou visualiza imagem salva em /uploads/
   ```
4. Escaneamento:
   ```bash
   Usuário escaneia com app WhatsApp
   WhatsApp valida token com servidor
   Conexão estabelecida via INSTANCE_SERVER_HOST
   ```

---

PROBLEMAS COMUNS E SOLUÇÕES

QR Code não aparece

Causa: SERVER_URL incorreto Solução:Verifique se a URL é acessível externamente

QR Code expira rapidamente

Causa: QRCODE_GENERATE_INTERVAL muito alto Solução:Diminua o intervalo (2000-5000 ms)

Conexão falha após escaneamento

Causa: INSTANCE_SERVER_HOST incorreto Solução:Certifique-se que aponta para o mesmo domínio

QR Code não atualiza

Causa: QRCODE_LIMIT muito baixo Solução:Aumente o limite (30-50 tentativas)

---

CONFIGURAÇÃO RECOMENDADA PARA REDE LOCAL

```env
# URL acessível na sua rede
SERVER_URL=https://seu-dominio-local.com

# Mesmo que SERVER_URL
INSTANCE_SERVER_HOST=https://seu-dominio-local.com

# Atualiza a cada 5 segundos
QRCODE_GENERATE_INTERVAL=5000

# Exibe no console para debug
QRCODE_DISPLAY=true

# 30 tentativas de geração
QRCODE_LIMIT=30

# Salva como imagem para acesso alternativo
SAVE_QRCODE=true
```

Lembre-se de adicionar seu-dominio-local.com ao arquivo hosts da sua máquina para que a rede local consiga resolver o nome do domínio.