---
title: Capabilities Catalog — mcp-intune
---

# mcp-intune

**Repositório:** [https://github.com/thiagorufino1/mcp-intune](https://github.com/thiagorufino1/mcp-intune)  
**Branch analisada:** main

## Descrição Geral

Servidor MCP para gerenciamento e operações do Microsoft Intune. Oferece acesso programático à API do Microsoft Graph para consultar, gerenciar e executar ações em dispositivos gerenciados pelo Intune.

## Tecnologias e Requisitos

- **Python:** ≥3.12
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

### 1. Consulta e Busca de Dispositivos

#### 1.1 Buscar Dispositivos
- **Tool:** `intune_search_devices`
- **Descrição:** Busca dispositivos gerenciados pelo Intune por nome (prefixo), serial (exato) ou UPN (exato)
- **Funcionalidades:**
  - Filtros opcionais: plataforma (Windows, iOS, Android, macOS)
  - Filtro de compliance_state (compliant, noncompliant, unknown)
  - Paginação com `top` (max 999)
- **API:** `/deviceManagement/managedDevices`
- **Arquivos:** `src/mcp_intune/tools/device/device_tools.py`, `src/mcp_intune/services/device/device_service.py`

#### 1.2 Visão Geral de Dispositivo
- **Tool:** `intune_get_device_overview`
- **Descrição:** Visão completa de um dispositivo incluindo compliance, usuários e políticas
- **Seções opcionais:**
  - hardware (specs técnicas)
  - primaryUser (usuário principal)
  - compliance (estado de conformidade)
  - policies (políticas aplicadas)
  - detectedApps (aplicativos detectados)
- **Arquivos:** `src/mcp_intune/tools/device/device_tools.py`

#### 1.3 Hardware do Dispositivo
- **Tool:** `intune_get_device_hardware`
- **Descrição:** Inventário de hardware (modelo, serial, armazenamento, RAM)
- **Arquivos:** `src/mcp_intune/tools/device/device_tools.py`

#### 1.4 Usuários do Dispositivo
- **Tool:** `intune_get_device_users`
- **Descrição:** Usuários associados ao dispositivo (primary user e logged-on users)
- **Arquivos:** `src/mcp_intune/tools/device/device_tools.py`

#### 1.5 Aplicativos Detectados
- **Tool:** `intune_get_detected_apps`
- **Descrição:** Lista todos os softwares detectados em um dispositivo
- **Parâmetros:** `max_items` (default 200, 0=ilimitado)
- **Arquivos:** `src/mcp_intune/tools/device/device_tools.py`

#### 1.6 Estado de Compliance
- **Tool:** `intune_get_compliance_state`
- **Descrição:** Estado de conformidade e lista de políticas de compliance com falha
- **Retorno:** Resumo de compliance + estados por política
- **Arquivos:** `src/mcp_intune/tools/device/device_tools.py`

#### 1.7 Status de Políticas
- **Tool:** `intune_get_policy_status`
- **Descrição:** Estados de atribuição de políticas de configuração
- **Filtro opcional:** `policy_type` (ex: 'windows10', 'iOS')
- **Arquivos:** `src/mcp_intune/tools/device/device_tools.py`

### 2. Ações em Dispositivos

#### 2.1 Ações Não-Destrutivas (Execução Imediata)

##### 2.1.1 Sincronizar Dispositivo
- **Tool:** `intune_request_sync`
- **Descrição:** Força sincronização de um dispositivo com o Intune
- **Uso:** Atualizar dados desatualizados ou forçar check-in de políticas
- **API:** POST `/deviceManagement/managedDevices/{id}/syncDevice`
- **Arquivos:** `src/mcp_intune/tools/device/action_tools.py`, `src/mcp_intune/services/device/action_service.py`

##### 2.1.2 Reiniciar Dispositivo
- **Tool:** `intune_request_restart`
- **Descrição:** Solicita reinicialização remota de um dispositivo
- **API:** POST `/deviceManagement/managedDevices/{id}/rebootNow`
- **Arquivos:** `src/mcp_intune/tools/device/action_tools.py`

##### 2.1.3 Scan do Defender
- **Tool:** `intune_request_scan`
- **Descrição:** Inicia scan do Windows Defender
- **Parâmetros:** `quick_scan` (True=quick, False=full)
- **API:** POST `/deviceManagement/managedDevices/{id}/windowsDefenderScan`
- **Arquivos:** `src/mcp_intune/tools/device/action_tools.py`

##### 2.1.4 Localizar Dispositivo
- **Tool:** `intune_request_locate`
- **Descrição:** Solicita localização de um dispositivo (deve estar online)
- **API:** POST `/deviceManagement/managedDevices/{id}/locateDevice`
- **Arquivos:** `src/mcp_intune/tools/device/action_tools.py`

#### 2.2 Ações Destrutivas (Requerem Aprovação)

O sistema implementa um workflow de aprovação em duas etapas para ações destrutivas:

##### 2.2.1 Retire (Aposentar)
- **Tool de Request:** `intune_request_retire`
- **Descrição:** Remove dados corporativos, mantém dados pessoais
- **Campos obrigatórios:** device_id, device_name, reason, ticket_id
- **Retorna:** `request_id` com status `pending_approval`
- **API (após aprovação):** POST `/deviceManagement/managedDevices/{id}/retire`
- **Arquivos:** `src/mcp_intune/tools/device/action_tools.py`, `src/mcp_intune/services/device/destructive_action_service.py`

##### 2.2.2 Wipe (Apagar)
- **Tool de Request:** `intune_request_wipe`
- **Descrição:** Apaga completamente o dispositivo (factory reset)
- **Parâmetros adicionais:** `keep_enrollment_data` (se True, dispositivo pode re-enrollar)
- **API (após aprovação):** POST `/deviceManagement/managedDevices/{id}/wipe`
- **Arquivos:** `src/mcp_intune/tools/device/action_tools.py`

##### 2.2.3 Delete (Deletar)
- **Tool de Request:** `intune_request_delete`
- **Descrição:** Remove registro do dispositivo do Intune
- **API (após aprovação):** DELETE `/deviceManagement/managedDevices/{id}`
- **Arquivos:** `src/mcp_intune/tools/device/action_tools.py`

#### 2.3 Workflow de Aprovação

##### 2.3.1 Listar Ações Pendentes
- **Tool:** `intune_list_pending_actions`
- **Descrição:** Lista todas as ações aguardando aprovação
- **Retorno:** request_id, operation, device_id, device_name, risk, reason, ticket_id, timestamps
- **Arquivos:** `src/mcp_intune/tools/device/action_tools.py`

##### 2.3.2 Aprovar Ação
- **Tool:** `intune_approve_action`
- **Descrição:** Aprova uma ação destrutiva pendente
- **Entrada:** `request_id`, `comment` (opcional)
- **Próximo passo:** Chamar `intune_execute_action`
- **Arquivos:** `src/mcp_intune/tools/device/action_tools.py`

##### 2.3.3 Negar Ação
- **Tool:** `intune_deny_action`
- **Descrição:** Nega uma ação destrutiva pendente
- **Entrada:** `request_id`, `comment` (opcional)
- **Arquivos:** `src/mcp_intune/tools/device/action_tools.py`

##### 2.3.4 Executar Ação Aprovada
- **Tool:** `intune_execute_action`
- **Descrição:** Executa ação previamente aprovada contra o Microsoft Graph
- **Entrada:** `request_id`
- **Valida:** Status de aprovação antes de executar
- **Arquivos:** `src/mcp_intune/tools/device/action_tools.py`, `src/mcp_intune/services/device/destructive_action_service.py`

### 3. Operações em Lote (Bulk)

#### 3.1 Ações em Múltiplos Dispositivos
- **Tool:** `intune_bulk_action`
- **Descrição:** Executa ação em múltiplos dispositivos simultaneamente
- **Ações suportadas:** sync, restart, retire (após aprovação)
- **Implementação:** Concorrência controlada para não sobrecarregar API
- **Arquivos:** `src/mcp_intune/tools/device/bulk_tools.py`

### 4. Windows Autopilot

#### 4.1 Listar Dispositivos Autopilot
- **Tool:** `intune_list_autopilot_devices`
- **Descrição:** Lista identidades de dispositivos Windows Autopilot registrados
- **Dados:** Serial numbers, profile assignment status
- **API:** `/deviceManagement/windowsAutopilotDeviceIdentities`
- **Arquivos:** `src/mcp_intune/tools/autopilot/autopilot_tools.py`, `src/mcp_intune/services/autopilot/autopilot_service.py`

#### 4.2 Buscar por Serial
- **Tool:** `intune_get_autopilot_device_by_serial`
- **Descrição:** Encontra dispositivo Autopilot específico por número de série
- **Uso:** Verificar se dispositivo já está registrado antes de importar
- **Arquivos:** `src/mcp_intune/tools/autopilot/autopilot_tools.py`

#### 4.3 Importar Dispositivo Autopilot
- **Tool:** `intune_import_autopilot_device`
- **Descrição:** Importa novo dispositivo para Windows Autopilot usando hardware hash
- **Parâmetros:**
  - serial_number (obrigatório)
  - hardware_hash (obrigatório, base64 do Get-WindowsAutoPilotInfo)
  - group_tag (opcional, para targeting de perfil)
  - assigned_user_principal_name (opcional, pré-atribuição)
- **API:** POST `/deviceManagement/importedWindowsAutopilotDeviceIdentities`
- **Arquivos:** `src/mcp_intune/tools/autopilot/autopilot_tools.py`

### 5. Gerenciamento de Updates

#### 5.1 Consultar Windows Updates
- **Tool:** Operações relacionadas a Windows Update for Business
- **Descrição:** Consulta status de updates, rings de deployment e estados de conformidade
- **API:** `/deviceManagement/windowsUpdateForBusinessConfigurations`
- **Arquivos:** `src/mcp_intune/tools/updates/updates_tools.py`, `src/mcp_intune/services/updates/updates_service.py`

### 6. Scripts e Remediações

#### 6.1 Gerenciar Scripts PowerShell
- **Tool:** Scripts relacionados à execução remota de PowerShell
- **Descrição:** Upload, listagem e execução de scripts PowerShell em dispositivos gerenciados
- **API:** `/deviceManagement/deviceManagementScripts`
- **Arquivos:** `src/mcp_intune/tools/scripts/scripts_tools.py`

### 7. Relatórios e Governança

#### 7.1 Exportar Relatórios
- **Tool:** Operações de exportação de relatórios do Intune
- **Descrição:** Exportação de inventário de dispositivos, compliance reports, app inventory
- **API:** `/deviceManagement/reports/exportJobs`
- **Arquivos:** `src/mcp_intune/tools/reporting/reporting_tools.py`

#### 7.2 Análise de Governança
- **Tool:** Operações de auditoria e governança
- **Descrição:** Análise de compliance, dispositivos órfãos, dispositivos inativos
- **Arquivos:** `src/mcp_intune/tools/governance/governance_tools.py`

### 8. Windows Autopatch

#### 8.1 Gerenciar Windows Autopatch
- **Tool:** Operações relacionadas ao Windows Autopatch
- **Descrição:** Consultar deployment rings, enrollment status e update readiness
- **API:** Endpoints específicos do Autopatch dentro do Graph
- **Arquivos:** `src/mcp_intune/tools/autopatch/autopatch_tools.py`

## Funcionalidades Técnicas

### Sistema de Aprovação
- **Localização:** `src/mcp_intune/approval/store.py`
- **Funcionalidade:**
  - Armazena requests de ações destrutivas em memória
  - Workflow: request → approve/deny → execute
  - Campos: request_id, operation, device_id, risk, reason, ticket_id, timestamps
  - Expiração automática de requests antigos

### Autenticação e Autorização
- **Implementação:** MSAL (Microsoft Authentication Library)
- **Fluxo:** Client Credentials (app-only)
- **Scopes:** Configurados por operação
- **Arquivo:** `src/mcp_intune/security/auth.py`

### Cliente HTTP e Retry
- **Cliente:** httpx.AsyncClient
- **Retry:** Automático com backoff exponencial (via tenacity)
- **Throttling:** Tratamento de 429 com Retry-After
- **Cache:** TTL-based para operações de leitura
- **Arquivo:** `src/mcp_intune/graph/client.py`

### Logging e Auditoria
- **Framework:** structlog
- **Formato:** JSON estruturado
- **Decorator:** `@audited` para logging de todas as tool calls
- **Arquivo:** `src/mcp_intune/logging_config.py`

### Tratamento de Erros
- **Erros customizados:** AuthError, NotFoundError, ThrottlingError, etc.
- **Arquivo:** `src/mcp_intune/graph/errors.py`

## Arquivos Analisados

- `pyproject.toml`
- `src/mcp_intune/server.py`
- `src/mcp_intune/config.py`
- `src/mcp_intune/tools/device/device_tools.py`
- `src/mcp_intune/tools/device/action_tools.py`
- `src/mcp_intune/tools/device/bulk_tools.py`
- `src/mcp_intune/tools/autopilot/autopilot_tools.py`
- `src/mcp_intune/tools/updates/updates_tools.py`
- `src/mcp_intune/tools/scripts/scripts_tools.py`
- `src/mcp_intune/tools/reporting/reporting_tools.py`
- `src/mcp_intune/tools/governance/governance_tools.py`
- `src/mcp_intune/tools/autopatch/autopatch_tools.py`
- `src/mcp_intune/approval/store.py`
- `src/mcp_intune/graph/client.py`

## Divergências com a Documentação

Nenhuma divergência significativa foi encontrada. A análise do código está alinhada com a arquitetura documentada.
