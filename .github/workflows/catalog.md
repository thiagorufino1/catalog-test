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

Leia o arquivo `mcps.json` na raiz do repositório atual. Ele lista, em
`repositories`, os repositórios públicos a avaliar (`repo` = `owner/nome`,
`branch`). Analise o código-fonte de **cada** um, lendo-o com as ferramentas do
GitHub. Não use o repositório atual como alvo: ele serve só para publicar o
resultado.

Seu objetivo é identificar as capacidades funcionais oferecidas pelo software,
baseando-se no código-fonte e não apenas no README ou na documentação existente.

Para cada capacidade identificada, informe:

- Nome da capacidade
- Descrição
- Funcionalidade implementada
- Principais operações disponíveis
- Arquivos do código que sustentam essa conclusão

Não invente capacidades que não possam ser comprovadas pelo código. Confira
contagens (ex.: número de tools) e requisitos (ex.: versão do Python) no código.

Escreva o resultado em Markdown, servido pelo GitHub Pages:

- Uma página por repositório em `docs/<nome-do-repo>.md`, com front matter
  `title: Capabilities Catalog — <nome-do-repo>`, um H1 com o nome, o link
  `https://github.com/<owner>/<repo>` e a branch analisada logo abaixo.
- `docs/index.md` com o título `Capabilities Catalog`, e uma lista com link para
  cada página e o link do repositório de origem.

Abra um pull request com essas alterações.
