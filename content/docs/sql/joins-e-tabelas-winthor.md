---
title: JOINs e tabelas (Winthor)
---

Referência das principais tabelas do Winthor e como relacioná-las nas queries. Use como mapa para montar JOINs corretos.

## Pedido de venda (cabeçalho × itens)

| Tabela | Papel | Chave principal | Chave para JOIN |
|--------|--------|------------------|-----------------|
| **PCPEDC** | Cabeçalho do pedido | NUMPED | NUMPED |
| **PCPEDI** | Itens do pedido | NUMPED + CODPROD (item) | NUMPED →

Um pedido tem vários itens. O JOIN é sempre pelo número do pedido:

```sql
FROM PCPEDC C
JOIN PCPEDI I ON C.NUMPED = I.NUMPED
```

**INNER JOIN** → só pedidos que têm itens (e itens que têm pedido).  
**LEFT JOIN** → se quiser pedidos mesmo sem itens (raro).

Campos úteis no cabeçalho: `DATA`, `CODCLI`, `CODUSUR`, `CODFILIAL`, `POSICAO`, `DTCANCEL`, `CONDVENDA`.  
Nos itens: `CODPROD`, `QT`, `PVENDA`, `VLCUSTOFIN`, `BONIFIC`.

---

## Produto e estoque

| Tabela | Papel | Chave principal | Chave para JOIN |
|--------|--------|------------------|-----------------|
| **PCPRODUT** | Cadastro de produto | CODPROD | CODPROD |
| **PCEST** | Estoque por produto e filial | CODPROD + CODFILIAL | CODPROD (e CODFILIAL) |

Estoque é por filial. Na maioria das análises usamos `CODFILIAL = '1'`:

```sql
FROM PCPRODUT P
LEFT JOIN PCEST E ON P.CODPROD = E.CODPROD AND E.CODFILIAL = '1'
```

**LEFT JOIN** → produto aparece mesmo sem registro em PCEST (estoque zero).  
Campos de estoque: `QTESTGER`, `QTRESERV`, `QTBLOQUEADA`.  
No produto: `QTUNITCX`, `PESOLIQ`, `CODEPTO`, `CODFAB`, `CODAUXILIAR` (EAN).

---

## Pessoas e estrutura

| Tabela | Papel | Chave para JOIN |
|--------|--------|------------------|
| **PCUSUARI** | Vendedores | CODUSUR (em PCPEDC) |
| **PCSUPERV** | Supervisores | CODSUPERVISOR (em PCUSUARI) |
| **PCCLIENT** | Clientes | CODCLI (em PCPEDC) |
| **PCDEPTO** | Departamentos | CODEPTO (em PCPRODUT) |

Exemplo: pedido + vendedor + supervisor

```sql
FROM PCPEDC C
JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
LEFT JOIN PCSUPERV S ON U.CODSUPERVISOR = S.CODSUPERVISOR
```

Exemplo: itens + produto + departamento

```sql
FROM PCPEDI I
JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
LEFT JOIN PCDEPTO D ON P.CODEPTO = D.CODEPTO
```

---

## Compras / entrada

| Tabela | Papel | Relacionamento |
|--------|--------|----------------|
| **PCNFENT** | Nota fiscal de entrada | NUMTRANSENT |
| **PCMOV** | Movimentação (entrada/saída) | NUMTRANSENT + CODPROD, CODOPER = 'E' para entrada |

```sql
FROM PCNFENT N
JOIN PCMOV M ON N.NUMTRANSENT = M.NUMTRANSENT AND M.CODOPER = 'E'
JOIN PCPRODUT P ON M.CODPROD = P.CODPROD
```

---

## Devoluções

Usar a view **VIEW_DEVOL_RESUMO_FATURAMENTO**: já traz valor e custo de devolução por produto/vendedor/data. Campos úteis: `CODPROD`, `CODUSUR`, `DTENT`, `VLDEVOLUCAO`, `VLCUSTOFIN`. JOIN por `CODPROD` (e opcionalmente `CODUSUR`) quando for somar com vendas.

---

## Resumo: cadeia pedido → valor

```
PCPEDC (pedido)
  └── JOIN PCPEDI ON NUMPED         → itens
        └── JOIN PCPRODUT ON CODPROD → produto, CODEPTO, QTUNITCX, PESOLIQ
  └── JOIN PCUSUARI ON CODUSUR     → vendedor, CODSUPERVISOR
  └── JOIN PCCLIENT ON CODCLI      → cliente (quando precisar nome/CNPJ)
```

Para estoque:

```
PCPRODUT (produto)
  └── LEFT JOIN PCEST ON CODPROD AND CODFILIAL = '1'  → estoque
  └── LEFT JOIN PCDEPTO ON CODEPTO                    → departamento
```

Para compras:

```
PCNFENT (NF entrada)
  └── JOIN PCMOV ON NUMTRANSENT AND CODOPER = 'E'    → itens entrada
        └── JOIN PCPRODUT ON CODPROD
        └── JOIN PCDEPTO ON P.CODEPTO
```
