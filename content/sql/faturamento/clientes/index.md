---
title: Faturamento por Cliente
---

Consulta base para listar clientes que compraram produtos específicos no período.

## Query base

```sql
SELECT
    CL.CODCLI,
    CL.CLIENTE,
    C.NUMNOTA,
    C.NUMPED,
    C.DATA,
    I.CODPROD,
    P.DESCRICAO AS PRODUTO,
    I.PVENDA,
    I.QT,
    ROUND(I.QT * I.PVENDA, 2) AS VALOR_TOTAL,
    C.CODUSUR,
    U.NOME AS VENDEDOR
FROM PCPEDC C
JOIN PCPEDI I ON C.NUMPED = I.NUMPED
JOIN PCCLIENT CL ON C.CODCLI = CL.CODCLI
JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
LEFT JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
WHERE C.DATA BETWEEN :data_inicio AND :data_fim
  AND I.CODPROD IN (:lista_codprod)
  AND C.CODFILIAL IN ('1', '98')
  AND C.POSICAO = 'F'
  AND C.DTCANCEL IS NULL
  AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
  AND NVL(I.BONIFIC, 'N') = 'N'
ORDER BY C.DATA DESC, CL.CODCLI, I.CODPROD
```

## Ajustes comuns

- Filtrar por preço: `ROUND(I.PVENDA, 2) = :preco`
- Filtrar por supervisor: `JOIN PCUSUARI U` e `U.CODSUPERVISOR IN (...)`
