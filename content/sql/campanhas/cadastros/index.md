---
title: Cadastros (Produtos, Vendedores, Supervisores)
---

Consultas auxiliares usadas no sistema das campanhas.

## Produtos

```sql
SELECT
    CODPROD,
    DESCRICAO,
    EMBALAGEM,
    QTUNITCX,
    CODFAB,
    CODEPTO
FROM PCPRODUT
WHERE CODPROD IN (:produtos)
ORDER BY CODPROD
```

## Vendedores

```sql
SELECT
    U.CODUSUR,
    U.NOME,
    U.CODSUPERVISOR,
    S.NOME AS NOME_SUPERVISOR
FROM PCUSUARI U
LEFT JOIN PCSUPERV S ON U.CODSUPERVISOR = S.CODSUPERVISOR
ORDER BY U.CODUSUR
```

## Supervisores

```sql
SELECT
    CODSUPERVISOR,
    NOME
FROM PCSUPERV
ORDER BY CODSUPERVISOR
```
