Documentação de Configuração do Sistema Budibase + Supabase Local

📋 Índice

1. Visão Geral
2. Pré-requisitos
3. Estrutura do Projeto
4. Configuração Inicial
5. Configuração do Docker Swarm
6. Configuração SSL
7. Implantação do Sistema
8. Verificação e Testes
9. Gerenciamento do Swarm
10. Solução de Problemas

---

🎯 Visão Geral

Este documento descreve a configuração de um sistema completo com:

· Budibase para criação de aplicativos
· Supabase local com PostgreSQL, Studio e Auth
· Nginx com SSL e domínio local cobom.app
· Docker Swarm para distribuição em duas máquinas

---

🛠️ Pré-requisitos

🔧 Software Necessário

```bash
# Em ambas as máquinas
sudo apt-get update && sudo apt-get upgrade -y
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
```

🌐 Configuração de Rede

```bash
# Configurar hosts em ambas as máquinas
sudo nano /etc/hosts

# Adicionar as linhas:
127.0.0.1   cobom.app
127.0.0.1   studio.cobom.app
# Para acesso entre máquinas, usar IPs reais
```

---

📁 Estrutura do Projeto

```
budibase-supabase-local/
├── docker-swarm.yml
├── nginx/
│   ├── nginx.conf
│   ├── ssl/
│   │   ├── cobom.app.crt
│   │   └── cobom.app.key
├── supabase/
│   └── scripts/
│       └── init-supabase.sql
├── config/
│   └── budibase.env
└── scripts/
    ├── init-swarm.sh
    ├── deploy.sh
    └── generate-passwords.sh
```

---

⚙️ Configuração Inicial

1. Clonar/Criar Estrutura

```bash
mkdir budibase-supabase-local
cd budibase-supabase-local
mkdir -p nginx/ssl supabase/scripts config scripts
```

2. Gerar Senhas Seguras

```bash
chmod +x scripts/generate-passwords.sh
./scripts/generate-passwords.sh
```

3. Configurar Variáveis de Ambiente

Edite config/budibase.env conforme necessário:

```env
BB_ADMIN_EMAIL=admin@cobom.app
BB_ADMIN_PASSWORD=sua_senha_admin_forte
```

---

🔄 Configuração do Docker Swarm

1. Inicializar Swarm na Máquina Principal

```bash
chmod +x scripts/init-swarm.sh
./scripts/init-swarm.sh
```

2. Adicionar Segunda Máquina

Execute o comando fornecido pelo output do script anterior na segunda máquina.

3. Verificar Nodes

```bash
docker node ls
```

---

🔐 Configuração SSL

1. Gerar Certificados Autoassinados

```bash
mkdir -p nginx/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/cobom.app.key \
  -out nginx/ssl/cobom.app.crt \
  -subj "/CN=cobom.app"
```

2. Para Certificados Válidos (Opcional)

Use Certbot ou importe certificados de uma CA válida para a pasta nginx/ssl/.

---

🚀 Implantação do Sistema

1. Implantar Stack Completa

```bash
chmod +x scripts/deploy.sh
./scripts/deploy.sh
```

2. Verificar Status

```bash
docker service ls
docker service ps budibase-supabase_budibase
docker service ps budibase-supabase_supabase-db
```

3. Acompanhar Logs

```bash
docker service logs -f budibase-supabase_budibase
docker service logs -f budibase-supabase_supabase-db
```

---

✅ Verificação e Testes

1. Testar Acesso aos Serviços

```bash
# Testar Budibase
curl -k https://cobom.app/health

# Testar Supabase Studio
curl -k https://studio.cobom.app

# Testar conexão com banco
docker exec -it $(docker ps -q -f name=supabase-db) psql -U postgres -c "\l"
```

2. Acessar Interfaces Web

· Budibase: https://cobom.app
· Supabase Studio: https://studio.cobom.app
· Credenciais padrão: admin@cobom.app / senha configurada no .env

---

⚡ Gerenciamento do Swarm

Comandos Úteis

