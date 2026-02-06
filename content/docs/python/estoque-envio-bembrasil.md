---
title: Envio de estoque BEMBRASIL
---

Script que busca estoque do **departamento 102** no Oracle e na planilha **VILOG**, soma os estoques por CODPROD e envia para a planilha Google Sheets **ESTOQUE_DIARIO_BEMBRASIL**, aba **ESTOQUE**.

## Fluxo

1. Busca estoque no Oracle (depto 102) – unidades e caixas.
2. Busca peso total (KG) na planilha VILOG, aba CALCULADO (coluna A = CODPROD, coluna J = PESO TOTAL KG).
3. Para cada produto: Oracle KG = caixas × QTUNITCX × PESOLIQ; soma com Vilog KG.
4. Ordena por estoque total KG (maior primeiro).
5. Limpa a aba ESTOQUE e insere cabeçalhos + linhas (Local Parceiro, Data, Empresa, EAN, CODFAB, Nome, Unidade KG, Qtd estoque KG).

## Como executar

```bash
python "ESTOQUE PRODUTOS\enviar_estoque_bembrasil.py"
```

Ou duplo clique em `EXECUTAR_ENVIAR_ESTOQUE.bat`.

## Configurações (no script)

- Oracle: host, porta, service, usuário/senha.
- Google: arquivo de credenciais (service account), nome da planilha e aba.
- `CODIGO_DEPTO = 102`, `LOCAL_PARCEIRO`, `EMPRESA_FIXA`.

## Estrutura da planilha destino

| Coluna | Campo              | Origem                    |
|--------|--------------------|---------------------------|
| A      | LOCAL PARCEIRO     | Fixo 37218268000104        |
| B      | DATA ATUAL         | DD/MM/YYYY                |
| C      | Empresa            | Fixo DICON ATACADAO...    |
| D      | EAN                | P.CODAUXILIAR             |
| E      | CODFAB             | P.CODFAB                  |
| F      | NOME DO PRODUTO    | P.DESCRICAO               |
| G      | UNIDADE DE MEDIDA  | KG                        |
| H      | QTD ESTOQUE (KG)   | Oracle KG + Vilog KG      |

## Dependências

- `oracledb`, `pandas`, `gspread`, `google-auth`
- Oracle Instant Client (ex.: `C:\oracle\instantclient_23_9`)
- Credenciais Google (JSON da service account) com acesso à planilha e à VILOG

## Observações

- Match Oracle ↔ Vilog é sempre por **CODPROD**.
- Inserção em lotes de 1000 linhas.
- Cabeçalhos formatados (negrito, fundo, linha congelada).
