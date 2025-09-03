# Comandos PostgreSQL - Guia de Referência

## 📋 Índice
1. [Comandos Básicos](#comandos-básicos)
2. [Comandos Para Consultas de Administração](#Comandos-Para-Consultas-de-Administração)
3. [Comandos Básicos para Gerenciamento de Usuários](#Comandos-Básicos-para-Gerenciamento-de-Usuários)
4. [Schemas no PostgreSQL](#Schemas-no-PostgreSQL.)
5. [DDL - Definição de Dados](#ddl---definição-de-dados)
6. [DML - Manipulação de Dados](#dml---manipulação-de-dados)
7. [Consultas e Cláusulas](#consultas-e-cláusulas)
8. [Funções](#funções)
9. [Joins](#joins)
10. [Transações](#transações)
11. [Controle de Acesso](#controle-de-acesso)
12. [Utilitários](#utilitários)

---

## Comandos Básicos

### Conectar ao PostgreSQL
```bash
psql -h hostname -p port -U username -d database_name
```
### Criação de usuários e privilégios
```bash
-- Criar usuário para aplicação web
CREATE USER web_app_user WITH 
    PASSWORD 'Str0ngP@ssw0rd!2024'
    NOSUPERUSER
    NOCREATEDB
    NOCREATEROLE
    NOINHERIT
    LOGIN
    CONNECTION LIMIT 50; -- Limite de conexões simultâneas

-- Conceder privilégios
GRANT CONNECT ON DATABASE my_production_db TO web_app_user;
GRANT USAGE ON SCHEMA public TO web_app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO web_app_user;
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO web_app_user;

-- Privilégios padrão para objetos futuros
ALTER DEFAULT PRIVILEGES IN SCHEMA public 
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO web_app_user;

ALTER DEFAULT PRIVILEGES IN SCHEMA public 
GRANT USAGE ON SEQUENCES TO web_app_user;
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
##Comandos Para Consultas de Administração

# Comandos PostgreSQL para Consultas de Administração

## Conexão e Informações Básicas

```sql
-- Conectar ao PostgreSQL
psql -U username -d database_name -h host -p port

-- Ver versão do PostgreSQL
SELECT version();

-- Ver data e hora atual
SELECT now();

-- Informações sobre a conexão atual
SELECT current_user, current_database(), inet_client_addr(), inet_client_port();
```

## Consulta de Usuários e Roles

```sql
-- Listar todos os usuários/roles
SELECT * FROM pg_roles;

-- Listar usuários com login
SELECT usename AS username, usesuper AS is_superuser, usecreatedb AS can_create_db 
FROM pg_user;

-- Informações detalhadas sobre um usuário específico
SELECT * FROM pg_user WHERE usename = 'nome_do_usuario';

-- Ver privilégios de um usuário
SELECT * FROM pg_default_acl WHERE defacluser = (SELECT oid FROM pg_roles WHERE rolname = 'nome_do_usuario');
```

## Consulta de Bancos de Dados

```sql
-- Listar todos os bancos de dados
SELECT datname, datistemplate, datallowconn, datconnlimit, pg_size_pretty(pg_database_size(datname)) as size
FROM pg_database
ORDER BY datname;

-- Informações detalhadas de um banco específico
SELECT * FROM pg_database WHERE datname = 'nome_do_banco';

-- Tamanho de todos os bancos
SELECT datname, pg_size_pretty(pg_database_size(datname)) as size
FROM pg_database
ORDER BY pg_database_size(datname) DESC;

-- Conexões ativas por banco
SELECT datname, count(*) as connections
FROM pg_stat_activity 
WHERE datname IS NOT NULL 
GROUP BY datname;
```

## Consulta de Schemas

```sql
-- Listar todos os schemas
SELECT nspname AS schema_name, nspowner::regrole AS owner, nspacl AS privileges
FROM pg_namespace
ORDER BY nspname;

-- Schemas do usuário atual
SELECT schema_name 
FROM information_schema.schemata 
ORDER BY schema_name;

-- Ver objetos por schema
SELECT nspname AS schema, 
       count(*) FILTER (WHERE relkind = 'r') AS tables,
       count(*) FILTER (WHERE relkind = 'v') AS views,
       count(*) FILTER (WHERE relkind = 'i') AS indexes,
       count(*) FILTER (WHERE relkind = 'S') AS sequences
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
GROUP BY nspname
ORDER BY nspname;
```

## Consulta de Privilégios

```sql
-- Privilégios em tabelas
SELECT grantee, table_schema, table_name, privilege_type
FROM information_schema.table_privileges
WHERE grantee = 'nome_do_usuario'
ORDER BY table_schema, table_name;

-- Privilégios em schemas
SELECT nspname AS schema, rolname AS grantee, privilege_type
FROM pg_namespace
CROSS JOIN pg_roles
CROSS JOIN unnest(ARRAY['USAGE','CREATE']) AS privilege_type
WHERE has_schema_privilege(rolname, nspname, privilege_type)
ORDER BY nspname, rolname;

-- Privilégios de usuários em bancos
SELECT datname AS database, rolname AS username, 
       CASE 
           WHEN has_database_privilege(rolname, datname, 'CONNECT') THEN 'CONNECT' 
           ELSE 'NO ACCESS' 
       END AS access
FROM pg_database, pg_roles
WHERE rolname NOT LIKE 'pg_%'
ORDER BY datname, rolname;

-- Verificar privilégios específicos de um usuário
SELECT has_schema_privilege('usuario', 'schema', 'USAGE');
SELECT has_table_privilege('usuario', 'tabela', 'SELECT');
SELECT has_database_privilege('usuario', 'banco', 'CONNECT');
```

## Consulta de Tabelas e Estrutura

```sql
-- Listar todas as tabelas
SELECT schemaname, tablename, tableowner, tablespace
FROM pg_tables
ORDER BY schemaname, tablename;

-- Listar tabelas de um schema específico
SELECT tablename 
FROM pg_tables 
WHERE schemaname = 'nome_do_schema'
ORDER BY tablename;

-- Estrutura de uma tabela
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_schema = 'nome_do_schema' AND table_name = 'nome_da_tabela'
ORDER BY ordinal_position;

-- Tamanho das tabelas
SELECT schemaname, relname AS table_name,
       pg_size_pretty(pg_total_relation_size(relid)) AS total_size,
       pg_size_pretty(pg_relation_size(relid)) AS table_size,
       pg_size_pretty(pg_total_relation_size(relid) - pg_relation_size(relid)) AS index_size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC;
```

## Consulta de Índices

```sql
-- Listar índices de uma tabela
SELECT indexname, indexdef
FROM pg_indexes
WHERE tablename = 'nome_da_tabela' AND schemaname = 'nome_do_schema'
ORDER BY indexname;

-- Índices e estatísticas
SELECT schemaname, tablename, indexname, 
       idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_all_indexes
WHERE schemaname = 'nome_do_schema'
ORDER BY idx_scan DESC;
```

## Consulta de Views

```sql
-- Listar todas as views
SELECT schemaname, viewname, viewowner, definition
FROM pg_views
ORDER BY schemaname, viewname;

-- Views de um schema específico
SELECT viewname 
FROM pg_views 
WHERE schemaname = 'nome_do_schema'
ORDER BY viewname;
```

## Consulta de Funções e Procedures

```sql
-- Listar funções
SELECT n.nspname as schema, p.proname as function, 
       pg_get_function_arguments(p.oid) as arguments,
       pg_get_function_result(p.oid) as returns
FROM pg_proc p
JOIN pg_namespace n ON p.pronamespace = n.oid
WHERE n.nspname NOT IN ('pg_catalog', 'information_schema')
ORDER BY n.nspname, p.proname;

-- Informações detalhadas de uma função
SELECT prosrc AS source_code, prolang::regclass AS language, 
       prosecdef AS security_definer, provolatile AS volatility
FROM pg_proc
WHERE proname = 'nome_da_funcao';
```

## Consulta de Sequences

```sql
-- Listar sequences
SELECT sequence_schema, sequence_name, data_type, 
       start_value, minimum_value, maximum_value, increment
FROM information_schema.sequences
ORDER BY sequence_schema, sequence_name;

-- Valor atual de uma sequence
SELECT last_value FROM nome_da_sequence;
```

## Consulta de Regras (Rules)

```sql
-- Listar regras
SELECT schemaname, tablename, rulename, definition
FROM pg_rules
ORDER BY schemaname, tablename, rulename;

-- Regras de uma tabela específica
SELECT rulename, definition
FROM pg_rules
WHERE tablename = 'nome_da_tabela' AND schemaname = 'nome_do_schema';
```

## Consulta de Triggers

```sql
-- Listar triggers
SELECT event_object_schema, event_object_table, trigger_name, 
       event_manipulation, action_statement, action_timing
FROM information_schema.triggers
ORDER BY event_object_schema, event_object_table, trigger_name;

-- Triggers de uma tabela
SELECT trigger_name, action_timing, event_manipulation, action_statement
FROM information_schema.triggers
WHERE event_object_table = 'nome_da_tabela' AND event_object_schema = 'nome_do_schema';
```

## Consulta de Conexões Ativas

```sql
-- Conexões ativas
SELECT pid, usename, application_name, client_addr, client_port, 
       backend_start, state, query
FROM pg_stat_activity
WHERE state IS NOT NULL;

-- Conexões por usuário
SELECT usename, count(*) as connections, 
       array_agg(DISTINCT datname) as databases
FROM pg_stat_activity 
WHERE usename IS NOT NULL 
GROUP BY usename;

-- Matar uma conexão específica
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid = 12345;
```

## Consulta de Estatísticas e Performance

```sql
-- Estatísticas de tabelas
SELECT schemaname, relname, seq_scan, seq_tup_read, 
       idx_scan, idx_tup_fetch, n_tup_ins, n_tup_upd, n_tup_del
FROM pg_stat_all_tables
ORDER BY seq_scan DESC;

-- Estatísticas de bancos
SELECT datname, numbackends, xact_commit, xact_rollback, 
       blks_read, blks_hit, tup_returned, tup_fetched, tup_inserted
FROM pg_stat_database;

-- Cache hit ratio
SELECT datname, 
       round(blks_hit::numeric / (blks_hit + blks_read) * 100, 2) as cache_hit_ratio
FROM pg_stat_database 
WHERE (blks_hit + blks_read) > 0;
```

## Consulta de Locks

```sql
-- Locks ativos
SELECT locktype, relation::regclass, mode, granted, pid, usename
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE relation IS NOT NULL;

-- Detectando deadlocks
SELECT blocked_locks.pid AS blocked_pid,
       blocking_locks.pid AS blocking_pid,
       blocked_activity.query AS blocked_query,
       blocking_activity.query AS blocking_query
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.DATABASE IS NOT DISTINCT FROM blocked_locks.DATABASE
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

## Consulta de Configurações

```sql
-- Configurações do servidor
SELECT name, setting, unit, context, vartype, source
FROM pg_settings
ORDER BY name;

-- Configurações modificáveis
SELECT name, setting, short_desc
FROM pg_settings
WHERE context IN ('user', 'superuser')
ORDER BY name;
```

## Consulta de Extensões

```sql
-- Listar extensões instaladas
SELECT extname, extversion, nspname
FROM pg_extension e
JOIN pg_namespace n ON e.extnamespace = n.oid
ORDER BY extname;

-- Extensões disponíveis
SELECT name, default_version, installed_version, comment
FROM pg_available_extensions
ORDER BY name;
```

## Consulta de Tablespaces

```sql
-- Listar tablespaces
SELECT spcname, pg_get_userbyid(spcowner) as owner, 
       pg_tablespace_location(oid) as location,
       spcacl as privileges
FROM pg_tablespace
ORDER BY spcname;

-- Tamanho dos tablespaces
SELECT spcname, 
       pg_size_pretty(pg_tablespace_size(oid)) as size
FROM pg_tablespace
ORDER BY pg_tablespace_size(oid) DESC;
```

## Dicas Úteis

```sql
-- Usar \x para formato expandido no psql
\x on

-- Usar \timing para ver tempo de execução
\timing on

-- Limitar número de linhas
SELECT * FROM tabela LIMIT 10;

-- Ver plano de execução
EXPLAIN ANALYZE SELECT * FROM tabela WHERE condicao;

-- Exportar resultados para arquivo
\o resultado.txt
SELECT * FROM tabela;
\o
```

Estes comandos fornecem uma base abrangente para administração e monitoramento de bancos PostgreSQL. Lembre-se de sempre testar comandos de modificação em ambientes de desenvolvimento antes de usar em produção.
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
## Schemas no PostgreSQL.

### 1. Criando um Schema Básico

```sql
-- Sintaxe básica
CREATE SCHEMA nome_do_schema;

-- Exemplo
CREATE SCHEMA vendas;
```

### 2. Criando Schema com Autorização

```sql
-- Criar schema atribuindo a um usuário específico
CREATE SCHEMA rh AUTHORIZATION usuario_rh;

-- Verificar usuários existentes
SELECT usename FROM pg_user;
```

### 3. Criando Schema com Comentário

```sql
CREATE SCHEMA financeiro;
COMMENT ON SCHEMA financeiro IS 'Schema para tabelas do departamento financeiro';
```

### 4. Criando Tabelas Dentro de Schemas

```sql
-- Criar tabela em schema específico
CREATE TABLE vendas.pedidos (
    id SERIAL PRIMARY KEY,
    cliente_id INTEGER,
    data_pedido DATE,
    valor_total DECIMAL(10,2)
);

-- Criar outra tabela no mesmo schema
CREATE TABLE vendas.clientes (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100),
    email VARCHAR(100)
);
```

### 5. Consultando Schemas Existentes

```sql
-- Listar todos os schemas
SELECT schema_name 
FROM information_schema.schemata 
ORDER BY schema_name;

-- Schemas do sistema e usuário
SELECT nspname AS schema_name, 
       nspowner::regrole AS owner,
       obj_description(oid, 'pg_namespace') AS description
FROM pg_namespace
ORDER BY nspname;
```

### 6. Permissões e Privilégios

```sql
-- Conceder permissões em um schema
GRANT USAGE ON SCHEMA vendas TO usuario_leitura;
GRANT SELECT ON ALL TABLES IN SCHEMA vendas TO usuario_leitura;

-- Conceder todos os privilégios
GRANT ALL PRIVILEGES ON SCHEMA vendas TO usuario_admin;
```

### 7. Alterando Schemas

```sql
-- Renomear schema
ALTER SCHEMA vendas RENAME TO comercial;

-- Alterar proprietário
ALTER SCHEMA rh OWNER TO novo_proprietario;
```

### 8. Excluindo Schemas

```sql
-- Excluir schema vazio
DROP SCHEMA nome_do_schema;

-- Excluir schema com todas as tabelas (CUIDADO!)
DROP SCHEMA nome_do_schema CASCADE;
```

### 9. Exemplo Prático Completo

```sql
-- Criar schema para e-commerce
CREATE SCHEMA ecommerce AUTHORIZATION admin_user;

-- Criar tabelas no schema
CREATE TABLE ecommerce.produtos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(200) NOT NULL,
    preco DECIMAL(10,2),
    estoque INTEGER DEFAULT 0
);

CREATE TABLE ecommerce.pedidos (
    id SERIAL PRIMARY KEY,
    cliente_id INTEGER,
    data_criacao TIMESTAMP DEFAULT NOW(),
    status VARCHAR(20)
);

-- Conceder permissões
GRANT USAGE ON SCHEMA ecommerce TO usuario_app;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA ecommerce TO usuario_app;

-- Adicionar comentário
COMMENT ON SCHEMA ecommerce IS 'Schema para sistema de e-commerce';
```

### 10. Configurando Search Path

```sql
-- Ver search path atual
SHOW search_path;

-- Alterar search path para incluir seu schema
SET search_path TO vendas, public;

-- Alterar permanentemente para um usuário
ALTER USER meu_usuario SET search_path = vendas, public;
```

### Dicas Importantes:

1. **Schemas padrão**: PostgreSQL vem com schemas como `public`, `information_schema`, `pg_catalog`
2. **Organização**: Use schemas para separar lógica de negócio (vendas, rh, financeiro)
3. **Segurança**: Controle de acesso por schema é mais granular
4. **Backup**: Você pode fazer backup de schemas específicos

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