```bash
# Escalar serviços
docker service scale budibase-supabase_budibase=3

# Ver logs em tempo real
docker service logs -f budibase-supabase_budibase

# Atualizar serviço
docker service update --image budibase/budibase:latest budibase-supabase_budibase

# Remover stack
docker stack rm budibase-supabase

# Ver recursos
docker node ps
docker service inspect budibase-supabase_budibase
```

Monitoramento

```bash
# Ver uso de recursos
docker stats

# Ver serviços por node
docker node ps $(docker node ls -q)
```

---

🐛 Solução de Problemas

Problemas Comuns e Soluções

1. Erro de Conexão entre Nodes

```bash
# Verificar se todas as portas necessárias estão abertas
sudo ufw allow 2377/tcp
sudo ufw allow 7946/tcp
sudo ufw allow 7946/udp
sudo ufw allow 4789/udp
```

2. Problemas de SSL

```bash
# Verificar certificados
openssl x509 -in nginx/ssl/cobom.app.crt -text -noout

# Testar configuração Nginx
docker exec -it $(docker ps -q -f name=nginx) nginx -t
```

3. Serviços Não Iniciam

```bash
# Ver logs detalhados
docker service logs --tail 100 budibase-supabase_budibase

# Ver eventos do serviço
docker service events budibase-supabase_budibase
```

4. Problemas de Volume

```bash
# Verificar volumes
docker volume ls

# Limpar volumes (cuidado!)
docker volume prune
```

5. Rede Overlay

```bash
# Verificar redes
docker network ls

# Inspecionar rede
docker network inspect budibase-network
```

Comandos de Diagnóstico

```bash
# Health check completo
curl -k https://cobom.app/health

# Verificar conectividade entre containers
docker exec -it $(docker ps -q -f name=budibase) ping supabase-db

# Verificar DNS resolution
docker exec -it $(docker ps -q -f name=budibase) nslookup supabase-db
```

---

🔄 Backup e Recuperação

Backup de Dados

```bash
# Backup do PostgreSQL
docker exec -it $(docker ps -q -f name=supabase-db) pg_dump -U postgres postgres > backup.sql

# Backup de volumes
docker run --rm -v supabase_data:/source -v $(pwd)/backup:/backup alpine tar czf /backup/supabase_backup.tar.gz /source
```

Restauração

```bash
# Restaurar PostgreSQL
cat backup.sql | docker exec -i $(docker ps -q -f name=supabase-db) psql -U postgres postgres

# Restaurar volume
docker run --rm -v supabase_data:/target -v $(pwd)/backup:/backup alpine tar xzf /backup/supabase_backup.tar.gz -C /target
```

---

📊 Monitoramento e Logs

Configurar Log Rotation

```bash
# Configurar driver de logs no Docker daemon
sudo nano /etc/docker/daemon.json

# Adicionar:
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Monitoramento com cAdvisor (Opcional)

```bash
docker run -d \
  --name=cadvisor \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  google/cadvisor:latest
```

---

🔒 Segurança

Melhores Práticas

1. Alterar senhas padrão no arquivo .env
2. Restringir acesso às portas de administração
3. Configurar firewall adequadamente
4. Monitorar logs regularmente
5. Manter imagens atualizadas

Atualizações de Segurança

```bash
# Atualizar todas as imagens
docker images | awk 'NR>1 {print $1 ":" $2}' | xargs -L1 docker pull

# Recriar serviços com imagens atualizadas
docker service update --image budibase/budibase:latest budibase-supabase_budibase
```

---

📞 Suporte

Recursos Úteis

· Documentação Docker Swarm
· Documentação Budibase
· Documentação Supabase

Verificação Final

Após implantação, verifique:

· ✅ Todos os serviços estão rodando
· ✅ SSL funciona corretamente
· ✅ Acesso via cobom.app e studio.cobom.app
· ✅ Conexão entre Budibase e Supabase
· ✅ Replicação entre nodes do Swarm

---

Nota: Este setup é para ambiente de desenvolvimento/local. Para produção, considere:

· Certificados SSL válidos
· Monitoramento mais robusto
· Backup automatizado
· Maior segurança de rede
· Load balancing adicional

Para qualquer problema, consulte a seção de Solução de Problemas ou verifique os logs dos serviços.
