---
title: Exportar produtos para Excel
---

Script que busca produtos no Oracle (Winthor) e gera um arquivo Excel formatado com: Código (CODPROD), Descrição, NCM (NBM), Código de Barras (CODAUXILIAR).

## Uso

```bash
python "ESTOQUE PRODUTOS\exportar_produtos_excel.py"
```

Ou duplo clique em `EXECUTAR_EXPORTAR_PRODUTOS.bat`.

## Arquivo gerado

- Nome: `Produtos_Oracle_YYYYMMDD_HHMMSS.xlsx`
- Pasta: mesmo diretório do script.

## Query utilizada

```sql
SELECT
    P.CODPROD AS CODIGO,
    P.DESCRICAO,
    P.NBM AS NCM,
    P.CODAUXILIAR AS CODIGO_BARRAS
FROM PCPRODUT P
ORDER BY P.DESCRICAO ASC
```

No script há filtros adicionais: `DTEXCLUSAO IS NULL`, `OBS2 <> 'FL'`, `REVENDA = 'S'`.

## Formatação do Excel

- Cabeçalho: fundo azul (#366092), texto branco, negrito.
- Bordas em todas as células.
- Primeira linha congelada.
- Filtro automático.
- Larguras: Código 12, Descrição 60, NCM 15, Código de Barras 20.

## Dependências

- `oracledb`, `pandas`, `openpyxl`
- Oracle Instant Client configurado
