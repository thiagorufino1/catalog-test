---
name: "Capabilities Catalog"
description: "Analisa código e gera um catálogo de capacidades."
on:
  workflow_dispatch:

engine:
  id: copilot
  model: gpt-5

permissions:
  contents: read

tools:
  github:
    toolsets: [default]
---

# Capabilities Catalog

Analise o código-fonte disponível nos repositórios autorizados.

Seu objetivo é identificar as capacidades funcionais oferecidas pelo software,
baseando-se no código-fonte e não apenas no README ou na documentação existente.

Para cada capacidade identificada, informe:

- Nome da capacidade
- Descrição
- Funcionalidade implementada
- Principais operações disponíveis
- Arquivos do código que sustentam essa conclusão

Não invente capacidades que não possam ser comprovadas pelo código.

Produza o resultado em Markdown.
