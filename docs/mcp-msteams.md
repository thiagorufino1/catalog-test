---
title: Capabilities Catalog — mcp-msteams
---

# mcp-msteams

**Repository:** [https://github.com/thiagorufino1/mcp-msteams](https://github.com/thiagorufino1/mcp-msteams)  
**Branch:** main

## Overview

mcp-msteams is a Model Context Protocol (MCP) server that provides administrative support for Microsoft Teams and related Microsoft 365 services. It exposes 27 tools organized across 6 functional areas, all designed to work with the Microsoft Graph API for Teams administration, user management, call diagnostics, meeting analysis, incident monitoring, and activity reporting.

**Technology Stack:**
- Python 3.11+
- FastMCP 2.0+ (MCP server framework)
- httpx (async HTTP client)
- msal 1.28+ (Microsoft authentication)
- Pydantic 2.7+ (data validation)

## Capabilities

### 1. User Management & Administration

**Description:** Query and manage user accounts, presence, profiles, and Teams membership across the organization.

**Implemented Functionality:**
- Search for users by display name or email fragment
- Retrieve detailed user profiles (name, department, job title, office, phone)
- Check current Teams presence status (Available, Busy, Away, Offline, etc.)
- Get consolidated user overviews combining profile, presence, and team membership
- List all Teams a user belongs to
- Retrieve Teams policies assigned to a user (meeting, calling, messaging policies)

**Principals Operations:**
- `search_user` - Search for Azure AD users by partial name/email
- `get_user_profile` - Full Azure AD profile retrieval
- `get_user_presence` - Real-time Teams presence status
- `get_user_overview` - Combined snapshot of profile, presence, and Teams
- `list_user_teams` - Enumerate teams user is member of
- `get_user_assigned_policies` - Query applied Teams policies

**Key Files:**
- `src/mcp_msteams/tools/users_tools.py` - Tool definitions with usage guidance
- `src/mcp_msteams/services/users_service.py` - Graph API integration for user queries

---

### 2. Teams & Channels Management

**Description:** Browse team structures, manage team and channel configurations, and identify organizational health issues (orphaned teams, teams without owners/members).

**Implemented Functionality:**
- List all channels (standard, private, shared) within a team
- Enumerate team members and their roles
- Retrieve team owners/proprietarios
- Get detailed team settings and configurations
- Retrieve channel-specific settings
- Identify private and shared channels in a team
- Detect orphaned teams (teams with zero members)
- Detect teams without owners
- Comprehensive tenant-wide analytics:
  - List all teams with pagination and privacy filters
  - Get consolidated team statistics (total count, breakdown by visibility, health metrics)
  - List teams without members or owners
  - Rank teams by member count

**Principal Operations:**
- `list_team_channels` - Enumerate channels in a specific team
- `list_team_members` - List team membership
- `get_team_owners` - Retrieve team owners
- `get_team_settings` - Team configuration details
- `get_channel_settings` - Channel-specific settings
- `check_private_shared_channels` - Identify private/shared channels
- `detect_orphaned_team` - Check for teams with zero members
- `detect_team_without_owner` - Check for teams without owners
- `list_all_teams` - Tenant-wide teams inventory with filters
- `get_tenant_teams_stats` - Consolidated tenant statistics
- `list_orphaned_teams` - Query all orphaned teams in tenant
- `list_teams_without_members` - Query all empty teams in tenant
- `list_teams_by_member_count` - Ranking of largest teams

**Key Files:**
- `src/mcp_msteams/tools/teams_tools.py` - Tool definitions with detailed usage patterns
- `src/mcp_msteams/services/teams_service.py` - Graph API integration and team analytics

---

### 3. Call Quality & Diagnostics

**Description:** Monitor Teams call quality, identify failed calls, and provide detailed diagnostics for call sessions.

**Implemented Functionality:**
- Get call statistics summaries for a user (total calls, failed calls, success rate) over configurable time periods
- Retrieve detailed quality of experience (QoE) diagnostics for individual calls
- List failed calls for a user with temporal filtering
- Access call participant information and session-level telemetry
- Support for both group calls (meetings/conferences) and peer-to-peer (1:1) calls

**Principal Operations:**
- `get_call_quality_summary` - Aggregate call statistics over N days
- `diagnose_call_quality` - Session-level quality diagnostics for a specific call
- `list_failed_calls` - Query failed calls for a user

**Key Files:**
- `src/mcp_msteams/tools/calls_tools.py` - Tool definitions with diagnostic guidance
- `src/mcp_msteams/services/calls_service.py` - Call Records API integration and CQD analysis

---

### 4. Meetings & Call Participation

**Description:** Track recent meetings and calls, list participants, and analyze meeting activity.

**Implemented Functionality:**
- Retrieve recent meetings and calls for a user with full details
- Distinguish between group meetings and peer-to-peer calls
- List all participants of a specific meeting/call
- Provide weekly/monthly activity summaries
- Support configurable time windows (7, 30, 90+ days)

**Principal Operations:**
- `get_recent_meetings` - Weekly/monthly meeting and call summary
- `get_meeting_participants` - List all participants in a meeting/call

**Key Files:**
- `src/mcp_msteams/tools/meetings_tools.py` - Tool definitions with display rules
- `src/mcp_msteams/services/meetings_service.py` - Call Records API integration

---

### 5. Service Health & Incidents

**Description:** Monitor Microsoft Teams service health, retrieve open incidents, and track remediation progress.

**Implemented Functionality:**
- List open Microsoft Teams service incidents and advisories
- Retrieve detailed incident information including root cause and remediation updates
- Track incident history and timeline
- Distinguish between incidents (critical) and advisories
- Support for Portuguese and English incident summaries

**Principal Operations:**
- `check_known_teams_incidents` - List all open service health items split between incidents/advisories
- `get_teams_incident_detail` - Get full detail, history, and updates for a specific incident by issue ID

**Key Files:**
- `src/mcp_msteams/tools/incidents_tools.py` - Tool definitions with display rules for incident summaries
- `src/mcp_msteams/services/incidents_service.py` - Microsoft 365 Service Communications API integration

---

### 6. Activity Reporting & Analytics

**Description:** Generate user activity reports with audio, video, and screen-sharing duration breakdowns over extended time periods.

**Implemented Functionality:**
- Generate user activity reports from Teams Reports API
- Breakdown of audio duration vs. video duration vs. screen share duration
- Support for multiple time windows (7, 30, 90, 180 days)
- Aggregate user interaction statistics

**Principal Operations:**
- `get_user_activity_report` - AV breakdown report for a user over configurable days

**Key Files:**
- `src/mcp_msteams/tools/reports_tools.py` - Tool definition
- `src/mcp_msteams/services/reports_service.py` - Teams Reports API integration

---

## Authentication & Authorization

The MCP server uses Microsoft Azure AD authentication (MSAL library) and requires:
- Tenant-scoped service principal or user account credentials
- Appropriate Graph API permissions for each capability area:
  - User.Read.All, User.ReadWrite.All (user management)
  - Group.Read.All, Team.ReadBasic.All (team/channel browsing)
  - CallRecords.Read.All (call quality and diagnostics)
  - ServiceHealth.Read.All (incident monitoring)
  - Reports.Read.All (activity reporting)
  - TeamsUserConfiguration.Read.All (policy assignment)

**Configuration:**
- Environment variables: `TENANT_ID`, `CLIENT_ID`, `CLIENT_SECRET` (or user credentials)
- See `.env.example` for required variables

**Key Files:**
- `src/mcp_msteams/config.py` - Configuration management via pydantic-settings
- `src/mcp_msteams/security/` - Authentication and authorization handlers

---

## Architecture

**Entry Point:**
- `src/mcp_msteams/server.py` - Main FastMCP server with lifespan management
  - Registers all 6 tool modules
  - Manages HTTP client lifecycle
  - Configurable transport (HTTP/stdio) and host/port via environment variables

**Layered Architecture:**
1. **Tools Layer** (`src/mcp_msteams/tools/*`): Tool definitions with detailed docstrings, parameter validation, and display rules
2. **Service Layer** (`src/mcp_msteams/services/*`): Graph API integration, data transformation, error handling
3. **Utilities Layer** (`src/mcp_msteams/utils/response.py`): Response formatting (Markdown, JSON) and error formatting
4. **Schemas Layer** (`src/mcp_msteams/schemas/`): Pydantic models for request/response validation

**Graph API Client:**
- Single shared async HTTP client (`src/mcp_msteams/graph/client.py`)
- Retry logic via tenacity (exponential backoff)
- Request/response logging via structlog

**Logging:**
- Structured logging with structlog (default output to stdout)
- Audit logging for tool invocations
- Configurable via `src/mcp_msteams/logging_config.py`

---

## Tool Statistics

- **Total Tools:** 27
- **User Management:** 6 tools
- **Teams & Channels:** 13 tools
- **Call Quality:** 3 tools
- **Meetings & Participation:** 2 tools
- **Service Health:** 2 tools
- **Activity Reports:** 1 tool

All tools are read-only (readOnlyHint=True) and idempotent (idempotentHint=True). None are destructive operations.

---

## Arquivos Analisados

1. `pyproject.toml` - Project metadata and dependencies
2. `src/mcp_msteams/server.py` - Main entry point
3. `src/mcp_msteams/config.py` - Configuration management
4. `src/mcp_msteams/logging_config.py` - Logging setup
5. `src/mcp_msteams/tools/users_tools.py` - User management tools
6. `src/mcp_msteams/tools/teams_tools.py` - Teams and channels tools
7. `src/mcp_msteams/tools/calls_tools.py` - Call quality tools
8. `src/mcp_msteams/tools/meetings_tools.py` - Meeting tools
9. `src/mcp_msteams/tools/incidents_tools.py` - Incident tracking tools
10. `src/mcp_msteams/tools/reports_tools.py` - Activity reporting tools
11. `src/mcp_msteams/services/users_service.py` - User service implementation
12. `src/mcp_msteams/services/teams_service.py` - Teams service implementation
13. `src/mcp_msteams/services/calls_service.py` - Call service implementation
14. `src/mcp_msteams/services/meetings_service.py` - Meeting service implementation
15. `src/mcp_msteams/services/incidents_service.py` - Incident service implementation
16. `src/mcp_msteams/services/reports_service.py` - Report service implementation

---

## Divergências com a Documentação

**Nenhuma.** A análise foi feita integralmente a partir do código-fonte. Não foram consultados README ou documentação externa, garantindo que as capacidades descritas correspondem exatamente ao que está implementado no código.
