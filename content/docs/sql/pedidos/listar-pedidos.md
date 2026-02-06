---
title: Listar pedidos (cabeçalho e resumo)
---

Consultas para listar pedidos (cabeçalho) com cliente, vendedor e valor total. Use os filtros de período, filial e posição; para venda faturada use `POSICAO = 'F'` e `DTCANCEL IS NULL`.

## Pedidos por período (com cliente e vendedor)

```sql
SELECT
    C.NUMPED,
    C.DATA,
    C.NUMNOTA,
    C.CODCLI,
    CL.CLIENTE,
    C.CODUSUR,
    U.NOME AS VENDEDOR,
    C.CODFILIAL,
    C.POSICAO,
    C.CONDVENDA
FROM PCPEDC C
LEFT JOIN PCCLIENT CL ON C.CODCLI = CL.CODCLI
LEFT JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
WHERE C.DATA BETWEEN :data_inicio AND :data_fim
  AND C.CODFILIAL IN ('1', '98')
  AND C.POSICAO = 'F'
  AND C.DTCANCEL IS NULL
  AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
ORDER BY C.DATA DESC, C.NUMPED
```

## Pedidos com valor total (resumo por pedido)

Valor total = soma dos itens (QT × PVENDA), só itens não bonificados. Usar subquery ou JOIN com agregação.

```sql
SELECT
    C.NUMPED,
    C.DATA,
    C.NUMNOTA,
    C.CODCLI,
    CL.CLIENTE,
    C.CODUSUR,
    U.NOME AS VENDEDOR,
    NVL(T.TOTAL_ITENS, 0) AS QTD_ITENS,
    ROUND(NVL(T.VALOR_PEDIDO, 0), 2) AS VALOR_PEDIDO
FROM PCPEDC C
LEFT JOIN PCCLIENT CL ON C.CODCLI = CL.CODCLI
LEFT JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
LEFT JOIN (
    SELECT
        I.NUMPED,
        COUNT(*) AS TOTAL_ITENS,
        SUM(CASE WHEN NVL(I.BONIFIC, 'N') = 'N' THEN ROUND(NVL(I.QT, 0) * NVL(I.PVENDA, 0), 2) ELSE 0 END) AS VALOR_PEDIDO
    FROM PCPEDI I
    GROUP BY I.NUMPED
) T ON C.NUMPED = T.NUMPED
WHERE C.DATA BETWEEN :data_inicio AND :data_fim
  AND C.CODFILIAL IN ('1', '98')
  AND C.POSICAO = 'F'
  AND C.DTCANCEL IS NULL
  AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
ORDER BY C.DATA DESC, C.NUMPED
```

## Filtrar por cliente

```sql
WHERE C.DATA BETWEEN :data_inicio AND :data_fim
  AND C.CODCLI = :codcli
  AND C.CODFILIAL IN ('1', '98')
  AND C.POSICAO = 'F'
  AND C.DTCANCEL IS NULL
  ...
```

Ou por nome (trecho):

```sql
WHERE C.DATA BETWEEN :data_inicio AND :data_fim
  AND UPPER(CL.CLIENTE) LIKE '%' || UPPER(:trecho_nome) || '%'
  ...
```

## Filtrar por vendedor

```sql
WHERE C.DATA BETWEEN :data_inicio AND :data_fim
  AND C.CODUSUR = :codusur
  ...
```

Ou por supervisor (todos os vendedores do supervisor):

```sql
FROM PCPEDC C
JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
WHERE C.DATA BETWEEN :data_inicio AND :data_fim
  AND U.CODSUPERVISOR IN (1, 2)
  ...
```

## Observações

- **POSICAO = 'F'** → pedido faturado.
- **DTCANCEL IS NULL** → pedido não cancelado.
- **CONDVENDA** → excluir condições que não são venda efetiva (ver [Padrões e filtros](../padroes-e-filtros/)).
- Para valor considerando condição de venda (zerar 5, 6, 11, 12), use o mesmo DECODE das queries de faturamento na subquery de totais.
