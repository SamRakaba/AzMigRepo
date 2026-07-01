# AGENTS.md — AzMigRepo Development Guide

Instructions for GitHub Copilot agents working in this Azure Migrate project. This project follows the [HVE Core](https://github.com/microsoft/hve-core) methodology (Research → Plan → Implement → Review) for all non-trivial work.

---

## Project Purpose

AzMigRepo is an **Azure Migrate** project for building Copilot Studio agents that process migration assessment data — CSV exports, application inventories, SQL Server inventories, and database inventories. It extends [SamRakaba/AZMrepo](https://github.com/SamRakaba/AZMrepo).

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
- Follow code style in `.github/copilot-instructions.md` section 4
- PowerShell for Azure/Windows automation, Python for data processing
- Always include error handling and structured logging
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

**Last Updated**: June 2026
