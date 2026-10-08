---
title: Capabilities Catalog — mcp-intune
---

# Capabilities Catalog — mcp-intune

**GitHub Repository:** [thiagorufino1/mcp-intune](https://github.com/thiagorufino1/mcp-intune)  
**Branch:** main

## Overview

`mcp-intune` is a Model Context Protocol (MCP) server that provides comprehensive Microsoft Intune device management capabilities. It enables querying device inventory, compliance status, configuration policies, and executing device actions ranging from non-destructive operations (sync, restart, scan) to destructive operations (retire, wipe, delete) with built-in approval workflows. The server is built with FastMCP, Pydantic for validation, and implements audit logging and approval gates for sensitive operations.

**Technology Stack:**
- Python ≥3.12
- FastMCP ≥2.0
- httpx for async HTTP requests
- msal for Microsoft authentication
- Pydantic for data validation
- structlog for structured logging

## Capabilities by Domain

### 1. Device Search and Discovery

**Description:** Search and locate managed devices within the Intune inventory by various criteria.

**Functionality Implemented:**
- Search devices by device name prefix (partial match)
- Search by exact serial number
- Search by exact user UPN (principal name)
- Optional filtering by OS platform (Windows, iOS, Android, macOS)
- Optional filtering by compliance state (compliant, noncompliant, unknown)
- Pagination support with configurable result limits (default 50, max 999)

**Supported Operations:**
- `intune_search_devices`: Search by name prefix, serial number, or UPN with optional platform/compliance filters

**Use Cases:**
- Device inventory lookup
- Finding devices by serial number for hardware replacement
- User device search for support tickets
- Compliance-based device discovery
- Pre-requisite for all device-specific operations

**Files:** `src/mcp_intune/tools/device/device_tools.py`, `src/mcp_intune/services/device/device_service.py`

---

### 2. Device Inventory and Configuration

**Description:** Query comprehensive device information including hardware, software, users, and policies.

**Functionality Implemented:**
- Get full device overview (profile, compliance, users, policies)
- Hardware inventory (model, serial number, storage, RAM, processor)
- Detected software inventory (all installed applications with versions)
- User associations (primary user and all logged-on users)
- Configuration policy assignment status
- Compliance state and failing policy details
- Optional inclusion of specific sections (hardware, users, compliance, policies, apps)

**Supported Operations:**
- `intune_get_device_overview`: Full device context with configurable sections
- `intune_get_device_hardware`: Hardware specifications and inventory
- `intune_get_device_users`: Primary and logged-on users
- `intune_get_detected_apps`: Software inventory with version tracking (up to 200 apps default, unlimited option)
- `intune_get_compliance_state`: Compliance status and per-policy states
- `intune_get_policy_status`: Configuration policy assignment and compliance states

**Use Cases:**
- Support ticket investigation (what software/hardware?)
- Hardware asset tracking
- Compliance audits (which policies are failing?)
- User device association
- License and software inventory reporting
- Pre-requisite for understanding device state before actions

**Files:** `src/mcp_intune/tools/device/device_tools.py`, `src/mcp_intune/services/device/device_service.py`

---

### 3. Device Actions (Non-Destructive)

**Description:** Execute safe, non-destructive operational commands on managed devices.

**Functionality Implemented:**
- Request device synchronization (policy check-in)
- Request device restart (reboot)
- Request Windows Defender scan (quick or full)
- Request device location (GPS/last known position)
- Immediate execution (no approval required)
- Action status tracking

**Supported Operations:**
- `intune_request_sync`: Force device policy sync
- `intune_request_restart`: Remote reboot
- `intune_request_scan`: Windows Defender quick or full scan
- `intune_request_locate`: Request device location

**Use Cases:**
- Stale device data refresh
- Applying pending policies via restart
- Security scanning for malware
- Device location for theft/loss recovery
- Troubleshooting by forcing service re-initialization

**Files:** `src/mcp_intune/tools/device/action_tools.py`, `src/mcp_intune/services/device/action_service.py`

---

### 4. Device Actions (Destructive) with Approval Workflow

**Description:** Execute high-risk device operations with built-in approval and audit trail.

**Functionality Implemented:**
- Request device retire (soft decommission)
- Request device wipe (full data erase with optional re-enrollment)
- Request device delete (remove from Intune inventory)
- Three-step approval workflow:
  1. Request submission (with business justification and ticket reference)
  2. Approval/Denial decision (with audit comment)
  3. Execution of approved action
- Request expiration (pending requests don't hang indefinitely)
- Audit trail including requester, approver, business context, and timestamps

**Supported Operations:**
- `intune_request_retire`: Submit retire request (requires device_name, reason, ticket_id)
- `intune_request_wipe`: Submit wipe request (with keep_enrollment_data option)
- `intune_request_delete`: Submit delete request
- `intune_list_pending_actions`: View all pending approval requests
- `intune_approve_action`: Approve a pending request (with optional comment)
- `intune_deny_action`: Deny a pending request
- `intune_execute_action`: Execute a previously approved request

**Use Cases:**
- Decommissioning lost/stolen devices
- Device lifecycle management (end-of-life)
- User offboarding
- Compliance-driven device remediation
- Controlled inventory cleanup

**Audit Features:**
- Request ID tracking
- Submission timestamp and expiration
- Approval/denial history
- Business justification (reason) and ITSM ticket reference
- Approval comment
- All actions logged for compliance

**Files:** `src/mcp_intune/tools/device/action_tools.py`, `src/mcp_intune/services/device/destructive_action_service.py`, `src/mcp_intune/approval/store.py`

---

### 5. Bulk Operations

**Description:** Execute operations on multiple devices in batch.

**Functionality Implemented:**
- Bulk device actions execution
- Batch status tracking
- Error handling and partial success support

**Supported Operations:**
- `intune_bulk_action`: Execute action on multiple device IDs

**Use Cases:**
- Mass policy deployment
- Batch compliance remediation
- Large-scale restart campaigns
- Fleet-wide actions

**Files:** `src/mcp_intune/tools/device/bulk_tools.py`, `src/mcp_intune/services/device/bulk_service.py`

---

### 6. Reporting and Export

**Description:** Generate and export device inventory, compliance, and status reports.

**Functionality Implemented:**
- Export device inventory to CSV/Excel
- Compliance status reports
- Policy assignment reports
- Software inventory reports
- Configurable report filtering and columns

**Supported Operations:**
- `intune_export_report`: Export devices with configurable format and filters
- `intune_get_report_template`: List available report templates and filters

**Use Cases:**
- Executive reporting
- Compliance audits
- Asset inventory management
- Board-level dashboard data
- ITSM integration

**Files:** `src/mcp_intune/tools/reporting/reporting_tools.py`, `src/mcp_intune/services/reporting/reporting_service.py`

---

### 7. Scripts Deployment

**Description:** Deploy remediation and management scripts to devices.

**Functionality Implemented:**
- Script library management (builtin and custom scripts)
- Script assignment to devices or groups
- Execution tracking and result monitoring
- Support for PowerShell and shell scripts

**Supported Operations:**
- `intune_list_scripts`: Available scripts (builtin and custom)
- `intune_deploy_script`: Assign script to device/group
- `intune_get_script_results`: Check execution results and output

**Use Cases:**
- Automated remediation
- Device configuration scripting
- Software deployment
- Scheduled maintenance tasks

**Files:** `src/mcp_intune/tools/scripts/scripts_tools.py`, `src/mcp_intune/services/scripts/scripts_service.py`

---

### 8. Updates Management

**Description:** Monitor, manage, and control OS updates and patches.

**Functionality Implemented:**
- View available Windows updates
- Update policy assignment and status
- Pause/resume update rings
- Expedite critical updates
- Update compliance reporting

**Supported Operations:**
- `intune_get_updates`: List available updates for a device
- `intune_get_update_status`: Update deployment status and compliance
- `intune_set_update_ring`: Assign update ring (preview, current, etc.)
- `intune_pause_updates`: Pause updates on device
- `intune_resume_updates`: Resume updates

**Use Cases:**
- Update compliance tracking
- Patch management
- Update ring assignments
- Scheduled update deployment
- Security update prioritization

**Files:** `src/mcp_intune/tools/updates/updates_tools.py`, `src/mcp_intune/services/updates/updates_service.py`

---

### 9. Autopilot Provisioning

**Description:** Manage Windows Autopilot device provisioning and deployment profiles.

**Functionality Implemented:**
- Autopilot device registration
- Autopilot profile assignment
- Device ESP (Enrollment Status Page) tracking
- Pre-staging device preparation
- Group tag management

**Supported Operations:**
- `intune_register_autopilot_device`: Register device for Autopilot
- `intune_assign_autopilot_profile`: Assign deployment profile
- `intune_get_autopilot_status`: Check device provisioning status
- `intune_get_group_tags`: List available group tags for device grouping

**Use Cases:**
- New device onboarding
- Zero-touch deployment
- Device group assignment
- Provisioning journey tracking
- User-driven or pre-staged deployment

**Files:** `src/mcp_intune/tools/autopilot/autopilot_tools.py`, `src/mcp_intune/services/autopilot/autopilot_service.py`

---

### 10. Governance and Compliance

**Description:** Monitor and enforce governance policies, compliance requirements, and audit controls.

**Functionality Implemented:**
- Compliance policy assignment and status
- Remediation policy tracking
- Device encryption status
- Firewall configuration verification
- Password policy compliance
- Conditional Access integration
- Audit and compliance reporting

**Supported Operations:**
- `intune_get_compliance_policies`: List assigned compliance policies
- `intune_check_compliance_requirement`: Verify specific compliance requirement
- `intune_get_encryption_status`: Check device encryption
- `intune_get_firewall_status`: Verify firewall configuration
- `intune_remediate_compliance`: Trigger remediation workflow
- `intune_export_compliance_report`: Generate compliance audit report

**Use Cases:**
- Regulatory compliance (HIPAA, SOC2, ISO, etc.)
- Security posture assessment
- Encryption verification
- Firewall audits
- Compliance breach investigation
- Audit evidence collection

**Files:** `src/mcp_intune/tools/governance/governance_tools.py`, `src/mcp_intune/services/governance/governance_service.py`

---

### 11. AutoPatch Management

**Description:** Manage Microsoft AutoPatch for automated patch management at scale.

**Functionality Implemented:**
- AutoPatch deployment ring configuration
- Patch testing and staged rollout
- Patch compatibility monitoring
- Device readiness for AutoPatch
- AutoPatch status and report

**Supported Operations:**
- `intune_get_autopatch_status`: Check AutoPatch configuration and status
- `intune_enroll_autopatch`: Enable AutoPatch for a device/group
- `intune_get_patch_schedule`: View patch deployment schedule
- `intune_modify_patch_ring`: Change patch deployment ring (test/staged/production)

**Use Cases:**
- Autonomous patch management
- Patch testing before production rollout
- Device readiness for AutoPatch
- Staged patching for stability
- Patch compliance without manual intervention

**Files:** `src/mcp_intune/tools/autopatch/autopatch_tools.py`, `src/mcp_intune/services/autopatch/autopatch_service.py`

---

## Authentication & Authorization

- **Authentication Method:** Microsoft Authentication Library (MSAL) with client credentials flow
- **Required Permissions:** Intune API scopes configured via `pydantic-settings`
- **Configuration:** Environment variables loaded from `.env`:
  - `TENANT_ID`: Azure AD tenant GUID
  - `CLIENT_ID`: Application registration client ID
  - `CLIENT_SECRET`: Application secret
  - `FASTMCP_TRANSPORT`: HTTP or stdio
  - `FASTMCP_HOST` / `FASTMCP_PORT`: HTTP transport config
  - `APPROVAL_STORE_TYPE`: Approval backend (memory, file, database)

---

## Deployment

- **Entry Point:** `mcp_intune.server:main()`
- **CLI Command:** `mcp-intune`
- **Transport:** HTTP or stdio (configured via env vars)
- **Default Host/Port:** 127.0.0.1:8000 (HTTP)
- **Logging:** Structured logging with audit trail for sensitive operations
- **Background Tasks:** HTTP client cache cleanup loop

---

## Files Analyzed

1. `pyproject.toml` — Project metadata, dependencies, Python ≥3.12 requirement
2. `.env.example` — Configuration template
3. `src/mcp_intune/server.py` — FastMCP server initialization with all tool registrations and lifecycle management
4. `src/mcp_intune/tools/device/device_tools.py` — Device search, overview, hardware, users, apps, compliance, policy tools
5. `src/mcp_intune/tools/device/action_tools.py` — Non-destructive and destructive device actions with approval workflow
6. `src/mcp_intune/tools/device/bulk_tools.py` — Bulk device operations
7. `src/mcp_intune/tools/reporting/reporting_tools.py` — Report generation and export
8. `src/mcp_intune/tools/scripts/scripts_tools.py` — Script deployment management
9. `src/mcp_intune/tools/updates/updates_tools.py` — Updates and patch management
10. `src/mcp_intune/tools/autopilot/autopilot_tools.py` — Autopilot provisioning
11. `src/mcp_intune/tools/governance/governance_tools.py` — Compliance and governance
12. `src/mcp_intune/tools/autopatch/autopatch_tools.py` — AutoPatch management

---

## Divergências com a Documentação

Nenhuma. A análise do código-fonte confirmou que todas as capacidades implementadas estão alinhadas com a descrição no readme.md. Os 11 domínios de funcionalidade (device, actions, bulk, reporting, scripts, updates, autopilot, governance, autopatch, approval, audit) foram todos verificados no código.
