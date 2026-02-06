---
title: Consultas úteis
---

Snippets genéricos e para Oracle (Winthor): descobrir estrutura, top N, datas, agregação e existência.

## Oracle: listar tabelas e colunas

```sql
SELECT OWNER, TABLE_NAME FROM ALL_TABLES WHERE OWNER = 'SEU_SCHEMA' ORDER BY TABLE_NAME;
```

```sql
SELECT COLUMN_NAME, DATA_TYPE, NULLABLE, DATA_LENGTH
FROM ALL_TAB_COLUMNS
WHERE TABLE_NAME = 'PCPEDC'
ORDER BY COLUMN_ID;
```

## Oracle: início e fim do mês

```sql
SELECT
    TRUNC(SYSDATE, 'MM') AS primeiro_dia,
    LAST_DAY(SYSDATE) AS ultimo_dia
FROM DUAL;
```

## Oracle: Top N (ROWNUM)

```sql
SELECT * FROM (
    SELECT P.CODPROD, P.DESCRICAO, SUM(I.QT) AS TOTAL
    FROM PCPEDI I
    JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
    GROUP BY P.CODPROD, P.DESCRICAO
    ORDER BY TOTAL DESC
) WHERE ROWNUM <= 10;
```

## Oracle: Top N por grupo (ROW_NUMBER)

```sql
SELECT * FROM (
    SELECT
        CODGRUPO,
        CODPROD,
        VALOR,
        ROW_NUMBER() OVER (PARTITION BY CODGRUPO ORDER BY VALOR DESC) AS rn
    FROM MinhaTabela
) WHERE rn <= 5;
```

## Contar por agrupamento

```sql
SELECT coluna, COUNT(*) AS total
FROM Tabela
WHERE condicao = 1
GROUP BY coluna
ORDER BY total DESC;
```

## EXISTS (só onde existe relacionamento)

```sql
SELECT P.CODPROD, P.DESCRICAO
FROM PCPRODUT P
WHERE EXISTS (
    SELECT 1 FROM PCPEDI I
    WHERE I.CODPROD = P.CODPROD AND I.NUMPED = :numped
);
```

## LEFT JOIN vs INNER: quando usar

- **INNER**: só linhas que existem nas duas tabelas (ex.: pedidos que têm itens).
- **LEFT**: todas da tabela da esquerda; da direita preenche onde houver match. Use para “produto mesmo sem estoque” ou “venda menos devolução” (devolução pode ser zero).

```sql
FROM PCPRODUT P
LEFT JOIN PCEST E ON P.CODPROD = E.CODPROD AND E.CODFILIAL = '1'
-- produtos sem registro em PCEST aparecem com E.* em NULL
```

## COALESCE / NVL (primeiro não nulo)

```sql
COALESCE(E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA, 0)
NVL(campo, 0)
```

## SQL Server (referência rápida)

Listar tabelas:

```sql
SELECT TABLE_SCHEMA, TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE' ORDER BY TABLE_SCHEMA, TABLE_NAME;
```

Listar colunas:

```sql
SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE, CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'NOME_DA_TABELA' ORDER BY ORDINAL_POSITION;
```

Início/fim do mês:

```sql
SELECT DATEFROMPARTS(YEAR(GETDATE()), MONTH(GETDATE()), 1) AS inicio_mes, EOMONTH(GETDATE()) AS fim_mes;
```
