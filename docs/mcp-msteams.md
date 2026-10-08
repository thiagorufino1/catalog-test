---
title: Capabilities Catalog — mcp-msteams
---

# mcp-msteams

**Repository:** [https://github.com/thiagorufino1/mcp-msteams](https://github.com/thiagorufino1/mcp-msteams)  
**Branch:** main  
**Language:** Python  
**Requirements:** Python >= 3.11

## Overview

MCP (Model Context Protocol) server for Microsoft Teams administration and support. Provides comprehensive tools for managing users, teams, call quality diagnostics, meetings, activity reports, and service health monitoring through Microsoft Graph API integration.

## Technical Architecture

- **Framework:** FastMCP 2.0+
- **Authentication:** MSAL (Microsoft Authentication Library) 1.28+
- **HTTP Client:** httpx 0.27+
- **Data Validation:** Pydantic 2.7+
- **Retry Logic:** tenacity 8.3+
- **Logging:** structlog 24.1+

**Entry point:** `mcp-msteams` command runs `mcp_msteams.server:main`

**Supporting files:**
- `src/mcp_msteams/server.py` - Main server initialization and tool registration
- `src/mcp_msteams/config.py` - Configuration management
- `src/mcp_msteams/graph/` - Microsoft Graph API client implementation
- `src/mcp_msteams/security/` - Authentication and security layer
- `src/mcp_msteams/services/` - Business logic services
- `src/mcp_msteams/schemas/` - Pydantic data models

## Capabilities

### 1. User Management

**Description:** Comprehensive user information retrieval including profiles, presence status, Teams membership, and assigned policies.

**Operations:**

- **get_user_overview** - Combined snapshot of user profile, presence, and Teams membership in a single API call. Recommended first step for any user-focused support session.
  
- **get_user_profile** - Full Azure AD profile details including name, department, job title, office, and phone number.
  
- **get_user_presence** - Current Teams presence status (Available, Busy, Away, Offline, etc.). Data cached for 30 seconds.
  
- **get_user_assigned_policies** - Teams policies assigned to a user (meeting policy, calling policy, messaging policy). Essential for troubleshooting missing features.
  
- **list_user_teams** - List all Microsoft Teams a user is member of, with team GUIDs required for other operations.
  
- **search_user** - Search Azure AD users by display name or email fragment. Returns up to 10 matches with UPNs needed for other tools.

**Supporting files:**
- `src/mcp_msteams/tools/users_tools.py` (7,420 bytes) - Tool definitions and parameter validation
- `src/mcp_msteams/services/users_service.py` - Business logic implementation
- `src/mcp_msteams/schemas/users.py` - User-specific Pydantic schemas

### 2. Teams Management

**Description:** Tenant-wide and per-team management including channels, members, owners, settings, and health analytics.

**Operations:**

**Per-Team Operations:**
- **list_team_channels** - List all channels in a team (standard, private, shared)
- **list_team_members** - List all members of a specific team
- **get_team_owners** - Return team owners/proprietários
- **get_team_settings** - Team configuration and settings
- **get_channel_settings** - Settings for a specific channel
- **check_private_shared_channels** - Identify private and shared channels
- **detect_orphaned_team** - Check if a team has zero members
- **detect_team_without_owner** - Check if a team has no owners

**Tenant-Wide Analytics:**
- **list_all_teams** - List all teams with pagination, privacy filters, and optional member/owner counts (default 20, max 100 per page)
- **get_tenant_teams_stats** - Consolidated statistics: total teams, public/private breakdown, orphaned teams count
- **list_orphaned_teams** - Teams with no owners (ownerless groups)
- **list_teams_without_members** - Empty teams (no members)
- **list_teams_by_member_count** - Largest teams ranked by member count (default top 10, max 50)

**Supporting files:**
- `src/mcp_msteams/tools/teams_tools.py` (17,221 bytes) - Tool definitions with extensive documentation
- `src/mcp_msteams/services/teams_service.py` - Business logic with pagination and filtering
- `src/mcp_msteams/schemas/teams.py` - Teams-specific Pydantic schemas

### 3. Call Quality Management

**Description:** Call quality diagnostics and analysis using Microsoft Call Quality Dashboard (CQD) telemetry data.

**Operations:**

- **get_call_quality_summary** - Summarize call statistics for a user over N days (total calls, failed calls, success rate). Note: Call Records API has up to 15 minutes latency.
  
- **diagnose_call_quality** - Detailed per-participant call quality diagnosis with telemetry data (network metrics, audio/video quality indicators).
  
- **list_failed_calls** - List calls that ended with failure result for a user over N days.

**Supporting files:**
- `src/mcp_msteams/tools/calls_tools.py` (4,551 bytes) - Call diagnostic tools
- `src/mcp_msteams/services/calls_service.py` - Call Records API integration
- `src/mcp_msteams/schemas/calls.py` - Call-related schemas

### 4. Meetings Management

**Description:** Meeting and call history retrieval with participant information.

**Operations:**

- **get_recent_meetings** - List recent calls and meetings for a user (both groupCall meetings/conferences and peerToPeer 1:1 calls). Returns detailed table with type breakdown. Supports 7, 30, 90, or 180 day ranges.
  
- **get_meeting_participants** - List all participants of a specific meeting/call by call_id with full details.

**Supporting files:**
- `src/mcp_msteams/tools/meetings_tools.py` (3,138 bytes) - Meeting tools
- `src/mcp_msteams/services/meetings_service.py` - Meeting data retrieval
- `src/mcp_msteams/schemas/meetings.py` - Meeting schemas

### 5. Activity Reports

**Description:** Teams usage reports and activity metrics from Microsoft Graph Reports API.

**Operations:**

- **get_user_activity_report** - Audio/video/screen share duration breakdown from Teams Reports API. Supports 7, 30, 90, or 180 day ranges. Note: Counts may differ from Call Records API.

**Supporting files:**
- `src/mcp_msteams/tools/reports_tools.py` (1,663 bytes) - Reports tools
- `src/mcp_msteams/services/reports_service.py` - Reports API integration
- `src/mcp_msteams/schemas/reports.py` - Report schemas

### 6. Service Health Monitoring

**Description:** Microsoft Teams service health incidents and advisories monitoring.

**Operations:**

- **check_known_teams_incidents** - List open Microsoft Teams service health items, split between incidents and advisories, with issue IDs and activity summaries.
  
- **get_teams_incident_detail** - Detailed structured information for a specific service health issue including full update history, root cause, mitigation steps, and current status.

**Supporting files:**
- `src/mcp_msteams/tools/incidents_tools.py` (5,330 bytes) - Service health tools
- `src/mcp_msteams/services/incidents_service.py` - Service Health API integration
- `src/mcp_msteams/schemas/incidents.py` - Incident schemas

## Tool Count

**Total MCP Tools Implemented: 26**

- Users: 6 tools
- Teams: 13 tools
- Call Quality: 3 tools
- Meetings: 2 tools
- Reports: 1 tool
- Service Health: 2 tools

All tools are read-only (`readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true`).

## Authentication & Permissions

The server uses MSAL for authentication with Microsoft Graph API. Required permissions vary by capability:

- **User operations:** User.Read.All, Presence.Read.All (presence not available on all license tiers)
- **Teams operations:** Group.Read.All, Team.ReadBasic.All
- **Call quality:** CallRecords.Read.All
- **Policies:** TeamsUserConfiguration.Read.All
- **Service health:** ServiceHealth.Read.All

Configuration via environment variables in `.env` file (see `.env.example`).

## Response Formats

All tools support dual response formats:
- **MARKDOWN** (default) - Human-readable formatted tables and text
- **JSON** - Structured data for programmatic consumption

Format controlled via `response_format` parameter on each tool.

## Utility Infrastructure

**Logging:** Structured logging with `structlog` for audit trails  
**Error Handling:** Consistent Graph API error responses with context  
**Retry Logic:** Automatic retry with exponential backoff via `tenacity`  
**HTTP Client:** Persistent connections and connection pooling via `httpx`

Supporting files:
- `src/mcp_msteams/logging_config.py` (2,648 bytes) - Structured logging setup
- `src/mcp_msteams/utils/response.py` - Response rendering utilities
- `src/mcp_msteams/utils/` - Additional utilities

## Development

**Testing:** pytest, pytest-asyncio, respx for HTTP mocking  
**Code Quality:** ruff (linting), mypy (type checking)  
**Python Version:** 3.11+ (specified in pyproject.toml)

Test files located in `tests/` directory.
