---
title: Compras por Produto (Top N)
---

Consulta base para listar os produtos mais comprados por valor no período.

## Query base

```sql
SELECT
    P.CODPROD,
    P.DESCRICAO AS PRODUTO,
    D.DESCRICAO AS DEPARTAMENTO,
    SUM(NVL(M.QT, 0)) AS QUANTIDADE,
    SUM(NVL(M.QT, 0) * NVL(M.PUNIT, 0)) AS VALOR_TOTAL
FROM PCNFENT N
JOIN PCMOV M ON N.NUMTRANSENT = M.NUMTRANSENT AND M.CODOPER = 'E'
JOIN PCPRODUT P ON M.CODPROD = P.CODPROD
JOIN PCDEPTO D ON P.CODEPTO = D.CODEPTO
WHERE N.DTENT >= :data_inicio
  AND N.DTENT < :data_fim
  AND D.DESCRICAO IS NOT NULL
GROUP BY P.CODPROD, P.DESCRICAO, D.DESCRICAO
ORDER BY VALOR_TOTAL DESC
FETCH FIRST :limite ROWS ONLY
```

## Observações

- Para filtrar departamento específico, adicione `AND D.CODEPTO = :codepto`
- `PCMOV.CODOPER = 'E'` garante apenas entradas
