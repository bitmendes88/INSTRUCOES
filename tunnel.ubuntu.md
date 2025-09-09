Vou te ajudar a configurar um Cloudflare Tunnel para seu site. Aqui está um guia passo a passo:

1. Instalação do Cloudflared

Primeiro, instale o Cloudflare Tunnel no Ubuntu:

```bash
# Adicione a chave GPG do Cloudflare
sudo mkdir -p /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null

# Adicione o repositório
echo "deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflared.list

# Atualize e instale
sudo apt-get update
sudo apt-get install cloudflared
```

2. Autenticação no Cloudflare

```bash
# Faça login no Cloudflare
cloudflared tunnel login
```

Isso abrirá uma janela no navegador para você autenticar e selecionar o domínio (app.cbi1.org).

3. Criar o Tunnel

```bash
# Crie um novo tunnel
cloudflared tunnel create app-tunnel
```

Anote o UUID do tunnel que será gerado.

4. Configurar o Tunnel

Crie o arquivo de configuração:

```bash
sudo mkdir -p /etc/cloudflared
sudo nano /etc/cloudflared/config.yml
```

Adicione o seguinte conteúdo (substitua TUUNEL_UUID pelo UUID do seu tunnel):

```yaml
tunnel: TUUNEL_UUID
credentials-file: /etc/cloudflared/credentials.json

ingress:
  - hostname: app.cbi1.org
    service: http://localhost:80
  - service: http_status:404
```

5. Mover as Credenciais

```bash
# Mova o arquivo de credenciais para o local correto
sudo mv ~/.cloudflared/TUUNEL_UUID.json /etc/cloudflared/credentials.json
```

Substitua TUUNEL_UUID pelo UUID real do seu tunnel.

6. Configurar DNS

```bash
# Crie o registro DNS no Cloudflare
cloudflared tunnel route dns app-tunnel app.cbi1.org
```

7. Configurar como Serviço

```bash
# Instale como serviço
sudo cloudflared service install
```

8. Iniciar o Serviço

```bash
# Inicie o serviço
sudo systemctl start cloudflared
sudo systemctl enable cloudflared
```

9. Verificar Status

```bash
# Verifique se o tunnel está funcionando
sudo systemctl status cloudflared
cloudflared tunnel list
```

10. Configuração do Nginx (Opcional)

Se você precisar configurar o Nginx no container, aqui está um exemplo básico:

```nginx
server {
    listen 80;
    server_name app.cbi1.org;
    
    location / {
        proxy_pass http://localhost:3000; # ou o serviço que você está executando
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Comandos Úteis para Troubleshooting

```bash
# Ver logs do tunnel
sudo journalctl -u cloudflared -f

# Testar o tunnel manualmente
cloudflared tunnel run app-tunnel

# Verificar conexões
netstat -tlnp | grep :80
```

Verificação Final

1. Acesse app.cbi1.org no navegador
2. Verifique os logs: sudo journalctl -u cloudflared -n 50
3. Teste a conectividade: curl -I http://localhost:80

Se encontrar problemas, verifique:

· Se o container Nginx está rodando na porta 80
· Se as credenciais estão no local correto
· Se o DNS está apontando corretamente

Precisa de ajuda com alguma etapa específica?