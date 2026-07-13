# AGENTS.md — AzMigRepo Agent Reference & Development Guide (v1.1)

> **Full implementation guide**: [`AZURE_MIGRATE_AGENTS_GUIDE.md`](AZURE_MIGRATE_AGENTS_GUIDE.md) (v1.3) — complete step-by-step instructions for building all 5 agents, Power Automate flows, testing, and troubleshooting. Updated for the **2026 Copilot Studio UI**.

---

## Agent Quick Reference

This project documents 5 Copilot Studio agents using a **GPT-4.1-first** architecture where the model handles all intelligence and Power Automate handles only file I/O.

| # | Agent | Purpose | Guide Section | Build Status |
|---|-------|---------|---------------|--------------|
| 1 | **File Upload Handler** | Accept CSV/XLSX uploads, GPT-4.1 sheet compliance verification, coordinate processing | [Agent 1](AZURE_MIGRATE_AGENTS_GUIDE.md#agent-1-file-upload-handler) | ✅ Topic built (`Topics/Welcome-and_upload.yml`) |
| 2 | **App Inventory Processor** | GPT-4.1 noise detection, deduplication, application consolidation | [Agent 2](AZURE_MIGRATE_AGENTS_GUIDE.md#agent-2-application-inventory-processor) | ✅ Topic + Read flow + prompt (proven CSV pattern) |
| 3 | **SQL Server Processor** | GPT-4.1 SQL instance consolidation, version grouping | [Agent 3](AZURE_MIGRATE_AGENTS_GUIDE.md#agent-3-sql-server-inventory-processor) | 🔧 Topic YAML + prompt authored; PA flow + AI Builder prompt pending in Studio |
| 4 | **Web App Processor** | GPT-4.1 web application consolidation by server type | [Agent 4](AZURE_MIGRATE_AGENTS_GUIDE.md#agent-4-web-app-inventory-processor) | 🔧 Topic YAML + prompt authored; PA flow + AI Builder prompt pending in Studio |
| 5 | **Report Generator** | Combine processed data, write **CSV report** to Blob, provide download link | [Agent 5](AZURE_MIGRATE_AGENTS_GUIDE.md#agent-5-report-generator) | 🔧 Topic YAML + prompt authored; PA flow + AI Builder prompt pending in Studio |

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

| Flow Name | Trigger Type | Purpose | Build Status |
|-----------|-------------|---------|--------------|
| **Handle File Upload (Blob Storage)** | Agent Tool | Save file to Blob, return path | ✅ Connected |
| **Validate Excel Sheet** | Agent Tool | Validate uploaded file, return metadata | ✅ Connected |
| **Read Application Inventory Data** | Agent Tool | Read raw app data for GPT-4.1 analysis | ✅ Created |
| **Read SQL Server Inventory Data** | Agent Tool | Read raw SQL CSV for GPT-4.1 analysis (CSV pattern) | 🔧 Topic wired; flow pending |
| **Read Web App Inventory Data** | Agent Tool | Read raw web app CSV for GPT-4.1 analysis (CSV pattern) | 🔧 Topic wired; flow pending |
| **Generate Consolidated Report** | Agent Tool | Write CSV report to Blob, return SAS download URL | 🔧 Topic wired; flow pending |
| **Azure Migrate Processing Orchestrator** | HTTP (optional) | Multi-file batch processing | ⬜ Not started |
| **Get Processing Status** | Utility | Check processing status | ⬜ Not started |

### Implementation artifacts & remaining Studio steps (2026-07-12)

Topic YAML and prompt text for the full chain are authored in the repo (`/task-implement`):

- **Topics** (paste into each topic's code editor → Save): `Topics/Welcome-and_upload.yml`, `Process_Application_Inventory.yaml`, `Process_SQL_Server_Inventory.yaml`, `Process_Web_App_Inventory.yaml`, `Generate_Report.yaml`.
- **Prompts** (create as AI Builder/prompt tools, paste body): `Prompts/Analyze_SQL_Server_Inventory.md`, `Analyze_Web_App_Inventory.md`, `Format_Consolidated_Report.md`.
- **Orchestration chain**: Welcome → Process Application Inventory → Process SQL Server Inventory → Process Web App Inventory → Generate Report (each step guards on `Global.uploadedFilePath`, size-checks, then redirects on success).

**Canonical Read-flow pattern (supersedes the guide's Excel-table steps for Agents 3–5):**
`Get blob content (V2)` → Compose **`Extract csv text`** = `base64ToString(body('Get_blob_content_V2')?['$content'])` → Compose **`CsvLength`** = `length(outputs('Extract_csv_text'))` → `Respond to the agent` with `raw<X>Data = if(greater(outputs('CsvLength'),350000), '', outputs('Extract_csv_text'))`, `rowCount` (coalesced), `status = if(greater(outputs('CsvLength'),350000),'TooLarge','DataReady')`. Never feed raw `.xlsx` binary to the model.

**Remaining to go live** (in Copilot Studio / Power Automate): create the 3 AI Builder prompts, build the 3 PA flows (2 Read + 1 Generate Report, CSV pattern), then **replace the `PLACEHOLDER-…-flowId` values** in the topic YAML with the real flow GUIDs, paste topics back, and test the chain. See `.copilot-tracking/plans/2026-07-12-complete-agent-implementation-plan.md`.

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
- Use **security-reviewer** for OWASP assessment on documentation handling credentials or sensitive data

### When writing documentation
- Use the **documentation** agent for auditing and authoring
- Use **prd-builder** or **brd-builder** for formal requirements
- Follow the format in `.github/copilot-instructions.md` section 7

### When committing
- Use `/git-commit` for Conventional Commit messages
- Follow format: `<type>: <subject>` (feat, fix, docs, refactor, test, chore)

## Security Practices

- Use **security-reviewer** (`/security-review`) before merging changes that handle credentials, storage connections, or assessment data
- Use **security-planner** for STRIDE threat modeling on new agent designs
- Assessment data may contain sensitive infrastructure details — handle accordingly

## Tracking

All HVE Core agent outputs go to `.copilot-tracking/` (gitignored — never commit these):
- `research/` — Research documents
- `plans/` and `details/` — Implementation plans
- `changes/` — Change logs and UI discovery notes
- `reviews/` — Review findings
- `security/` — OWASP reports

---

## Copilot Studio 2026 UI Quick Reference

> **Critical**: The guide (v1.3) and all instructions use the 2026 Copilot Studio UI. See `.github/copilot-instructions.md` §12 for the full mapping.

| Action | Where |
|--------|-------|
| Instructions / System prompt | Main page → **Instructions** tab |
| Add PA flow as tool | Main page → **Tools** tab → Add a tool → Workflows |
| Topics | **Top nav** → Topics |
| Create topic | Topics → **+ Add** → Topic → From blank |
| Orchestration mode | Settings → **Orchestration** section |
| Tool input mapping | Topic node → `...` → **Custom** |
| Test | Bottom-right → **Test your agent** |

---

## Lessons Learned — Pitfalls to Avoid

These are discovered during hands-on building. Apply them when creating or debugging agents.

### Copilot Studio Pitfalls

1. **Variable mapping uses `...` → Custom, NOT `{x}`** — The `{x}` picker works in message nodes, but for tool input mapping use the `...` menu → Custom.
2. **Power Fx formulas fail for File-type inputs** — `{ contentBytes: Topic.uploadedFiles.Content }` causes "The '.' operator cannot be used on Blob values". Select the File variable directly from the picker.
3. **Square bracket syntax creates Table, not File** — `[{ contentBytes: ... }]` causes "incorrect type table" error. Never wrap file inputs in `[]`.
4. **Model Responses is read-only in Classic mode** — The Settings → Model Responses field is greyed out when Classic orchestration is selected. This is expected.
5. **Output variables get auto-generated names** — When you add a tool node, outputs are auto-captured with names like `rawDataApp`. Rename them via the variable properties panel if needed.
6. **Wire file upload FIRST, test processing SECOND** — All processing topics depend on `Global.uploadedFilePath` and `Global.sessionId`. If those are empty, every PA tool call fails with "required parameter has a blank or empty value".
7. **Add condition guards at the top of processing topics** — Check if `Global.uploadedFilePath` is empty before calling any PA tool. If empty, show a message redirecting the user to upload first.
8. **Bind AI Builder prompt output as `predictionOutput.text`** — Agents 2–5 invoke their analysis/report prompt via `InvokeAIBuilderModelAction`. Bind the output key **`predictionOutput.text`** (String) to your Global; binding the whole `predictionOutput` record causes a type mismatch. Read it as `Topic.predictionOutput.text`.

### Power Automate Pitfalls

1. **Rename Compose actions BEFORE referencing them** — `outputs('Generate_Session_ID')` fails if the action is still named "Compose" (internal name = `Compose`).
2. **Every "Respond to the agent" output must be non-null** — Wrap dynamic values in `coalesce(expression, '')`.
3. **Both condition branches need "Respond to the agent"** — Missing response in any branch → `FlowActionException: output parameter missing from response data`.
4. **Azure Blob "Get content" asks for storage account name** — This is the connection setup, not the blob path. Select or create a connection first, then specify container and blob path.
5. **Managed Identity may not be available** — Some Power Automate environments don't offer Managed Identity for Blob Storage. Use **Entra ID (Service Principal)** with "Storage Blob Data Contributor" RBAC role.
6. **First test triggers consent popup** — First time a tool runs in the Copilot Studio test panel, a consent dialog appears for storage connections. Grant it once.
7. **Flow triggers must be "When an agent calls the flow"** — Only flows with this trigger appear in the Workflows list when adding tools in Copilot Studio.

### Architecture Lessons

1. **GPT-4.1 cannot parse binary .xlsx** — The model can analyze text/CSV data passed as strings, but PA flows must extract sheet data first. `Get blob content (V2)` on an `.xlsx` returns the whole workbook binary as base64 (~1M tokens) — unparseable. Process the sheet as `.csv` and decode with `base64ToString(body('Get_blob_content_V2')?['$content'])`.
2. **Set globals BEFORE topic redirects** — Redirects are immediate — the target topic sees globals as they were at redirect time.
3. **In-memory approach has size limits** — Global variables (JSON strings) work for typical exports (100-500 rows). For 1000+ rows, test to confirm or use blob storage.
4. **Keep PA flows minimal** — PA should only do blob read/write. All analysis, consolidation, and formatting stays in Copilot Studio topics.
5. **Guard CSV size vs. the 128K-token model limit** — Budget ≤ ~350K input chars. Read flow adds a `CsvLength` Compose and returns `status = "TooLarge"` (blank data) above the threshold; the processing topic branches on `Topic.status = "TooLarge"` to show trim/split guidance instead of failing the model with `InvalidPredictionInput`.
6. **Large files (60K+ rows) require chunking** — Single-pass ceiling is ~300–400 full-column or ~2,000–2,500 trimmed-column rows. For 60K+ rows: trim columns at export, batch (~2,000–2,500 rows/batch), analyze per batch, then dedup/merge across batches. See `AZURE_MIGRATE_AGENTS_GUIDE.md` Step 1.8.
7. **Reuse the proven CSV+prompt pattern for Agents 2–5** — SQL (3), Web App (4) and Report (5) mirror Agent 2 exactly: agent-called Read flow → `Extract csv text` → `CsvLength` guard → `Respond to the agent` (`raw<X>Data`/`rowCount`/`status`) → topic guard → AI Builder prompt (`predictionOutput.text`) → store `Global.consolidated<X>` → redirect to next topic. Never reintroduce Excel `List rows`/Parse JSON reads or Excel-template report writes — both failed for Agent 2.
8. **Agent 5 emits a sectioned CSV, not Excel** — The *Format Consolidated Report* prompt formats the three globals into one CSV (`## Applications` / `## SQL Server Instances` / `## Web Applications`); a single agent-called flow writes `reports/<sessionId>/…csv` and returns a 24h SAS `downloadUrl` (`coalesce`-wrapped). No HTTP trigger, no email, no Excel template, no Azure Functions. Chain redirects App→SQL→Web→Report.
9. **Replace PLACEHOLDER flowId GUIDs** — `Topics/Process_SQL_Server_Inventory.yaml`, `Process_Web_App_Inventory.yaml`, and `Generate_Report.yaml` use `PLACEHOLDER-…-flowId` so they parse. After building each PA flow in Studio, swap in the real flow GUID before pasting the topic back and testing.

### Development Process Lessons

1. **Pull ALL source files before researching** — Initial research produced incorrect findings because full implementation guides weren't in the repo. Always start with complete sources.
2. **HVE RPI cycle catches fabrication** — The Research phase caught fabricated PowerShell cmdlets and fake references. Never skip research.
3. **Validate guides against actual UI interactively** — The guide was written for old UI. Step-by-step building with "what do you see?" questions is the only reliable way to document the current UI.
4. **Document UI discoveries immediately** — Use `.copilot-tracking/changes/` to capture UI mapping discoveries as they happen.

---

## Skills Reference

### HVE Core Skills (Development Methodology)

| Skill | Command | Purpose |
|-------|---------|---------|
| **Research** | `/task-research <topic>` | Investigate before implementing — produces evidence-based findings |
| **Plan** | `/task-plan` | Create actionable implementation plan from research |
| **Implement** | `/task-implement` | Execute plan with change tracking |
| **Review** | `/task-review` | Validate implementation against research and plan |
| **Security Review** | `/security-review` | OWASP assessment on credential/data handling |
| **Git Commit** | `/git-commit` | Create Conventional Commit messages |
| **RPI Agent** | `rpi-agent` | Full Research → Plan → Implement → Review cycle (autonomous) |
| **Code Review** | `code-review` agent | Multi-perspective review (functional, security, standards) |
| **Documentation** | `documentation` agent | Doc audit, drift detection, authoring |

### Copilot Studio Build Skills (Agent-Specific)

These skills are derived from lessons learned during implementation and should be applied when building agents:

| Skill | When To Use | Key Actions |
|-------|-------------|-------------|
| **Topic YAML Authoring** | Any time a topic is created or modified (preferred over UI narration) | 1) Get/keep the topic YAML in `Topics/<Name>.yml`, 2) Edit YAML surgically, 3) Validate it parses (`yaml.safe_load`), 4) User pastes back into topic **code editor** → Save, 5) Test. See `.github/copilot-instructions.md` §14. |
| **Topic Wiring** | Creating a new topic flow | 1) Add trigger phrases, 2) Add welcome message, 3) Add tool call with `...` → Custom mapping, 4) Set globals, 5) Add condition guards, 6) Save |
| **Tool Registration** | Adding PA flow as tool | 1) Ensure flow has "When an agent calls the flow" trigger, 2) Go to Tools tab → Add a tool → Workflows, 3) Select flow, 4) Verify inputs/outputs |
| **Condition Guard** | Protecting processing topics | 1) Add condition at top: `Global.uploadedFilePath` is equal to `""`, 2) TRUE branch → redirect message, 3) FALSE branch → proceed with processing |
| **PA Flow Creation** | Building data extraction flows | 1) "When an agent calls the flow" trigger, 2) Add inputs (filePath, sessionId), 3) Get blob content, 4) "Respond to the agent" with ALL outputs + coalesce() |
| **Variable Debugging** | When tool calls fail with empty params | 1) Check if globals are populated (test Welcome & Upload first), 2) Check variable scope is Global not Topic, 3) Check connection consent granted |

