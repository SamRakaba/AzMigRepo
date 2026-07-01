# AGENTS.md — AzMigRepo Agent Reference & Development Guide

> **Full implementation guide**: [`AZURE_MIGRATE_AGENTS_GUIDE.md`](AZURE_MIGRATE_AGENTS_GUIDE.md) (232KB) — complete step-by-step instructions for building all 5 agents, Power Automate flows, testing, and troubleshooting.

---

## Agent Quick Reference

This project documents 5 Copilot Studio agents using a **GPT-4.1-first** architecture where the model handles all intelligence and Power Automate handles only file I/O.

| # | Agent | Purpose | Guide Section |
|---|-------|---------|---------------|
| 1 | **File Upload Handler** | Accept CSV/XLSX uploads, GPT-4.1 sheet compliance verification, coordinate processing | [Agent 1](AZURE_MIGRATE_AGENTS_GUIDE.md#agent-1-file-upload-handler) |
| 2 | **App Inventory Processor** | GPT-4.1 noise detection, deduplication, application consolidation | [Agent 2](AZURE_MIGRATE_AGENTS_GUIDE.md#agent-2-application-inventory-processor) |
| 3 | **SQL Server Processor** | GPT-4.1 SQL instance consolidation, version grouping | [Agent 3](AZURE_MIGRATE_AGENTS_GUIDE.md#agent-3-sql-server-inventory-processor) |
| 4 | **Web App Processor** | GPT-4.1 web application consolidation by server type | [Agent 4](AZURE_MIGRATE_AGENTS_GUIDE.md#agent-4-web-app-inventory-processor) |
| 5 | **Report Generator** | Combine processed data, write Excel to Blob, provide download link | [Agent 5](AZURE_MIGRATE_AGENTS_GUIDE.md#agent-5-report-generator) |

## Global Variables

All agents share state via Global variables (JSON strings in Copilot Studio):

| Variable | Type | Purpose | Set By |
|----------|------|---------|--------|
| `Global.sessionId` | String | Unique session identifier | PA flow (Agent 1) |
| `Global.uploadStatus` | String | File upload status | Agent 1 |
| `Global.uploadedFilePath` | String | Blob path to uploaded file | PA flow (Agent 1) |
| `Global.detectedSheets` | String | Sheet names found in file (JSON) | Agent 1 (GPT-4.1) |
| `Global.consolidatedApplications` | String | Unique application list (JSON) | Agent 2 (GPT-4.1) |
| `Global.applicationDedupCSV` | String | CSV dedup report | Agent 2 (GPT-4.1) |
| `Global.consolidatedSQLInstances` | String | Unique SQL Server list (JSON) | Agent 3 (GPT-4.1) |
| `Global.consolidatedWebApps` | String | Unique web app list (JSON) | Agent 4 (GPT-4.1) |
| `Global.processingStatus` | String | Workflow status | Agents 1-5 |
| `Global.errorMessage` | String | Error messages | Any agent |
| `Global.downloadUrl` | String | Report download URL | Agent 5 (PA flow) |

## Power Automate Flows

| Flow Name | Trigger Type | Purpose |
|-----------|-------------|---------|
| **Handle File Upload (Blob Storage)** | Agent Tool | Save file to Blob, return path |
| **Read Application Inventory Data** | Agent Tool | Read raw app data for GPT-4.1 analysis |
| **Read SQL Server Inventory Data** | Agent Tool | Read raw SQL data for GPT-4.1 analysis |
| **Read Web App Inventory Data** | Agent Tool | Read raw web app data for GPT-4.1 analysis |
| **Generate Consolidated Report** | Agent Tool | Write Excel to Blob, return download URL |
| **Azure Migrate Processing Orchestrator** | HTTP (optional) | Multi-file batch processing |
| **Get Processing Status** | Utility | Check processing status |

---

## Development Methodology

Use the **HVE Core RPI workflow** when building or modifying this project:

1. **Research** (`/task-research <topic>`) — Investigate before coding. Produce evidence-based findings.
2. **Plan** (`/task-plan`) — Create implementation plan from research. Never skip to code.
3. **Implement** (`/task-implement`) — Execute the plan with change tracking.
4. **Review** (`/task-review`) — Validate against research and plan specs.

For quick, small tasks use **rpi-agent** which handles the full cycle autonomously.

## How Copilot Should Work Here

### When researching
- Use `/task-research` to investigate Azure Migrate APIs, CSV schemas, or Copilot Studio patterns before implementing
- Cite Azure documentation and the parent [AZMrepo](https://github.com/SamRakaba/AZMrepo) as primary references

### When planning
- Use `/task-plan` to create actionable plans with checklist items
- Plans should reference the copilot-instructions.md conventions (sections 2–8)

### When implementing
- Follow conventions in `.github/copilot-instructions.md` section 4
- Markdown for documentation, JSON for flow contracts and variable schemas
- Reference `AZURE_MIGRATE_AGENTS_GUIDE.md` for implementation details
- Use `/task-implement` for tracked execution

### When reviewing
- Use `/task-review` or the **code-review** agent to validate changes
- Apply perspectives: `functional` (logic), `security` (credentials/data), `standards` (conventions)
- Use **security-reviewer** for OWASP assessment on scripts handling credentials or sensitive data

### When writing documentation
- Use the **documentation** agent for auditing and authoring
- Use **prd-builder** or **brd-builder** for formal requirements
- Follow the format in `.github/copilot-instructions.md` section 7

### When committing
- Use `/git-commit` for Conventional Commit messages
- Follow format: `<type>: <subject>` (feat, fix, docs, refactor, test, chore)

## Security Practices

- Use **security-reviewer** (`/security-review`) before merging scripts that handle credentials, storage connections, or assessment data
- Use **security-planner** for STRIDE threat modeling on new agent designs
- Assessment data may contain sensitive infrastructure details — handle accordingly

## Tracking

All HVE Core agent outputs go to `.copilot-tracking/` (gitignored — never commit these):
- `research/` — Research documents
- `plans/` and `details/` — Implementation plans
- `changes/` — Change logs
- `reviews/` — Review findings
- `security/` — OWASP reports

## HVE Core Setup

Install HVE Core to use the agents, skills, and prompts referenced above:

```bash
# VS Code Extension
ext install ise-hve-essentials.hve-core

# OR Copilot CLI Plugin
copilot plugin marketplace add microsoft/hve-core
copilot plugin install hve-core@hve-core
```

---

**Last Updated**: July 2026
