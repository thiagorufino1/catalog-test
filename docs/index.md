---
title: Capabilities Catalog
---

# Capabilities Catalog

This catalog documents the functional capabilities of Microsoft Graph-based MCP servers for Teams administration and Intune device management. Each project has been analyzed by reading the source code implementation to verify actual capabilities, not just documentation promises.

## Repositories Analyzed

### [mcp-msteams](mcp-msteams.md)

**GitHub:** [thiagorufino1/mcp-msteams](https://github.com/thiagorufino1/mcp-msteams)  
**Branch:** main

MCP server for Microsoft Teams administrative support and operations. Provides capabilities for user management, team administration, call quality diagnostics, meeting analytics, activity reporting, and service health monitoring.

**Key Domains:**
- User Management (search, profiles, presence, policies)
- Team Management (channels, members, owners, settings, analytics)
- Call Quality Diagnostics (summary, failed calls, participant-level analysis)
- Meeting Analytics (call history, participants)
- Activity Reporting (usage breakdown by communication type)
- Service Health Monitoring (incidents, advisories, detail tracking)

---

### [mcp-intune](mcp-intune.md)

**GitHub:** [thiagorufino1/mcp-intune](https://github.com/thiagorufino1/mcp-intune)  
**Branch:** main

MCP server for Microsoft Intune device management and lifecycle control. Provides comprehensive device inventory, compliance monitoring, configuration management, and device actions with approval workflows for destructive operations.

**Key Domains:**
- Device Discovery & Inventory (search, hardware, software, users, policies)
- Device Actions (non-destructive: sync, restart, scan, locate)
- Destructive Device Actions (retire, wipe, delete with approval workflow)
- Bulk Operations (batch device actions)
- Reporting & Export (inventory and compliance reports)
- Scripts Deployment (remediation and management scripts)
- Updates Management (OS patches and update rings)
- Autopilot Provisioning (zero-touch deployment)
- Governance & Compliance (policy enforcement, audit trails)
- AutoPatch Management (autonomous patching)

---

## Methodology

Each repository was analyzed according to the mandatory method:

1. **Complete file tree enumeration** — Listed all files and directories in the project
2. **Configuration analysis** — Read dependency manifests (`pyproject.toml`), environment templates (`.env.example`)
3. **Entry point review** — Examined `server.py` to understand tool registration and architecture
4. **Implementation verification** — Read each tool module to confirm actual functionality (not just declarations)
5. **Service layer inspection** — Reviewed service implementations to understand Graph API calls, permissions required, data transformation
6. **Comparison with docs** — Verified that README and documentation match the actual code implementation

**Verification Principle:** Only capabilities demonstrated in the source code are documented. Documented capabilities must be traceable to actual implementation in the codebase.

---

## How to Use This Catalog

- **For mcp-msteams capabilities:** See [mcp-msteams.md](mcp-msteams.md)
- **For mcp-intune capabilities:** See [mcp-intune.md](mcp-intune.md)

Each catalog page includes:
- Overview of the project and tech stack
- Detailed breakdown by functional domain
- List of supported operations (tools) in each domain
- Use cases and practical examples
- Files analyzed for each capability
- Divergences between documentation and actual code (if any)
