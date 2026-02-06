---
title: Pedidos (PCPEDC / PCPEDI)
---

Consultas para listar e filtrar pedidos de venda no Winthor. Cabeçalho em **PCPEDC**, itens em **PCPEDI**; o vínculo é sempre **NUMPED**.

## Páginas

- [Listar pedidos (cabeçalho e resumo)](listar-pedidos/) – Por período, filial, posição; com cliente, vendedor e valor total
- [Itens do pedido](itens-do-pedido/) – Detalhe por NUMPED; filtros por cliente e vendedor

## Tabelas

| Tabela | Papel | Chave |
|--------|--------|--------|
| **PCPEDC** | Cabeçalho do pedido | NUMPED |
| **PCPEDI** | Itens do pedido | NUMPED + CODPROD (linha do item) |

Campos principais no cabeçalho: `NUMPED`, `DATA`, `NUMNOTA`, `CODCLI`, `CODUSUR`, `CODFILIAL`, `POSICAO`, `DTCANCEL`, `CONDVENDA`.  
Nos itens: `CODPROD`, `QT`, `PVENDA`, `VLCUSTOFIN`, `BONIFIC`.  
Ver [JOINs e tabelas](../joins-e-tabelas-winthor/) e [Padrões e filtros](../padroes-e-filtros/) para filtros de venda válida.
