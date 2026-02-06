---
title: Estoque (PCEST)
---

Consultas e padrões para puxar estoque no Winthor (Oracle) de várias formas: por CODFAB, por categoria, por departamento, em caixas, em KG, e integração Oracle + Vilog.

## Formas de puxar estoque (SQL)

- [Regra de estoque em caixas](estoque-regra-caixas/) – Fórmula base (QTESTGER, QTRESERV, QTBLOQUEADA, QTUNITCX)
- [Estoque por CODFAB](estoque-por-codfab/) – Por código do fabricante, com departamento e EAN
- [Estoque por categoria](estoque-por-categoria/) – Congelado/Resfriado (100/101) e demais categorias
- [Estoque por departamento](estoque-por-departamento/) – Query Oracle por CODEPTO (ex.: depto 102)
- [Estoque Oracle + Vilog (KG)](estoque-oracle-vilog/) – Soma estoque Oracle com peso da planilha Vilog
- [Exportar produtos (lista)](exportar-produtos/) – CODPROD, Descrição, NCM, EAN para Excel

## Scripts Python (estoque)

- [Envio de estoque BEMBRASIL](../../python/estoque-envio-bembrasil/) – Oracle + Vilog → Google Sheets
- [Exportar produtos para Excel](../../python/estoque-exportar-excel/) – Listagem formatada
- [App de atualização de estoque](../../python/estoque-app-atualizacao/) – Atualização automática (5h) e manual

## Tabelas principais

- `PCEST` – estoque por produto/filial (QTESTGER, QTRESERV, QTBLOQUEADA)
- `PCPRODUT` – produto (QTUNITCX, PESOLIQ, CODEPTO, CODFAB, CODCATEGORIA)
- `PCDEPTO` – departamento
