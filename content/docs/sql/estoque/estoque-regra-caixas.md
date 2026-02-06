---
title: Regra de estoque em caixas
---

Fórmula padrão para estoque em caixas no Winthor.

## Fórmula

```
ESTOQUE_CX = (QTESTGER - QTRESERV - QTBLOQUEADA) / QTUNITCX
```

- **QTESTGER** – quantidade em estoque (unidades)
- **QTRESERV** – quantidade reservada
- **QTBLOQUEADA** – quantidade bloqueada
- **QTUNITCX** – unidades por caixa (PCPRODUT)

Se `QTUNITCX` for 0 ou NULL, usar apenas a diferença em unidades (sem dividir por caixa).

## Em SQL (genérico)

```sql
COALESCE(
    (E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA) / NULLIF(P.QTUNITCX, 1),
    (E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA),
    0
) AS ESTOQUE_CX
```

Sempre considerar **CODFILIAL** no `PCEST` (ex.: `E.CODFILIAL = '1'`).
