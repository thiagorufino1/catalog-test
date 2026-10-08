---
name: "Capabilities Catalog"
description: "Analisa código e gera um catálogo de capacidades."
on:
  workflow_dispatch:

engine:
  id: claude
  model: claude-sonnet-4-5

permissions:
  contents: read

tools:
  github:
    toolsets: [repos]

safe-outputs:
  create-pull-request:
    title-prefix: "[catalog] "
    draft: false
---

# Capabilities Catalog

Analise o código-fonte do repositório `thiagorufino1/mcp-msteams` (público, branch `main`), lendo-o com as ferramentas do GitHub (não o repositório atual, que serve só para publicar o resultado).

Seu objetivo é identificar as capacidades funcionais oferecidas pelo software,
baseando-se no código-fonte e não apenas no README ou na documentação existente.

Para cada capacidade identificada, informe:

- Nome da capacidade
- Descrição
- Funcionalidade implementada
- Principais operações disponíveis
- Arquivos do código que sustentam essa conclusão

Não invente capacidades que não possam ser comprovadas pelo código.

Escreva o resultado em Markdown no arquivo `docs/index.md` (página servida
pelo GitHub Pages, com front matter `title: Capabilities Catalog — <nome do repo>`). O topo da página deve ter um H1 com o nome do repositório avaliado e, logo abaixo,
o link para ele (`https://github.com/<owner>/<repo>`) e a branch analisada. Abra um
pull request com essa alteração.
