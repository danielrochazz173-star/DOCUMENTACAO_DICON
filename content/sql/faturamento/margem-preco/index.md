---
title: Margem e Preço Ideal
---

Esta página concentra o cálculo de margem por produto e a lógica para preço ideal.

## Fórmulas

- Margem (%):
  `((valor_vendido - custo_total) / valor_vendido) * 100`

- Preço ideal:
  `preco_ideal = custo_medio / (1 - margem_minima/100)`

## Query base (margem por produto)

```sql
WITH VENDAS_MES AS (
    SELECT
        I.CODPROD,
        P.DESCRICAO,
        SUM(ROUND(NVL(I.QT, 0) * NVL(I.PVENDA, 0), 2)) AS VALOR_VENDIDO,
        SUM(ROUND(NVL(I.QT, 0) * NVL(I.VLCUSTOFIN, 0), 2)) AS CUSTO_TOTAL,
        SUM(NVL(I.QT, 0)) AS QUANTIDADE_TOTAL,
        MIN(I.PVENDA) AS PRECO_MIN_VENDIDO,
        MAX(I.PVENDA) AS PRECO_MAX_VENDIDO,
        AVG(NVL(I.VLCUSTOFIN, 0)) AS CUSTO_MEDIO
    FROM PCPEDC C
    JOIN PCPEDI I ON C.NUMPED = I.NUMPED
    JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
    WHERE EXTRACT(YEAR FROM C.DATA) = :ano
      AND EXTRACT(MONTH FROM C.DATA) = :mes
      AND C.CODFILIAL IN ('1', '98')
      AND C.POSICAO = 'F'
      AND C.DTCANCEL IS NULL
      AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
      AND NVL(I.BONIFIC, 'N') = 'N'
      AND I.PVENDA > 0
    GROUP BY I.CODPROD, P.DESCRICAO
    HAVING SUM(ROUND(NVL(I.QT, 0) * NVL(I.PVENDA, 0), 2)) > 0
)
SELECT
    V.CODPROD,
    V.DESCRICAO,
    ROUND(V.VALOR_VENDIDO, 2) AS VALOR_VENDIDO,
    ROUND(V.CUSTO_TOTAL, 2) AS CUSTO_TOTAL,
    CASE
        WHEN V.QUANTIDADE_TOTAL > 0 THEN
            ROUND(V.VALOR_VENDIDO / V.QUANTIDADE_TOTAL, 2)
        ELSE 0
    END AS PRECO_VENDIDO,
    ROUND(V.PRECO_MIN_VENDIDO, 2) AS PRECO_MIN_VENDIDO,
    ROUND(V.PRECO_MAX_VENDIDO, 2) AS PRECO_MAX_VENDIDO,
    ROUND(V.CUSTO_MEDIO, 2) AS CUSTO_MEDIO,
    V.QUANTIDADE_TOTAL,
    CASE
        WHEN V.VALOR_VENDIDO > 0 THEN
            ROUND(((V.VALOR_VENDIDO - V.CUSTO_TOTAL) / V.VALOR_VENDIDO) * 100, 2)
        ELSE 0
    END AS MARGEM_ATUAL,
    COALESCE(
        (E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA) / NULLIF(P.QTUNITCX, 1),
        (E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA),
        0
    ) AS ESTOQUE_CX
FROM VENDAS_MES V
JOIN PCPRODUT P ON V.CODPROD = P.CODPROD
LEFT JOIN PCEST E ON V.CODPROD = E.CODPROD AND E.CODFILIAL = '1'
ORDER BY V.VALOR_VENDIDO DESC
```

## Preço ideal (após consulta)

O preço ideal é calculado no Python com base no `CUSTO_MEDIO`:

```python
preco_ideal = round(custo_medio / (1 - margem_minima / 100), 2)
```
