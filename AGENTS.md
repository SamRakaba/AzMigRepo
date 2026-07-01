# AGENTS.md — AzMigRepo Agent Definitions

This file defines the agents, skills, and orchestration patterns for the AzMigRepo project. It leverages **HVE Core (Hypervelocity Engineering)** agents from [microsoft/hve-core](https://github.com/microsoft/hve-core) for structured AI-assisted development workflows.

> **HVE Core Integration**: This project uses the HVE Core framework for repeatable, standards-aligned AI workflows. Install via the [VS Code extension](https://marketplace.visualstudio.com/items?itemName=ise-hve-essentials.hve-core) or CLI plugin (`copilot plugin marketplace add microsoft/hve-core`).

---

## Agent Overview

### HVE Core Agents — RPI Workflow

| Agent | Purpose | Key Constraint |
|-------|---------|---------------|
| **rpi-agent** | Autonomous agent with subagent delegation for complex tasks | Requires subagent tool enabled |
| **task-researcher** | Produces research documents with evidence-based recommendations | Research-only; never plans or implements |
| **task-planner** | Creates plan + details files from research | Requires research first; never implements code |
| **task-implementor** | Executes implementation plans with subagent delegation | Requires completed plan files |
| **task-reviewer** | Validates implementation against research and plan specs | Requires research/plan artifacts |
| **task-challenger** | Adversarial questioning of completed implementations | Experimental; no hints or leading questions |

### HVE Core Agents — Documentation & Planning

| Agent | Purpose | Key Constraint |
|-------|---------|---------------|
| **adr-creation** | Interactive Architecture Decision Record coaching | Socratic coaching approach |
| **brd-builder** | Creates Business Requirements Documents with references | Solution-agnostic requirements focus |
| **documentation** | Documentation audit, drift, authoring, and validation | Uses shared documentation skill |
| **prd-builder** | Creates Product Requirements Documents through guided Q&A | Iterative questioning; state-tracked sessions |
| **product-manager-advisor** | Requirements discovery, story quality, prioritization | Principles over format; delegates to builders |
| **system-architecture-reviewer** | Reviews system designs for trade-offs and ADR alignment | Scoped review; delegates security concerns |
| **ux-ui-designer** | JTBD analysis, user journey mapping, accessibility | Research artifacts only |

### HVE Core Agents — Security & Compliance

| Agent | Purpose | Key Constraint |
|-------|---------|---------------|
| **security-planner** | STRIDE-based security model analysis with standards mapping | Six-phase conversational workflow |
| **security-reviewer** | OWASP vulnerability assessment with subagent verification | Delegates reference reading to subagents |
| **sssc-planner** | Supply chain security (OpenSSF, SLSA, Sigstore, SBOM) | Six-phase conversational workflow |
| **rai-planner** | Responsible AI assessment (Microsoft RAI + NIST AI RMF) | Six-phase conversational workflow |

### HVE Core Agents — Code & Review

| Agent | Purpose | Key Constraint |
|-------|---------|---------------|
| **code-review** | Human-gated review with five perspective subagents | Operator confirms scope; review-only |
| **prompt-builder** | Engineers and validates instruction/prompt files | Dual-persona with auto-testing |
| **memory** | Persists repository facts for future tasks | Stores only durable, actionable facts |

### HVE Core Agents — Generators

| Agent | Purpose | Key Constraint |
|-------|---------|---------------|
| **gen-jupyter-notebook** | Creates structured EDA notebooks from data sources | Requires data dictionaries |
| **gen-streamlit-dashboard** | Develops multi-page Streamlit dashboards | Uses Context7 for documentation |
| **gen-data-spec** | Generates data dictionaries and profiles | Produces JSON and markdown artifacts |

### HVE Core Agents — Platform Integration

| Agent | Purpose | Key Constraint |
|-------|---------|---------------|
| **github-backlog-manager** | GitHub backlog management with community interaction | Uses MCP GitHub tools |
| **jira-backlog-manager** | Jira backlog management with workflow dispatch | Uses Jira skill planning workflows |
| **ado-prd-to-wit** | Analyzes PRDs and plans Azure DevOps work item hierarchies | Planning-only; does not create items |

---

## HVE Core RPI Workflow

The **Research → Plan → Implement → Review** (RPI) methodology from HVE Core structures how tasks are approached in this project:

### RPI Example

```
Example: "Add CSV export validation for assessment files"

1. /task-research csv-export-validation
   → task-researcher produces research doc at:
     .copilot-tracking/research/YYYY-MM-DD-csv-export-validation-research.md

2. /task-plan (attach research)
   → task-planner produces:
     .copilot-tracking/plans/YYYY-MM-DD-task-plan.instructions.md
     .copilot-tracking/details/YYYY-MM-DD-task-details.md

3. /task-implement
   → task-implementor executes plan, tracks changes at:
     .copilot-tracking/changes/YYYY-MM-DD-task-changes.md

4. /task-review (or select task-reviewer)
   → task-reviewer validates against research + plan:
     .copilot-tracking/reviews/YYYY-MM-DD-csv-export-validation-review.md
```

### RPI Quick Path

For smaller tasks, use **rpi-agent** which autonomously handles the full cycle:
1. Select **rpi-agent** from agent picker
2. Provide your request
3. Agent researches, implements, and verifies autonomously

---

## HVE Core Skills

Skills add reusable tool capabilities to agents. The following HVE Core skill categories are available:

### RPI Skills
| Skill | Purpose |
|-------|---------|
| `rpi-research` | Deep research with evidence-based recommendations |
| `rpi-plan` | Create implementation plans from research |
| `rpi-implement` | Execute plans with tracking |
| `rpi-review` | Validate implementations against specs |
| `rpi-quick` | Full end-to-end RPI flow |

### Prompt Engineering Skills
| Skill | Purpose |
|-------|---------|
| `prompt-analyze` | Analyze and evaluate prompt/instruction quality |
| `prompt-builder` | Engineer and validate instruction/prompt files |
| `prompt-refactor` | Refactor and improve existing prompts |

### Security Skills
| Skill | Purpose |
|-------|---------|
| `owasp-top-10` | OWASP Top 10 web vulnerability checks |
| `owasp-llm` | OWASP LLM vulnerability assessment |
| `owasp-agentic` | OWASP agentic AI security checks |
| `owasp-infrastructure` | Infrastructure security assessment |
| `owasp-cicd` | CI/CD pipeline security checks |
| `secure-by-design` | Secure by Design principles assessment |

### Documentation & Architecture Skills
| Skill | Purpose |
|-------|---------|
| `documentation` | Documentation audit, authoring, validation |
| `architecture-diagrams` | Generate architecture diagrams |

### Coding Standards Skills
| Skill | Purpose |
|-------|---------|
| `coding-standards` | Apply coding conventions automatically |

### Project Planning Skills
| Skill | Purpose |
|-------|---------|
| `project-planning` | Project planning and tracking |

### Platform Skills
| Skill | Purpose |
|-------|---------|
| `github` | GitHub integration (issues, PRs, backlog) |
| `jira` | Jira integration (issues, workflows) |
| `gitlab` | GitLab MR, pipeline, and job inspection |

### Accessibility & RAI Skills
| Skill | Purpose |
|-------|---------|
| `accessibility` | Accessibility compliance checks |
| `rai` | Responsible AI assessment skills |
| `design-thinking` | Design thinking methodology support |

---

## HVE Core Prompts

Prompts provide repeatable workflow entry points via `/prompt-name` syntax:

### Onboarding & Planning
| Prompt | Invocation | Purpose |
|--------|-----------|---------|
| Task Research | `/task-research <topic>` | Research a topic with evidence |
| Task Plan | `/task-plan` | Create implementation plan from research |
| Task Implement | `/task-implement` | Execute plans with tracking |

### Source Control & Commit Quality
| Prompt | Invocation | Purpose |
|--------|-----------|---------|
| Git Commit | `/git-commit` | Stage all changes and create Conventional Commit |
| Git Commit Message | `/git-commit-message` | Generate commit message for staged changes |
| Git Merge | `/git-merge` | Merge, rebase, conflict handling |
| Git Setup | `/git-setup` | Verification-first Git configuration |

### Security
| Prompt | Invocation | Purpose |
|--------|-----------|---------|
| Security Review | `/security-review` | Full OWASP assessment against codebase |
| Security Review - Web | `/security-review-web` | OWASP Top 10 web vulnerability scan |
| Security Review - LLM | `/security-review-llm` | OWASP LLM/Agentic assessment |
| Incident Response | `/incident-response` | Azure incident triage and RCA |
| Risk Register | `/risk-register` | Qualitative risk assessment with P×I matrix |

### Data Science
| Prompt | Invocation | Purpose |
|--------|-----------|---------|
| Synthetic Data | `/synth-data-generate` | Generate synthetic data for testing |

### Pull Requests
| Prompt | Invocation | Purpose |
|--------|-----------|---------|
| Pull Request | `/pull-request` | PR description and review assistance |

---

## Agent Workflows

### Autonomous Task Completion
1. Select **rpi-agent** → provide request → agent researches, implements, verifies

#### Feature Planning (RPI)
1. **task-researcher** → create research doc
2. **task-planner** → generate plan + details files
3. **task-implementor** → execute plan with tracking
4. **task-reviewer** → validate against specs

#### Code Review
1. Select **code-review** → confirm scope → choose perspectives (`functional`, `standards`, `accessibility`, `security`, `pr`, or `full`) and depth tier (`basic`, `standard`, `comprehensive`)
2. Receive merged `review.md` under `.copilot-tracking/reviews/code-reviews/<branch>/`

#### Security Assessment
1. Select **security-reviewer** → auto-profiles codebase → runs applicable OWASP skills → verifies findings → generates report
2. Modes: `audit` (full scan), `diff` (changed files only), `plan` (pre-implementation risk)

#### Documentation Workflow
1. Select **documentation** → reviews existing docs → identifies drift/gaps → authors updates
2. For formal docs: **prd-builder** (product requirements) or **brd-builder** (business requirements)

---

## Tracking Artifacts

HVE Core agents create artifacts under `.copilot-tracking/` (gitignored by default):

| Path | Created By | Content |
|------|-----------|---------|
| `.copilot-tracking/research/` | task-researcher | Research documents |
| `.copilot-tracking/plans/` | task-planner | Implementation plans |
| `.copilot-tracking/details/` | task-planner | Step-by-step execution details |
| `.copilot-tracking/changes/` | task-implementor | Change tracking logs |
| `.copilot-tracking/reviews/` | task-reviewer, code-review | Review findings |
| `.copilot-tracking/security/` | security-reviewer | OWASP assessment reports |
| `.copilot-tracking/adrs/` | adr-creation | Architecture Decision Records |
| `.copilot-tracking/memory/` | memory | Session continuity context |
| `.copilot-tracking/documentation/` | documentation | Documentation session tracking |
| `.copilot-tracking/security-plans/` | security-planner | STRIDE security plans |
| `.copilot-tracking/rai-plans/` | rai-planner | Responsible AI assessments |
| `.copilot-tracking/sssc-plans/` | sssc-planner | Supply chain security plans |

---

## Skill Reference Quick Table

| Skill ID | Category | Description |
|----------|----------|-------------|
| `rpi-research` | RPI Workflow | Evidence-based research |
| `rpi-plan` | RPI Workflow | Implementation planning |
| `rpi-implement` | RPI Workflow | Plan execution with tracking |
| `rpi-review` | RPI Workflow | Spec validation |
| `rpi-quick` | RPI Workflow | Full end-to-end RPI cycle |
| `prompt-analyze` | Prompt Engineering | Prompt quality analysis |
| `prompt-builder` | Prompt Engineering | Prompt file creation |
| `prompt-refactor` | Prompt Engineering | Prompt improvement |
| `documentation` | Documentation | Doc audit and authoring |
| `architecture-diagrams` | Documentation | Architecture visualization |
| `coding-standards` | Standards | Convention enforcement |
| `project-planning` | Planning | Project tracking |
| `owasp-top-10` | Security | Web vulnerability checks |
| `owasp-llm` | Security | LLM vulnerability assessment |
| `owasp-agentic` | Security | Agentic AI security |
| `owasp-infrastructure` | Security | Infrastructure security |
| `owasp-cicd` | Security | CI/CD pipeline security |
| `secure-by-design` | Security | Secure design principles |
| `accessibility` | Accessibility | Accessibility compliance |
| `rai` | Responsible AI | RAI assessment |
| `design-thinking` | Design | Design thinking methods |
| `github` | Platform | GitHub integration |
| `jira` | Platform | Jira integration |
| `gitlab` | Platform | GitLab integration |

---

## Setup — Installing HVE Core

### VS Code Extension (Recommended)
```
# Install from VS Code Marketplace
ext install ise-hve-essentials.hve-core
```

### Copilot CLI Plugin
```bash
copilot plugin marketplace add microsoft/hve-core
copilot plugin install hve-core@hve-core
```

### What Gets Installed
- Custom agents (available in agent picker dropdown)
- Skills (invocable via `/skill-name`)
- Prompts (invocable via `/prompt-name`)
- Instructions (auto-applied based on file patterns)
- Tracking directory structure (`.copilot-tracking/`)

---

**Last Updated**: June 2026
