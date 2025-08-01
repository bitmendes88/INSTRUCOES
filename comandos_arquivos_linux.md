# 📁 Comandos Linux para Gestão de Arquivos e Pastas

Este guia apresenta os principais comandos do terminal Linux relacionados à criação, movimentação, cópia, exclusão e manipulação de permissões de arquivos e pastas — tanto localmente quanto em ambiente cliente-servidor (SSH, SCP, etc.).

---

## 🛠️ Criação

### Criar um diretório
```bash
mkdir nome_do_diretorio
```
Cria um diretório vazio.

### Criar um diretório com subdiretórios
```bash
mkdir -p pai/filho/nieto
```
Cria diretórios aninhados.

### Criar um arquivo vazio
```bash
touch nome_do_arquivo.txt
```
Cria um novo arquivo em branco.

---

## ✏️ Renomear ou Mover

### Mover ou renomear arquivo ou diretório
```bash
mv antigo.txt novo.txt
```
Renomeia `antigo.txt` para `novo.txt`.

```bash
mv arquivo.txt /caminho/para/destino/
```
Move o arquivo para outra pasta.

---

## 📋 Cópia

### Copiar arquivos
```bash
cp arquivo.txt copia.txt
```
Cria uma cópia do arquivo.

### Copiar diretório recursivamente
```bash
cp -r pasta1/ pasta2/
```
Copia todos os arquivos e subpastas de `pasta1` para `pasta2`.

---

## ❌ Remoção

### Remover arquivo
```bash
rm arquivo.txt
```

### Remover diretório vazio
```bash
rmdir nome_da_pasta
```

### Remover diretório com arquivos
```bash
rm -r nome_da_pasta
```
Use com **cuidado**.

---

## 🔐 Permissões e Propriedades

### Ver permissões
```bash
ls -l
```

### Alterar permissões
```bash
chmod +x script.sh
```
Adiciona permissão de execução.

```bash
chmod 755 arquivo.sh
```
Define permissões específicas (rwxr-xr-x).

### Alterar dono (usuário e grupo)
```bash
chown usuario:grupo arquivo.txt
```

---

## 🔍 Localização e Pesquisa

### Listar arquivos
```bash
ls
```

### Listar com detalhes e arquivos ocultos
```bash
ls -la
```

### Buscar arquivos por nome
```bash
find /caminho -name "arquivo.txt"
```

### Buscar dentro dos arquivos
```bash
grep "palavra" arquivo.txt
```

---

## 🧭 Navegação

```bash
cd /caminho/desejado   # entra em diretório
cd ..                  # volta um nível
pwd                    # mostra caminho atual
```

---

## 🌐 Cliente/Servidor (SSH, SCP, SFTP)

### Acessar servidor remoto via SSH
```bash
ssh usuario@ip_do_servidor
```

### Copiar arquivo do cliente para o servidor
```bash
scp arquivo.txt usuario@ip:/caminho/no/servidor/
```

### Copiar arquivo do servidor para o cliente
```bash
scp usuario@ip:/caminho/arquivo.txt ./
```

### Usar SFTP (modo interativo)
```bash
sftp usuario@ip
```
Dentro do SFTP:
```bash
put arquivo.txt        # Envia arquivo
get arquivo.txt        # Baixa arquivo
ls                     # Lista arquivos no servidor
lcd /caminho/local     # Muda pasta local
```

---

## 🧪 Dicas úteis

### Ver tamanho de pasta
```bash
du -sh nome_da_pasta/
```

### Ver espaço em disco
```bash
df -h
```

### Exibir conteúdo de um arquivo
```bash
cat arquivo.txt
less arquivo.txt
```

---

> 📝 **Dica final:** use `man comando` para ver o manual de qualquer comando no terminal.  
> Exemplo:
```bash
man cp
```

---
