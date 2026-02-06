---
title: Exportar produtos (lista)
---

Consulta para listagem de produtos (código, descrição, NCM, código de barras) usada na exportação para Excel.

## Query

```sql
SELECT
    P.CODPROD AS CODIGO,
    P.DESCRICAO,
    P.NBM AS NCM,
    P.CODAUXILIAR AS CODIGO_BARRAS
FROM PCPRODUT P
WHERE P.DTEXCLUSAO IS NULL
  AND (P.OBS2 <> 'FL' OR P.OBS2 IS NULL)
  AND P.REVENDA = 'S'
ORDER BY P.DESCRICAO ASC
```

## Uso

- Base do script `exportar_produtos_excel.py`: gera Excel com Código, Descrição, NCM, Código de Barras.
- Sem filtro de departamento: todos os produtos válidos.
