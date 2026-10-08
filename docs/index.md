---
title: Capabilities Catalog
---

# Capabilities Catalog

Catálogo de capacidades funcionais de servidores MCP (Model Context Protocol) para administração do Microsoft 365.

## Repositórios Analisados

### 1. [mcp-msteams](mcp-msteams.md)
**Repositório:** [https://github.com/thiagorufino1/mcp-msteams](https://github.com/thiagorufino1/mcp-msteams)  
**Descrição:** Servidor MCP para administração e suporte do Microsoft Teams.

**Principais Capacidades:**
- Gerenciamento de usuários (perfil, presença, políticas)
- Gerenciamento de Teams (canais, membros, configurações)
- Análise tenant-wide (estatísticas, Teams órfãos)
- Qualidade de chamadas e diagnóstico
- Reuniões e participantes
- Relatórios de atividade
- Monitoramento de incidentes do serviço Microsoft Teams

**Tecnologias:** Python 3.11+, FastMCP, Microsoft Graph API

---

### 2. [mcp-intune](mcp-intune.md)
**Repositório:** [https://github.com/thiagorufino1/mcp-intune](https://github.com/thiagorufino1/mcp-intune)  
**Descrição:** Servidor MCP para gerenciamento e operações do Microsoft Intune.

**Principais Capacidades:**
- Consulta e busca de dispositivos gerenciados
- Ações em dispositivos (sync, restart, retire, wipe, delete)
- Workflow de aprovação para ações destrutivas
- Operações em lote (bulk actions)
- Windows Autopilot (listagem, importação)
- Gerenciamento de updates
- Scripts PowerShell remotos
- Relatórios e governança
- Windows Autopatch

**Tecnologias:** Python 3.12+, FastMCP, Microsoft Graph API

---

## Informações Técnicas Comuns

Ambos os repositórios compartilham:

- **Arquitetura:** FastMCP server com tools expostas via MCP
- **Autenticação:** Azure AD (MSAL) com Client Credentials flow
- **API:** Microsoft Graph API v1.0
- **Cliente HTTP:** httpx com retry automático e throttling
- **Logging:** structlog com formato JSON
- **Cache:** TTL-based para operações de leitura
- **Validação:** Pydantic para schemas e tipos

## Metodologia de Análise

Este catálogo foi gerado através da análise sistemática do código-fonte de cada repositório:

1. Leitura dos manifestos de dependências (`pyproject.toml`)
2. Análise dos pontos de entrada (`server.py`)
3. Leitura completa de todos os arquivos de tools
4. Análise dos services e implementação das capacidades
5. Verificação de clientes de API e autenticação

**Importante:** As capacidades documentadas foram identificadas através da leitura do código-fonte real, não apenas da documentação README. Quando há divergência entre documentação e código, o código prevalece.
