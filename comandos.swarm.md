# Comandos Rápidos - Docker Swarm

## Inicialização do Swarm
```bash
# Inicializar swarm (manager)
docker swarm init

# Inicializar swarm com IP específico
docker swarm init --advertise-addr <IP>

# Juntar nó worker ao swarm
docker swarm join --token <TOKEN> <IP>:2377

# Juntar nó manager ao swarm
docker swarm join-token manager

# Listar nós do swarm
docker node ls

# Ver informações do swarm
docker info
```

## Gerenciamento de Nós
```bash
# Promover nó para manager
docker node promote <NODE>

# Rebaixar nó para worker
docker node demote <NODE>

# Remover nó do swarm
docker node rm <NODE>

# Drenar nó (parar serviços e não aceitar novos)
docker node update --availability drain <NODE>

# Ver detalhes do nó
docker node inspect <NODE>
```

## Serviços e Stacks
```bash
# Criar serviço
docker service create --name <SERVICE> <IMAGE>

# Criar serviço com replicas
docker service create --name <SERVICE> --replicas 3 <IMAGE>

# Listar serviços
docker service ls

# Ver detalhes do serviço
docker service inspect <SERVICE>

# Ver logs do serviço
docker service logs <SERVICE>

# Escalar serviço
docker service scale <SERVICE>=5

# Atualizar serviço
docker service update --image <NEW_IMAGE> <SERVICE>

# Remover serviço
docker service rm <SERVICE>

# Deploy de stack (docker-compose)
docker stack deploy -c docker-compose.yml <STACK_NAME>

# Listar stacks
docker stack ls

# Listar serviços da stack
docker stack services <STACK_NAME>

# Remover stack
docker stack rm <STACK_NAME>
```

## Rede no Swarm
```bash
# Criar rede overlay
docker network create --driver overlay <NETWORK>

# Listar redes
docker network ls

# Conectar serviço à rede
docker service create --network <NETWORK> --name <SERVICE> <IMAGE>
```

## Segredos e Configurações
```bash
# Criar secret
echo "senha" | docker secret create <SECRET_NAME> -

# Listar secrets
docker secret ls

# Usar secret em serviço
docker service create --secret <SECRET_NAME> --name <SERVICE> <IMAGE>

# Criar config
echo "config" | docker config create <CONFIG_NAME> -

# Usar config em serviço
docker service create --config <CONFIG_NAME> --name <SERVICE> <IMAGE>
```

## Verificação e Troubleshooting
```bash
# Ver tasks de um serviço
docker service ps <SERVICE>

# Ver nodes com detalhes
docker node ls -q | xargs docker node inspect

# Verificar saúde do swarm
docker node ls

# Forçar saída do swarm (em caso de problemas)
docker swarm leave --force
```

## Comandos Úteis
```bash
# Visualizar serviço no browser
docker service inspect --format='{{.Endpoint.Ports}}' <SERVICE>

# Ver variáveis de ambiente do serviço
docker service inspect --format='{{.Spec.TaskTemplate.ContainerSpec.Env}}' <SERVICE>

# Backup do swarm (configurações)
docker swarm init --force-new-cluster
```

## Dicas Rápidas
- Use `--constraint` para colocar serviços em nós específicos
- Use `--placement-pref` para espalhar réplicas uniformemente
- Monitore com `docker service logs -f <SERVICE>`
- Para produção, sempre use healthchecks nos serviços
- Mantenha número ímpar de managers para consenso
