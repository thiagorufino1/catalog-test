# Capabilities Catalog

Gera, de forma automática, um **catálogo de capacidades** de repositórios de software e o publica como um site no GitHub Pages.

Um agente de IA (GitHub Agentic Workflows, motor Claude) lê o **código-fonte** dos repositórios listados em [mcps.json](mcps.json), entende o que cada um realmente faz e escreve uma página por repositório. O resultado chega como Pull Request, para revisão humana antes de ser publicado.

- Site publicado: https://thiagorufino1.github.io/catalog-test/
- Definição do workflow: [.github/workflows/catalog.md](.github/workflows/catalog.md)

## Como funciona

```
mcps.json ──► Workflow (manual) ──► Agente lê o código ──► Pull Request ──► Revisão ──► Merge ──► GitHub Pages
```

| # | Etapa | O que acontece |
|---|-------|----------------|
| 1 | **Lista de repositórios** | [mcps.json](mcps.json) define quais repositórios (`owner/repo` + `branch`) serão avaliados. Adicionar um repositório é adicionar uma linha. |
| 2 | **Disparo** | O workflow roda por `workflow_dispatch` (botão "Run workflow" em Actions, ou `gh workflow run "Capabilities Catalog"`). Não há agendamento. |
| 3 | **Activation** | Job de preparação do gh-aw: valida o ambiente e monta o contexto do agente. |
| 4 | **Agent** | O agente (Claude) roda em sandbox, com acesso de **somente leitura** à API do GitHub. Para cada repositório do `mcps.json`: lista a árvore de arquivos, lê manifesto e configuração, ponto de entrada e a **implementação** de cada capacidade (serviços, clientes de API, autenticação, segurança). README e `docs/` só servem de comparação. |
| 5 | **Escrita das páginas** | O agente escreve `docs/<repo>.md` (capacidades, operações, arquivos que sustentam cada conclusão, "Arquivos analisados" e "Divergências com a documentação") e `docs/index.md` (índice com links). |
| 6 | **Detection** | Job de segurança do gh-aw: examina a saída do agente em busca de vazamento de segredos ou conteúdo malicioso antes de qualquer escrita. |
| 7 | **Safe outputs** | O agente **não escreve no repositório**. Ele só propõe; este job, com permissões próprias, abre o Pull Request (prefixo `[catalog]`) com as alterações em `docs/`. |
| 8 | **Revisão humana** | Uma pessoa confere o conteúdo contra o código (contagens, versões, permissões) e faz o merge. |
| 9 | **Publicação** | O GitHub Pages serve a pasta `/docs` da `main`. Cada merge republica o site. |

### Por que é seguro

- O agente tem apenas `contents: read`. Toda escrita passa por `safe-outputs`.
- O token de leitura de repositórios privados tem escopo mínimo (somente Contents: read nos repositórios desejados).
- Nada é publicado sem PR e revisão.
- Atenção: o agente lê código de terceiros e está sujeito a *prompt injection*. Por isso a revisão do PR é parte do processo, não opcional.

## Estrutura do repositório

| Arquivo | Função |
|---------|--------|
| [mcps.json](mcps.json) | Lista de repositórios a catalogar. |
| [.github/workflows/catalog.md](.github/workflows/catalog.md) | **Fonte do workflow**: configuração (motor, modelo, permissões, saídas) e o prompt do agente. É o arquivo que você edita. |
| [.github/workflows/catalog.lock.yml](.github/workflows/catalog.lock.yml) | Workflow compilado que o GitHub executa de fato. **Gerado**, não edite à mão. |
| [.github/aw/actions-lock.json](.github/aw/actions-lock.json) | Versões fixadas (SHA) das actions usadas. Gerado. |
| `docs/` | Saída: páginas servidas pelo Pages. Criada pelos PRs do agente. |

## Configuração

Em [catalog.md](.github/workflows/catalog.md):

```yaml
engine:
  id: claude
  model: claude-haiku-4-5-20251001   # trocar o modelo = editar esta linha
permissions:
  contents: read
tools:
  github:
    toolsets: [repos]
safe-outputs:
  create-pull-request:
    title-prefix: "[catalog] "
```

**Depois de qualquer edição no `.md`, rode `gh aw compile`** para regenerar o `.lock.yml`. Sem isso, nada muda.

Para catalogar outro repositório, edite o `mcps.json`:

```json
{
  "repositories": [
    { "repo": "owner/repo-publico", "branch": "main" },
    { "repo": "owner/repo-privado", "branch": "main" }
  ]
}
```

## Como implantar (replicar)

### Pré-requisitos

