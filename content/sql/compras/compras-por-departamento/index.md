---
title: Compras por Departamento
---

Consulta base para total de compras por departamento.

## Query base

```sql
SELECT
    D.CODEPTO,
    D.DESCRICAO AS DEPARTAMENTO,
    SUM(NVL(M.QT, 0) * NVL(M.PUNIT, 0)) AS VALOR_TOTAL
FROM PCNFENT N
JOIN PCMOV M ON N.NUMTRANSENT = M.NUMTRANSENT AND M.CODOPER = 'E'
JOIN PCPRODUT P ON M.CODPROD = P.CODPROD
JOIN PCDEPTO D ON P.CODEPTO = D.CODEPTO
WHERE N.DTENT >= :data_inicio
  AND N.DTENT < :data_fim
  AND D.DESCRICAO IS NOT NULL
GROUP BY D.CODEPTO, D.DESCRICAO
ORDER BY VALOR_TOTAL DESC
```

## Observações

- Para filtrar departamento específico, adicione `AND D.CODEPTO = :codepto`
