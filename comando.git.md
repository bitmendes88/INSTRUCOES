# Guia Completo de Comandos Git para Iniciantes

![Git Logo](https://git-scm.com/images/logo@2x.png)

## O que é Git?

Git é um sistema de controle de versão distribuído que permite rastrear alterações no código-fonte durante o desenvolvimento de software. Ele ajuda equipes a colaborarem em projetos, mantendo um histórico completo de todas as modificações.

## Índice

1. [Configuração Inicial](#configuração-inicial)
2. [Criando um Repositório](#criando-um-repositório)
3. [Estados dos Arquivos](#estados-dos-arquivos)
4. [Comandos Básicos](#comandos-básicos)
5. [Trabalhando com Branches](#trabalhando-com-branches)
6. [Trabalhando com Repositórios Remotos](#trabalhando-com-repositórios-remotos)
7. [Desfazendo Alterações](#desfazendo-alterações)
8. [Histórico e Logs](#histórico-e-logs)
9. [Boas Práticas](#boas-práticas)

## Configuração Inicial

Antes de começar a usar o Git, você precisa configurar suas informações pessoais:

```bash
# Configurar nome de usuário
git config --global user.name "Seu Nome"

# Configurar email
git config --global user.email "seu.email@exemplo.com"

# Configurar editor padrão (opcional)
git config --global core.editor "code --wait"  # VS Code

# Verificar configurações
git config --list

# Configurar branch padrão (recomendado para novos projetos)
git config --global init.defaultBranch main
```

## Criando um Repositório

### Inicializando um novo repositório

```bash
# Criar um novo diretório e inicializar o Git
mkdir meu-projeto
cd meu-projeto
git init

# Ou inicializar Git em um diretório existente
cd projeto-existente
git init
```

### Clonando um repositório existente

```bash
# Clonar via HTTPS
git clone https://github.com/usuario/repositorio.git

# Clonar via SSH (requer chave SSH configurada)
git clone git@github.com:usuario/repositorio.git

# Clonar para um diretório específico
git clone https://github.com/usuario/repositorio.git nome-do-diretorio
```

## Estados dos Arquivos

No Git, os arquivos podem estar em três estados principais:

1. **Modificado (Modified)**: Arquivo foi alterado mas não commitado
2. **Preparado (Staged)**: Arquivo foi marcado para ser commitado
3. **Consolidado (Committed)**: Arquivo está seguro no banco de dados local

![Estados do Git](https://git-scm.com/book/en/v2/images/areas.png)

## Comandos Básicos

### Verificando o status

```bash
# Verificar status dos arquivos
git status

# Status resumido
git status -s
```

### Adicionando arquivos

```bash
# Adicionar arquivo específico
git add arquivo.txt

# Adicionar todos os arquivos modificados e novos
git add .

# Adicionar todos os arquivos (incluindo deletados)
git add -A

# Adicionar arquivos interativamente
git add -i
```

### Fazendo commits

```bash
# Commit com mensagem
git commit -m "Mensagem do commit"

# Adicionar e commitar em um comando (apenas para arquivos já rastreados)
git commit -am "Mensagem do commit"

# Alterar o último commit (se não foi enviado para remoto)
git commit --amend
```

### Verificando diferenças

```bash
# Ver diferenças não preparadas
git diff

# Ver diferenças preparadas
git diff --staged

# Ver diferenças entre commits
git diff commit1 commit2
```

## Trabalhando com Branches

Branches permitem trabalhar em funcionalidades isoladas sem afetar o código principal.

```bash
# Listar branches
git branch

# Listar branches remotas também
git branch -a

# Criar nova branch
git branch nova-feature

# Mudar para uma branch
git checkout nova-feature

# Criar e mudar para nova branch
git checkout -b nova-feature

# Renomear branch atual
git branch -m novo-nome

# Deletar branch (após merge)
git branch -d nome-da-branch

# Forçar deleção de branch (mesmo sem merge)
git branch -D nome-da-branch

# Ver branches merged com a atual
git branch --merged

# Ver branches não merged com a atual
git branch --no-merged
```

### Merge e Rebase

```bash
# Fazer merge de uma branch na atual
git merge nome-da-branch

# Fazer rebase da branch atual na main
git checkout minha-branch
git rebase main

# Continuar rebase após resolver conflitos
git rebase --continue

# Abortar rebase
git rebase --abort
```

## Trabalhando com Repositórios Remotos

### Configurando remotos

```bash
# Ver repositórios remotos
git remote -v

# Adicionar repositório remoto
git remote add origin https://github.com/usuario/repositorio.git

# Alterar URL do remote
git remote set-url origin nova-url

# Remover remote
git remote remove origin
```

### Enviando e recebendo alterações

```bash
# Enviar branch para remote
git push origin nome-da-branch

# Enviar e definir upstream (para tracking)
git push -u origin nome-da-branch

# Baixar alterações do remote (sem merge)
git fetch origin

# Baixar e integrar alterações (fetch + merge)
git pull origin nome-da-branch

# Forçar push (use com cuidado!)
git push -f origin nome-da-branch
```

## Desfazendo Alterações

```bash
# Desfazer modificações em arquivo não preparado
git checkout -- arquivo.txt

# Remover arquivo da área de stage (mas manter alterações)
git reset HEAD arquivo.txt

# Reverter um commit (cria novo commit desfazendo as alterações)
git revert hash-do-commit

# Resetar para commit específico (cuidado: altera histórico)
git reset --soft hash-do-commit   # mantém alterações em stage
git reset --mixed hash-do-commit  # mantém alterações não stage (padrão)
git reset --hard hash-do-commit   # descarta todas as alterações
```

## Histórico e Logs

```bash
# Ver histórico de commits
git log

# Histórico resumido (uma linha por commit)
git log --oneline

# Histórico com gráfico de branches
git log --oneline --graph --all

# Histórico com estatísticas de arquivos modificados
git log --stat

# Ver alterações específicas de um commit
git show hash-do-commit

# Procurar no histórico por termo
git log --grep="termo de busca"

# Ver histórico de um arquivo específico
git log -- caminho/do/arquivo
```

## Boas Práticas

### Mensagens de commit

Siga estas diretrizes para mensagens de commit claras e úteis:

1. **Assunto** (até 50 caracteres): Descrição concisa do que foi feito
2. **Corpo** (opcional): Explicação detalhada do porquê das mudanças
3. **Use o imperativo**: "Adiciona funcionalidade" em vez de "Adicionei funcionalidade"

Exemplo:
```
Adiciona autenticação de usuário

- Implementa sistema de login com JWT
- Adiciona middleware de verificação de token
- Cria rotas protegidas para dashboard
```

### Fluxo de trabalho recomendado

1. Sempre faça `git pull` antes de começar a trabalhar
2. Crie branches para novas funcionalidades (`git checkout -b feature/nome`)
3. Faça commits pequenos e frequentes com mensagens claras
4. Teste seu código antes de fazer push
5. Faça review do código antes de merge para a branch principal

### Arquivo .gitignore

Crie um arquivo `.gitignore` para evitar que arquivos desnecessários sejam commitados:

```
# Dependências
node_modules/
vendor/

# Arquivos de ambiente
.env
.env.local

# Logs
*.log
logs/

# Arquivos do sistema
.DS_Store
Thumbs.db

# Arquivos de IDE
.vscode/
.idea/
*.swp
*.swo
```

## Recursos Adicionais

- [Documentação Oficial do Git](https://git-scm.com/doc)
- [GitHub Learning Lab](https://lab.github.com/)
- [Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials)
- [Visual Git Guide](http://marklodato.github.io/visual-git-guide/index-en.html)

## Conclusão

Este guia cobre os comandos essenciais do Git para iniciantes. Pratique regularmente e logo estará usando Git com confiança. Lembre-se: sempre verifique o status com `git status` quando estiver em dúvida sobre o que está acontecendo no seu repositório!

**Dica**: Use `git comando --help` para ver a documentação detalhada de qualquer comando Git.
