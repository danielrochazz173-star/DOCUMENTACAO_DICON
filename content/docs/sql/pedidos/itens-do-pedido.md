---
title: Itens do pedido
---

Consultas para listar os itens de um ou mais pedidos (PCPEDI), com produto, quantidade, preço e valor. Use **NUMPED** para trazer um pedido específico ou mantenha filtros de período/cliente/vendedor.

## Itens de um pedido (por NUMPED)

```sql
SELECT
    C.NUMPED,
    C.DATA,
    C.NUMNOTA,
    C.CODCLI,
    CL.CLIENTE,
    C.CODUSUR,
    U.NOME AS VENDEDOR,
    I.CODPROD,
    P.DESCRICAO AS PRODUTO,
    P.EMBALAGEM,
    I.QT,
    I.PVENDA,
    ROUND(I.QT * I.PVENDA, 2) AS VALOR_LINHA,
    I.VLCUSTOFIN,
    NVL(I.BONIFIC, 'N') AS BONIFIC
FROM PCPEDC C
JOIN PCPEDI I ON C.NUMPED = I.NUMPED
JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
LEFT JOIN PCCLIENT CL ON C.CODCLI = CL.CODCLI
LEFT JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
WHERE C.NUMPED = :numped
ORDER BY I.CODPROD
```

## Itens de vários pedidos (por período)

Mesma query, trocando o filtro:

```sql
WHERE C.DATA BETWEEN :data_inicio AND :data_fim
  AND C.CODFILIAL IN ('1', '98')
  AND C.POSICAO = 'F'
  AND C.DTCANCEL IS NULL
  AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
  AND NVL(I.BONIFIC, 'N') = 'N'
ORDER BY C.DATA DESC, C.NUMPED, I.CODPROD
```

## Itens por cliente (produtos comprados por um cliente no período)

```sql
SELECT
    C.NUMPED,
    C.DATA,
    CL.CODCLI,
    CL.CLIENTE,
    I.CODPROD,
    P.DESCRICAO AS PRODUTO,
    I.QT,
    I.PVENDA,
    ROUND(I.QT * I.PVENDA, 2) AS VALOR_TOTAL,
    U.NOME AS VENDEDOR
FROM PCPEDC C
JOIN PCPEDI I ON C.NUMPED = I.NUMPED
JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
JOIN PCCLIENT CL ON C.CODCLI = CL.CODCLI
LEFT JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
WHERE C.DATA BETWEEN :data_inicio AND :data_fim
  AND C.CODCLI = :codcli
  AND C.CODFILIAL IN ('1', '98')
  AND C.POSICAO = 'F'
  AND C.DTCANCEL IS NULL
  AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
  AND NVL(I.BONIFIC, 'N') = 'N'
ORDER BY C.DATA DESC, I.CODPROD
```

## Itens por vendedor

```sql
WHERE C.DATA BETWEEN :data_inicio AND :data_fim
  AND C.CODUSUR = :codusur
  AND C.CODFILIAL IN ('1', '98')
  AND C.POSICAO = 'F'
  AND C.DTCANCEL IS NULL
  AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
  AND NVL(I.BONIFIC, 'N') = 'N'
ORDER BY C.DATA DESC, C.NUMPED, I.CODPROD
```

## Itens por produto (quem comprou um produto no período)

```sql
SELECT
    C.NUMPED,
    C.DATA,
    CL.CODCLI,
    CL.CLIENTE,
    I.CODPROD,
    P.DESCRICAO AS PRODUTO,
    I.QT,
    I.PVENDA,
    ROUND(I.QT * I.PVENDA, 2) AS VALOR_TOTAL,
    U.NOME AS VENDEDOR
FROM PCPEDC C
JOIN PCPEDI I ON C.NUMPED = I.NUMPED
JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
JOIN PCCLIENT CL ON C.CODCLI = CL.CODCLI
LEFT JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
WHERE C.DATA BETWEEN :data_inicio AND :data_fim
  AND I.CODPROD = :codprod
  AND C.CODFILIAL IN ('1', '98')
  AND C.POSICAO = 'F'
  AND C.DTCANCEL IS NULL
  AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
  AND NVL(I.BONIFIC, 'N') = 'N'
ORDER BY C.DATA DESC, CL.CODCLI
```

## Observações

- **BONIFIC = 'N'** → item conta como venda; bonificado costuma ser zerado ou excluído.
- **VALOR_LINHA** = QT × PVENDA (arredondar em 2 casas).
- Para custo por linha: `I.QT * I.VLCUSTOFIN`; margem depois (valor - custo) / valor.
