# Comandos PostgreSQL - Guia de Referência

## 📋 Índice
1. [Comandos Básicos](#comandos-básicos)
2. [Comandos Básicos para Gerenciamento de Usuários](#Comandos-Básicos-para-Gerenciamento-de-Usuários)
3. [DDL - Definição de Dados](#ddl---definição-de-dados)
4. [DML - Manipulação de Dados](#dml---manipulação-de-dados)
5. [Consultas e Cláusulas](#consultas-e-cláusulas)
6. [Funções](#funções)
7. [Joins](#joins)
8. [Transações](#transações)
9. [Controle de Acesso](#controle-de-acesso)
10. [Utilitários](#utilitários)

---

## Comandos Básicos

### Conectar ao PostgreSQL
```bash
psql -h hostname -p port -U username -d database_name
```

### Comandos no psql
```sql
\l          -- Listar bancos de dados
\c dbname   -- Conectar a um banco
\dt         -- Listar tabelas
\d table    -- Descrever tabela
\du         -- Listar usuários
\q          -- Sair do psql
```
---
## Comandos Básicos para Gerenciamento de Usuários

### 1. Conectar como superusuário
```bash
# Conectar ao PostgreSQL (como postgres ou outro superusuário)
docker exec -it nome_container psql -U postgres -d postgres
```

### 2. Criar um novo usuário
```sql
-- Criar usuário com senha
CREATE USER nome_usuario WITH PASSWORD 'senha_segura';

-- Criar usuário com validade de senha
CREATE USER nome_usuario WITH PASSWORD 'senha_segura' VALID UNTIL '2025-12-31';

-- Criar usuário com opções adicionais
CREATE USER nome_usuario WITH 
  PASSWORD 'senha_segura'
  NOSUPERUSER
  NOCREATEDB
  NOCREATEROLE
  INHERIT
  LOGIN;
```

### 3. Alterar usuário existente
```sql
-- Alterar senha
ALTER USER nome_usuario WITH PASSWORD 'nova_senha';

-- Adicionar permissões
ALTER USER nome_usuario CREATEDB CREATEROLE;

-- Remover permissões
ALTER USER nome_usuario NOCREATEDB NOCREATEROLE;

-- Renomear usuário
ALTER USER nome_antigo RENAME TO novo_nome;

-- Definir data de expiração
ALTER USER nome_usuario VALID UNTIL '2025-12-31';
```

### 4. Excluir usuário
```sql
-- Remover usuário (só funciona se o usuário não tiver objetos)
DROP USER nome_usuario;

-- Remover usuário e todos os seus objetos
DROP USER nome_usuario CASCADE;
```

## 🛡️ Conceder Privilégios (GRANT)

### Privilégios em Bancos de Dados
```sql
-- Conceder todos os privilégios em um banco específico
GRANT ALL PRIVILEGES ON DATABASE nome_banco TO nome_usuario;

-- Conceder conexão a um banco
GRANT CONNECT ON DATABASE nome_banco TO nome_usuario;

-- Conceder criação de banco de dados
ALTER USER nome_usuario CREATEDB;
```

### Privilégios em Esquemas (Schemas)
```sql
-- Conceder uso do schema
GRANT USAGE ON SCHEMA nome_schema TO nome_usuario;

-- Conceder todos os privilégios no schema
GRANT ALL PRIVILEGES ON SCHEMA nome_schema TO nome_usuario;

-- Conceder criação no schema
GRANT CREATE ON SCHEMA nome_schema TO nome_usuario;
```

### Privilégios em Tabelas
```sql
-- Conceder todos os privilégios em todas as tabelas do schema
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA nome_schema TO nome_usuario;

-- Conceder privilégios específicos em uma tabela
GRANT SELECT, INSERT, UPDATE ON TABLE nome_tabela TO nome_usuario;

-- Conceder privilégios em colunas específicas
GRANT SELECT (coluna1, coluna2), UPDATE (coluna1) ON TABLE nome_tabela TO nome_usuario;
```

### Privilégios em Sequências
```sql
-- Conceder uso de sequências
GRANT USAGE, SELECT ON SEQUENCE nome_sequencia TO nome_usuario;

-- Conceder todos os privilégios em todas as sequências
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA nome_schema TO nome_usuario;
```

## 🔍 Revogar Privilégios (REVOKE)

### Revogar privilégios
```sql
-- Revogar todos os privilégios de um usuário
REVOKE ALL PRIVILEGES ON DATABASE nome_banco FROM nome_usuario;

-- Revogar privilégios específicos
REVOKE INSERT, UPDATE ON TABLE nome_tabela FROM nome_usuario;

-- Revogar todos os privilégios em todas as tabelas
REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA nome_schema FROM nome_usuario;
```

## 👥 Grupos e Roles

### Criar e gerenciar grupos
```sql
-- Criar um grupo (role)
CREATE ROLE nome_grupo;

-- Adicionar usuário ao grupo
GRANT nome_grupo TO nome_usuario;

-- Remover usuário do grupo
REVOKE nome_grupo FROM nome_usuario;

-- Conceder privilégios ao grupo
GRANT SELECT ON ALL TABLES IN SCHEMA public TO nome_grupo;
```

## 🎯 Exemplos Práticos Completos

### Exemplo 1: Usuário com acesso somente leitura
```sql
-- Criar usuário leitor
CREATE USER leitor WITH PASSWORD 'senha_leitura';

-- Conceder privilégios
GRANT CONNECT ON DATABASE meu_banco TO leitor;
GRANT USAGE ON SCHEMA public TO leitor;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO leitor;

-- Para futuras tabelas também
ALTER DEFAULT PRIVILEGES IN SCHEMA public 
GRANT SELECT ON TABLES TO leitor;
```

### Exemplo 2: Usuário com acesso completo a um schema específico
```sql
-- Criar usuário desenvolvedor
CREATE USER desenvolvedor WITH PASSWORD 'senha_dev';

-- Conceder privilégios completos no schema app
GRANT CONNECT ON DATABASE meu_banco TO desenvolvedor;
GRANT ALL PRIVILEGES ON SCHEMA app TO desenvolvedor;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA app TO desenvolvedor;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA app TO desenvolvedor;

-- Para futuros objetos também
ALTER DEFAULT PRIVILEGES IN SCHEMA app 
GRANT ALL PRIVILEGES ON TABLES TO desenvolvedor;
```

### Exemplo 3: Usuário administrador de um banco específico
```sql
-- Criar usuário admin
CREATE USER admin_banco WITH PASSWORD 'senha_admin';

-- Tornar proprietário do banco (cuidado com esta permissão)
ALTER DATABASE meu_banco OWNER TO admin_banco;

-- Ou conceder todos os privilégios
GRANT ALL PRIVILEGES ON DATABASE meu_banco TO admin_banco;
GRANT ALL PRIVILEGES ON SCHEMA public TO admin_banco;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO admin_banco;
```

## 📊 Consultar Privilégios Existente

### Verificar privilégios de usuários
```sql
-- Listar todos os usuários
\du

-- Listar privilégios detalhados
SELECT * FROM pg_roles;

-- Ver privilégios em tabelas
SELECT grantee, table_name, privilege_type 
FROM information_schema.table_privileges 
WHERE grantee = 'nome_usuario';

-- Ver privilégios em bancos de dados
SELECT datname, datacl FROM pg_database;

-- Ver privilégios em schemas
SELECT nspname, nspacl FROM pg_namespace;
```

## 🐳 Exemplo com Docker

### Script completo para criar usuário no container
```bash
#!/bin/bash
# create-user.sh

CONTAINER_NAME="meu-postgres"
DB_NAME="meu_banco"
NEW_USER="novo_usuario"
PASSWORD="senha_segura123"

docker exec -it $CONTAINER_NAME psql -U postgres -d postgres << EOF
CREATE USER $NEW_USER WITH PASSWORD '$PASSWORD';
GRANT CONNECT ON DATABASE $DB_NAME TO $NEW_USER;
GRANT USAGE ON SCHEMA public TO $NEW_USER;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO $NEW_USER;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE ON TABLES TO $NEW_USER;
EOF

echo "Usuário $NEW_USER criado com sucesso!"
```

## ⚠️ Boas Práticas e Segurança

### Recomendações importantes
```sql
-- Use senhas fortes
CREATE USER app_user WITH PASSWORD 'S3nh4F0rt3!2024';

-- Limite privilégios ao mínimo necessário
GRANT SELECT ON TABLE relatorios TO usuario_leitura;

-- Revogue privilégios padrão
REVOKE ALL ON DATABASE meu_banco FROM PUBLIC;

-- Use grupos para gerenciamento fácil
CREATE ROLE leitores;
GRANT SELECT ON ALL TABLES TO leitores;
GRANT leitores TO usuario1, usuario2;

-- Defina expiração para usuários temporários
ALTER USER usuario_temporario VALID UNTIL '2024-12-31';
```

### Comando para testar conexão do novo usuário
```bash
# Testar conexão do novo usuário
docker exec -it nome_container psql -U novo_usuario -d nome_banco -c "SELECT current_user;"
```

---

## DDL - Definição de Dados

### Bancos de Dados
```sql
CREATE DATABASE nome_banco;
DROP DATABASE nome_banco;
ALTER DATABASE nome_banco RENAME TO novo_nome;
```

### Tabelas
```sql
CREATE TABLE nome_tabela (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    idade INTEGER CHECK (idade >= 0),
    data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

DROP TABLE nome_tabela;

ALTER TABLE nome_tabela ADD COLUMN nova_coluna VARCHAR(50);
ALTER TABLE nome_tabela DROP COLUMN coluna;
ALTER TABLE nome_tabela RENAME COLUMN antigo_nome TO novo_nome;
ALTER TABLE nome_tabela ALTER COLUMN coluna TYPE novo_tipo;
```

### Índices
```sql
CREATE INDEX idx_nome ON tabela (coluna);
CREATE UNIQUE INDEX idx_email ON usuarios (email);
DROP INDEX idx_nome;
```

### Views
```sql
CREATE VIEW view_name AS SELECT col1, col2 FROM tabela WHERE condição;
DROP VIEW view_name;
```

---

## DML - Manipulação de Dados

### INSERT
```sql
INSERT INTO tabela (col1, col2, col3) VALUES (valor1, valor2, valor3);
INSERT INTO tabela VALUES (valor1, valor2, valor3);
INSERT INTO tabela (col1, col2) SELECT col1, col2 FROM outra_tabela;
```

### UPDATE
```sql
UPDATE tabela SET col1 = novo_valor WHERE condição;
UPDATE tabela SET col1 = valor1, col2 = valor2 WHERE id = 1;
```

### DELETE
```sql
DELETE FROM tabela WHERE condição;
DELETE FROM tabela; -- CUIDADO: remove todos os registros!
TRUNCATE TABLE tabela; -- Mais rápido para limpar tabela
```

---

## Consultas e Cláusulas

### SELECT
```sql
SELECT * FROM tabela;
SELECT col1, col2 FROM tabela;
SELECT DISTINCT coluna FROM tabela;
```

### WHERE
```sql
SELECT * FROM tabela WHERE condição;
SELECT * FROM tabela WHERE coluna = valor;
SELECT * FROM tabela WHERE coluna > valor;
SELECT * FROM tabela WHERE coluna BETWEEN valor1 AND valor2;
SELECT * FROM tabela WHERE coluna IN (valor1, valor2, valor3);
SELECT * FROM tabela WHERE coluna LIKE 'padrao%';
SELECT * FROM tabela WHERE coluna IS NULL;
```

### ORDER BY
```sql
SELECT * FROM tabela ORDER BY coluna ASC;
SELECT * FROM tabela ORDER BY coluna DESC;
SELECT * FROM tabela ORDER BY coluna1 ASC, coluna2 DESC;
```

### LIMIT e OFFSET
```sql
SELECT * FROM tabela LIMIT 10;
SELECT * FROM tabela LIMIT 10 OFFSET 20; -- Página 3 (pula 20 registros)
```

### GROUP BY e HAVING
```sql
SELECT coluna, COUNT(*) FROM tabela GROUP BY coluna;
SELECT coluna, AVG(valor) FROM tabela GROUP BY coluna HAVING AVG(valor) > 100;
```

---

## Funções

### Funções de Agregação
```sql
SELECT COUNT(*) FROM tabela;
SELECT SUM(coluna) FROM tabela;
SELECT AVG(coluna) FROM tabela;
SELECT MIN(coluna) FROM tabela;
SELECT MAX(coluna) FROM tabela;
```

### Funções de String
```sql
SELECT UPPER(nome) FROM tabela;
SELECT LOWER(nome) FROM tabela;
SELECT CONCAT(nome, ' ', sobrenome) AS nome_completo FROM tabela;
SELECT SUBSTRING(nome FROM 1 FOR 5) FROM tabela;
SELECT LENGTH(nome) FROM tabela;
```

### Funções de Data/Hora
```sql
SELECT CURRENT_DATE;
SELECT CURRENT_TIME;
SELECT CURRENT_TIMESTAMP;
SELECT EXTRACT(YEAR FROM data_nascimento) FROM tabela;
SELECT AGE(data_nascimento) FROM tabela;
SELECT NOW() + INTERVAL '1 day';
```

---

## Joins

### INNER JOIN
```sql
SELECT * FROM tabela1
INNER JOIN tabela2 ON tabela1.id = tabela2.tabela1_id;
```

### LEFT JOIN
```sql
SELECT * FROM tabela1
LEFT JOIN tabela2 ON tabela1.id = tabela2.tabela1_id;
```

### RIGHT JOIN
```sql
SELECT * FROM tabela1
RIGHT JOIN tabela2 ON tabela1.id = tabela2.tabela1_id;
```

### FULL JOIN
```sql
SELECT * FROM tabela1
FULL JOIN tabela2 ON tabela1.id = tabela2.tabela1_id;
```

### CROSS JOIN
```sql
SELECT * FROM tabela1 CROSS JOIN tabela2;
```

---

## Transações

```sql
BEGIN; -- Inicia transação

UPDATE conta SET saldo = saldo - 100 WHERE id = 1;
UPDATE conta SET saldo = saldo + 100 WHERE id = 2;

COMMIT; -- Confirma transação
-- ou
ROLLBACK; -- Desfaz transação
```

### Savepoints
```sql
BEGIN;
UPDATE tabela SET coluna = valor1;
SAVEPOINT meu_savepoint;
UPDATE tabela SET coluna = valor2;
ROLLBACK TO SAVEPOINT meu_savepoint;
COMMIT;
```

---

## Controle de Acesso

### Usuários e Roles
```sql
CREATE ROLE nome_role LOGIN PASSWORD 'senha';
ALTER ROLE nome_role WITH SUPERUSER;
DROP ROLE nome_role;
```

### Privilégios
```sql
GRANT SELECT, INSERT ON tabela TO role;
GRANT ALL PRIVILEGES ON DATABASE banco TO role;
REVOKE INSERT ON tabela FROM role;
```

---

## Utilitários

### EXPLAIN
```sql
EXPLAIN SELECT * FROM tabela WHERE coluna = valor;
EXPLAIN ANALYZE SELECT * FROM tabela WHERE coluna = valor;
```

### COPY (Importar/Exportar)
```sql
-- Exportar para CSV
COPY tabela TO '/caminho/arquivo.csv' WITH CSV HEADER;

-- Importar de CSV
COPY tabela FROM '/caminho/arquivo.csv' WITH CSV HEADER;
```

### Backup e Restore
```bash
# Terminal - Backup
pg_dump -U usuario -h host banco > backup.sql

# Terminal - Restore
psql -U usuario -h host banco < backup.sql
```

---

## Extensões Úteis

```sql
-- Instalar extensão
CREATE EXTENSION nome_extensão;

-- Extensões comuns
CREATE EXTENSION pgcrypto;  -- Funções criptográficas
CREATE EXTENSION uuid-ossp; -- Geração de UUIDs
CREATE EXTENSION postgis;   -- Dados geográficos
```

---

## ⚠️ Dicas e Boas Práticas

1. **Sempre use WHERE em UPDATE/DELETE** para evitar modificar dados acidentalmente
2. **Use transações** para operações críticas
3. **Faça backups regularmente**
4. **Use índices** em colunas frequentemente consultadas
5. **Teste consultas com EXPLAIN** para otimizar performance

---

## 📚 Recursos Adicionais

- [Documentação Oficial PostgreSQL](https://www.postgresql.org/docs/)
- [PSQL Command Cheat Sheet](https://www.postgresql.org/docs/current/app-psql.html)

*Este documento foi gerado para PostgreSQL 13+. Alguns comandos podem variar entre versões.*