- Repositório no GitHub (destino do catálogo) com GitHub Actions habilitado.
- [GitHub CLI](https://cli.github.com/) (`gh`) autenticado.
- Chave de API da Anthropic.

### Passo a passo

1. **Criar o repositório** e clonar. Instalar a extensão do gh-aw:
   ```bash
   gh extension install github/gh-aw
   ```
2. **Copiar** para o novo repositório: `.github/workflows/catalog.md` e `mcps.json`. Ajustar o `mcps.json` com os seus repositórios.
3. **Compilar o workflow**:
   ```bash
   gh aw compile
   ```
   Isso gera `catalog.lock.yml` e `.github/aw/actions-lock.json`. Se o compilador pedir revisão de novos secrets, confira e rode `gh aw compile --approve`.
4. **Criar o secret da API do modelo**:
   ```bash
   gh secret set ANTHROPIC_API_KEY
   ```
5. **Permitir que Actions crie PRs**: Settings → Actions → General → "Allow GitHub Actions to create and approve pull requests". (Ou via API: `gh api -X PUT repos/<owner>/<repo>/actions/permissions/workflow -f default_workflow_permissions=read -F can_approve_pull_request_reviews=true`.)
6. **Commit e push** de tudo para a `main`.
7. **Primeira execução**:
   ```bash
   gh workflow run "Capabilities Catalog"
   gh run watch
   ```
8. **Revisar e fazer merge** do PR `[catalog] ...` aberto pelo agente.
9. **Habilitar o Pages**: Settings → Pages → Deploy from a branch → `main` / `/docs`.

### Repositórios privados

O `GITHUB_TOKEN` padrão só enxerga o próprio repositório. Para ler repositórios privados:

1. Criar um **fine-grained PAT** (Settings → Developer settings), com acesso apenas aos repositórios desejados e permissão **Contents: Read-only** (Metadata: Read vem junto). Use expiração curta.
2. Salvar como secret com o nome especial que o gh-aw já reconhece (não exige mudar o workflow):
   ```bash
   gh secret set GH_AW_GITHUB_MCP_SERVER_TOKEN
   ```
3. Adicionar o repositório ao `mcps.json`.

> **Cuidado com a publicação.** Em contas e planos que só têm Pages público, o catálogo de um repositório privado ficaria visível na internet (nomes de arquivos, estrutura, permissões). Use Pages privado (GitHub Enterprise Cloud) ou não faça merge/publique esse conteúdo.

### Uso em organização (GitHub Enterprise Cloud)

Recomendações ao levar para uma empresa:

- Trocar o PAT pessoal por um **GitHub App** da organização (Contents: Read-only, instalado só nos repositórios a catalogar). O gh-aw suporta `tools.github.github-app` e gera um token de curta duração a cada execução:
  ```yaml
  tools:
    github:
      toolsets: [repos]
      github-app:
        client-id: ${{ vars.APP_ID }}
        private-key: ${{ secrets.APP_PRIVATE_KEY }}
        owner: "sua-org"
        repositories: ["*"]   # o alcance real é o da instalação do App
  ```
- Guardar `APP_ID` / `APP_PRIVATE_KEY` e a chave da API como secrets/variables **da organização**.
- Habilitar **Pages com acesso privado** (visibilidade restrita à organização) antes do primeiro merge.
- Proteger a `main` (revisão obrigatória) e manter o merge dos PRs do catálogo como etapa humana.
- Validar com Segurança a política de envio de código ao provedor do modelo.

## Operação

| Tarefa | Como |
|--------|------|
| Atualizar o catálogo | `gh workflow run "Capabilities Catalog"`, revisar o PR e fazer merge. |
| Adicionar/remover repositório | Editar `mcps.json` e rodar o workflow. |
| Trocar o modelo | Editar `engine.model` em `catalog.md`, `gh aw compile`, commit. |
| Mudar o que o agente analisa/escreve | Editar o prompt (corpo do `catalog.md`), `gh aw compile`, commit. |
| Ver o que o agente leu | Logs da execução em Actions; procure as chamadas `get_file_contents`. |

## Limitações conhecidas

- **Qualidade depende do modelo e da revisão.** O agente pode errar contagens, versões e detalhes. Nas primeiras execuções ele afirmou, por exemplo, número de operações e requisitos incorretos que só a conferência com o código revelou. Modelos menores tendem a ler menos arquivos: confira a seção "Arquivos analisados" de cada página.
- Execução **manual**: não há agendamento nem gatilho por mudança nos repositórios avaliados.
- O catálogo é um retrato do código na data da execução.
- Cada execução consome créditos da API do modelo.
