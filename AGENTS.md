# AGENTS.md — AzMigRepo HVE Agent Definitions

This file defines the agents, skills, and orchestration patterns for Azure Migrate Hyper-V Environment (HVE) workflows in this project. It combines **domain-specific HVE agents** with **HVE Core (Hypervelocity Engineering)** agents from [microsoft/hve-core](https://github.com/microsoft/hve-core) for structured AI-assisted development workflows.

> **HVE Core Integration**: This project uses the HVE Core framework for repeatable, standards-aligned AI workflows. Install via the [VS Code extension](https://marketplace.visualstudio.com/items?itemName=ise-hve-essentials.hve-core) or CLI plugin (`copilot plugin marketplace add microsoft/hve-core`).

---

## Agent Overview

### Domain Agents — Azure Migrate HVE

| Agent | Purpose | Trigger Examples |
|-------|---------|-----------------|
| **HVE Discovery Agent** | Discover Hyper-V hosts, clusters, and guest VMs | "discover Hyper-V hosts", "scan datacenter", "list HVE inventory" |
| **Assessment Agent** | Evaluate migration readiness for Azure targets | "assess readiness", "run HVE assessment", "check migration eligibility" |
| **CSV Processor Agent** | Process Azure Migrate export CSV files | "process assessment export", "consolidate CSV data", "clean inventory file" |
| **Migration Planner Agent** | Plan migration waves and estimate costs | "plan migration waves", "estimate Azure costs", "group workloads" |
| **Dependency Mapper Agent** | Map application dependencies across VMs | "map dependencies", "show VM connections", "analyze app topology" |
| **Report Generator Agent** | Generate Excel reports with findings | "generate report", "create assessment spreadsheet", "export summary" |

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

## Agent 1: HVE Discovery Agent

### Role
Discovers and inventories Hyper-V hosts, clusters, and guest VMs. Collects metadata including CPU, memory, disk, network adapters, and OS details.

### Skills
- `discover-hyperv-hosts` — Scan a subnet or host list for Hyper-V servers
- `collect-vm-inventory` — Gather guest VM metadata from discovered hosts
- `collect-host-metrics` — Gather host-level capacity and utilization data
- `detect-clusters` — Identify Hyper-V failover clusters
- `validate-appliance` — Check Azure Migrate appliance connectivity and status

### Tools (Copilot Studio)
- **Power Automate Flow**: `Flow-HVE-DiscoverHosts` — Runs PowerShell remoting against target hosts
- **Power Automate Flow**: `Flow-HVE-CollectVMInventory` — Collects VM details via Hyper-V WMI/CIM

### Inputs
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `target_hosts` | string (CSV) | Yes | Comma-separated Hyper-V host names or IPs |
| `credential_vault` | string | Yes | Key Vault name for Hyper-V admin credentials |
| `include_performance` | boolean | No | Collect performance counters (default: false) |

### Outputs
| Field | Type | Description |
|-------|------|-------------|
| `discovered_hosts` | integer | Number of Hyper-V hosts found |
| `total_vms` | integer | Total guest VMs discovered |
| `inventory_file_url` | string | Download link to inventory CSV/Excel |
| `status` | string | Success / Partial / Failed |
| `errors` | string | Any errors encountered |

### Sample Conversation
```
User: "Discover Hyper-V hosts in my datacenter"
Agent: "I'll scan for Hyper-V hosts. Please provide:
        1. Host names or IP range to scan
        2. Key Vault name with admin credentials"
User: "Hosts: HV-HOST-01, HV-HOST-02, HV-HOST-03. Vault: kv-migration-prod"
Agent: "Scanning 3 hosts... Found 3 Hyper-V hosts with 47 guest VMs.
        📄 Inventory report: [Download](PLACEHOLDER)
        Would you like to run a readiness assessment on these VMs?"
```

---

## Agent 2: Assessment Agent

### Role
Evaluates discovered Hyper-V VMs for migration readiness against Azure targets (Azure VM, Azure VMware Solution, AKS).

### Skills
- `assess-azure-vm-readiness` — Check VM compatibility with Azure VM SKUs
- `assess-avs-readiness` — Evaluate for Azure VMware Solution migration
- `assess-containerization` — Evaluate workloads for AKS containerization
- `identify-blockers` — Flag migration blockers (unsupported OS, features)
- `recommend-sku` — Recommend Azure VM sizes based on utilization

### Tools (Copilot Studio)
- **Power Automate Flow**: `Flow-HVE-RunAssessment` — Calls Azure Migrate REST API
- **Power Automate Flow**: `Flow-HVE-GetAssessmentResults` — Retrieves assessment results

### Inputs
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `migrate_project` | string | Yes | Azure Migrate project name |
| `assessment_target` | enum | Yes | `AzureVM`, `AVS`, or `AKS` |
| `sizing_criteria` | enum | No | `performance-based` or `as-on-premises` (default: performance) |
| `resource_group` | string | Yes | Resource group of the Migrate project |

### Outputs
| Field | Type | Description |
|-------|------|-------------|
| `ready_count` | integer | VMs ready for migration |
| `conditionally_ready` | integer | VMs with conditions/warnings |
| `not_ready_count` | integer | VMs with blockers |
| `unknown_count` | integer | VMs with insufficient data |
| `assessment_url` | string | Link to full assessment in Azure portal |
| `export_file_url` | string | Download link to assessment CSV |

---

## Agent 3: CSV Processor Agent

### Role
Processes Azure Migrate export CSV files — cleans data, removes duplicates, consolidates inventories, and produces structured output.

### Skills
- `parse-assessment-csv` — Parse and validate Azure Migrate CSV exports
- `deduplicate-inventory` — Remove duplicate VM entries across assessments
- `consolidate-servers` — Merge server data from multiple discovery sources
- `consolidate-databases` — Aggregate database inventory (SQL on Hyper-V VMs)
- `validate-data-quality` — Flag missing or inconsistent fields

### Tools (Copilot Studio)
- **Power Automate Flow**: `Flow-CSV-ParseAndClean` — Parses uploaded CSV, removes noise
- **Power Automate Flow**: `Flow-CSV-ConsolidateInventory` — Merges multiple CSV files

### Inputs
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `uploaded_file` | file | Yes | CSV file uploaded by user |
| `processing_mode` | enum | Yes | `application`, `sql_server`, `database`, or `full` |
| `dedup_strategy` | enum | No | `hostname`, `ip_address`, or `both` (default: hostname) |

### Outputs
| Field | Type | Description |
|-------|------|-------------|
| `records_processed` | integer | Total records in source file |
| `duplicates_removed` | integer | Duplicate entries removed |
| `clean_records` | integer | Records in output file |
| `output_file_url` | string | Download link to processed file |
| `data_quality_notes` | string | Warnings about data issues |

---

## Agent 4: Migration Planner Agent

### Role
Plans migration waves, estimates Azure costs, and groups workloads by dependency and priority.

### Skills
- `plan-migration-waves` — Group VMs into migration waves based on dependencies
- `estimate-azure-costs` — Calculate monthly Azure cost for migrated workloads
- `prioritize-workloads` — Rank workloads by migration complexity and business value
- `generate-timeline` — Create migration timeline with milestones
- `compare-targets` — Compare cost/fit across Azure VM, AVS, and AKS

### Tools (Copilot Studio)
- **Power Automate Flow**: `Flow-Plan-EstimateCosts` — Calls Azure Pricing API
- **Power Automate Flow**: `Flow-Plan-GroupWorkloads` — Groups VMs by dependency graph

### Inputs
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `assessment_data` | file/URL | Yes | Assessment results (CSV or assessment URL) |
| `target_region` | string | Yes | Azure region for cost estimation |
| `migration_timeline` | string | No | Desired completion date |
| `max_wave_size` | integer | No | Maximum VMs per migration wave (default: 20) |

### Outputs
| Field | Type | Description |
|-------|------|-------------|
| `total_waves` | integer | Number of migration waves |
| `estimated_monthly_cost` | string | Total estimated Azure monthly cost |
| `migration_plan_url` | string | Download link to migration plan Excel |
| `wave_summary` | string | Summary of each wave |

---

## Agent 5: Dependency Mapper Agent

### Role
Maps application dependencies across Hyper-V VMs using Azure Migrate dependency analysis or network connection data.

### Skills
- `map-vm-dependencies` — Visualize inbound/outbound connections per VM
- `identify-app-groups` — Group VMs into application tiers (web/app/db)
- `detect-cross-host-deps` — Find dependencies spanning multiple Hyper-V hosts
- `export-dependency-graph` — Export dependency data as CSV or JSON

### Tools (Copilot Studio)
- **Power Automate Flow**: `Flow-Dep-GetDependencyData` — Retrieves dependency data from Azure Migrate
- **Power Automate Flow**: `Flow-Dep-AnalyzeConnections` — Analyzes network connections

---

## Agent 6: Report Generator Agent

### Role
Generates formatted Excel reports summarizing discovery, assessment, and migration planning results.

### Skills
- `generate-discovery-report` — Summary of discovered Hyper-V hosts and VMs
- `generate-assessment-report` — Readiness breakdown with SKU recommendations
- `generate-cost-report` — Cost estimation with comparisons across targets
- `generate-executive-summary` — High-level summary for stakeholders
- `format-spreadsheet` — Apply formatting, charts, and pivot tables

### Tools (Copilot Studio)
- **Power Automate Flow**: `Flow-Report-GenerateExcel` — Creates formatted Excel workbook
- **Power Automate Flow**: `Flow-Report-AddCharts` — Adds charts and visualizations

---

## HVE Core RPI Workflow — How It Applies Here

The **Research → Plan → Implement → Review** (RPI) methodology from HVE Core structures how migration tasks are approached in this project:

### RPI for Azure Migrate Tasks

```
Example: "Add Hyper-V cluster discovery support"

1. /task-research hyper-v-cluster-discovery
   → task-researcher produces research doc at:
     .copilot-tracking/research/YYYY-MM-DD-hyper-v-cluster-discovery-research.md

2. /task-plan (attach research)
   → task-planner produces:
     .copilot-tracking/plans/YYYY-MM-DD-task-plan.instructions.md
     .copilot-tracking/details/YYYY-MM-DD-task-details.md

3. /task-implement
   → task-implementor executes plan, tracks changes at:
     .copilot-tracking/changes/YYYY-MM-DD-task-changes.md

4. /task-review (or select task-reviewer)
   → task-reviewer validates against research + plan:
     .copilot-tracking/reviews/YYYY-MM-DD-cluster-discovery-review.md
```

### RPI Quick Path

For smaller tasks, use **rpi-agent** which autonomously handles the full cycle:
1. Select **rpi-agent** from agent picker
2. Provide request (e.g., "Add CSV export validation for Azure Migrate assessment files")
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

## Agent Orchestration

### Domain Agent Chain (Copilot Studio)

Agents can be chained for end-to-end Azure Migrate workflows:

```
1. HVE Discovery Agent → discovers hosts and VMs
2. Assessment Agent → evaluates readiness
3. Dependency Mapper Agent → maps app dependencies
4. CSV Processor Agent → cleans and consolidates data
5. Migration Planner Agent → plans waves and estimates costs
6. Report Generator Agent → produces final Excel deliverable
```

#### Orchestration via Copilot Studio Topics

Create a **master topic** in Copilot Studio that chains agents:

```
Trigger: "Run full HVE assessment"
Flow:
  1. Call HVE Discovery Agent → get inventory
  2. Call Assessment Agent → get readiness
  3. Call Dependency Mapper → get dependencies
  4. Call Migration Planner → get plan + costs
  5. Call Report Generator → produce final report
  6. Return report download link to user
```

### HVE Core Agent Workflows

#### Autonomous Task Completion
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

### Domain Skills (Azure Migrate HVE)

| Skill ID | Agent | Description |
|----------|-------|-------------|
| `discover-hyperv-hosts` | Discovery | Scan for Hyper-V servers |
| `collect-vm-inventory` | Discovery | Gather VM metadata |
| `collect-host-metrics` | Discovery | Host capacity data |
| `detect-clusters` | Discovery | Find failover clusters |
| `validate-appliance` | Discovery | Check appliance status |
| `assess-azure-vm-readiness` | Assessment | Azure VM compatibility |
| `assess-avs-readiness` | Assessment | AVS migration fit |
| `assess-containerization` | Assessment | AKS readiness |
| `identify-blockers` | Assessment | Migration blockers |
| `recommend-sku` | Assessment | Azure VM size recs |
| `parse-assessment-csv` | CSV Processor | Parse Migrate exports |
| `deduplicate-inventory` | CSV Processor | Remove duplicates |
| `consolidate-servers` | CSV Processor | Merge server data |
| `consolidate-databases` | CSV Processor | Aggregate DB inventory |
| `validate-data-quality` | CSV Processor | Data quality checks |
| `plan-migration-waves` | Planner | Group migration waves |
| `estimate-azure-costs` | Planner | Azure cost estimation |
| `prioritize-workloads` | Planner | Workload ranking |
| `generate-timeline` | Planner | Migration timeline |
| `compare-targets` | Planner | Target comparison |
| `map-vm-dependencies` | Dependency | VM dependency map |
| `identify-app-groups` | Dependency | App tier grouping |
| `detect-cross-host-deps` | Dependency | Cross-host dependencies |
| `export-dependency-graph` | Dependency | Export dependency data |
| `generate-discovery-report` | Report | Discovery summary |
| `generate-assessment-report` | Report | Readiness report |
| `generate-cost-report` | Report | Cost report |
| `generate-executive-summary` | Report | Executive summary |
| `format-spreadsheet` | Report | Excel formatting |

### HVE Core Skills

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
