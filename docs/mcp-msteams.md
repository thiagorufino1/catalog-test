---
title: Capabilities Catalog — mcp-msteams
---

# mcp-msteams

**Repositório:** [https://github.com/thiagorufino1/mcp-msteams](https://github.com/thiagorufino1/mcp-msteams)  
**Branch analisada:** main

## Descrição Geral

Servidor MCP (Model Context Protocol) para administração e suporte do Microsoft Teams. Oferece acesso programático ao Microsoft Graph API para consulta de usuários, teams, chamadas, reuniões, relatórios e incidentes do Microsoft 365.

## Tecnologias e Requisitos

- **Python:** ≥3.11
- **Dependências principais:**
  - fastmcp ≥2.0 (framework MCP)
  - httpx ≥0.27 (cliente HTTP assíncrono)
  - msal ≥1.28 (autenticação Microsoft)
  - pydantic ≥2.7 (validação de dados)
  - tenacity ≥8.3 (retry com backoff)
  - structlog ≥24.1 (logging estruturado)
- **Autenticação:** Azure Active Directory (tenant_id, client_id, client_secret)
- **Transporte:** HTTP (padrão) ou stdio

## Capacidades Funcionais

### 1. Gerenciamento de Usuários

#### 1.1 Visão Geral de Usuário
- **Tool:** `get_user_overview`
- **Descrição:** Combina perfil, presença e participação em Teams em uma única chamada
- **Operações:**
  - Consulta perfil do Azure AD
  - Status de presença no Teams (via `/users/{upn}/presence`)
  - Lista de Teams participantes
- **Entrada:** UPN (email completo)
- **Saída:** Objeto combinado com profile, presence, joinedTeams
- **Arquivos:** `src/mcp_msteams/tools/users_tools.py`, `src/mcp_msteams/services/users_service.py`

#### 1.2 Perfil de Usuário
- **Tool:** `get_user_profile`
- **Descrição:** Retorna perfil completo do Azure AD (nome, cargo, departamento, telefone)
- **API:** `/users/{upn}`
- **Arquivos:** `src/mcp_msteams/tools/users_tools.py`, `src/mcp_msteams/services/users_service.py`

#### 1.3 Presença
- **Tool:** `get_user_presence`
- **Descrição:** Consulta status de presença no Teams (Available, Busy, Away, Offline, etc.)
- **API:** `/users/{upn}/presence`
- **Cache:** 30 segundos (configurável via `cache_ttl_presence`)
- **Permissão:** Presence.Read.All
- **Arquivos:** `src/mcp_msteams/tools/users_tools.py`, `src/mcp_msteams/services/users_service.py`

#### 1.4 Políticas Atribuídas
- **Tool:** `get_user_assigned_policies`
- **Descrição:** Lista políticas do Teams atribuídas ao usuário (meeting, calling, messaging)
- **Uso:** Troubleshooting de recursos faltantes (gravação, chamadas externas)
- **Permissão:** TeamsUserConfiguration.Read.All
- **Arquivos:** `src/mcp_msteams/tools/users_tools.py`, `src/mcp_msteams/services/users_service.py`

#### 1.5 Listar Teams do Usuário
- **Tool:** `list_user_teams`
- **Descrição:** Lista todos os Teams dos quais o usuário é membro, incluindo team IDs (GUIDs)
- **API:** `/users/{upn}/joinedTeams`
- **Arquivos:** `src/mcp_msteams/tools/users_tools.py`, `src/mcp_msteams/services/users_service.py`

#### 1.6 Busca de Usuários
- **Tool:** `search_user`
- **Descrição:** Busca usuários por nome ou fragmento de email
- **Retorno:** Até 10 usuários com displayName, UPN, cargo e departamento
- **Uso:** Resolução de nome para UPN antes de outras operações
- **Arquivos:** `src/mcp_msteams/tools/users_tools.py`, `src/mcp_msteams/services/users_service.py`

### 2. Gerenciamento de Teams

#### 2.1 Canais de um Team
- **Tool:** `list_team_channels`
- **Descrição:** Lista todos os canais de um Team (standard, private, shared)
- **API:** `/teams/{team_id}/channels`
- **Arquivos:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

#### 2.2 Membros de um Team
- **Tool:** `list_team_members`
- **Descrição:** Lista todos os membros de um Team específico
- **API:** `/teams/{team_id}/members`
- **Arquivos:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

#### 2.3 Proprietários de um Team
- **Tool:** `get_team_owners`
- **Descrição:** Retorna os proprietários (owners) de um Team
- **API:** `/groups/{team_id}/owners`
- **Arquivos:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

#### 2.4 Configurações de um Team
- **Tool:** `get_team_settings`
- **Descrição:** Retorna configurações e metadados de um Team
- **API:** `/teams/{team_id}`
- **Arquivos:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

#### 2.5 Configurações de Canal
- **Tool:** `get_channel_settings`
- **Descrição:** Retorna configurações de um canal específico dentro de um Team
- **API:** `/teams/{team_id}/channels/{channel_id}`
- **Arquivos:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

#### 2.6 Verificar Canais Privados/Compartilhados
- **Tool:** `check_private_shared_channels`
- **Descrição:** Identifica canais privados e compartilhados em um Team
- **Arquivos:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

#### 2.7 Detectar Team Órfão (sem membros)
- **Tool:** `detect_orphaned_team`
- **Descrição:** Verifica se um Team específico não possui membros
- **Arquivos:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

#### 2.8 Detectar Team sem Proprietário
- **Tool:** `detect_team_without_owner`
- **Descrição:** Verifica se um Team específico não possui owners
- **Arquivos:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

### 3. Análise Tenant-Wide de Teams

#### 3.1 Listar Todos os Teams
- **Tool:** `list_all_teams`
- **Descrição:** Lista todos os Teams do tenant com paginação
- **Funcionalidades:**
  - Filtro por visibilidade (Public/Private)
  - Incluir contagens de membros e owners (opcional)
  - Paginação com `top` e `skip`
- **API:** `/groups` com filtros
- **Limitação:** Filtro de visibilidade é client-side (Graph API não suporta `$filter=visibility`)
- **Arquivos:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

#### 3.2 Estatísticas do Tenant
- **Tool:** `get_tenant_teams_stats`
- **Descrição:** Retorna estatísticas consolidadas de todos os Teams
- **Métricas:**
  - Total de Teams
  - Breakdown por visibilidade (public/private)
  - Teams órfãos (sem owners)
  - Teams sem membros
- **Arquivos:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

#### 3.3 Listar Teams Órfãos
- **Tool:** `list_orphaned_teams`
- **Descrição:** Lista todos os Teams do tenant sem owners
- **API:** `/groups` com filtro `owners/$count eq 0`
- **Arquivos:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

#### 3.4 Listar Teams sem Membros
- **Tool:** `list_teams_without_members`
- **Descrição:** Lista Teams sem membros
- **Implementação:** Scan paginado (Graph API não suporta filtro direto)
- **Limite:** Configurável via `graph_team_scan_max_teams`
- **Arquivos:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

#### 3.5 Ranking por Membros
- **Tool:** `list_teams_by_member_count`
- **Descrição:** Lista os maiores Teams ordenados por número de membros
- **Parâmetros:** `top` (default 10, max 50)
- **Arquivos:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

### 4. Qualidade de Chamadas

#### 4.1 Resumo de Qualidade
- **Tool:** `get_call_quality_summary`
- **Descrição:** Estatísticas de chamadas de um usuário nos últimos N dias
- **Métricas:** Total, falhas, taxa de sucesso
- **API:** `/communications/callRecords`
- **Latência:** Até 15 minutos para chamadas recentes
- **Arquivos:** `src/mcp_msteams/tools/calls_tools.py`, `src/mcp_msteams/services/calls_service.py`

#### 4.2 Diagnóstico de Chamada
- **Tool:** `diagnose_call_quality`
- **Descrição:** Diagnóstico detalhado de qualidade de uma chamada específica
- **Dados:** Métricas de rede, latência, jitter, perda de pacotes por participante
- **Entrada:** call_id e UPN (ou "all" para todos os participantes)
- **API:** `/communications/callRecords/{call_id}/sessions`
- **Arquivos:** `src/mcp_msteams/tools/calls_tools.py`, `src/mcp_msteams/services/calls_service.py`

#### 4.3 Listar Chamadas com Falha
- **Tool:** `list_failed_calls`
- **Descrição:** Lista chamadas que terminaram com falha para um usuário
- **Período:** Últimos N dias (default 7)
- **Arquivos:** `src/mcp_msteams/tools/calls_tools.py`, `src/mcp_msteams/services/calls_service.py`

### 5. Reuniões

#### 5.1 Reuniões Recentes
- **Tool:** `get_recent_meetings`
- **Descrição:** Lista chamadas e reuniões recentes de um usuário
- **Tipos:** groupCall (reuniões) e peerToPeer (chamadas 1:1)
- **Período:** Últimos N dias (default 7)
- **Arquivos:** `src/mcp_msteams/tools/meetings_tools.py`, `src/mcp_msteams/services/meetings_service.py`

#### 5.2 Participantes de Reunião
- **Tool:** `get_meeting_participants`
- **Descrição:** Lista todos os participantes de uma reunião específica
- **Entrada:** call_id
- **Arquivos:** `src/mcp_msteams/tools/meetings_tools.py`, `src/mcp_msteams/services/meetings_service.py`

### 6. Relatórios

#### 6.1 Relatório de Atividade do Usuário
- **Tool:** `get_user_activity_report`
- **Descrição:** Breakdown de duração de áudio, vídeo e compartilhamento de tela
- **API:** `/reports/getTeamsUserActivityUserDetail(period='{period}')`
- **Períodos:** 7, 30, 90 ou 180 dias
- **Nota:** Contagens de reuniões/chamadas podem divergir da API callRecords
- **Arquivos:** `src/mcp_msteams/tools/reports_tools.py`, `src/mcp_msteams/services/reports_service.py`

### 7. Incidentes do Serviço

#### 7.1 Verificar Incidentes Conhecidos
- **Tool:** `check_known_teams_incidents`
- **Descrição:** Lista incidentes abertos do Microsoft Teams (incidents e advisories)
- **API:** `/admin/serviceAnnouncement/issues`
- **Filtro:** Apenas issues abertas do serviço "Microsoft Teams"
- **Arquivos:** `src/mcp_msteams/tools/incidents_tools.py`, `src/mcp_msteams/services/incidents_service.py`

#### 7.2 Detalhes de Incidente
- **Tool:** `get_teams_incident_detail`
- **Descrição:** Retorna detalhes completos de um incidente específico
- **Dados:** Título, causa raiz, atualizações cronológicas, status
- **API:** `/admin/serviceAnnouncement/issues/{issue_id}`
- **Arquivos:** `src/mcp_msteams/tools/incidents_tools.py`, `src/mcp_msteams/services/incidents_service.py`

## Funcionalidades Técnicas

### Autenticação e Autorização
- **Implementação:** MSAL (Microsoft Authentication Library)
- **Fluxo:** Client Credentials (app-only)
- **Scopes:** Configurados por operação (ex: Presence.Read.All, Team.ReadBasic.All)
- **Arquivo:** `src/mcp_msteams/security/auth.py`

### Cliente HTTP e Retry
- **Cliente:** httpx.AsyncClient
- **Retry:** Automático com backoff exponencial (via tenacity)
- **Throttling:** Tratamento de 429 com Retry-After
- **Timeouts:** connect=10s, read=30s, write=10s, pool=5s
- **Limites:** max_connections=100, max_keepalive_connections=50
- **Arquivo:** `src/mcp_msteams/graph/client.py`

### Cache
- **Estratégia:** In-memory TTL-based
- **Configurações:**
  - Presença: 30s
  - Usuário: 300s
  - Teams: 600s
  - Políticas: 900s
  - Chamadas: 120s
  - Incidentes: 300s
  - Devices: 300s
- **Arquivo:** `src/mcp_msteams/graph/cache.py`

### Tratamento de Erros
- **Erros customizados:**
  - `AuthError` (401, 403)
  - `NotFoundError` (404)
  - `ThrottlingError` (429)
  - `ServiceUnavailableError` (5xx)
  - `GraphValidationError` (400)
- **Arquivo:** `src/mcp_msteams/graph/errors.py`

### Logging e Auditoria
- **Framework:** structlog
- **Formato:** JSON
- **Decorator:** `@audited` para logging de todas as tool calls
- **Arquivo:** `src/mcp_msteams/logging_config.py`

## Arquivos Analisados

- `pyproject.toml`
- `src/mcp_msteams/server.py`
- `src/mcp_msteams/config.py`
- `src/mcp_msteams/tools/users_tools.py`
- `src/mcp_msteams/tools/teams_tools.py`
- `src/mcp_msteams/tools/calls_tools.py`
- `src/mcp_msteams/tools/meetings_tools.py`
- `src/mcp_msteams/tools/reports_tools.py`
- `src/mcp_msteams/tools/incidents_tools.py`
- `src/mcp_msteams/graph/client.py`
- `src/mcp_msteams/graph/endpoints.py`
- `src/mcp_msteams/graph/cache.py`
- `src/mcp_msteams/graph/errors.py`

## Divergências com a Documentação

Nenhuma divergência significativa foi encontrada. A análise do código está alinhada com a arquitetura documentada.
