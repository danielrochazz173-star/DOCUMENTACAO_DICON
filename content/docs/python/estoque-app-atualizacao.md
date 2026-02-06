---
title: App de atualização de estoque
---

Aplicativo gráfico (tkinter) que atualiza estoque e produtos no Google Sheets em ciclo automático e permite disparar atualização manual.

## O que atualiza

1. **Estoque BEMBRASIL (Depto 102)** → planilha ESTOQUE_DIARIO_BEMBRASIL (mesmo fluxo do `enviar_estoque_bembrasil.py`).
2. **Produtos VILOG** → planilha VILOG, aba PRODUTOS (script `sincronizar_produtos_vilog.py`).

## Como executar

```bash
python "ESTOQUE PRODUTOS\app_atualizacao_estoque.py"
```

Ou duplo clique em `EXECUTAR_APP.bat`.

## Funcionalidades

- **Atualização automática** a cada 5 horas (configurável: variável `INTERVALO_HORAS` no código).
- **Atualização manual**: botões “Atualizar BEMBRASIL”, “Atualizar VILOG”, “Atualizar Ambos”.
- **Status**: última atualização, próxima atualização, tempo restante, estado (Aguardando / Atualizando / Concluído / Erro).
- Execução em thread para não travar a interface; barra de progresso durante a atualização.

## Interface

- Seção Estoque BEMBRASIL (Depto 102) com botão e status.
- Seção Produtos VILOG com botão e status.
- Botão “Atualizar Ambos” para rodar os dois fluxos em sequência.

## Requisitos

- Python 3.x com tkinter.
- Módulos do projeto: `enviar_estoque_bembrasil`, `sincronizar_produtos_vilog` (no path).
