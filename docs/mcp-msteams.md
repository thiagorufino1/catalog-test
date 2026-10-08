---
title: Capabilities Catalog — mcp-msteams
---

# Capabilities Catalog — mcp-msteams

**GitHub Repository:** [thiagorufino1/mcp-msteams](https://github.com/thiagorufino1/mcp-msteams)  
**Branch:** main

## Overview

`mcp-msteams` is a Model Context Protocol (MCP) server that provides administrative and support capabilities for Microsoft Teams. It integrates with Microsoft Graph API to enable Teams management, user oversight, call quality diagnostics, meeting analytics, and service health monitoring. The server is built with FastMCP, Pydantic for data validation, and structured logging for audit trails.

**Technology Stack:**
- Python ≥3.11
- FastMCP ≥2.0
- httpx for async HTTP requests
- msal for Microsoft authentication
- Pydantic for data validation
- structlog for structured logging

## Capabilities by Domain

### 1. User Management

**Description:** Query and analyze individual user profiles, presence status, Teams memberships, and assigned policies within the Microsoft Teams tenant.

**Functionality Implemented:**
- User search by display name or email
- Fetch comprehensive user profiles (name, department, job title, office, phone)
- Check real-time Teams presence status (Available, Busy, Away, Offline, etc.)
- List all Teams a user is member of with team GUIDs
- Retrieve assigned Teams policies (meeting, calling, messaging policies)
- Combined user overview snapshot (profile + presence + teams membership in one call)

**Supported Operations:**
- `search_user`: Find users by display name or email fragment (returns up to 10 matches with UPN, display name, job title, department)
- `get_user_profile`: Retrieve full Azure AD profile data
- `get_user_presence`: Check current Teams presence status
- `list_user_teams`: Get all Teams membership with team IDs (GUIDs)
- `get_user_assigned_policies`: List Teams policies (meeting, calling, messaging) assigned to user
- `get_user_overview`: Single combined call returning profile, presence, and Teams membership

**Use Cases:**
- User support investigations
- Policy troubleshooting (e.g., user can't record meetings, no external calling)
- Team membership audits
- Presence-based routing decisions

**Files:** `src/mcp_msteams/tools/users_tools.py`, `src/mcp_msteams/services/users_service.py`

---

### 2. Team Management

**Description:** Query and analyze Teams structure, membership, settings, and health metrics at both per-team and tenant-wide scope.

**Functionality Implemented:**
- List channels (standard, private, shared) in a specific Team
- List members of a Team with user details
- Retrieve Team owners
- Get Team settings and configuration
- Get channel-level settings
- Identify private and shared channels
- Detect orphaned Teams (no members or no owners)
- Tenant-wide analytics: total teams, breakdown by visibility (public/private), orphaned count
- Comprehensive statistics combining multiple scan types
- Pagination support for large tenant browsing

**Supported Operations:**
- `list_team_channels`: Get all channels in a Team
- `list_team_members`: List Team membership
- `get_team_owners`: Retrieve Team owners
- `get_team_settings`: Get Team configuration
- `get_channel_settings`: Get channel-level configuration
- `check_private_shared_channels`: Identify private and shared channels
- `detect_orphaned_team`: Check if a specific Team has zero members
- `detect_team_without_owner`: Check if a Team lacks owners
- `list_all_teams`: Paginated list of all Teams with member/owner counts
- `get_tenant_teams_stats`: Tenant-wide statistics (total, by visibility, orphaned count)
- `list_orphaned_teams`: Find Teams without owners
- `list_teams_without_members`: Find empty Teams (full tenant scan)
- `list_teams_by_member_count`: Rank Teams by member count

**Use Cases:**
- Team governance audits
- Detecting and managing orphaned Teams
- Inventory and health reporting
- Team structure investigation
- Compliance monitoring

**Files:** `src/mcp_msteams/tools/teams_tools.py`, `src/mcp_msteams/services/teams_service.py`

---

### 3. Call Quality and Diagnostics

**Description:** Analyze call quality, identify failed calls, and diagnose performance issues at the participant level.

**Functionality Implemented:**
- Summarize call statistics for a user over configurable time period
- Identify failed calls and their failure reasons
- Diagnose session-level call quality for individual participants
- Access to Call Records API with up to 15-minute latency
- Per-participant telemetry: packet loss, jitter, latency, bandwidth

**Supported Operations:**
- `get_call_quality_summary`: Summarize call statistics (total calls, failed calls, success rate) over N days
- `diagnose_call_quality`: Detailed diagnosis for a specific call with participant-level metrics (can diagnose for all participants or a specific UPN)
- `list_failed_calls`: List calls that ended with failure result

**Use Cases:**
- Troubleshooting user call quality issues
- Identifying systematic call problems (pattern analysis)
- Performance investigations before escalation
- Quality metrics for compliance/SLA reporting

**Files:** `src/mcp_msteams/tools/calls_tools.py`, `src/mcp_msteams/services/calls_service.py`

---

### 4. Meeting Analytics

**Description:** Retrieve meeting history, participant information, and meeting details for audit and analysis.

**Functionality Implemented:**
- List recent calls and meetings for a user (both group calls and 1:1 peer-to-peer calls)
- Retrieve participant list for a specific meeting/call with join/leave times
- Meeting type classification (meetings vs. 1:1 calls)
- Duration tracking and interaction counts
- Weekly and historical summaries (configurable day ranges)

**Supported Operations:**
- `get_recent_meetings`: List calls/meetings over N days with type classification (reunião vs. chamada 1:1), duration, and participant counts
- `get_meeting_participants`: Get participant list for a specific call_id with join/leave times

**Use Cases:**
- Meeting history audits
- Participant tracking
- Compliance documentation
- User activity reports
- Communication pattern analysis

**Files:** `src/mcp_msteams/tools/meetings_tools.py`, `src/mcp_msteams/services/meetings_service.py`

---

### 5. Activity Reporting

**Description:** Generate activity breakdown reports from the Teams Reports API for detailed usage analytics.

**Functionality Implemented:**
- Audio/video/screen share duration breakdown per user
- Activity reports over multiple time periods (7, 30, 90, 180 days)
- Usage statistics differentiated by communication type
- Historical trends

**Supported Operations:**
- `get_user_activity_report`: Audio/video/screen share duration breakdown for a user over N days (7, 30, 90, or 180 days)

**Use Cases:**
- Detailed usage analytics
- Communication method tracking
- Long-term activity trends
- Meeting vs. call time allocation

**Files:** `src/mcp_msteams/tools/reports_tools.py`, `src/mcp_msteams/services/reports_service.py`

---

### 6. Service Health and Incident Monitoring

**Description:** Monitor Microsoft Teams service health, track ongoing incidents, and retrieve detailed incident information.

**Functionality Implemented:**
- List only open Teams service incidents and advisories
- Distinguish between incidents and advisories
- Retrieve detailed information for specific incidents including:
  - Root cause analysis
  - Update history with timestamps
  - Investigation findings
  - Mitigation and remediation steps
  - ETA and next actions
- Support for both English and Portuguese (pt-BR) incident details

**Supported Operations:**
- `check_known_teams_incidents`: List all open Teams service incidents and advisories with summary
- `get_teams_incident_detail`: Get structured detail for specific incident by issue ID including full update history

**Use Cases:**
- Service status monitoring
- Incident impact assessment
- Communication to affected users
- Troubleshooting validation (is this a service issue?)
- Compliance and SLA documentation

**Files:** `src/mcp_msteams/tools/incidents_tools.py`, `src/mcp_msteams/services/incidents_service.py`

---

## Authentication & Authorization

- **Authentication Method:** Microsoft Authentication Library (MSAL) with client credentials flow
- **Required Permissions:** Graph API scopes configured in `.env` and loaded via `pydantic-settings`
- **Configuration:** Environment variables in `.env.example`:
  - `TENANT_ID`: Azure AD tenant GUID
  - `CLIENT_ID`: Application registration client ID
  - `CLIENT_SECRET`: Application secret (kept secure)
  - `FASTMCP_TRANSPORT`: HTTP or stdio
  - `FASTMCP_HOST` / `FASTMCP_PORT`: HTTP transport configuration

---

## Deployment

- **Entry Point:** `mcp_msteams.server:main()`
- **CLI Command:** `mcp-msteams`
- **Transport:** HTTP (default, configurable via env vars) or stdio
- **Default Host/Port:** 127.0.0.1:8000 (HTTP)
- **Logging:** Structured logging via `structlog` with audit trail for all tool invocations

---

## Files Analyzed

1. `pyproject.toml` — Project metadata, dependencies, Python version requirement (≥3.11)
2. `.env.example` — Configuration template for authentication and deployment
3. `src/mcp_msteams/server.py` — FastMCP server initialization and tool registration
4. `src/mcp_msteams/tools/users_tools.py` — User management tools implementation
5. `src/mcp_msteams/tools/teams_tools.py` — Team management and analytics tools
6. `src/mcp_msteams/tools/calls_tools.py` — Call quality and diagnostics
7. `src/mcp_msteams/tools/meetings_tools.py` — Meeting analytics and participants
8. `src/mcp_msteams/tools/reports_tools.py` — Activity reporting
9. `src/mcp_msteams/tools/incidents_tools.py` — Service health incident monitoring

---

## Divergências com a Documentação

Nenhuma. A análise do código-fonte confirmou que as ferramentas implementadas correspondem ao escopo descrito no README, com ênfase em capacidades de suporte administrativo para Teams.
