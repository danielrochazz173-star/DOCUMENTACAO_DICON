---
title: DICON - Documentação
cascade:
  - type: docs
---

Área de documentação interna da **DICON**. Consultas SQL e snippets para o dia a dia.

## Seções principais

- **[Conexão Oracle e Ambiente](sql/oracle-conexao/)** – Padrão de conexão e configuração segura
- **[Faturamento](sql/faturamento/)** – Cálculo completo, margem, devoluções, vendedores, clientes e departamentos
- **[Compras](sql/compras/)** – Compras por produto e departamento, positivação e compra x venda
- **[Campanhas](sql/campanhas/)** – Queries usadas no sistema Camil
- **[Estoque](sql/estoque/)** – Estoque por CODFAB, categoria, departamento, Oracle+Vilog (KG), exportar produtos

## Acessos rápidos (Faturamento)

- **[Faturamento líquido (base de cálculo)](sql/faturamento/faturamento-liquido/)**
- **[Margem e preço ideal](sql/faturamento/margem-preco/)**
- **[Devoluções (view oficial)](sql/faturamento/devolucoes/)**
- **[Faturamento por vendedor](sql/faturamento/vendedores/)**
- **[Faturamento por cliente](sql/faturamento/clientes/)**
- **[Faturamento por departamento](sql/faturamento/departamentos/)**

## Acessos rápidos (Compras)

- **[Compras por produto (Top N)](sql/compras/compras-por-produto/)**
- **[Compras por departamento](sql/compras/compras-por-departamento/)**
- **[Positivação por produto](sql/compras/positivacao-produto/)**
- **[Comparativo compra x venda](sql/compras/compra-venda/)**

## Conceitos (TI / desenvolvedores)

- **[JOINs e tabelas Winthor](sql/joins-e-tabelas-winthor/)** – Mapa das tabelas (PCPEDC, PCPEDI, PCEST, etc.) e como fazer os JOINs
- **[Padrões e filtros de negócio](sql/padroes-e-filtros/)** – Filtros de pedido, NVL, DECODE, datas, CTE

Use a **busca** no topo para achar rápido o que precisar.
