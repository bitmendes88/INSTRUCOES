# Comandos PostgreSQL - Guia de Referência

## 📋 Índice
1. [Comandos Básicos](#comandos-básicos)
2. [DDL - Definição de Dados](#ddl---definição-de-dados)
3. [DML - Manipulação de Dados](#dml---manipulação-de-dados)
4. [Consultas e Cláusulas](#consultas-e-cláusulas)
5. [Funções](#funções)
6. [Joins](#joins)
7. [Transações](#transações)
8. [Controle de Acesso](#controle-de-acesso)
9. [Utilitários](#utilitários)

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
