---
title: Faturamento Líquido (Base de Cálculo)
---

Base oficial para faturamento líquido. Este padrão é usado nos scripts de produtos e relatórios comparativos.

## Fórmulas oficiais

- Faturamento bruto: `QT * PVENDA`
- Faturamento líquido: `faturamento_bruto - devolucoes`
- Custo bruto: `QT * VLCUSTOFIN`
- Custo líquido: `custo_bruto - custo_devolucao`

## Query base (produto)

```sql
WITH VENDAS_VALIDAS AS (
    SELECT
        I.CODPROD,
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
    JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
    WHERE C.DATA BETWEEN :data_inicio AND :data_fim
      AND C.CODFILIAL IN ('1', '98')
      AND C.POSICAO = 'F'
      AND C.DTCANCEL IS NULL
      AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
      AND U.CODSUPERVISOR IN (1, 2)
    GROUP BY I.CODPROD
),
DEVOLUCOES AS (
    SELECT
        D.CODPROD,
        SUM(NVL(D.VLDEVOLUCAO, 0)) AS VALOR_DEVOLVIDO,
        SUM(NVL(D.VLCUSTOFIN, 0)) AS CUSTO_DEVOLVIDO
    FROM VIEW_DEVOL_RESUMO_FATURAMENTO D
    JOIN PCUSUARI U ON D.CODUSUR = U.CODUSUR
    WHERE D.DTENT BETWEEN :data_inicio AND :data_fim
      AND U.CODSUPERVISOR IN (1, 2)
    GROUP BY D.CODPROD
)
SELECT
    P.CODPROD,
    P.DESCRICAO,
    (NVL(V.VALOR_BRUTO, 0) - NVL(D.VALOR_DEVOLVIDO, 0)) AS FATURAMENTO_LIQUIDO,
    (NVL(V.CUSTO_BRUTO, 0) - NVL(D.CUSTO_DEVOLVIDO, 0)) AS CUSTO_LIQUIDO
FROM PCPRODUT P
LEFT JOIN VENDAS_VALIDAS V ON P.CODPROD = V.CODPROD
LEFT JOIN DEVOLUCOES D ON P.CODPROD = D.CODPROD
WHERE (NVL(V.VALOR_BRUTO, 0) > 0 OR NVL(D.VALOR_DEVOLVIDO, 0) > 0)
ORDER BY FATURAMENTO_LIQUIDO DESC
```

## Observações de negócio

- Devoluções sempre via `VIEW_DEVOL_RESUMO_FATURAMENTO`
- Margem deve ser calculada no Python para evitar arredondamentos no SQL
- Usar `TO_DATE` quando for montar datas em string
