---
title: Capabilities Catalog
---

# Capabilities Catalog

This document catalogs the functional capabilities provided by this repository, based on source code analysis.

## 1. Automated Code Analysis and Documentation Generation

**Description:** Automated workflow system that analyzes source code repositories and generates structured capability catalogs using AI.

**Implemented Functionality:**
- Analyzes source code in authorized repositories
- Identifies functional capabilities from actual code (not just documentation)
- Generates structured documentation in Markdown format
- Automatically creates pull requests with the generated documentation
- Uses Claude Sonnet 4.5 AI model for intelligent code analysis

**Main Operations:**
- `workflow_dispatch`: Manual trigger for catalog generation
- AI-powered source code analysis
- Documentation generation in Markdown format
- Pull request creation with title prefix "[catalog]"
- Safe output validation and controls

**Supporting Files:**
- `.github/workflows/catalog.md` (lines 1-44): Workflow definition with configuration
- `.github/workflows/catalog.lock.yml` (lines 1-1879): Compiled GitHub Actions workflow
- `.github/aw/actions-lock.json` (lines 1-9): Action version locks

---

## 2. GitHub Actions Workflow Orchestration

**Description:** Complete GitHub Actions workflow infrastructure with job orchestration, artifact management, and multi-stage execution.

**Implemented Functionality:**
- Two-job workflow architecture (activation + agent)
- Activation job performs validation and setup
- Agent job executes AI-powered analysis
- Artifact upload/download between jobs
- Timeout management (60 minutes for agent job)
- Concurrency control with queue management

**Main Operations:**
- Secret validation (ANTHROPIC_API_KEY)
- OAuth token verification
- Daily AI credits guardrail checks
- Workflow lock file validation
- Version compatibility checks
- Base branch restoration for PR context

**Supporting Files:**
- `.github/workflows/catalog.lock.yml` (lines 82-107): Activation job definition
- `.github/workflows/catalog.lock.yml` (lines 381-424): Agent job definition and outputs
- `.github/workflows/catalog.lock.yml` (lines 56-64): Workflow triggers and inputs

---

## 3. MCP (Model Context Protocol) Server Integration

**Description:** Integration with multiple MCP servers for AI capabilities, including GitHub API access and safe output management.

**Implemented Functionality:**
- GitHub MCP Server for repository operations
- Safe Outputs MCP Server for controlled PR creation
- MCP Gateway for server orchestration
- Container-based MCP server isolation
- Guard policies for security controls
- Read-only GitHub operations with explicit permissions

**Main Operations:**
- GitHub repository content retrieval
- Commit and release information access
- Code search across repositories
- Pull request creation with validation
- Missing tool/data reporting
- No-operation (noop) reporting for transparency

**Supporting Files:**
- `.github/workflows/catalog.lock.yml` (lines 711-831): MCP Gateway configuration
- `.github/workflows/catalog.lock.yml` (lines 14-16): GitHub MCP toolsets configuration
- `.github/workflows/catalog.md` (lines 15-16): MCP tools declaration

---

## 4. Security and Firewall Controls

**Description:** Multi-layered security infrastructure with firewall, sandboxing, and policy enforcement for safe AI operations.

**Implemented Functionality:**
- Container-based firewall using Squid proxy
- Network isolation for AI agent execution
- Docker-in-Docker security controls
- Allowed domain filtering
- Shell expansion guard to prevent injection
- Temporary file isolation in `/tmp/gh-aw/agent/`

**Main Operations:**
- Firewall container deployment (ghcr.io/github/gh-aw-firewall)
- Agent container isolation
- API proxy for controlled external access
- Security policy validation
- Lockdown mode for GitHub MCP Server

**Supporting Files:**
- `.github/workflows/catalog.lock.yml` (lines 47-53): Container image declarations
- `.github/workflows/catalog.lock.yml` (lines 136-142): Firewall configuration
- `.github/workflows/catalog.lock.yml` (lines 510-520): Automatic lockdown determination

---

## 5. AI Credits and Usage Management

**Description:** Token usage tracking and rate limiting system to control AI model consumption and costs.

**Implemented Functionality:**
- Daily AI credits threshold enforcement (default: 5000 credits)
- Per-workflow AI credits limit (default: 1000 credits)
- Usage scan observations and artifact storage
- Token usage parsing and reporting
- Rate limit error detection
- Cache miss tracking

**Main Operations:**
- Restore daily AIC scan observations from cache
- Check daily workflow token guardrail
- Publish AIC usage scan observations
- Parse token usage from agent execution
- Detect rate limit errors
- Track ambient context usage

**Supporting Files:**
- `.github/workflows/catalog.lock.yml` (lines 89-90): Daily AI credits configuration
- `.github/workflows/catalog.lock.yml` (lines 154-191): Guardrail check implementation
- `.github/workflows/catalog.lock.yml` (lines 181-182): Credit limit variables

---

## 6. Safe Output Management

**Description:** Controlled output system for GitHub operations with validation, constraints, and review requirements.

**Implemented Functionality:**
- Pull request creation with configurable constraints (max 1 PR)
- Title prefix enforcement ("[catalog]")
- Protected files policy (request-review mode)
- Patch size limits (max 4096 bytes)
- Dot folder protection
- Draft PR control
- Missing tool/data reporting
- Incomplete task reporting

