---
title: Capabilities Catalog — mcp-msteams
---

# Capabilities Catalog — mcp-msteams

**Repositório avaliado:** [thiagorufino1/mcp-msteams](https://github.com/thiagorufino1/mcp-msteams) (branch `main`)

## Overview

This document catalogs the functional capabilities of the Microsoft Teams MCP (Model Context Protocol) Server, based on source code analysis of the [`thiagorufino1/mcp-msteams`](https://github.com/thiagorufino1/mcp-msteams) repository.

The MCP server provides a comprehensive set of tools for managing and monitoring Microsoft Teams through the Microsoft Graph API, organized into six main capability areas.

---

## 1. User Management

**Description**: Provides comprehensive user profile and presence information retrieval capabilities.

**Functionality**: Enables searching for users, retrieving detailed user profiles, monitoring Teams presence status, checking assigned policies, and listing user team memberships.

### Operations

| Operation | Description | Implementation |
|-----------|-------------|----------------|
| **get_user_overview** | Combined snapshot of user profile, presence, and Teams membership | Single round-trip aggregation |
| **get_user_profile** | Full Azure AD profile (name, department, job title, office, phone) | Direct Graph API query |
| **get_user_presence** | Current Teams presence status (Available, Busy, Away, Offline) | Presence.Read.All permission |
| **get_user_assigned_policies** | Teams policies assigned to user (meeting, calling, messaging) | TeamsUserConfiguration.Read.All |
| **list_user_teams** | All Microsoft Teams the user is a member of with team IDs | Group membership query |
| **search_user** | Search users by display name or email fragment | Azure AD search |

### Supporting Files

- `src/mcp_msteams/tools/users_tools.py` (7,420 bytes) - Tool definitions and parameter validation
- `src/mcp_msteams/services/users_service.py` (7,576 bytes) - Business logic implementation
- `src/mcp_msteams/schemas/users.py` - Data schemas for user operations

---

## 2. Teams Management

**Description**: Comprehensive Microsoft Teams organization and administration capabilities at both team-specific and tenant-wide levels.

**Functionality**: Enables listing and analyzing Teams, channels, members, owners, and settings across the organization. Includes advanced analytics for detecting orphaned teams, identifying configuration issues, and generating tenant-wide statistics.

### Per-Team Operations

| Operation | Description | Implementation |
|-----------|-------------|----------------|
| **list_team_channels** | List all channels in a Team (standard, private, shared) | Channel enumeration by team GUID |
| **list_team_members** | List all members of a specific Team | Member query with pagination |
| **get_team_owners** | Return the owners of a specific Team | Filter members by owner role |
| **get_team_settings** | Configuration and settings for a specific Team | Team properties retrieval |
| **get_channel_settings** | Settings for a specific channel within a Team | Channel-level configuration |
| **check_private_shared_channels** | Identify private and shared channels in a Team | Channel type filtering |
| **detect_orphaned_team** | Check if a Team has zero members | Member count validation |
| **detect_team_without_owner** | Check if a Team has no owners assigned | Owner count validation |

### Tenant-Wide Analytics

| Operation | Description | Implementation |
|-----------|-------------|----------------|
| **list_all_teams** | List all Teams in tenant with pagination and filters | Paginated group enumeration |
| **get_tenant_teams_stats** | Consolidated statistics for all Teams in tenant | Parallel aggregation queries |
| **list_orphaned_teams** | All Teams with no owners (orphaned groups) | Filter by owner count = 0 |
| **list_teams_without_members** | Teams with no members (empty groups) | Client-side filtering scan |
| **list_teams_by_member_count** | Largest Teams ranked by member count | Full tenant scan with sorting |

### Supporting Files

- `src/mcp_msteams/tools/teams_tools.py` (17,221 bytes) - Extensive tool definitions
- `src/mcp_msteams/services/teams_service.py` (18,504 bytes) - Core implementation logic
- `src/mcp_msteams/schemas/teams.py` - Data schemas for Teams operations

---

## 3. Call Quality Diagnostics

**Description**: Advanced call quality monitoring and diagnostics using Microsoft Teams Call Records API.

**Functionality**: Provides call quality summaries, detailed diagnostics for individual calls, and failure tracking with support for multi-participant analysis.

### Operations

| Operation | Description | Implementation |
|-----------|-------------|----------------|
| **get_call_quality_summary** | Call statistics for a user over N days (total, failed, success rate) | Aggregation over callRecords |
| **diagnose_call_quality** | Detailed call quality diagnostics for a participant | CQD telemetry analysis |
| **list_failed_calls** | List calls that ended with failure result for a user | Filter by failure status |

### Key Features

- Session-level quality metrics (audio, video, screen share)
- Network diagnostics (latency, jitter, packet loss)
- Participant-specific or call-wide analysis
- Support for both 1:1 calls and group meetings
- Up to 15 minutes latency for recent calls

### Supporting Files

- `src/mcp_msteams/tools/calls_tools.py` (4,551 bytes) - Tool definitions
- `src/mcp_msteams/services/calls_service.py` (26,282 bytes) - Complex diagnostic logic
- `src/mcp_msteams/schemas/calls.py` - Data schemas for call operations

---

## 4. Meeting Management

**Description**: Meeting and call listing capabilities with participant information.

**Functionality**: Retrieves recent meetings and calls for users, distinguishing between group meetings and 1:1 calls, with participant roster details.

### Operations

| Operation | Description | Implementation |
|-----------|-------------|----------------|
| **get_recent_meetings** | List recent calls and meetings (groupCall and peerToPeer) | callRecords query with type filter |
| **get_meeting_participants** | List all participants of a specific meeting/call | Participant roster retrieval |

### Supported Call Types

- **Group Calls**: Multi-participant meetings and conferences
- **Peer-to-Peer**: 1:1 calls between two users

### Supporting Files

- `src/mcp_msteams/tools/meetings_tools.py` (3,138 bytes) - Tool definitions
- `src/mcp_msteams/services/meetings_service.py` (7,226 bytes) - Meeting query implementation
- `src/mcp_msteams/schemas/meetings.py` - Data schemas

---

## 5. Activity Reports

**Description**: Teams usage activity reports with audio/video/screen share duration breakdowns.

**Functionality**: Provides detailed activity metrics from the Microsoft Teams Reports API for extended time periods.

### Operations

| Operation | Description | Implementation |
|-----------|-------------|----------------|
| **get_user_activity_report** | Audio/video/screen share duration breakdown | Teams Reports API query |

### Supported Time Periods

- 7 days (default)
- 30 days
- 90 days
- 180 days

### Metrics Provided

- Audio duration
- Video duration
- Screen share duration
- Meeting counts (note: may differ from callRecords API)

### Supporting Files

- `src/mcp_msteams/tools/reports_tools.py` (1,663 bytes) - Tool definitions
- `src/mcp_msteams/services/reports_service.py` (3,987 bytes) - Reports API integration
- `src/mcp_msteams/schemas/reports.py` - Data schemas

---

## 6. Service Health Monitoring

**Description**: Microsoft Teams service health and incident tracking capabilities.

**Functionality**: Monitors Microsoft 365 service health for Teams-related incidents and advisories, providing both overview and detailed incident information.

### Operations

| Operation | Description | Implementation |
|-----------|-------------|----------------|
| **check_known_teams_incidents** | List open Microsoft Teams service health items | Filter serviceHealth by Teams |
| **get_teams_incident_detail** | Detailed information for a specific service health issue | Issue detail with update history |

### Information Provided

- Open incidents vs advisories distinction
- Issue IDs for tracking
- Impact summaries
- Root cause analysis
- Mitigation steps
- Fix progress and ETAs
- Update history in chronological order

### Supporting Files

- `src/mcp_msteams/tools/incidents_tools.py` (5,330 bytes) - Tool definitions
- `src/mcp_msteams/services/incidents_service.py` (11,416 bytes) - Service health API integration
- `src/mcp_msteams/schemas/incidents.py` - Data schemas

---

## Architecture

### Server Infrastructure

**File**: `src/mcp_msteams/server.py` (1,201 bytes)

The MCP server is built using FastMCP framework with the following characteristics:

- **Server Name**: `teams-admin-support-mcp`
- **Transport Support**: HTTP (default) and stdio
- **Lifecycle Management**: Async context manager with HTTP client cleanup
- **Tool Registration**: Modular registration from six tool modules

### Configuration Management

**File**: `src/mcp_msteams/config.py` (953 bytes)

Handles Microsoft Graph API authentication and configuration:

- Azure AD tenant ID, client ID, and client secret (`pydantic-settings`, `.env` supported)
- Transport/host/port (`FASTMCP_TRANSPORT`, default `http`), log level/format
- Per-domain cache TTLs (presence, user, teams, policies, calls, incidents, devices) and Graph concurrency/pagination limits
- Graph permissions are **not** requested in code: the client credentials flow uses the `.default` scope, so effective permissions are those granted in the Azure App Registration (see `security/permissions.py`)

### Security Features

**Directory**: `src/mcp_msteams/security/`

- `auth.py`: token acquisition (MSAL client credentials)
- `input_validation.py` and `utils/sanitization.py`: input validation and sanitization
- `permissions.py`: scope mapping per feature domain (all resolve to `.default`)
- All tools are read-only (only `Read` Graph permissions)

### Utilities

**Directory**: `src/mcp_msteams/utils/`

- Response formatting (Markdown and JSON)
- Error handling and Graph API error responses
- Data transformation helpers

### Logging

**File**: `src/mcp_msteams/logging_config.py` (2,648 bytes)

- Structured logging (structlog) with audit trail
- Operation tracking with `@audited` decorator
- Configurable log levels

### Microsoft Graph API Integration

**Directory**: `src/mcp_msteams/graph/`

- HTTP client (`client.py`), response cache (`cache.py`), endpoint definitions (`endpoints.py`) and Graph error mapping (`errors.py`)
- Graph API endpoint handling
- Request/response processing

---

## Data Output Formats

The server supports two response formats for all operations:

1. **Markdown** (default): Human-readable formatted output with tables and sections
2. **JSON**: Structured machine-readable output

---

## Dependencies and Requirements

Based on code analysis, the system requires:

- **Python**: 3.11+ (`requires-python = ">=3.11"`)
- **FastMCP** (`fastmcp>=2.0`): MCP server framework
- **msal** (client credentials flow), **httpx**, **pydantic / pydantic-settings**, **tenacity** (retries), **structlog**
- **Microsoft Graph API**: Access to organizational data
- **Azure AD Application**: With appropriate permissions configured

### Required Permissions

- `User.Read.All` - User profile and search
- `Group.Read.All` - Teams and channel information
- `Presence.Read.All` - User presence status
- `TeamsUserConfiguration.Read.All` - Policy assignments
- `CallRecords.Read.All` - Call quality data
- `ServiceHealth.Read.All` - Service incidents

Additional permissions referenced in `security/permissions.py` comments: `Team.ReadBasic.All`, `TeamMember.Read.All`, `Directory.Read.All`, `Channel.ReadBasic.All`, `ChannelSettings.Read.All`, `ChannelMessage.Read.All`, `Reports.Read.All`, `OnlineMeetings.Read.All`.

---

## Summary

The Microsoft Teams MCP Server provides **27 distinct operations** across **6 capability areas**, enabling comprehensive Teams administration, monitoring, and support through a unified MCP interface. All capabilities are backed by production-quality service implementations ranging from 4KB to 26KB, with proper error handling, parameter validation, and response formatting.
