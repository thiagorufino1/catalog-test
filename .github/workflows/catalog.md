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

## Regras de veracidade

- O catálogo deve refletir **exatamente** o código. Não invente nada: nenhuma
  tool, operação, permissão, parâmetro, número ou versão que você não tenha visto
  no código.
- **Não confie no README nem em `docs/`**: podem estar desatualizados ou errados.
  Use-os, no máximo, para encontrar divergências. Em caso de conflito, o código
  vence.
- Confira contagens (ex.: número de tools) e requisitos (ex.: versão do Python)
  no código.
- Se algo não pôde ser verificado, escreva "não verificado" em vez de supor.

## Saída

Escreva em português (pt-BR), em Markdown, servido pelo GitHub Pages. Mantenha
nomes técnicos (tools, arquivos, permissões) exatamente como no código.

Todas as páginas começam com uma linha de navegação que funciona como abas,
com a mesma lista em todas, na mesma ordem: `Summary` primeiro e depois um link
por repositório, ex.: `[Summary](index.md) | [mcp-a](mcp-a.md) | [mcp-b](mcp-b.md)`.

### `docs/index.md` (aba Summary)

Front matter `title: Capabilities Catalog`, a linha de navegação, o H1
`Capabilities Catalog` e uma breve frase dizendo quantos repositórios foram
analisados. Em seguida, **uma seção por repositório, todas com exatamente o mesmo
formato**:

- H2 com o nome do repositório (link para a página dele) e, abaixo, o link do
  GitHub (`https://github.com/<owner>/<repo>`) e a branch.
- **Descrição**: 1 a 3 frases.
- **Principais capacidades**: lista curta, uma linha por domínio.
- **Tecnologias**: linguagem/versão e dependências principais.
- **Ferramentas**: o total, e **uma tabela com todas as tools** (colunas
  `Tool | Descrição`, uma linha por tool, na ordem do código).

Não inclua seções de informações técnicas comuns nem de metodologia.

### `docs/<nome-do-repo>.md` (uma aba por repositório)

Front matter `title: Capabilities Catalog — <nome-do-repo>`, a linha de
navegação, H1 com o nome, link do GitHub e branch. Depois, nesta ordem:

1. **Visão geral**: o que o software faz, em um parágrafo, com o total de
   tools/capacidades.
2. **Uma seção numerada por domínio funcional** (ex.: "1. Gestão de Usuários").
   Em cada uma: descrição; funcionalidade; **tabela de operações**
   (`Operação | Descrição | Implementação`, uma linha por tool, com o nome exato
   e como é implementada, ex.: endpoint ou permissão usados); características
   relevantes (limites, paginação, parâmetros, aprovações); e **Arquivos de
   suporte** (os arquivos de código daquele domínio que você leu).
3. **Arquitetura**: servidor e transporte, configuração, segurança, utilitários,
   logging e integração com APIs externas, cada um com o arquivo correspondente.
4. **Formatos de saída**, se o código define mais de um.
5. **Dependências e requisitos**: versão da linguagem, dependências e
   **permissões exigidas** (as que estão no código/configuração).
6. **Resumo**: totais conferidos (tools, domínios).
7. **Arquivos analisados**: somente os arquivos que você de fato abriu.
8. **Divergências com a documentação**: onde README/docs não batem com o código,
   ou "nenhuma".

Abra um pull request com essas alterações.
