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

## Método obrigatório (por repositório)

1. Liste a árvore completa de arquivos do repositório.
2. Leia o manifesto de dependências e a configuração (ex.: `pyproject.toml`,
   `package.json`, `config.*`, `.env.example`).
3. Leia o ponto de entrada e o registro de capacidades (ex.: servidor, rotas,
   registro de tools).
4. **Leia a implementação de cada capacidade**, não só a sua declaração: abra os
   arquivos de serviços/lógica de negócio, clientes de API, autenticação e
   segurança. Entenda o que o código realmente faz (chamadas externas feitas,
   permissões exigidas, filtros, paginação, cache, erros tratados).
5. **Abra todos os arquivos que registram capacidades** (todos os módulos de
   tools/rotas/comandos), um por um. Não basta listar a pasta nem ler só alguns
   arquivos: se um módulo não foi aberto, suas capacidades não podem entrar no
   catálogo nem ser deduzidas pelo nome do arquivo.
6. **Inventário e conferência:** monte a lista completa dos nomes registrados no
   código (ex.: cada `name="..."` de tool). Cada nome deve aparecer no catálogo,
   exatamente como no código, e nenhum nome pode aparecer se não estiver no
   código. Informe no topo de cada página o total de tools/capacidades
   encontradas e confira que a contagem da página bate com o inventário.
7. Só use README e `docs/` para comparar, nunca como fonte. Se o README divergir
   do código, o código vence: registre a divergência na seção "Divergências".
8. Se você não abriu o arquivo, você não pode afirmar nada sobre ele. Não deduza
   comportamento pelo nome de uma função.

Seu objetivo é identificar as capacidades funcionais oferecidas pelo software,
baseando-se no código-fonte e não apenas no README ou na documentação existente.

Para cada capacidade identificada, informe:

- Nome da capacidade
- Descrição
- Funcionalidade implementada
- Principais operações disponíveis
- Arquivos do código que sustentam essa conclusão (somente arquivos que você leu)

Não invente capacidades que não possam ser comprovadas pelo código. Confira
contagens (ex.: número de tools) e requisitos (ex.: versão do Python) no código.

Escreva o resultado em Markdown, servido pelo GitHub Pages:

- Uma página por repositório em `docs/<nome-do-repo>.md`, com front matter
  `title: Capabilities Catalog — <nome-do-repo>`, um H1 com o nome, o link
  `https://github.com/<owner>/<repo>` e a branch analisada logo abaixo.
- `docs/index.md` com o título `Capabilities Catalog`, e uma lista com link para
  cada página e o link do repositório de origem.

Cada página de repositório deve terminar com duas seções: "Arquivos analisados"
(lista dos arquivos que você de fato leu) e "Divergências com a documentação"
(onde README/docs não batem com o código, ou "nenhuma").

Abra um pull request com essas alterações.
