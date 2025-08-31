# Guia Completo de Comandos Docker

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

## Índice
- [Introdução](#introdução)
- [Instalação](#instalação)
- [Comandos Básicos](#comandos-básicos)
- [Gerenciamento de Containers](#gerenciamento-de-containers)
- [Gerenciamento de Imagens](#gerenciamento-de-imagens)
- [Docker Compose](#docker-compose)
- [Redes](#redes)
- [Volumes](#volumes)
- [Dockerfile](#dockerfile)
- [Logs e Inspeção](#logs-e-inspeção)
- [Limpeza e Manutenção](#limpeza-e-manutenção)
- [Dicas e Boas Práticas](#dicas-e-boas-práticas)

## Introdução

Docker é uma plataforma aberta para desenvolver, enviar e executar aplicações em containers. Containers permitem empacotar uma aplicação com todas as suas dependências em uma unidade padronizada para desenvolvimento.

## Instalação

### Ubuntu/Debian
```bash
# Atualizar repositórios
sudo apt-get update

# Instalar dependências
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common

# Adicionar chave GPG oficial do Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Adicionar repositório do Docker
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Instalar Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Verificar instalação
sudo docker --version
```

### Windows/macOS
Baixe e instale o [Docker Desktop](https://www.docker.com/products/docker-desktop) para Windows ou macOS.

## Comandos Básicos

```bash
# Verificar versão do Docker
docker --version
docker version
docker info

# Executar um container hello-world para testar
docker run hello-world

# Listar containers em execução
docker ps

# Listar todos os containers (incluindo parados)
docker ps -a

# Listar imagens disponíveis localmente
docker images
```

## Gerenciamento de Containers

```bash
# Executar um container
docker run [opções] imagem [comando] [args]

# Exemplos:
docker run -it ubuntu /bin/bash           # Modo interativo
docker run -d nginx                       # Modo detached (em segundo plano)
docker run -p 8080:80 nginx               # Mapear porta host:container
docker run -v /host/path:/container/path  # Mapear volume
docker run --name meu-container nginx     # Nomear container
docker run -e VAR=valor nginx             # Definir variável de ambiente

# Parar um container
docker stop [container_id ou nome]

# Iniciar um container parado
docker start [container_id ou nome]

# Reiniciar um container
docker restart [container_id ou nome]

# Pausar um container
docker pause [container_id ou nome]

# Despausar um container
docker unpause [container_id ou nome]

# Remover um container
docker rm [container_id ou nome]

# Remover container parado forcadamente
docker rm -f [container_id ou nome]

# Executar comando em container em execução
docker exec [opções] container comando [args]

# Exemplos:
docker exec -it meu-container /bin/bash   # Terminal interativo
docker exec meu-container ls -la          # Executar comando específico

# Copiar arquivos do/para container
docker cp [container_id]:/caminho/arquivo /caminho/local
docker cp /caminho/local [container_id]:/caminho/arquivo

# Inspecionar container
docker inspect [container_id ou nome]

# Ver processos em execução no container
docker top [container_id ou nome]

# Ver estatísticas de uso de recursos
docker stats [container_id ou nome]
```

## Gerenciamento de Imagens

```bash
# Buscar imagens no Docker Hub
docker search [termo]

# Baixar (pull) uma imagem
docker pull [imagem]:[tag]

# Listar imagens locais
docker images
docker image ls

# Inspecionar uma imagem
docker inspect [imagem]

# Ver histórico de uma imagem
docker history [imagem]

# Remover uma imagem
docker rmi [imagem]
docker image rm [imagem]

# Remover imagens não utilizadas
docker image prune

# Remover todas as imagens não utilizadas (incluindo dangling)
docker image prune -a

# Taggear uma imagem
docker tag [imagem_origem] [novo_nome]:[tag]

# Salvar imagem para arquivo tar
docker save -o [arquivo.tar] [imagem]

# Carregar imagem de arquivo tar
docker load -i [arquivo.tar]

# Login no Docker Hub/registry
docker login

# Logout do Docker Hub/registry
docker logout

# Push de imagem para registry
docker push [usuario]/[imagem]:[tag]
```

## Docker Compose

Docker Compose é uma ferramenta para definir e executar aplicações multi-container.

```bash
# Instalação do Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verificar versão
docker-compose --version

# Comandos básicos
docker-compose up              # Criar e iniciar containers
docker-compose up -d           # Executar em segundo plano
docker-compose down            # Parar e remover containers
docker-compose ps              # Listar containers
docker-compose logs            # Ver logs
docker-compose exec [serviço] [comando]  # Executar comando no serviço
docker-compose build           # Construir imagens
docker-compose pull            # Baixar imagens
docker-compose restart         # Reiniciar containers
docker-compose pause           # Pausar containers
docker-compose unpause         # Despausar containers
```

### Exemplo de docker-compose.yml
```yaml
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - minha-rede

  db:
    image: postgres:13
    environment:
      POSTGRES_DB: meu_db
      POSTGRES_USER: usuario
      POSTGRES_PASSWORD: senha
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - minha-rede

volumes:
  db-data:

networks:
  minha-rede:
    driver: bridge
```

## Redes

```bash
# Listar redes
docker network ls

# Criar uma rede
docker network create [nome_rede]

# Inspecionar uma rede
docker network inspect [nome_rede]

# Conectar container a uma rede
docker network connect [nome_rede] [container]

# Desconectar container de uma rede
docker network disconnect [nome_rede] [container]

# Remover uma rede
docker network rm [nome_rede]

# Tipos de rede:
# - bridge: Rede padrão para containers
# - host: Remove isolamento de rede entre container e host
# - none: Desabilita toda a rede
# - overlay: Para comunicação entre containers em diferentes hosts (Docker Swarm)
# - macvlan: Permite atribuir endereço MAC ao container
```

## Volumes

```bash
# Listar volumes
docker volume ls

# Criar volume
docker volume create [nome_volume]

# Inspecionar volume
docker volume inspect [nome_volume]

# Remover volume
docker volume rm [nome_volume]

# Remover volumes não utilizados
docker volume prune

# Usar volumes em containers
docker run -v [nome_volume]:/caminho/container [imagem]
docker run --mount source=[nome_volume],target=/caminho/container [imagem]
```

## Dockerfile

Um Dockerfile é um arquivo de texto que contém todas as instruções para construir uma imagem Docker.

### Exemplo de Dockerfile
```dockerfile
# Imagem base
FROM node:16-alpine

# Definir diretório de trabalho
WORKDIR /app

# Copiar arquivos de dependências
COPY package*.json ./

# Instalar dependências
RUN npm install

# Copiar código fonte
COPY . .

# Expor porta
EXPOSE 3000

# Definir variável de ambiente
ENV NODE_ENV=production

# Comando para executar a aplicação
CMD ["npm", "start"]
```

### Comandos para construir imagens
```bash
# Construir imagem a partir do Dockerfile
docker build -t [nome_imagem]:[tag] .

# Construir a partir de um Dockerfile em caminho específico
docker build -t [nome_imagem]:[tag] -f /caminho/Dockerfile .

# Construir com argumentos
docker build --build-arg VAR=valor -t [nome_imagem]:[tag] .
```

### Instruções do Dockerfile
- `FROM`: Define a imagem base
- `RUN`: Executa comandos durante a construção
- `COPY`: Copia arquivos do host para a imagem
- `ADD`: Similar ao COPY, mas com funcionalidades adicionais
- `CMD`: Comando padrão ao executar o container
- `ENTRYPOINT`: Configura o container para executar como um executável
- `ENV`: Define variáveis de ambiente
- `ARG`: Define variáveis de construção
- `EXPOSE`: Expõe portas
- `WORKDIR`: Define o diretório de trabalho
- `USER`: Define o usuário para instruções seguintes
- `VOLUME`: Cria ponto de montagem de volume
- `LABEL`: Adiciona metadados à imagem
- `HEALTHCHECK`: Define como verificar a saúde do container

## Logs e Inspeção

```bash
# Ver logs de um container
docker logs [container_id ou nome]

# Seguir logs (similar a tail -f)
docker logs -f [container_id ou nome]

# Ver logs com timestamps
docker logs -t [container_id ou nome]

# Ver últimas N linhas de logs
docker logs --tail [N] [container_id ou nome]

# Inspecionar container ou imagem (JSON format)
docker inspect [container_id ou nome|imagem]

# Ver eventos do Docker em tempo real
docker events

# Ver mudanças nos arquivos do container
docker diff [container_id ou nome]

# Ver uso de disco
docker system df

# Ver informações detalhadas de uso de disco
docker system df -v
```

## Limpeza e Manutenção

```bash
# Remover todos os containers parados
docker container prune

# Remover todas as imagens não utilizadas
docker image prune -a

# Remover todos os volumes não utilizados
docker volume prune

# Remover todos os recursos não utilizados (containers, imagens, redes, volumes)
docker system prune

# Remover todos os recursos não utilizados (forçado, sem confirmação)
docker system prune -f

# Remover tudo (containers, imagens, volumes, redes)
docker system prune -a --volumes

# Limpar builder cache
docker builder prune
```

## Dicas e Boas Práticas

1. **Use imagens oficiais** sempre que possível
2. **Especifique tags** em vez de usar `latest`
3. **Minimize o número de camadas** combinando comandos RUN
4. **Use .dockerignore** para excluir arquivos desnecessários
5. **Não execute processos como root** dentro do container
6. **Use volumes** para dados persistentes
7. **Uma função por container** - siga a filosofia de microserviços
8. **Documente** com LABEL e mantenha Dockerfiles legíveis
9. **Use multi-stage builds** para imagens menores
10. **Monitore** recursos com `docker stats`

### Exemplo de Multi-stage Build
```dockerfile
# Estágio de construção
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Estágio de produção
FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm install --production
EXPOSE 3000
CMD ["npm", "start"]
```

### Exemplo de .dockerignore
```
node_modules
npm-debug.log
.git
.gitignore
README.md
Dockerfile
.dockerignore
.env
```

## Recursos Adicionais

- [Documentação Oficial do Docker](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/) - Repositório de imagens
- [Best Practices for Writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Play with Docker](https://labs.play-with-docker.com/) - Ambiente de prática online

---

*Este documento é um guia de referência e pode não incluir todos os comandos ou opções disponíveis. Consulte a documentação oficial para informações completas e atualizadas.*
