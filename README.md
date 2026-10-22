# Guia Definitivo de SQL Joins (2, 3 e 4 Tabelas)

Um guia prático e teórico completo sobre a utilização de Junções (*Joins*) em bancos de dados relacionais (RDBMS), abrangendo desde o relacionamento básico de 2 tabelas até relacionamentos complexos envolvendo 3 e 4 tabelas.

---

## 📋 Sumário
1. [Fundamentação Teórica](#1-fundamentação-teórica)
2. [Sintaxe Generalizada](#2-sintaxe)
3. [Domínio de Aplicação do Banco de Dados](#3-domínio-de-aplicação-do-banco-de-dados)
4. [Estrutura das Tabelas DDL e DML](#4-estrutura-das-tabelas)
5. [Exemplos Práticos Explicados](#5-exemplos-práticos-explicados)
6. [Exercícios de Fixação com Gabarito](#6-exercícios-de-fixação-com-gabarito)
7. [Boas Práticas e Performance](#7-boas-práticas)
8. [Referências](#8-referências)

---

## 1. Fundamentação Teórica

Em bancos de dados relacionais baseados em SQL (como PostgreSQL, MySQL, SQL Server e Oracle), os dados são normalizados e distribuídos em múltiplas tabelas para evitar redundância e garantir a integridade dos dados (Regras de Codd / Formas Normais). 

As junções (**SQL Joins**) são operações baseadas na teoria dos conjuntos e na álgebra relacional que permitem restruturar esses dados dispersos, combinando colunas de uma ou mais tabelas com base em uma condição de relacionamento pré-estabelecida (geralmente chave primária vs. chave estrangeira).

### Tipos Principais de Joins:

*   **`INNER JOIN`**: Retorna apenas os registros que possuem correspondência em **ambas** as tabelas. É a interseção perfeita dos conjuntos ($A \cap B$).
*   **`LEFT JOIN` (ou `LEFT OUTER JOIN`)**: Retorna **todos** os registros da tabela à esquerda, juntamente com os registros correspondentes da tabela à direita. Caso não haja correspondência, o lado direito preenche com valores `NULL`.
*   **`RIGHT JOIN` (ou `RIGHT OUTER JOIN`)**: Retorna **todos** os registros da tabela à direita, juntamente com os correspondentes da esquerda. Caso não haja correspondência, o lado esquerdo preenche com valores `NULL`.
*   **`FULL OUTER JOIN`**: Retorna todos os registros quando há uma correspondência na tabela da esquerda ou da direita. Preenche com `NULL` onde não houver combinação ($A \cup B$).

---

## 2. SINTAXE

### Sintaxe para 2 Tabelas:
```sql
SELECT 
    t1.coluna1, 
    t2.coluna2
FROM tabela_a t1
[INNER | LEFT | RIGHT | FULL] JOIN tabela_b t2
    ON t1.chave_estrangeira = t2.chave_primaria;
```

### Sintaxe para 3 Tabelas:
```sql
SELECT 
    t1.coluna1, 
    t2.coluna2, 
    t3.coluna3
FROM tabela_a t1
INNER JOIN tabela_b t2 ON t1.t2_id = t2.id
INNER JOIN tabela_c t3 ON t2.t3_id = t3.id;
```

### Sintaxe para 4 Tabelas:
```sql
SELECT 
    t1.coluna1, 
    t2.coluna2, 
    t3.coluna3,
    t4.coluna4
FROM tabela_a t1
INNER JOIN tabela_b t2 ON t1.t2_id = t2.id
INNER JOIN tabela_c t3 ON t2.t3_id = t3.id
LEFT JOIN  tabela_d t4 ON t3.t4_id = t4.id;
```

---

## 3. Domínio da Aplicação do Banco de Dados

O domínio escolhido para este guia é um **Sistema de E-commerce e Gestão de Vendas**, englobando Clientes, Pedidos, Itens de Pedido e Produtos. 

### Modelo Entidade-Relacionamento (DER Lógico):
*   **`clientes`** $(1) \leftarrow (N)$ **`pedidos`**: Um cliente pode fazer múltiplos pedidos.
*   **`pedidos`** $(1) \leftarrow (N)$ **`itens_pedido`**: Um pedido contém um ou mais itens.
*   **`produtos`** $(1) \leftarrow (N)$ **`itens_pedido`**: Um produto pode estar presente em vários itens de pedidos.

---

## 4. Estrutura das Tabelas

Abaixo estão os scripts DDL (Criação) e DML (Inserção) com **4 tabelas populadas** para testes.

```sql
-- 1. Tabela de Clientes
CREATE TABLE clientes (
    cliente_id INT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cidade VARCHAR(50)
);

-- 2. Tabela de Produtos
CREATE TABLE produtos (
    produto_id INT PRIMARY KEY,
    nome_produto VARCHAR(100) NOT NULL,
    preco DECIMAL(10,2) NOT NULL
);

-- 3. Tabela de Pedidos
CREATE TABLE pedidos (
    pedido_id INT PRIMARY KEY,
    cliente_id INT,
    data_pedido DATE NOT NULL,
    FOREIGN KEY (cliente_id) REFERENCES clientes(cliente_id)
);

-- 4. Tabela de Itens do Pedido (Tabela de Junção/Reduzida)
CREATE TABLE itens_pedido (
    item_id INT PRIMARY KEY,
    pedido_id INT,
    produto_id INT,
    quantidade INT NOT NULL,
    FOREIGN KEY (pedido_id) REFERENCES pedidos(pedido_id),
    FOREIGN KEY (produto_id) REFERENCES produtos(produto_id)
);

-- INSERÇÕES DE DADOS (DML)

INSERT INTO clientes VALUES 
(1, 'Ana Silva', 'São Paulo'),
(2, 'Carlos Oliveira', 'Rio de Janeiro'),
(3, 'Beatriz Souza', 'Belo Horizonte'),
(4, 'João Pedro', 'Curitiba'); -- Cliente sem pedidos

INSERT INTO produtos VALUES 
(101, 'Notebook Pro', 4500.00),
(102, 'Mouse sem Fio', 150.00),
(103, 'Teclado Mecânico', 350.00),
(104, 'Monitor 27"', 1200.00); -- Produto nunca vendido

INSERT INTO pedidos VALUES 
(1001, 1, '2026-01-10'),
(1002, 2, '2026-01-12'),
(1003, 1, '2026-01-15'),
(1004, 3, '2026-01-20');

INSERT INTO itens_pedido VALUES 
(1, 1001, 101, 1), -- Pedido 1001: 1 Notebook
(2, 1001, 102, 2), -- Pedido 1001: 2 Mouses
(3, 1002, 103, 1), -- Pedido 1002: 1 Teclado
(4, 1003, 102, 1), -- Pedido 1003: 1 Mouse
(5, 1004, 101, 1); -- Pedido 1004: 1 Notebook
```

---

## 5. Exemplos Práticos Explicados

### Exemplo 1: INNER JOIN entre 2 Tabelas
*Objetivo:* Listar o nome do cliente e a data de todos os pedidos realizados.

```sql
SELECT 
    c.nome AS nome_cliente,
    p.pedido_id,
    p.data_pedido
FROM clientes c
INNER JOIN pedidos p ON c.cliente_id = p.cliente_id;
```
* **Explicação:** Conecta a tabela `clientes` ($c$) com `pedidos` ($p$). Clientes que não realizaram pedidos (ex: João Pedro) são filtrados e não aparecem no resultado.

---

### Exemplo 2: LEFT JOIN entre 2 Tabelas
*Objetivo:* Listar TODOS os clientes cadastrados e a data dos seus pedidos, inclusive aqueles que nunca compraram.

```sql
SELECT 
    c.nome AS nome_cliente,
    p.pedido_id,
    p.data_pedido
FROM clientes c
LEFT JOIN pedidos p ON c.cliente_id = p.cliente_id;
```
* **Explicação:** Preserva todos os registros da tabela à esquerda (`clientes`). O cliente "João Pedro" aparecerá no resultado com valores `NULL` nas colunas `pedido_id` e `data_pedido`.

---

### Exemplo 3: INNER JOIN entre 3 Tabelas
*Objetivo:* Listar o nome do cliente, o ID do pedido e o nome dos produtos comprados nesses pedidos.

```sql
SELECT 
    c.nome AS nome_cliente,
    p.pedido_id,
    prod.nome_produto
FROM clientes c
INNER JOIN pedidos p ON c.cliente_id = p.cliente_id
INNER JOIN itens_pedido item ON p.pedido_id = item.pedido_id
INNER JOIN produtos prod ON item.produto_id = prod.produto_id;
```
* **Explicação:** A consulta parte de `clientes`, junta com `pedidos` via `cliente_id`, depois associa a tabela intermediária `itens_pedido` via `pedido_id` e finaliza associando a `produtos` via `produto_id`.

---

### Exemplo 4: LEFT JOIN entre 3 Tabelas (Análise de Produtos Não Vendidos)
*Objetivo:* Listar todos os produtos e identificar se eles já foram vendidos em algum pedido de cliente.

```sql
SELECT 
    prod.nome_produto,
    item.pedido_id,
    c.nome AS nome_comprador
FROM produtos prod
LEFT JOIN itens_pedido item ON prod.produto_id = item.produto_id
LEFT JOIN pedidos p ON item.pedido_id = p.pedido_id
LEFT JOIN clientes c ON p.cliente_id = c.cliente_id;
```
* **Explicação:** Manteve-se o `LEFT JOIN` encadeado para garantir que o produto "Monitor 27"" (nunca vendido) apareça no resultado final com dados da compra como `NULL`.

---

### Exemplo 5: Encadeamento Completo de 4 Tabelas com Agregação
*Objetivo:* Relatório completo de vendas contendo Nome do Cliente, ID do Pedido, Nome do Produto, Quantidade, Valor Unitário e Subtotal por Item.

```sql
SELECT 
    c.nome AS cliente,
    p.pedido_id,
    p.data_pedido,
    prod.nome_produto,
    item.quantidade,
    prod.preco AS valor_unitario,
    (item.quantidade * prod.preco) AS subtotal
FROM clientes c
INNER JOIN pedidos p ON c.cliente_id = p.cliente_id
INNER JOIN itens_pedido item ON p.pedido_id = item.pedido_id
INNER JOIN produtos prod ON item.produto_id = prod.produto_id
ORDER BY p.pedido_id ASC;
```
* **Explicação:** Este relatório consolida todo o ecossistema relacional da aplicação. Ele une as 4 tabelas e faz um cálculo derivado em tempo de execução multiplicando a quantidade vendida pelo preço do produto.

---

## 6. Exercícios de Fixação com Gabarito

### Exercícios:

1. **(Duas tabelas)** Escreva uma consulta que traga todos os produtos e os itens de pedido correspondentes, mostrando inclusive os produtos que jamais foram vendidos.
2. **(Duas tabelas)** Mostre o nome dos clientes e a quantidade total de pedidos efetuados por cada um. Agrupe pelo nome do cliente e ordene do maior para o menor.
3. **(Três tabelas)** Selecione o nome do cliente, a data do pedido e a quantidade comprada para todos os pedidos que contenham o produto 'Notebook Pro'.
4. **(Quatro tabelas)** Crie uma consulta para exibir o total gasto por cliente (soma de todos os seus pedidos). Exiba apenas o Nome do Cliente e o Total Gasto Geral.
5. **(Quatro tabelas + Filtro)** Mostre os nomes dos clientes do 'Rio de Janeiro' e o nome dos produtos que eles compraram.

---

### Gabarito Resposta:

<details>
<summary>Clique aqui para ver os gabaritos dos exercícios</summary>

#### Resposta 1:
```sql
SELECT 
    p.nome_produto, 
    i.pedido_id, 
    i.quantidade
FROM produtos p
LEFT JOIN itens_pedido i ON p.produto_id = i.produto_id;
```

#### Resposta 2:
```sql
SELECT 
    c.nome, 
    COUNT(p.pedido_id) AS total_pedidos
FROM clientes c
LEFT JOIN pedidos p ON c.cliente_id = p.cliente_id
GROUP BY c.cliente_id, c.nome
ORDER BY total_pedidos DESC;
```

#### Resposta 3:
```sql
SELECT 
    c.nome, 
    p.data_pedido, 
    item.quantidade
FROM clientes c
INNER JOIN pedidos p ON c.cliente_id = p.cliente_id
INNER JOIN itens_pedido item ON p.pedido_id = item.pedido_id
INNER JOIN produtos prod ON item.produto_id = prod.produto_id
WHERE prod.nome_produto = 'Notebook Pro';
```

#### Resposta 4:
```sql
SELECT 
    c.nome AS cliente,
    SUM(item.quantidade * prod.preco) AS total_gasto
FROM clientes c
INNER JOIN pedidos p ON c.cliente_id = p.cliente_id
INNER JOIN itens_pedido item ON p.pedido_id = item.pedido_id
INNER JOIN produtos prod ON item.produto_id = prod.produto_id
GROUP BY c.cliente_id, c.nome;
```

#### Resposta 5:
```sql
SELECT DISTINCT
    c.nome AS cliente,
    prod.nome_produto
FROM clientes c
INNER JOIN pedidos p ON c.cliente_id = p.cliente_id
INNER JOIN itens_pedido item ON p.pedido_id = item.pedido_id
INNER JOIN produtos prod ON item.produto_id = prod.produto_id
WHERE c.cidade = 'Rio de Janeiro';
```
</details>

---

## 7. Melhores Práticas

1. **Uso Obrigatório de Aliases**: Em consultas com múltiplos *Joins*, sempre utilize apelidos curtos para as tabelas (ex: `clientes c`, `pedidos p`). Isso evita ambiguidade de colunas e melhora a legibilidade.
2. **Atente-se para Índices nas FKs**: Certifique-se de que as colunas utilizadas nas condições do `ON` (chaves primárias e estrangeiras) possuam índices criados para otimizar os algoritmos de busca do SGBD (*Hash Join*, *Nested Loop*, *Merge Join*).
3. **Evite `SELECT *`**: Sempre liste explicitamente as colunas necessárias. O uso de `SELECT *` consome mais memória, trafega dados desnecessários na rede e prejudica a performance.
4. **Filtros no lugar certo**: Use a cláusula `ON` exclusivamente para a lógica de junção de tabelas e a cláusula `WHERE` para filtragem de dados de negócio.
5. **Mantenha a consistência do LEFT JOIN**: Se você iniciar uma sequência de junções com `LEFT JOIN`, as junções subsequentes que dependerem da tabela da direita geralmente também devem usar `LEFT JOIN`, para evitar que um `INNER JOIN` acidental descarte as linhas preservadas com valores `NULL`.

---

## 8. Referências

* **PostgreSQL Documentation**: [Table Expressions - Joins](https://www.postgresql.org/docs/current/queries-table-expressions.html)
* **MySQL 8.0 Reference Manual**: [JOIN Clause Syntax](https://dev.mysql.com/doc/refman/8.0/en/join.html)
* **W3Schools SQL Joins Tutorial**: [SQL Joins](https://www.w3schools.com/sql/sql_join.asp)
* **SILBERSCHATZ, A.; KORTH, H. F.; SUDARSHAN, S.** *Sistema de Banco de Dados*. 7ª ed. McGraw-Hill, 2020.
* **Modelo Linguístico de Inteligência Artificial**: Gemini 3 Flash / OpenAI GPT-4o (Auxílio na estruturação técnica, validação teórica e elaboração de exemplos práticos do documento).
