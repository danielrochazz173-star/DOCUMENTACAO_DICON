---
title: Estoque por CODFAB
---

Consulta padrão para estoque em caixas por código do fabricante, com departamento e EAN.

## Query

```sql
SELECT
    P.CODPROD,
    P.DESCRICAO,
    P.CODFAB,
    P.CODAUXILIAR AS EAN,
    P.EMBALAGEM,
    P.PESOLIQ,
    P.QTUNITCX,
    D.DESCRICAO AS DEPARTAMENTO,
    D.CODEPTO,
    COALESCE(
        (E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA) / NULLIF(P.QTUNITCX, 1),
        (E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA),
        0
    ) AS ESTOQUE_CX
FROM PCPRODUT P
LEFT JOIN PCDEPTO D ON P.CODEPTO = D.CODEPTO
LEFT JOIN PCEST E ON P.CODPROD = E.CODPROD AND E.CODFILIAL = '1'
WHERE P.CODFAB = :codfab
  AND P.DTEXCLUSAO IS NULL
  AND (P.OBS2 <> 'FL' OR P.OBS2 IS NULL)
  AND P.REVENDA = 'S'
ORDER BY P.DESCRICAO
```

## Filtros

- **CODFAB** – código do fabricante (bind :codfab)
- Produto ativo, não FL, de revenda
- Filial 1 no PCEST
