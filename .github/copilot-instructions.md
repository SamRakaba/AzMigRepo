# GitHub Copilot Repo Instructions — SamRakaba/AzMigRepo

Effective date: 2026-06-30
Repo: SamRakaba/AzMigRepo
Primary focus: Azure Migrate assessment data processing and Copilot Studio agents
Parent project: [SamRakaba/AZMrepo](https://github.com/SamRakaba/AZMrepo)
Dev methodology: [HVE Core](https://github.com/microsoft/hve-core) — Research → Plan → Implement → Review

## 1) Role

You are GitHub Copilot working inside this repository. Your job is to help build, edit, and improve:

- **Azure Migrate data processing** — CSV exports, inventory consolidation, assessment reports
- **Copilot Studio agent definitions** with Power Automate tool integrations
- **Scripts and automation** for migration data workflows
- **Documentation** that is accurate, step-by-step, and actionable

Follow the HVE Core RPI methodology: research before planning, plan before implementing, review after implementing. Use `/task-research`, `/task-plan`, `/task-implement`, `/task-review` for non-trivial work.

## 2) Non-negotiable accuracy rules

### 2.1 No fabrication of product UI, APIs, or steps
- Do not invent Copilot Studio UI labels, Azure Migrate API endpoints, or cmdlet parameters you cannot verify.
- When unsure, ask a clarifying question or present alternatives.

### 2.2 No fabricated values
Never generate fake:
- Subscription IDs, tenant IDs, resource group names, storage account names
- SAS tokens, connection strings, credentials

All example values must be labeled **PLACEHOLDER — replace this** or **EXAMPLE only**.

### 2.3 No fabricated variables
Do not claim variables exist unless the repo or user explicitly defines them.

## 3) Domain context — Azure Migrate

### 3.1 Key Azure Migrate concepts
- **Discovery**: Collection of server metadata and performance data
- **Assessment**: Readiness evaluation against Azure target types
- **Dependency analysis**: Agent-based or agentless dependency mapping
- **Migration waves**: Grouping workloads for phased migration

## 4) Code style preferences

- **PowerShell**: For Windows automation and Azure management
- **Python**: For data processing, CSV/Excel handling, report generation, API calls
- **Markdown**: For documentation with clear structure and examples
- **Naming**: Descriptive names (e.g., `Get-MigrationInventory`, `consolidate_assessment`)
- **Error handling**: Always include try/catch with meaningful messages
- **Logging**: Use structured logging for all automation scripts

## 5) Copilot Studio & Power Automate constraints

### 5.1 Tools (not Actions)
Refer to Copilot Studio integrations as **Tools** (current UI terminology).

### 5.2 "Respond to the agent" — never null
Every Power Automate flow branch must end with "Respond to the agent" with non-null values.

### 5.3 File processing happens in Power Automate
The copilot does not parse CSV/Excel directly. File processing is delegated to Power Automate flows.

### 5.4 Storage must be explicit
Never assume Azure Blob vs SharePoint. Ask the user or document both paths.

## 6) Security & compliance

- Storage containers/libraries must not be public
- Use managed identity where possible; if SAS tokens are needed, use short expiry and minimal permissions
- Never instruct users to paste secrets into docs or code
- Assessment data may contain sensitive infrastructure details — handle accordingly

## 7) Documentation format

Structure all documentation as:
1. **Prerequisites / Assumptions**
2. **Exact Steps (numbered)**
3. **Inputs / Outputs contracts**
4. **Validation checklist**
5. **Troubleshooting**
6. **Placeholders to replace**

## 8) Git commit format

```
<type>: <subject>

Types: feat, fix, docs, style, refactor, test, chore
Examples:
- feat: Add assessment data export script
- docs: Update migration workflow guide
- fix: Handle missing fields in inventory export
```

## 9) Development methodology — HVE Core

This project uses the [HVE Core](https://github.com/microsoft/hve-core) RPI workflow as its development methodology. HVE Core is a tooling/methodology layer — it is not part of the project itself.

### How to use it
- `/task-research <topic>` — Research before implementing new features
- `/task-plan` — Create actionable implementation plans from research
- `/task-implement` — Execute plans with change tracking
- `/task-review` — Validate implementations against specs
- `/security-review` — Run OWASP assessment on scripts handling credentials or data
- `/git-commit` — Create Conventional Commit messages
- **code-review** agent — Multi-perspective review (functional, security, standards)
- **documentation** agent — Doc audit, drift detection, authoring

### Tracking
All HVE Core agent outputs go to `.copilot-tracking/` which is gitignored. Never commit tracking artifacts.

---
End of repo instructions.
