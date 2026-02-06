---
title: Faturamento por Departamento
---

Consulta padrão para faturamento líquido por departamento.

## Query base

```sql
WITH VENDAS_VALIDAS AS (
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
    JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
    WHERE C.DATA >= :data_inicio
      AND C.DATA < :data_fim
      AND C.CODFILIAL IN ('1', '98')
      AND C.POSICAO = 'F'
      AND C.DTCANCEL IS NULL
      AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
    GROUP BY P.CODEPTO
),
DEVOLUCOES AS (
    SELECT
        P.CODEPTO,
        SUM(NVL(D.VLDEVOLUCAO, 0)) AS VALOR_DEVOLVIDO
    FROM VIEW_DEVOL_RESUMO_FATURAMENTO D
    JOIN PCPRODUT P ON D.CODPROD = P.CODPROD
    JOIN PCUSUARI U ON D.CODUSUR = U.CODUSUR
    WHERE D.DTENT >= :data_inicio
      AND D.DTENT < :data_fim
    GROUP BY P.CODEPTO
)
SELECT
    DEP.CODEPTO,
    DEP.DESCRICAO AS DEPARTAMENTO,
    (NVL(V.VALOR_BRUTO, 0) - NVL(D.VALOR_DEVOLVIDO, 0)) AS VALOR_VENDA
FROM PCDEPTO DEP
LEFT JOIN VENDAS_VALIDAS V ON DEP.CODEPTO = V.CODEPTO
LEFT JOIN DEVOLUCOES D ON DEP.CODEPTO = D.CODEPTO
WHERE (NVL(V.VALOR_BRUTO, 0) > 0 OR NVL(D.VALOR_DEVOLVIDO, 0) > 0)
ORDER BY VALOR_VENDA DESC
```

## Observações

- Para filtrar por supervisor, adicionar `U.CODSUPERVISOR IN (...)` nas CTEs
