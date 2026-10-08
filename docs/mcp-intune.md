---
title: Capabilities Catalog — mcp-intune
---

# mcp-intune

**Repositório:** [https://github.com/thiagorufino1/mcp-intune](https://github.com/thiagorufino1/mcp-intune)  
**Branch analisada:** main

## Descrição Geral

Servidor MCP para suporte e operações do Microsoft Intune, construído sobre o Microsoft Graph. Expõe **47 tools** em nove domínios: busca e inventário de dispositivos, ações em dispositivo, ações em lote, governança (LAPS, BitLocker, RBAC), relatórios, scripts de remediação, gestão de atualizações, Autopilot e Windows Autopatch.

Operações sensíveis (retire, wipe, delete, senha LAPS, chave BitLocker, execução de remediação) nunca rodam direto: criam uma solicitação de aprovação, que precisa ser aprovada e depois executada em etapa separada.

## Tecnologias e Requisitos

- **Python:** ≥3.12
- **Dependências:** `fastmcp>=2.0`, `httpx`, `msal` (autenticação Microsoft), `pydantic` / `pydantic-settings`, `tenacity` (retry), `structlog`
- **Transporte:** HTTP (padrão) ou stdio, definido por `fastmcp_transport`
- **Configuração** (`config.py`): TTL de cache (dispositivo 60 s, política 120 s, apps 300 s), TTL de aprovação (`approval_ttl_seconds`, padrão 3600 s), timeouts e retries do Graph (padrão 4), tamanho de página padrão (`graph_default_top`, 50) e `allow_beta_apis` (padrão `False`)
- Várias tools usam endpoints **beta** do Graph e só funcionam com `ALLOW_BETA_APIS=true`

## Capacidades Funcionais

### 1. Busca e Inventário de Dispositivos

| Tool | O que faz |
|------|-----------|
| `intune_search_devices` | Busca por nome (prefixo), serial (exato) ou UPN (exato) |
| `intune_get_device_overview` | Visão completa: compliance, usuários, políticas. Primeira chamada recomendada |
| `intune_get_device_hardware` | Modelo, serial, armazenamento, RAM |
| `intune_get_device_users` | Usuário principal e usuários que fizeram logon |
| `intune_get_detected_apps` | Software detectado no dispositivo (`max_items`, `0` = sem limite) |
| `intune_get_compliance_state` | Estado de compliance e estado por política (por que o dispositivo está não conforme) |
| `intune_get_policy_status` | Estado de atribuição das políticas de configuração |

**Arquivos:** `src/mcp_intune/tools/device/device_tools.py`

### 2. Ações em Dispositivo

Imediatas (não destrutivas):

| Tool | O que faz |
|------|-----------|
| `intune_request_sync` | Força check-in de políticas |
| `intune_request_restart` | Reinício remoto |
| `intune_request_scan` | Varredura do Windows Defender (`quick_scan` true = rápida, false = completa) |
| `intune_request_locate` | Solicita a localização (dispositivo precisa estar online) |

Destrutivas, **exigem aprovação** (`device_id`, `device_name`, `reason`, `ticket_id`):

| Tool | O que faz |
|------|-----------|
| `intune_request_retire` | Solicita retire |
| `intune_request_wipe` | Solicita wipe completo |
| `intune_request_delete` | Solicita remoção do registro do dispositivo no Intune |

Fluxo de aprovação (compartilhado por todas as tools que exigem aprovação):

| Tool | O que faz |
|------|-----------|
| `intune_list_pending_actions` | Lista solicitações aguardando decisão |
| `intune_approve_action` | Aprova (não executa); comentário opcional para auditoria |
| `intune_deny_action` | Nega a solicitação |
| `intune_execute_action` | Executa uma solicitação aprovada; falha se não aprovada |

As solicitações expiram após `approval_ttl_seconds` e ficam num armazenamento em memória (`approval/store.py`), portanto não sobrevivem a um reinício do servidor.

**Arquivos:** `src/mcp_intune/tools/device/action_tools.py`, `src/mcp_intune/approval/`

### 3. Ações em Lote

| Tool | O que faz |
|------|-----------|
| `intune_bulk_sync` | Sync de até 20 dispositivos em uma chamada |
| `intune_bulk_restart` | Reinício de até 20 dispositivos em uma chamada |

**Arquivos:** `src/mcp_intune/tools/device/bulk_tools.py`

### 4. Governança e Credenciais

Acesso a senhas de administrador local e chaves de recuperação, mais revisão de RBAC. A recuperação de segredos passa por aprovação e gera evento de auditoria no Microsoft Entra ID.

| Tool | O que faz |
|------|-----------|
| `intune_get_laps_metadata` | Metadados do LAPS (última rotação, nome da conta); **não** retorna a senha |
| `intune_request_laps_secret` | Solicita a senha LAPS (exige aprovação) |
| `intune_execute_laps_secret` | Recupera a senha LAPS após aprovação |
| `intune_find_bitlocker_keys` | Lista metadados de chaves BitLocker; **não** retorna a chave |
| `intune_request_bitlocker_key` | Solicita chave de recuperação BitLocker (exige aprovação) |
| `intune_execute_bitlocker_key` | Recupera a chave após aprovação |
| `intune_list_role_definitions` | Funções RBAC do Intune (internas e personalizadas) |
| `intune_list_role_assignments` | Atribuições de função, com membros e scope tags |

**Arquivos:** `src/mcp_intune/tools/governance/governance_tools.py`

### 5. Relatórios e Auditoria

| Tool | O que faz |
|------|-----------|
| `intune_list_report_catalog` | Nomes de relatórios disponíveis |
| `intune_export_report` | Cria um job de exportação de relatório (retorna o job id) |
| `intune_get_report_status` | Consulta o job; quando concluído, traz a URL de download |
| `intune_get_audit_events` | Eventos de auditoria (quem fez o quê, quando), com filtro de dias e UPN do ator |
| `intune_get_endpoint_analytics` | Scores gerais do Endpoint Analytics (0–100) |

**Arquivos:** `src/mcp_intune/tools/reporting/reporting_tools.py`

### 6. Scripts de Remediação

Exige `ALLOW_BETA_APIS=true`.

| Tool | O que faz |
|------|-----------|
| `intune_list_remediations` | Lista scripts de remediação proativa |
| `intune_get_remediation_run_state` | Estado de execução por dispositivo de um script |
| `intune_request_remediation_run` | Solicita execução sob demanda (exige aprovação) |

**Arquivos:** `src/mcp_intune/tools/scripts/scripts_tools.py`

### 7. Gestão de Atualizações

Todas as tools deste domínio são somente leitura.

| Tool | O que faz | API beta |
|------|-----------|----------|
| `intune_list_update_rings` | Anéis do Windows Update for Business (deferral e pausa) | não |
| `intune_get_update_ring` | Um anel com suas atribuições de grupo | não |
| `intune_list_feature_update_profiles` | Perfis de feature update (versão alvo do Windows) | sim |
| `intune_list_quality_update_profiles` | Perfis de quality update (patch mensal) | sim |
| `intune_list_driver_update_profiles` | Perfis de driver update (aprovação automática ou manual) | sim |

**Arquivos:** `src/mcp_intune/tools/updates/updates_tools.py`

### 8. Windows Autopilot

| Tool | O que faz |
|------|-----------|
| `intune_list_autopilot_devices` | Identidades de dispositivos Autopilot registrados |
| `intune_get_autopilot_device_by_serial` | Localiza um dispositivo pelo serial exato |
| `intune_import_autopilot_device` | Importa um dispositivo usando serial e hardware hash |

`intune_import_autopilot_device` é a única operação de escrita deste domínio.

**Arquivos:** `src/mcp_intune/tools/autopilot/autopilot_tools.py`

### 9. Windows Autopatch

Somente leitura. Exige a permissão `WindowsUpdates.Read.All`.

| Tool | O que faz |
|------|-----------|
| `intune_list_autopatch_deployments` | Deployments do serviço de implantação do Windows Update for Business |
| `intune_get_autopatch_deployment` | Detalhes de um deployment |
| `intune_list_updatable_assets` | Dispositivos registrados como updatable assets |

**Arquivos:** `src/mcp_intune/tools/autopatch/autopatch_tools.py`

## Arquivos analisados

- `pyproject.toml`, `src/mcp_intune/config.py`, `src/mcp_intune/server.py`
- `src/mcp_intune/approval/store.py` e `approval/models.py`
- Os 9 arquivos `src/mcp_intune/tools/**/*_tools.py` (as 47 tools, conferidas por nome e docstring)

Não foram lidos `services/` e `graph/`: endpoints e permissões exatas do Graph não estão detalhados aqui, além do que as docstrings das tools indicam.

## Divergências com a Documentação

A versão inicial desta página, gerada pelo agente, continha nomes de tools que não existem no código e omitia 26 tools reais. Esta versão foi corrigida manualmente contra o código-fonte. A comparação com o README do repositório não foi feita.
