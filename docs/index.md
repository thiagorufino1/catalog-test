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
- Ações em dispositivos (sync, restart, scan, locate; retire, wipe e delete com aprovação)
- Workflow de aprovação (aprovar, negar, executar) com expiração
- Operações em lote (sync e restart, até 20 dispositivos)
- Governança: LAPS, chaves BitLocker (com aprovação) e RBAC
- Relatórios, auditoria e Endpoint Analytics
- Scripts de remediação proativa
- Gestão de atualizações (anéis e perfis de update)
- Windows Autopilot (listagem, importação)
- Windows Autopatch

**Ferramentas expostas:** 47

**Tecnologias:** Python 3.12+, FastMCP, Microsoft Graph API

---

## Informações Técnicas Comuns

Ambos os repositórios compartilham:

- **Arquitetura:** FastMCP server com tools expostas via MCP
- **Autenticação:** Azure AD (MSAL) com Client Credentials flow
- **API:** Microsoft Graph API (o `mcp-intune` também usa endpoints beta, com `ALLOW_BETA_APIS=true`)
- **Cliente HTTP:** httpx com retry automático e throttling
- **Logging:** structlog com formato JSON
- **Cache:** TTL-based para operações de leitura
- **Validação:** Pydantic para schemas e tipos

## Metodologia de Análise

Este catálogo foi gerado por um agente de IA que lê o código-fonte dos repositórios (manifesto de dependências, ponto de entrada e arquivos de tools) e depois revisado por uma pessoa. A lista de tools de cada página foi conferida contra os nomes registrados no código: `mcp-msteams` com 27 e `mcp-intune` com 47.

**Importante:** As capacidades vêm do código-fonte, não do README. Quando há divergência entre documentação e código, o código prevalece. Detalhes de implementação dos serviços e do cliente do Graph podem não estar cobertos; veja a seção "Arquivos analisados" de cada página.
