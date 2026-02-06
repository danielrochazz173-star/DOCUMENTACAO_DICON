---
title: Devoluções (View Oficial)
---

Devoluções devem ser obtidas pela `VIEW_DEVOL_RESUMO_FATURAMENTO`. Essa view já entrega custo e valores corretos.

## Query base

```sql
SELECT
    D.CODPROD,
    D.CODUSUR,
    D.DTENT,
    SUM(NVL(D.VLDEVOLUCAO, 0)) AS VALOR_DEVOLVIDO,
    SUM(NVL(D.VLCUSTOFIN, 0)) AS CUSTO_DEVOLVIDO
FROM VIEW_DEVOL_RESUMO_FATURAMENTO D
WHERE D.DTENT BETWEEN :data_inicio AND :data_fim
GROUP BY D.CODPROD, D.CODUSUR, D.DTENT
ORDER BY D.DTENT
```

## Boas práticas

- Nunca calcular custo de devolução proporcionalmente em SQL
- Sempre cruzar devoluções com vendedores ou produtos quando precisar de segmentação
