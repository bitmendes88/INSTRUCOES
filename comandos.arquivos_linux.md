# Guia Completo: Comandos de Gerenciamento de Arquivos e Pastas no Ubuntu

## Índice
- [Introdução](#introdução)
- [Navegação entre Pastas](#navegação-entre-pastas)
- [Listagem de Conteúdo](#listagem-de-conteúdo)
- [Criação de Arquivos e Pastas](#criação-de-arquivos-e-pastas)
- [Cópia de Arquivos e Pastas](#cópia-de-arquivos-e-pastas)
- [Movimento e Renomeação](#movimento-e-renomeação)
- [Exclusão de Arquivos e Pastas](#exclusão-de-arquivos-e-pastas)
- [Visualização de Conteúdo](#visualização-de-conteúdo)
- [Permissões e Propriedade](#permissões-e-propriedade)
- [Busca de Arquivos](#busca-de-arquivos)
- [Espaço em Disco](#espaço-em-disco)
- [Links Simbólicos e Físicos](#links-simbólicos-e-físicos)
- [Compactação e Descompactação](#compactação-e-descompactação)
- [Dicas e Boas Práticas](#dicas-e-boas-práticas)

## Introdução

O Ubuntu, como distribuição Linux, utiliza comandos de terminal para gerenciamento eficiente de arquivos e pastas. Este guia aborda os principais comandos e suas aplicações.

## Navegação entre Pastas

### `pwd` - Mostrar diretório atual
```bash
pwd
```
Exibe o caminho completo do diretório em que você está atualmente.

### `cd` - Mudar de diretório
```bash
cd /caminho/para/diretorio  # Vai para o diretório especificado
cd ~                       # Vai para o diretório home
cd ..                      # Volta um nível
cd -                       # Volta ao diretório anterior
cd                         # Vai para o diretório home
```

## Listagem de Conteúdo

### `ls` - Listar arquivos e pastas
```bash
ls                 # Lista simples
ls -l              # Lista detalhada (formato longo)
ls -a              # Mostra arquivos ocultos
ls -la             # Combinação dos dois anteriores
ls -lh             # Lista com tamanhos legíveis (KB, MB, GB)
ls -t              # Ordena por data de modificação (mais recente primeiro)
ls -R              # Lista recursivamente subdiretórios
ls --color=auto    # Lista com cores para diferentes tipos de arquivos
```

## Criação de Arquivos e Pastas

### `mkdir` - Criar diretórios
```bash
mkdir pasta                      # Cria uma pasta
mkdir -p pasta/subpasta         # Cria estrutura de pastas com subpastas
mkdir "pasta com espacos"       # Cria pasta com espaços no nome
```

### `touch` - Criar arquivos vazios
```bash
touch arquivo.txt               # Cria um arquivo vazio
touch arquivo1.txt arquivo2.txt # Cria múltiplos arquivos
```

## Cópia de Arquivos e Pastas

### `cp` - Copiar arquivos e diretórios
```bash
cp origem.txt destino.txt          # Copia um arquivo
cp -r pasta/ nova_pasta/          # Copia diretório recursivamente
cp -v arquivo.txt destino/        # Copia com modo verboso
cp -i arquivo.txt destino/        # Pergunta antes de sobrescrever
cp -u origem.txt destino.txt      # Copia apenas se origem for mais recente
cp *.txt destino/                 # Copia todos os arquivos .txt
```

## Movimento e Renomeação

### `mv` - Mover ou renomear arquivos e pastas
```bash
mv arquivo.txt novo_nome.txt      # Renomeia um arquivo
mv arquivo.txt pasta/             # Move para outra pasta
mv -i arquivo.txt pasta/          # Pergunta antes de sobrescrever
mv -v arquivo.txt pasta/          # Move com modo verboso
mv *.txt pasta/                   # Move todos os arquivos .txt
```

## Exclusão de Arquivos e Pastas

### `rm` - Remover arquivos
```bash
rm arquivo.txt                    # Remove um arquivo
rm -i arquivo.txt                 # Remove com confirmação
rm -f arquivo.txt                 # Remove forçadamente (sem confirmação)
rm *.txt                         # Remove todos os arquivos .txt
```

### `rmdir` - Remover diretórios vazios
```bash
rmdir pasta/                      # Remove pasta vazia
```

### `rm -r` - Remover diretórios com conteúdo
```bash
rm -r pasta/                     # Remove pasta e todo seu conteúdo
rm -rf pasta/                    # Remove forçadamente (CUIDADO!)
```

⚠️ **ATENÇÃO**: O comando `rm -rf` é extremamente perigoso. Use com cautela, especialmente com `sudo`.

## Visualização de Conteúdo

### `cat` - Concatenar e exibir arquivos
```bash
cat arquivo.txt                  # Exibe todo o conteúdo
cat -n arquivo.txt              # Exibe com numeração de linhas
cat arquivo1.txt arquivo2.txt   # Concatena múltiplos arquivos
```

### `less` e `more` - Visualização paginada
```bash
less arquivo.txt                # Visualiza com navegação (setas, Page Up/Down)
more arquivo.txt               # Visualiza página por página
```

### `head` e `tail` - Exibir início ou fim do arquivo
```bash
head arquivo.txt                # Mostra as primeiras 10 linhas
head -n 20 arquivo.txt         # Mostra as primeiras 20 linhas
tail arquivo.txt                # Mostra as últimas 10 linhas
tail -n 15 arquivo.txt         # Mostra as últimas 15 linhas
tail -f arquivo.log            # Monitora arquivo em tempo real (útil para logs)
```

## Permissões e Propriedade

### `chmod` - Alterar permissões
```bash
chmod +x script.sh             # Torna o arquivo executável
chmod 755 arquivo.txt          # Define permissões com notação octal
chmod u+rwx,g+rx,o+r arquivo.txt # Define permissões por categoria
chmod -R 755 pasta/            # Altera permissões recursivamente
```

### `chown` - Alterar proprietário
```bash
chown usuario arquivo.txt       # Muda o proprietário do arquivo
chown usuario:grupo arquivo.txt # Muda proprietário e grupo
chown -R usuario pasta/        # Altera recursivamente
```

### `chgrp` - Alterar grupo
```bash
chgrp grupo arquivo.txt         # Muda o grupo do arquivo
```

## Busca de Arquivos

### `find` - Buscar arquivos e diretórios
```bash
find . -name "*.txt"           # Busca por arquivos .txt no diretório atual
find /home -name "arquivo*"    # Busca a partir do diretório /home
find . -type f -name "*.txt"   # Busca apenas arquivos (não pastas)
find . -size +10M              # Busca arquivos maiores que 10MB
find . -mtime -7               # Busca arquivos modificados nos últimos 7 dias
find . -empty                  # Busca arquivos e pastas vazios
```

### `locate` - Buscar rapidamente (usa banco de dados)
```bash
locate arquivo.txt             # Busca rápida (atualize o banco com `sudo updatedb`)
```

### `which` e `whereis` - Localizar executáveis
```bash
which python                   # Mostra o caminho do executável
whereis python                 # Mostra executável, código-fonte e página man
```

## Espaço em Disco

### `df` - Espaço livre em discos
```bash
df -h                          # Mostra espaço em disco de forma legível
df -i                          # Mostra informações sobre inodes
```

### `du` - Uso de espaço por arquivos e pastas
```bash
du -sh pasta/                  # Mostra uso total de espaço da pasta
du -h --max-depth=1            # Mostra uso de espaço com profundidade 1
du -ah pasta/ | sort -rh | head -n 10 # Top 10 maiores arquivos/pastas
```

## Links Simbólicos e Físicos

### `ln` - Criar links
```bash
ln -s alvo link_simbolico      # Cria link simbólico
ln alvo link_fisico            # Cria link físico (hard link)
```

## Compactação e Descompactação

### `tar` - Arquivar e compactar
```bash
tar -cvf arquivo.tar pasta/    # Cria arquivo tar
tar -xvf arquivo.tar           # Extrai arquivo tar
tar -czvf arquivo.tar.gz pasta/ # Cria tar compactado com gzip
tar -xzvf arquivo.tar.gz       # Extrai tar.gz
tar -cjvf arquivo.tar.bz2 pasta/ # Cria tar compactado com bzip2
tar -xjvf arquivo.tar.bz2      # Extrai tar.bz2
```

### `gzip`/`gunzip` e `bzip2`/`bunzip2` - Compactação
```bash
gzip arquivo.txt               # Compacta para arquivo.txt.gz
gunzip arquivo.txt.gz          # Descompacta
bzip2 arquivo.txt              # Compacta para arquivo.txt.bz2
bunzip2 arquivo.txt.bz2        # Descompacta
```

### `zip`/`unzip` - Compactação ZIP
```bash
zip arquivo.zip arquivo.txt    # Compacta para ZIP
zip -r pasta.zip pasta/        # Compacta pasta recursivamente
unzip arquivo.zip              # Descompacta
```

## Dicas e Boas Práticas

1. **Use o Tab para autocompletar**: Pressione Tab para completar nomes de arquivos e pastas
2. **Use aspas para nomes com espaços**: `cd "pasta com espaços"`
3. **Cuidado com `rm -rf`**: Sempre verifique o caminho antes de executar
4. **Use `-i` para confirmação**: `rm -i`, `cp -i`, `mv -i` para operações críticas
5. **Backup regular**: Sempre mantenha backups importantes antes de operações em massa
6. **Use histórico de comandos**: Pressione ↑ para recuperar comandos anteriores
7. **Redirecione saídas**: Use `>` e `>>` para salvar saídas de comandos em arquivos

### Exemplos úteis:
```bash
# Criar backup de uma pasta
tar -czvf backup_$(date +%Y%m%d).tar.gz /caminho/da/pasta

# Encontrar e excluir arquivos temporários
find . -name "*.tmp" -type f -delete

# Monitorar crescimento de arquivo de log
tail -f /var/log/syslog

# Copiar preservando permissões e timestamps
cp -p arquivo.txt destino/
```

Este guia cobre os comandos essenciais para gerenciamento de arquivos e pastas no Ubuntu. Pratique em um ambiente seguro antes de usar em produção!
