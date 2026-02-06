---
title: Padrões e filtros de negócio
---

Convenções usadas nas queries do Winthor: filtros obrigatórios, tratamento de nulo, condições de venda e datas. Aplicar esses padrões evita duplicar venda, bonificação e pedido cancelado.

## Filtros de pedido (venda válida)

| Condição | Motivo |
|----------|--------|
| `C.POSICAO = 'F'` | Pedido faturado (posição final) |
| `C.DTCANCEL IS NULL` | Pedido não cancelado |
| `NVL(I.BONIFIC, 'N') = 'N'` | Item não bonificado (conta como venda) |
| `C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)` | Exclui condições que não são venda efetiva (consignado, bonificação, etc.) |
| `C.CODFILIAL IN ('1', '98')` | Filiais consideradas no faturamento (ajustar conforme regra) |

Condições de venda que zeram valor (quando entram no DECODE): 5, 6, 11, 12 (bonificação, cortesia, etc.).

---

## NVL e DECODE (Oracle)

**NVL** – substitui NULL por um valor default:

```sql
NVL(I.QT, 0)           -- se QT for NULL, usa 0
NVL(I.BONIFIC, 'N')    -- se BONIFIC for NULL, trata como 'N'
```

**DECODE** – “se valor = X então A, senão B” (evita vários OR):

```sql
DECODE(C.CONDVENDA,
    5, 0,    -- condição 5 → zero
    6, 0,
    11, 0,
    12, 0,
    ROUND(NVL(I.QT, 0) * NVL(I.PVENDA, 0), 2)  -- demais → calcula
) AS VALOR_BRUTO
```

Assim você zera o valor para condições especiais e calcula para as demais.

---

## Datas (Oracle)

Intervalo fechado (início e fim no mesmo dia):

```sql
WHERE C.DATA BETWEEN :data_inicio AND :data_fim
```

Intervalo “início inclusive, fim exclusive” (evita contar duas vezes meia-noite):

```sql
WHERE C.DATA >= :data_inicio AND C.DATA < :data_fim
```

Converter string para data (evitar ambiguidade):

```sql
WHERE C.DATA BETWEEN TO_DATE(:data_inicio, 'DD/MM/YYYY') AND TO_DATE(:data_fim, 'DD/MM/YYYY')
```

Início e fim do mês (Oracle):

```sql
TRUNC(SYSDATE, 'MM')                    AS primeiro_dia_mes
LAST_DAY(SYSDATE)                      AS ultimo_dia_mes
EXTRACT(YEAR FROM C.DATA) = :ano
EXTRACT(MONTH FROM C.DATA) = :mes
```

---

## CTE (WITH) – organizar a query

Quebrar a lógica em blocos nomeados deixa a query legível e reutilizável:

```sql
WITH VENDAS_VALIDAS AS (
    SELECT I.CODPROD, SUM(...) AS VALOR_BRUTO
    FROM PCPEDC C
    JOIN PCPEDI I ON C.NUMPED = I.NUMPED
    WHERE ... (filtros de pedido)
    GROUP BY I.CODPROD
),
DEVOLUCOES AS (
    SELECT D.CODPROD, SUM(D.VLDEVOLUCAO) AS VALOR_DEVOLVIDO
    FROM VIEW_DEVOL_RESUMO_FATURAMENTO D
    WHERE D.DTENT BETWEEN :data_inicio AND :data_fim
    GROUP BY D.CODPROD
)
SELECT
    P.CODPROD,
    (NVL(V.VALOR_BRUTO, 0) - NVL(D.VALOR_DEVOLVIDO, 0)) AS FATURAMENTO_LIQUIDO
FROM PCPRODUT P
LEFT JOIN VENDAS_VALIDAS V ON P.CODPROD = V.CODPROD
LEFT JOIN DEVOLUCOES D ON P.CODPROD = D.CODPROD
```

Cada CTE faz uma “consulta intermediária”; o SELECT final só junta e calcula.

---

## Produto ativo (cadastro)

Para não trazer produto excluído ou fora de uso:

```sql
WHERE P.DTEXCLUSAO IS NULL
  AND (P.OBS2 <> 'FL' OR P.OBS2 IS NULL)
  AND P.REVENDA = 'S'
```

Ajuste conforme regra da base (ex.: outros códigos em OBS2).

---

## Agregação e arredondamento

Soma de valores monetários: use **ROUND** no item para não acumular erro:

```sql
SUM(ROUND(NVL(I.QT, 0) * NVL(I.PVENDA, 0), 2)) AS VALOR_BRUTO
```

Margem e percentuais: preferir calcular fora do SQL (ex.: em relatório) para controle de casas decimais.
