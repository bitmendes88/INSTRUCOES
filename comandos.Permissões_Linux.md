# Comandos de Permissões no Ubuntu

## Introdução

No Ubuntu (e sistemas Linux em geral), as permissões de arquivos e diretórios são fundamentais para a segurança do sistema. Elas controlam quem pode ler, escrever e executar arquivos, garantindo que apenas usuários autorizados tenham acesso a recursos específicos.

## Estrutura Básica de Permissões

As permissões no Linux são divididas em três categorias:

1. **Permissões do proprietário (owner)** - usuário dono do arquivo
2. **Permissões do grupo (group)** - grupo associado ao arquivo
3. **Permissões de outros (others)** - todos os demais usuários

Cada categoria pode ter três tipos de permissão:
- **r (read)** - Leitura
- **w (write)** - Escrita
- **x (execute)** - Execução

## Visualizando Permissões

### Comando `ls -l`

```bash
ls -l arquivo.txt
```

Exemplo de saída:
```
-rwxr-xr-- 1 usuario grupo 2048 Jan 10 10:30 arquivo.txt
```

A string de permissões (`-rwxr-xr--`) é dividida assim:
- Primeiro caractere: tipo de arquivo (`-` para arquivo regular, `d` para diretório)
- Próximos 3: permissões do proprietário (`rwx`)
- Próximos 3: permissões do grupo (`r-x`)
- Últimos 3: permissões de outros (`r--`)

## Alterando Permissões

### Comando `chmod` (Change Mode)

**Sintaxe:**
```bash
chmod [opções] modo arquivo
```

#### Método Numérico (Octal)

Cada permissão tem um valor numérico:
- r (read) = 4
- w (write) = 2
- x (execute) = 1

Exemplos:
```bash
# Permissões: proprietário (rwx), grupo (r-x), outros (r--)
chmod 754 arquivo.txt

# Permissões: proprietário (rw-), grupo (r--), outros (r--)
chmod 644 arquivo.txt

# Permissões completas para todos
chmod 777 arquivo.txt

# Apenas proprietário tem acesso total
chmod 700 arquivo.txt
```

#### Método Simbólico

Usa símbolos para modificar permissões:
- `u` = usuário/proprietário
- `g` = grupo
- `o` = outros
- `a` = todos
- `+` = adiciona permissão
- `-` = remove permissão
- `=` = define permissão exata

Exemplos:
```bash
# Adiciona permissão de execução para todos
chmod a+x script.sh

# Remove permissão de escrita do grupo
chmod g-w arquivo.txt

# Define permissões exatas: proprietário (rwx), grupo (r-x), outros (---)
chmod u=rwx,g=rx,o= script.sh

# Adiciona permissão de leitura para grupo e outros
chmod go+r arquivo.conf
```

### Comando `chown` (Change Owner)

Altera o proprietário e grupo de um arquivo/diretório.

**Sintaxe:**
```bash
chown [opções] proprietário:grupo arquivo
```

Exemplos:
```bash
# Altera apenas o proprietário
chown usuario arquivo.txt

# Altera proprietário e grupo
chown usuario:grupo arquivo.txt

# Altera apenas o grupo
chown :novogrupo arquivo.txt

# Altera recursivamente (para diretórios)
chown -R usuario:grupo pasta/
```

### Comando `chgrp` (Change Group)

Altera apenas o grupo de um arquivo/diretório.

**Sintaxe:**
```bash
chgrp [opções] grupo arquivo
```

Exemplo:
```bash
chgrp novogrupo arquivo.txt
chgrp -R novogrupo pasta/
```

## Permissões Especiais

### SUID (Set User ID)

Quando aplicado a um executável, faz com que seja executado com as permissões do proprietário.

```bash
# Define SUID (representado como 's' na permissão do proprietário)
chmod u+s arquivo
# ou
chmod 4755 arquivo
```

### SGID (Set Group ID)

Para executáveis: executa com permissões do grupo.
Para diretórios: arquivos criados herdam o grupo do diretório.

```bash
# Define SGID (representado como 's' na permissão do grupo)
chmod g+s arquivo
# ou
chmod 2755 arquivo
```

### Sticky Bit

Usado em diretórios para permitir que apenas o proprietário do arquivo possa renomear ou excluir seus próprios arquivos.

```bash
# Define sticky bit (representado como 't' nas permissões de outros)
chmod +t diretorio
# ou
chmod 1755 diretorio
```

## Umask

A umask define as permissões padrão para novos arquivos e diretórios.

**Verificar umask atual:**
```bash
umask
```

**Definir umask:**
```bash
umask 022  # Permissões padrão: arquivos (644), diretórios (755)
```

## Exemplos Práticos Comuns

```bash
# Script executável apenas pelo proprietário
chmod 700 meu_script.sh

# Arquivo de configuração legível por todos, editável apenas pelo proprietário
chmod 644 config.conf

# Diretório compartilhado onde todos podem adicionar arquivos
chmod 1777 /compartilhado  # Com sticky bit

# Dar permissão de execução a um script para todos os usuários
chmod a+x script.py

# Alterar proprietário e dar permissões completas apenas a ele
chown usuario:usuario arquivo.conf
chmod 600 arquivo.conf
```

## Dicas de Segurança

1. **Princípio do menor privilégio**: conceda apenas as permissões necessárias
2. **Evite usar `chmod 777`** - é uma prática perigosa de segurança
3. **Use `chmod -R` com cuidado** - alterações recursivas podem afetar muitos arquivos
4. **Para scripts, `chmod +x`** é necessário para executá-los
5. **Verifique permissões regularmente** com `ls -l`

## Conclusão

O entendimento e uso adequado das permissões no Ubuntu é essencial para manter um sistema seguro e funcionando corretamente. Pratique esses comandos em um ambiente seguro antes de aplicá-los em sistemas de produção.
