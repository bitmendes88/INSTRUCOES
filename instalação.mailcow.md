# Guia Completo de Instalação do Mailcow com Docker

## 📋 Pré-requisitos
- Servidor Ubuntu 20.04/22.04 LTS (recomendado)
- Domínio válido apontando para o IP do servidor
- Acesso root ou usuário com sudo
- Docker e Docker Compose instalados
- Mínimo 4GB de RAM (6GB+ recomendado)

## 🔧 Passo 1: Preparação do Sistema

### Atualizar o sistema
```bash
sudo apt update && sudo apt upgrade -y
```

### Instalar dependências
```bash
sudo apt install curl git apt-transport-https ca-certificates software-properties-common -y
```

### Configurar hostname (substitua mail.seudominio.com)
```bash
sudo hostnamectl set-hostname mail.seudominio.com
echo "127.0.0.1 mail.seudominio.com" | sudo tee -a /etc/hosts
```

## 🐳 Passo 2: Instalar Docker e Docker Compose

### Instalar Docker
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

### Instalar Docker Compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Verificar instalação
```bash
docker --version
docker-compose --version
```

## 📧 Passo 3: Instalar o Mailcow

### Clonar o repositório
```bash
cd /opt
sudo git clone https://github.com/mailcow/mailcow-dockerized
cd mailcow-dockerized
```

### Executar script de configuração
```bash
sudo ./generate_config.sh
```
- Quando solicitado, digite o **FQDN** (mail.seudominio.com)
- Confirme as configurações

### Iniciar o Mailcow
```bash
sudo docker-compose pull
sudo docker-compose up -d
```

### Verificar se todos os containers estão rodando
```bash
sudo docker-compose ps
```
Deve mostrar todos os containers com status "Up"

## 🌐 Passo 4: Configuração de Firewall e DNS

### Configurar firewall (UFW)
```bash
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw allow 25/tcp    # SMTP
sudo ufw allow 587/tcp   # Submission
sudo ufw allow 465/tcp   # SMTPS
sudo ufw allow 110/tcp   # POP3
sudo ufw allow 995/tcp   # POP3S
sudo ufw allow 143/tcp   # IMAP
sudo ufw allow 993/tcp   # IMAPS
```

### Configurar DNS (no seu registrador de domínio)
```
mail IN A SEU_IP
autodiscover IN CNAME mail
autoconfig IN CNAME mail
```

## ⚙️ Passo 5: Configuração Inicial do Mailcow

### Acessar o painel de administração
Abra no navegador: `https://mail.seudominio.com`

- Login: `admin`
- Senha: `moohoo` (altere imediatamente após o primeiro login)

### Configurar Let's Encrypt (SSL)
Edite o arquivo de configuração:
```bash
sudo nano /opt/mailcow-dockerized/mailcow.conf
```

Altere as linhas:
```
SKIP_LETS_ENCRYPT=n
ACME_CA_URI=https://acme-v02.api.letsencrypt.org/directory
```

Aplique as mudanças:
```bash
cd /opt/mailcow-dockerized
sudo docker-compose down
sudo docker-compose up -d
```

## 📨 Passo 6: Configuração de Domínio e E-mails

### Adicionar domínio
1. Acesse o painel do Mailcow
2. Vá em "Mail Setup" > "Domains"
3. Clique em "Add Domain"
4. Digite seu domínio e clique em "Add"

### Criar usuário de e-mail
1. Vá em "Mail Setup" > "Mailboxes"
2. Clique em "Add Mailbox"
3. Preencha os dados (usuário, nome, senha)
4. Clique em "Add"

## 🔍 Passo 7: Testes e Verificação

### Testar serviços
```bash
# Verificar se os containers estão saudáveis
sudo docker-compose logs --tail=50

# Testar conectividade SMTP
telnet mail.seudominio.com 25

# Testar conectividade IMAP
telnet mail.seudominio.com 143
```

### Testar configuração DNS
```bash
nslookup -type=mx seudominio.com
nslookup -type=txt seudominio.com
```

## 🛠️ Passo 8: Manutenção e Atualizações

### Atualizar o Mailcow
```bash
cd /opt/mailcow-dockerized
sudo ./update.sh --check
sudo ./update.sh
sudo docker-compose pull
sudo docker-compose up -d
```

### Backup
```bash
# Backup completo
cd /opt/mailcow-dockerized
sudo ./helper-scripts/backup_and_restore.sh backup full
```

### Monitoramento
```bash
# Verificar uso de recursos
sudo docker stats

# Verificar logs
sudo docker-compose logs -f
```

## ❗ Solução de Problemas Comuns

### Containers não iniciam
```bash
sudo docker-compose down
sudo docker-compose up -d
```

### Problemas de SSL
```bash
# Forçar renovação do certificado
sudo docker-compose exec acme-mailcow acme.sh --force --renew
```

### Espaço em disco
```bash
# Limpar imagens não utilizadas
sudo docker system prune -a
```

## 📚 Próximos Passos
1. Configurar políticas de spam e antivírus
2. Configurar autenticação DMARC/DKIM/SPF
3. Implementar backup automatizado
4. Configurar clientes de e-mail (Outlook, Thunderbird, mobile)

## 🔗 Links Úteis
- [Documentação Oficial do Mailcow](https://mailcow.github.io/mailcow-dockerized-docs/)
- [Comunidade Mailcow](https://community.mailcow.email/)
- [Wiki do Mailcow](https://github.com/mailcow/mailcow-dockerized/wiki)

---

**Nota**: Lembre-se de substituir `mail.seudominio.com` pelo seu domínio real em todos os comandos e configurações.
