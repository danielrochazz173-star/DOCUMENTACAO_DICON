---
title: Estoque Oracle + Vilog (KG)
---

Padrão para somar estoque do Oracle com o peso total da planilha Vilog (aba CALCULADO) e obter estoque total em KG.

## Fontes

1. **Oracle** – estoque em caixas por produto (PCEST + PCPRODUT), filtrado por departamento.
2. **Vilog** – planilha Google Sheets `VILOG`, aba `CALCULADO`: coluna A = CODPROD, coluna J = PESO TOTAL (KG).

Match é sempre por **CODPROD** (não por CODFAB).

## Cálculo Oracle (KG)

```
ESTOQUE_KG_ORACLE = ESTOQUE_CX_ORACLE × QTUNITCX × PESOLIQ
```

Exemplo: 100 caixas × 7 un/cx × 2 kg/un = 1.400 kg.

## Cálculo Vilog

O peso já vem em KG na coluna J da aba CALCULADO. Apenas somar ao Oracle por CODPROD.

## Estoque total (KG)

```
ESTOQUE_KG_TOTAL = ESTOQUE_KG_ORACLE + ESTOQUE_KG_VILOG
```

## Query Oracle (base para o script)

```sql
SELECT
    P.CODPROD,
    P.DESCRICAO,
    P.CODAUXILIAR AS EAN,
    P.CODFAB,
    P.PESOLIQ,
    P.QTUNITCX,
    COALESCE((E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA) / NULLIF(P.QTUNITCX, 1),
             (E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA), 0) AS ESTOQUE_CX_ORACLE
FROM PCPRODUT P
LEFT JOIN PCEST E ON P.CODPROD = E.CODPROD AND E.CODFILIAL = '1'
WHERE P.DTEXCLUSAO IS NULL
  AND (P.OBS2 <> 'FL' OR P.OBS2 IS NULL)
  AND P.REVENDA = 'S'
  AND P.CODEPTO = 102
ORDER BY P.DESCRICAO ASC
```

No Python: ler Vilog (A=CODPROD, J=KG), fazer o match por CODPROD e somar Oracle KG + Vilog KG; ordenar por estoque total KG (desc).