### Azure MCP & Azure Copilot Skills (analysis/implementation only)

> **Tooling, not solution.** These are for Copilot's own research, verification, and implementation work. They are **not** part of the shipped agent design — the solution stays GPT-4.1-first + Power Automate file I/O only (no Azure Functions/custom compute). See `.github/copilot-instructions.md` §15 for the full boundary and rules.

Use during **Research** and **Review** to ground documentation in fact (avoid fabrication), and to verify the Azure setup the Power Automate flows depend on (Blob Storage, containers, RBAC, Entra ID service principal). Never write real subscription IDs, resource names, or secrets into the repo — keep example values labeled **PLACEHOLDER**/**EXAMPLE**.

| Capability | Azure MCP tool group / Copilot skill | Use for |
|-----------|--------------------------------------|---------|
| Grounding facts | `azure-documentation` | **Primary** — search Microsoft Learn (incl. Azure Migrate export schemas) to cite instead of fabricating |
| Blob Storage | `azure-storage` (MCP + skill) | Inspect containers/paths & storage-account config the flows use |
| RBAC | `azure-role` (MCP), `azure-rbac` (skill) | Confirm Storage Blob Data Contributor & assignment code |
| Secrets/SAS | `azure-keyvault` | Check secret handling (never surface values into docs) |
| Resource discovery | `azure-group_list`, `azure-subscription_list` (MCP), `azure-resource-lookup` (skill) | Find storage accounts & related resources |
| Model facts | `azure-foundry` | Verify GPT-4.1 availability/config |
| Diagnostics | `azure-monitor`, `azure-applicationinsights` (MCP), `azure-diagnostics` (skill) | Ground log-location / troubleshooting guidance |
| Best practices | `azure-get_azure_bestpractices`, `azure-wellarchitectedframework`, `azure-compliance` | Ground security/reliability guidance & citations |

Azure MCP tools are **deferred** — run the tool-search tool to load a tool's schema before calling it; never guess its signature.

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

**Last Updated**: July 12, 2026
