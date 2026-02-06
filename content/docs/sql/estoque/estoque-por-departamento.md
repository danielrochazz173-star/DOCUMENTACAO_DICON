---
title: Estoque por departamento
---

Consulta de estoque filtrada por departamento (CODEPTO). Usada no envio para BEMBRASIL (depto 102) e em outros fluxos por setor.

## Query Oracle (estoque em unidades e caixas)

```sql
SELECT
    P.CODPROD,
    P.DESCRICAO,
    P.CODAUXILIAR AS EAN,
    P.CODFAB,
    P.PESOLIQ,
    P.QTUNITCX,
    COALESCE(E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA, 0) AS ESTOQUE_UN_ORACLE,
    COALESCE(
        (E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA) / NULLIF(P.QTUNITCX, 1),
        (E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA),
        0
    ) AS ESTOQUE_CX_ORACLE
FROM PCPRODUT P
LEFT JOIN PCEST E ON P.CODPROD = E.CODPROD AND E.CODFILIAL = '1'
WHERE P.DTEXCLUSAO IS NULL
  AND (P.OBS2 <> 'FL' OR P.OBS2 IS NULL)
  AND P.REVENDA = 'S'
  AND P.CODEPTO = :coddepto
ORDER BY P.DESCRICAO ASC
```

## Filtros

- **:coddepto** – código do departamento (ex.: 102 para BEMBRASIL)
- Produto não excluído, não FL, de revenda
- Filial 1

## Estoque em KG (por departamento)

Para obter peso em KG por produto:

```
ESTOQUE_KG = ESTOQUE_CX_ORACLE × QTUNITCX × PESOLIQ
```

Isso é usado no script de envio para Google Sheets (BEMBRASIL).