**Main Operations:**
- `create_pull_request`: Create PRs with validation
- `missing_tool`: Report missing tool capabilities
- `missing_data`: Report missing data issues
- `noop`: Report no-action completion
- `report_incomplete`: Signal incomplete task execution

**Supporting Files:**
- `.github/workflows/catalog.lock.yml` (lines 544-556): Safe outputs config generation
- `.github/workflows/catalog.lock.yml` (lines 557-710): Safe outputs tools schema
- `.github/workflows/catalog.md` (lines 18-21): Safe output configuration

---

## 7. Git and Repository Management

**Description:** Git operations management for branch handling, credential configuration, and workspace management.

**Implemented Functionality:**
- Repository checkout with sparse checkout support
- PR branch checkout and restoration
- Git credential configuration via GCM
- Base branch folder preservation
- Agent config folder management (.claude, .agents, .github)
- Workspace isolation

**Main Operations:**
- Full repository checkout
- Sparse checkout for .github, .agents, .claude folders
- PR branch checkout with context preservation
- Git credential helper configuration
- Base branch restoration after PR checkout
- Working tree management

**Supporting Files:**
- `.github/workflows/catalog.lock.yml` (lines 214-226): Initial sparse checkout
- `.github/workflows/catalog.lock.yml` (lines 457-460): Full repository checkout
- `.github/workflows/catalog.lock.yml` (lines 485-499): PR branch checkout
- `.github/workflows/catalog.lock.yml` (lines 478-483): Git credentials configuration

---

## 8. OpenTelemetry Integration

**Description:** Distributed tracing and observability infrastructure for monitoring workflow execution.

**Implemented Functionality:**
- OTLP (OpenTelemetry Protocol) endpoint configuration
- Trace ID and span ID propagation across jobs
- Service name identification (gh-aw.catalog)
- Resource attributes for context
- Telemetry headers masking for security
- Custom OTLP endpoints support

**Main Operations:**
- Generate trace IDs for workflow runs
- Propagate span IDs between jobs
- Mask sensitive telemetry headers
- Configure OTLP exporters
- Track workflow execution across distributed components

**Supporting Files:**
- `.github/workflows/catalog.lock.yml` (lines 73-79): OTEL environment configuration
- `.github/workflows/catalog.lock.yml` (lines 122-123): Telemetry header masking
- `.github/workflows/catalog.lock.yml` (lines 819-828): Gateway OpenTelemetry config

---

## 9. Claude AI Engine Integration

**Description:** Integration with Anthropic's Claude AI model (Sonnet 4.5) for intelligent code analysis.

**Implemented Functionality:**
- Claude Code CLI installation and execution
- Model configuration (claude-sonnet-4-5)
- Anthropic API key management
- Agent version tracking (2.1.273)
- Prompt generation and interpolation
- Context management and token budgeting

**Main Operations:**
- Install Claude Code CLI via npm
- Configure Claude engine with model selection
- Generate AI prompts from templates
- Interpolate variables in prompts
- Execute Claude agent for analysis
- Parse and collect AI-generated output

**Supporting Files:**
- `.github/workflows/catalog.lock.yml` (lines 507-508): Claude Code CLI installation
- `.github/workflows/catalog.lock.yml` (lines 127-129): Engine and model configuration
- `.github/workflows/catalog.md` (lines 7-9): Engine declaration
- `.github/workflows/catalog.lock.yml` (lines 202-206): API key validation

---

## 10. Workflow Compilation and Version Management

**Description:** Workflow definition compilation system with version tracking and compatibility validation.

**Implemented Functionality:**
- Markdown-to-YAML workflow compilation
- Compiler version tracking (v0.89.21)
- Schema version management (v4)
- Frontmatter and body hashing for integrity
- Strict compilation mode
- Stale lock file detection
- Version update checking

**Main Operations:**
- Compile `.md` workflow files to `.lock.yml`
- Validate workflow lock file timestamps
- Check compiler version compatibility
- Detect stale lock files
- Report version update requirements
- Maintain compilation metadata

**Supporting Files:**
- `.github/workflows/catalog.lock.yml` (lines 1-3): Compilation metadata headers
- `.github/workflows/catalog.lock.yml` (lines 233-246): Lock file validation
- `.github/workflows/catalog.lock.yml` (lines 247-260): Version compatibility check
- `.github/workflows/catalog.md` (lines 1-22): Source workflow definition

---

## Summary

This repository provides a **self-contained GitHub Actions workflow system** for automated code analysis and documentation generation using AI. The system integrates multiple capabilities:

- **AI-Powered Analysis**: Uses Claude Sonnet 4.5 for intelligent code understanding
- **Security-First Design**: Multi-layered security with firewall, sandboxing, and policy controls
- **Cost Management**: Token tracking and rate limiting for AI usage
- **Safe Operations**: Controlled GitHub operations with validation and review workflows
- **Observability**: Full distributed tracing with OpenTelemetry
- **Extensibility**: MCP server architecture for pluggable capabilities

The workflow is designed for safe, controlled, and observable AI-powered repository analysis and documentation generation.
