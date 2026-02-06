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

## Tabelas principais

| Tabela | Chave | Uso |
|--------|--------|-----|
| **PCEST** | CODPROD + CODFILIAL | Estoque: QTESTGER, QTRESERV, QTBLOQUEADA. Sempre filtrar CODFILIAL (ex.: '1'). |
| **PCPRODUT** | CODPROD | Produto: QTUNITCX, PESOLIQ, CODEPTO, CODFAB, CODAUXILIAR (EAN), CODCATEGORIA. JOIN com PCEST por CODPROD (e CODFILIAL). |
| **PCDEPTO** | CODEPTO | Departamento. JOIN a partir de PCPRODUT.CODEPTO. |

Estoque em caixas: `(QTESTGER - QTRESERV - QTBLOQUEADA) / NULLIF(QTUNITCX, 1)`. Ver [Regra de estoque em caixas](estoque-regra-caixas/) e [JOINs e tabelas](/docs/sql/joins-e-tabelas-winthor/).
