# Guia Completo: Criando Autoridade Certificadora (AC) Própria para Domínios Locais

## 📋 Índice
1. [Visão Geral](#visão-geral)
2. [Pré-requisitos](#pré-requisitos)
3. [Estrutura de Diretórios](#estrutura-de-diretórios)
4. [Passo a Passo: Criar AC Raiz](#passo-a-passo-criar-ac-raiz)
5. [Passo a Passo: Gerar Certificados para Domínios](#passo-a-passo-gerar-certificados-para-domínios)
6. [Instalar AC nos Dispositivos Clientes](#instalar-ac-nos-dispositivos-clientes)
7. [Configurar Servidores Web](#configurar-servidores-web)
8. [Verificação e Testes](#verificação-e-testes)
9. [Manutenção e Gerenciamento](#manutenção-e-geração)
10. [Script de Automação](#script-de-automação)

---

## 🎯 Visão Geral

Este guia ensina a criar uma Autoridade Certificadora (AC) própria para gerar certificados SSL/TLS válidos em rede local para os domínios:
- `cbi1.org`
- `mail.cbi1.org` 
- `supabase.cbi1.org`
- `budibase.cbi1.org`

## ⚙️ Pré-requisitos

- Linux/Unix ou WSL no Windows
- OpenSSL instalado
- Acesso root/sudo para instalar certificados
- Servidores web (Apache/Nginx) configurados

## 📁 Estrutura de Diretórios

```bash
/opt/ca/
├── root/                  # AC Raiz (mantenha seguro!)
│   ├── private/
│   │   └── ca.key        # Chave privada da AC
│   └── certs/
│       └── ca.crt        # Certificado da AC
├── domains/               # Certificados para domínios
│   ├── cbi1.org/
│   ├── mail.cbi1.org/
│   ├── supabase.cbi1.org/
│   └── budibase.cbi1.org/
└── scripts/
    └── generate-cert.sh  # Script de automação
```

## 🔐 Passo a Passo: Criar AC Raiz

### 1. Criar diretórios e configuração
```bash
sudo mkdir -p /opt/ca/root/{private,certs}
sudo mkdir -p /opt/ca/domains/{cbi1.org,mail.cbi1.org,supabase.cbi1.org,budibase.cbi1.org}
sudo mkdir /opt/ca/scripts
```

### 2. Criar arquivo de configuração OpenSSL
```bash
sudo nano /opt/ca/root/openssl.cnf
```

```ini
[ req ]
default_bits        = 4096
distinguished_name  = req_distinguished_name
x509_extensions     = v3_ca
prompt              = no
encrypt_key         = no

[ req_distinguished_name ]
countryName             = BR
stateOrProvinceName     = Sao Paulo
localityName            = Sao Paulo
organizationName        = CBI Organizacao
organizationalUnitName  = TI
commonName              = CBI Root CA
emailAddress            = admin@cbi1.org

[ v3_ca ]
subjectKeyIdentifier    = hash
authorityKeyIdentifier  = keyid:always,issuer:always
basicConstraints        = critical, CA:TRUE
keyUsage                = critical, digitalSignature, cRLSign, keyCertSign
```

### 3. Gerar AC Raiz
```bash
cd /opt/ca/root

# Gerar chave privada da AC
sudo openssl genrsa -aes256 -out private/ca.key 4096

# Gerar certificado autoassinado (válido por 10 anos)
sudo openssl req -new -x509 -days 3650 -key private/ca.key \
  -out certs/ca.crt -config openssl.cnf

# Criar versão sem senha para servidores (opcional)
sudo openssl rsa -in private/ca.key -out private/ca-unencrypted.key
```

## 🌐 Passo a Passo: Gerar Certificados para Domínios

### 1. Criar arquivo de configuração para domínios
```bash
sudo nano /opt/ca/domains/openssl.cnf
```

```ini
[ req ]
default_bits        = 2048
distinguished_name  = req_distinguished_name
req_extensions      = req_ext
prompt              = no

[ req_distinguished_name ]
countryName             = BR
stateOrProvinceName     = Sao Paulo
localityName            = Sao Paulo
organizationName        = CBI Organizacao
organizationalUnitName  = TI
commonName              = cbi1.org

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = cbi1.org
DNS.2 = www.cbi1.org
DNS.3 = mail.cbi1.org
DNS.4 = supabase.cbi1.org
DNS.5 = budibase.cbi1.org
IP.1  = 192.168.1.100
IP.2  = 192.168.1.101
```

### 2. Gerar certificado para cbi1.org (com todos SANs)
```bash
cd /opt/ca/domains/cbi1.org

# Gerar chave privada
sudo openssl genrsa -out cbi1.org.key 2048

# Gerar Certificate Signing Request (CSR)
sudo openssl req -new -key cbi1.org.key -out cbi1.org.csr \
  -config /opt/ca/domains/openssl.cnf

# Assinar com AC (válido por 1 ano)
sudo openssl x509 -req -days 365 -in cbi1.org.csr \
  -CA /opt/ca/root/certs/ca.crt -CAkey /opt/ca/root/private/ca.key \
  -CAcreateserial -out cbi1.org.crt -extensions req_ext \
  -extfile /opt/ca/domains/openssl.cnf

# Verificar certificado
sudo openssl x509 -in cbi1.org.crt -text -noout
```

### 3. Copiar para outros diretórios de domínio
```bash
sudo cp /opt/ca/domains/cbi1.org/cbi1.org.crt /opt/ca/domains/mail.cbi1.org/
sudo cp /opt/ca/domains/cbi1.org/cbi1.org.key /opt/ca/domains/mail.cbi1.org/
sudo cp /opt/ca/domains/cbi1.org/cbi1.org.crt /opt/ca/domains/supabase.cbi1.org/
sudo cp /opt/ca/domains/cbi1.org/cbi1.org.key /opt/ca/domains/supabase.cbi1.org/
sudo cp /opt/ca/domains/cbi1.org/cbi1.org.crt /opt/ca/domains/budibase.cbi1.org/
sudo cp /opt/ca/domains/cbi1.org/cbi1.org.key /opt/ca/domains/budibase.cbi1.org/
```

## 💻 Instalar AC nos Dispositivos Clientes

### Windows
1. Copiar `ca.crt` para o computador
2. Executar `certmgr.msc`
3. Ir em "Autoridades de Certificação Raiz Confiáveis" → "Certificados"
4. Clicar direito → "Importar" e selecionar `ca.crt`

### Linux (Debian/Ubuntu)
```bash
sudo cp /opt/ca/root/certs/ca.crt /usr/local/share/ca-certificates/cbi-ca.crt
sudo update-ca-certificates
```

### Linux (RedHat/CentOS)
```bash
sudo cp /opt/ca/root/certs/ca.crt /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust
```

### macOS
```bash
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain /opt/ca/root/certs/ca.crt
```

### Android
1. Copiar `ca.crt` para o dispositivo
2. Configurações → Segurança → Criptografia e credenciais
3. Instalar certificado → CA certificate

## 🖥️ Configurar Servidores Web

### Nginx (exemplo para supabase.cbi1.org)
```nginx
server {
    listen 443 ssl;
    server_name supabase.cbi1.org;
    
    ssl_certificate /opt/ca/domains/supabase.cbi1.org/cbi1.org.crt;
    ssl_certificate_key /opt/ca/domains/supabase.cbi1.org/cbi1.org.key;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    
    # ... resto da configuração
}

server {
    listen 80;
    server_name supabase.cbi1.org;
    return 301 https://$server_name$request_uri;
}
```

### Apache (exemplo para budibase.cbi1.org)
```apache
<VirtualHost *:443>
    ServerName budibase.cbi1.org
    
    SSLEngine on
    SSLCertificateFile /opt/ca/domains/budibase.cbi1.org/cbi1.org.crt
    SSLCertificateKeyFile /opt/ca/domains/budibase.cbi1.org/cbi1.org.key
    
    # ... resto da configuração
</VirtualHost>

<VirtualHost *:80>
    ServerName budibase.cbi1.org
    Redirect permanent / https://budibase.cbi1.org/
</VirtualHost>
```

## ✅ Verificação e Testes

### Testar configuração SSL
```bash
# Verificar certificado
openssl s_client -connect supabase.cbi1.org:443 -CAfile /opt/ca/root/certs/ca.crt

# Verificar com curl
curl -I --cacert /opt/ca/root/certs/ca.crt https://supabase.cbi1.org

# Testar todos domínios
domains=("cbi1.org" "mail.cbi1.org" "supabase.cbi1.org" "budibase.cbi1.org")
for domain in "${domains[@]}"; do
    echo "Testing $domain..."
    openssl s_client -connect $domain:443 -servername $domain -CAfile /opt/ca/root/certs/ca.crt | grep "Verify return code"
done
```

### Verificar se o certificado está correto
```bash
openssl x509 -in /opt/ca/domains/cbi1.org/cbi1.org.crt -text -noout | grep -A1 "Subject Alternative Name"
```

## 🔄 Manutenção e Gerenciamento

### Renovar certificado (antes de expirar)
```bash
# Recriar CSR (se necessário)
sudo openssl req -new -key /opt/ca/domains/cbi1.org/cbi1.org.key \
  -out /opt/ca/domains/cbi1.org/cbi1.org.csr \
  -config /opt/ca/domains/openssl.cnf

# Assinar novamente
sudo openssl x509 -req -days 365 -in /opt/ca/domains/cbi1.org/cbi1.org.csr \
  -CA /opt/ca/root/certs/ca.crt -CAkey /opt/ca/root/private/ca.key \
  -CAcreateserial -out /opt/ca/domains/cbi1.org/cbi1.org.crt \
  -extensions req_ext -extfile /opt/ca/domains/openssl.cnf

# Reiniciar servidores web
sudo systemctl restart nginx
sudo systemctl restart apache2
```

### Revogar certificado (em caso de comprometimento)
```bash
# Criar lista de revogação (se necessário)
openssl ca -gencrl -keyfile /opt/ca/root/private/ca.key \
  -cert /opt/ca/root/certs/ca.crt -out /opt/ca/root/private/ca.crl
```

## 🤖 Script de Automação

```bash
sudo nano /opt/ca/scripts/generate-cert.sh
```

```bash
#!/bin/bash

# Configurações
CA_DIR="/opt/ca"
DOMAINS=("cbi1.org" "mail.cbi1.org" "supabase.cbi1.org" "budibase.cbi1.org")
IP_ADDRESSES=("192.168.1.100" "192.168.1.101")
VALID_DAYS=365

echo "=== Gerador de Certificados CBI ==="

# Gerar CSR e certificado
cd $CA_DIR/domains/cbi1.org

echo "Gerando chave privada..."
openssl genrsa -out cbi1.org.key 2048

echo "Gerando CSR..."
openssl req -new -key cbi1.org.key -out cbi1.org.csr -config ../openssl.cnf

echo "Assinando certificado com AC..."
openssl x509 -req -days $VALID_DAYS -in cbi1.org.csr \
  -CA $CA_DIR/root/certs/ca.crt -CAkey $CA_DIR/root/private/ca.key \
  -CAcreateserial -out cbi1.org.crt -extensions req_ext \
  -extfile ../openssl.cnf

# Copiar para outros domínios
for domain in "${DOMAINS[@]:1}"; do
  echo "Copiando certificado para $domain..."
  cp cbi1.org.crt ../$domain/
  cp cbi1.org.key ../$domain/
done

echo "=== Certificados gerados com sucesso! ==="
echo "Lembre-se de reiniciar os servidores web:"
echo "sudo systemctl restart nginx apache2"
```

### Tornar script executável
```bash
sudo chmod +x /opt/ca/scripts/generate-cert.sh
```

## 🔒 Considerações de Segurança

1. **Proteja a chave da AC**: Mantenha `/opt/ca/root/private/` seguro
2. **Backup regular**: Faça backup da AC e chaves privadas
3. **Validade**: Certificados de servidor - 1 ano; AC - 10 anos
4. **Monitoramento**: Monitore datas de expiração dos certificados
5. **AC offline**: Para máxima segurança, mantenha a AC offline e só use para assinar

## 📝 Notas Finais

- Todos os domínios compartilham o mesmo certificado com múltiplos SANs
- Apenas a AC raiz precisa ser instalada nos clientes
- O mesmo certificado pode ser usado em todos os servidores
- Renove os certificados anualmente antes da expiração

Este setup proporciona SSL/TSSL válido para todos os domínios especificados sem avisos de segurança nos navegadores.
