# Configuração do Cloudflare Tunnel para app.cbi1.org e mail.cbi1.org

## Pré-requisitos
- Domínio cbi1.org configurado na Cloudflare
- Acesso administrativo à conta da Cloudflare
- Servidor Linux com acesso à internet
- Permissões de root/sudo no servidor

## Passo a passo da configuração

### 1. Configuração DNS na Cloudflare

Acesse o dashboard da Cloudflare e configure os registros DNS:

- **Para app.cbi1.org**: CNAME apontando para seu túnel (será configurado posteriormente)
- **Para mail.cbi1.org**: CNAME apontando para seu túnel

### 2. Instalação do Cloudflared no Linux

```bash
# Adicione o repositório da Cloudflare
wget -q https://packages.cloudflare.com/cloudflare-main.gpg -O /usr/share/keyrings/cloudflare-main.gpg
echo "deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://packages.cloudflare.com/cloudflared $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflared.list

# Atualize e instale o cloudflared
sudo apt update
sudo apt install cloudflared
```

### 3. Autenticação do Cloudflared

```bash
# Execute o comando de login
cloudflared tunnel login
```

Isso abrirá uma janela no navegador para autenticar e autorizar o acesso à sua conta Cloudflare.

### 4. Criar o Tunnel

```bash
# Crie um novo túnel
cloudflared tunnel create <NOME_DO_TUNNEL>
# Exemplo: cloudflared tunnel create meu-tunnel
```

Anote o UUID do túnel e o caminho do arquivo de credenciais JSON gerado.

### 5. Configurar o Arquivo de Configuração

Crie ou edite o arquivo de configuração em `~/.cloudflared/config.yml`:

```yaml
tunnel: <UUID_DO_TUNNEL>
credentials-file: <CAMINHO_DO_ARQUIVO_DE_CREDENCIAIS>

ingress:
  - hostname: app.cbi1.org
    service: http://localhost:3000  # Altere para a porta do seu app
  - hostname: mail.cbi1.org
    service: http://localhost:8080   # Altere para a porta do seu mail server
  - service: http_status:404
```

### 6. Configurar os Registros DNS no Cloudflared

```bash
# Crie os registros DNS para apontar para o túnel
cloudflared tunnel route dns <NOME_DO_TUNNEL> app.cbi1.org
cloudflared tunnel route dns <NOME_DO_TUNNEL> mail.cbi1.org
```

### 7. Executar o Tunnel como Serviço

```bash
# Instale o cloudflared como serviço
cloudflared service install
```

### 8. Iniciar e Gerenciar o Serviço

```bash
# Inicie o serviço
sudo systemctl start cloudflared

# Verifique o status
sudo systemctl status cloudflared

# Habilite para iniciar automaticamente
sudo systemctl enable cloudflared
```

### 9. Verificar a Configuração

```bash
# Verifique os logs para confirmar que está funcionando
journalctl -u cloudflared -f

# Teste a conexão
cloudflared tunnel list
```

## Configurações Adicionais

### Para Aplicações Web (HTTPS)
Certifique-se de que sua aplicação local está configurada para aceitar tráfego do tunnel. O Cloudflare fornecerá certificados SSL automaticamente.

### Configuração Avançada
Para configurações mais complexas, você pode editar o arquivo de configuração para incluir:
- Regras de origem
- Políticas de segurança
- Configurações de cache
- Configurações de rede

## Solução de Problemas Comuns

1. **Erro de conexão**: Verifique se o serviço local está rodando na porta especificada
2. **Erro de DNS**: Confirme se os registros CNAME foram criados corretamente na Cloudflare
3. **Problemas de autenticação**: Execute `cloudflared tunnel login` novamente

## Documentação Oficial
Consulte a documentação completa em: https://developers.cloudflare.com/cloudflare-one/connections/connect-apps

## Notas Importantes
- Mantenha o cloudflared atualizado para garantir segurança e performance
- Monitore os logs regularmente para detectar problemas
- Considere configurar backups do arquivo de configuração e credenciais

Seu tunnel agora deve estar funcionando e acessível através de `app.cbi1.org` e `mail.cbi1.org`.
