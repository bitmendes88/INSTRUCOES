# Autohospedagem do Supabase com Docker

Este guia explica como auto-hospedar o Supabase usando Docker.

## Visão Geral

O Supabase fornece uma configuração Docker para auto-hospedagem, incluindo:
- Serviços principais do Supabase em contêineres
- Configurações personalizáveis
- Scripts para implantação e gerenciamento

## Pré-requisitos

- Docker Engine 19.03.0+
- Docker Compose 1.28.0+
- Pelo menos 4 GB de RAM (recomendado 8 GB)
- CPU com suporte a AVX (maioria das CPUs modernas)

## Começando

### 1. Clone o repositório

```bash
git clone --depth 1 https://github.com/supabase/supabase
cd supabase/docker
```

### 2. Configure o ambiente

Copie o arquivo de ambiente de exemplo:

```bash
cp .env.example .env
```

Edite o arquivo `.env` com suas configurações. As variáveis importantes incluem:

```env
POSTGRES_PASSWORD=senha-super-segura
JWT_SECRET=seu-segredo-jwt-super-seguro
SITE_URL=http://localhost:3000
ADDITIONAL_REDIRECT_URLS=
API_EXTERNAL_URL=http://localhost:8000
```

### 3. Inicie os serviços

Execute o script de inicialização:

```bash
./scripts/start.sh
```

Isso irá:
- Inicializar o volume do PostgreSQL se não existir
- Iniciar todos os serviços do Supabase

### 4. Acesse o estúdio

Abra seu navegador em http://localhost:8000 para acessar o Supabase Studio.

## Serviços incluídos

A configuração do Docker inclui:

- **PostgreSQL** - Banco de dados principal com extensões PostGIS
- **Studio** - Interface web do Supabase
- **Auth** - Serviço de autenticação
- **Rest** - API REST
- **Realtime** - Serviço em tempo real
- **Storage** - Gerenciamento de armazenamento
- **Edge Functions** - Funções serverless
- **Meta** - Gerenciamento de migrações
- **Logflare** - Agregação de logs
- **Kong** - API Gateway

## Configuração

### Portas

As portas padrão são:

- **5432** - PostgreSQL
- **8000** - Studio/Kong
- **8001** - Kong Admin API

### Volumes

Os volumes Docker são usados para persistência:

- `db-data` - Dados do PostgreSQL
- `db-wal` - Write-ahead logs do PostgreSQL

### Variáveis de ambiente

Principais variáveis para personalização:

- `POSTGRES_PASSWORD` - Senha do usuário postgres
- `JWT_SECRET` - Segredo para tokens JWT
- `SITE_URL` - URL do frontend
- `API_EXTERNAL_URL` - URL externa da API
- `ADDITIONAL_REDIRECT_URLS` - URLs adicionais para redirecionamento de autenticação

## Gerenciamento

### Parar serviços

```bash
./scripts/stop.sh
```

### Reiniciar serviços

```bash
./scripts/restart.sh
```

### Visualizar logs

```bash
docker-compose logs -f
```

### Backup e recuperação

Use `pg_dump` para backup do PostgreSQL:

```bash
docker exec -t supabase-db pg_dump -U postgres > backup.sql
```

Para restaurar:

```bash
docker exec -i supabase-db psql -U postgres < backup.sql
```

## Solução de problemas

### Verificar se a CPU suporta AVX

```bash
grep avx /proc/cpuinfo
```

Se não houver saída, sua CPU não suporta AVX e você precisará usar uma versão diferente do PostgreSQL.

### Erros de inicialização

Verifique os logs para diagnosticar problemas:

```bash
docker-compose logs [serviço]
```

### Limpar completamente

Para reiniciar do zero:

```bash
./scripts/stop.sh
docker-compose down -v
rm -rf volumes/
```

## Atualização

Para atualizar para uma versão mais recente:

1. Pare os serviços: `./scripts/stop.sh`
2. Atualize o repositório: `git pull origin master`
3. Inicie os serviços: `./scripts/start.sh`

## Considerações de produção

Para implantações em produção:

1. Use um proxy reverso (Nginx, Traefik)
2. Configure SSL/TLS
3. Implemente backup regular
4. Monitore o desempenho
5. Ajuste as configurações de recursos
6. Configure monitoramento e alertas

## Recursos adicionais

- [Documentação oficial do Supabase](https://supabase.com/docs)
- [Repositório GitHub](https://github.com/supabase/supabase)
- [Comunidade Discord](https://discord.supabase.com)

## Apoio

Para problemas relacionados à auto-hospedagem:
- Verifique issues do GitHub
- Junte-se à comunidade Discord
- Consulte a documentação oficial
