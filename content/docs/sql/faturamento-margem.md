---
title: Faturamento e Margem - Padrões SQL
---

Esta página consolida os padrões de cálculo e filtros usados nas análises de faturamento e margem.

## Fórmulas oficiais

- Faturamento bruto: `QT * PVENDA`
- Faturamento líquido: `faturamento_bruto - devolucoes`
- Custo bruto: `QT * VLCUSTOFIN`
- Custo líquido: `custo_bruto - custo_devolucao`
- Margem (%): `((fat_liq - custo_liq) / fat_liq) * 100`

## Tabelas e views principais

| Tabela / View | Uso no faturamento |
|---------------|---------------------|
| **PCPEDC** | Cabeçalho do pedido. JOIN com PCPEDI por `NUMPED`. Filtros: DATA, POSICAO, DTCANCEL, CONDVENDA, CODFILIAL. |
| **PCPEDI** | Itens do pedido. Traz CODPROD, QT, PVENDA, VLCUSTOFIN, BONIFIC. |
| **PCUSUARI** | Vendedor (CODUSUR). Usado para filtrar por CODSUPERVISOR. |
| **PCPRODUT** | Dados do produto (para listar descrição, departamento). |
| **PCDEPTO** | Departamento (quando precisar agrupar por setor). |
| **VIEW_DEVOL_RESUMO_FATURAMENTO** | Devoluções com valor e custo. JOIN por CODPROD (e opcionalmente CODUSUR). Campos: VLDEVOLUCAO, VLCUSTOFIN, DTENT. |

## Filtros de negócio obrigatórios

- `POSICAO = 'F'`
- `DTCANCEL IS NULL`
- `BONIFIC = 'N'`
- `CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)`
- `CODFILIAL IN ('1', '98')` quando aplicável

## Padrão de devoluções

Usar `VIEW_DEVOL_RESUMO_FATURAMENTO` para custo de devolução.
Evitar calcular custo de devolução proporcionalmente no SQL.

## Esqueleto de query (modelo)

```sql
WITH VENDAS_VALIDAS AS (
    SELECT
        I.CODPROD,
        SUM(NVL(I.QT, 0) * NVL(I.PVENDA, 0)) AS VALOR_BRUTO,
        SUM(NVL(I.QT, 0) * NVL(I.VLCUSTOFIN, 0)) AS CUSTO_BRUTO
    FROM PCPEDC C
    JOIN PCPEDI I ON C.NUMPED = I.NUMPED
    WHERE C.DATA BETWEEN :data_inicio AND :data_fim
      AND C.POSICAO = 'F'
      AND C.DTCANCEL IS NULL
      AND NVL(I.BONIFIC, 'N') = 'N'
      AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
    GROUP BY I.CODPROD
),
DEVOLUCOES AS (
    SELECT
        D.CODPROD,
        SUM(NVL(D.VLDEVOLUCAO, 0)) AS VALOR_DEVOLVIDO,
        SUM(NVL(D.VLCUSTOFIN, 0)) AS CUSTO_DEVOLVIDO
    FROM VIEW_DEVOL_RESUMO_FATURAMENTO D
    WHERE D.DTENT BETWEEN :data_inicio AND :data_fim
    GROUP BY D.CODPROD
)
SELECT
    V.CODPROD,
    (NVL(V.VALOR_BRUTO, 0) - NVL(D.VALOR_DEVOLVIDO, 0)) AS FATURAMENTO_LIQUIDO,
    (NVL(V.CUSTO_BRUTO, 0) - NVL(D.CUSTO_DEVOLVIDO, 0)) AS CUSTO_LIQUIDO
FROM VENDAS_VALIDAS V
LEFT JOIN DEVOLUCOES D ON V.CODPROD = D.CODPROD;
```

## Observações importantes

- Calcular margem no Python para evitar arredondamentos no SQL
- Usar datas com `TO_DATE` quando necessário para evitar ambiguidades
