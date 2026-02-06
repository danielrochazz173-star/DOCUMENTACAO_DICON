---
title: Faturamento por Vendedor
---

Consulta oficial para faturamento líquido por vendedor, com devoluções.

## Query base

```sql
WITH VENDAS_VALIDAS AS (
    SELECT
        C.CODUSUR,
        SUM(
            CASE
                WHEN NVL(I.BONIFIC, 'N') = 'N' THEN
                    DECODE(C.CONDVENDA,
                        5, 0, 6, 0, 11, 0, 12, 0,
                        ROUND(NVL(I.QT, 0) * NVL(I.PVENDA, 0), 2)
                    )
                ELSE 0
            END
        ) AS VALOR_BRUTO,
        SUM(
            CASE
                WHEN NVL(I.BONIFIC, 'N') = 'N' THEN
                    DECODE(C.CONDVENDA,
                        5, 0, 6, 0, 11, 0, 12, 0,
                        NVL(I.QT, 0) * NVL(I.VLCUSTOFIN, 0)
                    )
                ELSE 0
            END
        ) AS CUSTO_BRUTO
    FROM PCPEDC C
    JOIN PCPEDI I ON C.NUMPED = I.NUMPED
    WHERE C.DATA BETWEEN :data_inicio AND :data_fim
      AND C.CODFILIAL IN ('1', '98')
      AND C.POSICAO = 'F'
      AND C.DTCANCEL IS NULL
      AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
    GROUP BY C.CODUSUR
),
DEVOLUCOES AS (
    SELECT
        D.CODUSUR,
        SUM(NVL(D.VLDEVOLUCAO, 0)) AS VALOR_DEVOLVIDO,
        SUM(NVL(D.VLCUSTOFIN, 0)) AS CUSTO_DEVOLVIDO
    FROM VIEW_DEVOL_RESUMO_FATURAMENTO D
    WHERE D.DTENT BETWEEN :data_inicio AND :data_fim
    GROUP BY D.CODUSUR
)
SELECT
    U.CODUSUR AS CODIGO_VENDEDOR,
    U.NOME AS NOME_VENDEDOR,
    (NVL(V.VALOR_BRUTO, 0) - NVL(D.VALOR_DEVOLVIDO, 0)) AS FATURAMENTO_LIQUIDO,
    (NVL(V.CUSTO_BRUTO, 0) - NVL(D.CUSTO_DEVOLVIDO, 0)) AS CUSTO_LIQUIDO
FROM PCUSUARI U
LEFT JOIN VENDAS_VALIDAS V ON U.CODUSUR = V.CODUSUR
LEFT JOIN DEVOLUCOES D ON U.CODUSUR = D.CODUSUR
WHERE (NVL(V.VALOR_BRUTO, 0) > 0 OR NVL(D.VALOR_DEVOLVIDO, 0) > 0)
ORDER BY FATURAMENTO_LIQUIDO DESC
```

## Observações

- Para filtrar por supervisor, adicione `JOIN PCUSUARI U` na CTE e `U.CODSUPERVISOR IN (...)`
- Margem deve ser calculada fora do SQL
