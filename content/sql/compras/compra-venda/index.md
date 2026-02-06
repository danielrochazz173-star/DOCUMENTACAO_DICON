---
title: Comparativo Compra x Venda
---

Consulta usada para comparar compras (entrada) e vendas (faturamento) por período e por departamento.

## Total de compras (entrada)

```sql
SELECT
    SUM(NVL(M.QT, 0) * NVL(M.PUNIT, 0)) AS TOTAL_COMPRA
FROM PCNFENT N
JOIN PCMOV M ON N.NUMTRANSENT = M.NUMTRANSENT AND M.CODOPER = 'E'
WHERE N.DTENT >= :data_inicio
  AND N.DTENT < :data_fim
```

## Total de vendas (faturamento líquido)

```sql
WITH VENDAS_VALIDAS AS (
    SELECT
        SUM(
            CASE
                WHEN NVL(I.BONIFIC, 'N') = 'N' THEN
                    DECODE(C.CONDVENDA,
                        5, 0, 6, 0, 11, 0, 12, 0,
                        ROUND(NVL(I.QT, 0) * NVL(I.PVENDA, 0), 2)
                    )
                ELSE 0
            END
        ) AS VALOR_BRUTO
    FROM PCPEDC C
    JOIN PCPEDI I ON C.NUMPED = I.NUMPED
    JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
    WHERE C.DATA >= :data_inicio
      AND C.DATA < :data_fim
      AND C.CODFILIAL IN ('1', '98')
      AND C.POSICAO = 'F'
      AND C.DTCANCEL IS NULL
      AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
),
DEVOLUCOES AS (
    SELECT
        SUM(NVL(D.VLDEVOLUCAO, 0)) AS VALOR_DEVOLVIDO
    FROM VIEW_DEVOL_RESUMO_FATURAMENTO D
    WHERE D.DTENT >= :data_inicio
      AND D.DTENT < :data_fim
)
SELECT
    (NVL(V.VALOR_BRUTO, 0) - NVL(D.VALOR_DEVOLVIDO, 0)) AS TOTAL_VENDA
FROM VENDAS_VALIDAS V
CROSS JOIN DEVOLUCOES D
```

## Compra x Venda por departamento

```sql
WITH COMPRAS AS (
    SELECT
        D.CODEPTO,
        SUM(NVL(M.QT, 0) * NVL(M.PUNIT, 0)) AS VALOR_COMPRA
    FROM PCNFENT N
    JOIN PCMOV M ON N.NUMTRANSENT = M.NUMTRANSENT AND M.CODOPER = 'E'
    JOIN PCPRODUT P ON M.CODPROD = P.CODPROD
    JOIN PCDEPTO D ON P.CODEPTO = D.CODEPTO
    WHERE N.DTENT >= :data_inicio
      AND N.DTENT < :data_fim
    GROUP BY D.CODEPTO
),
VENDAS AS (
    SELECT
        P.CODEPTO,
        SUM(
            CASE
                WHEN NVL(I.BONIFIC, 'N') = 'N' THEN
                    DECODE(C.CONDVENDA,
                        5, 0, 6, 0, 11, 0, 12, 0,
                        ROUND(NVL(I.QT, 0) * NVL(I.PVENDA, 0), 2)
                    )
                ELSE 0
            END
        ) AS VALOR_BRUTO
    FROM PCPEDC C
    JOIN PCPEDI I ON C.NUMPED = I.NUMPED
    JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
    WHERE C.DATA >= :data_inicio
      AND C.DATA < :data_fim
      AND C.CODFILIAL IN ('1', '98')
      AND C.POSICAO = 'F'
      AND C.DTCANCEL IS NULL
      AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
    GROUP BY P.CODEPTO
)
SELECT
    D.CODEPTO,
    DEP.DESCRICAO AS DEPARTAMENTO,
    NVL(C.VALOR_COMPRA, 0) AS VALOR_COMPRA,
    NVL(V.VALOR_BRUTO, 0) AS VALOR_VENDA
FROM PCDEPTO DEP
LEFT JOIN COMPRAS C ON DEP.CODEPTO = C.CODEPTO
LEFT JOIN VENDAS V ON DEP.CODEPTO = V.CODEPTO
ORDER BY (NVL(C.VALOR_COMPRA, 0) - NVL(V.VALOR_BRUTO, 0)) DESC
```
