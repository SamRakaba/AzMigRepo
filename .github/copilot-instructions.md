# GitHub Copilot Repo Instructions — SamRakaba/AzMigRepo

Effective date: 2026-07-12 (v1.5)
Repo: SamRakaba/AzMigRepo
Primary focus: Implementation documentation for Azure Migrate CSV Processing agents built in Microsoft Copilot Studio
Parent project: [SamRakaba/AZMrepo](https://github.com/SamRakaba/AZMrepo)
Source of truth: `AZURE_MIGRATE_AGENTS_GUIDE.md` (v1.3, full step-by-step implementation)
Dev methodology: [HVE Core](https://github.com/microsoft/hve-core) — Research → Plan → Implement → Review
Copilot Studio UI version: **2026 redesign** (see §12 for UI mapping)

## 1) Role

You are GitHub Copilot working inside this repository. Your job is to help build, edit, and improve:

- **Azure Migrate CSV processing documentation** — agent implementation guides, inventory consolidation workflows, assessment report generation
- **Copilot Studio agent definitions** — 5 agents with Power Automate tool integrations (see §3 for architecture)
- **Power Automate flow documentation** — flow contracts, input/output schemas, Blob Storage I/O patterns
- **Documentation** that is accurate, step-by-step, and actionable — no fabricated UI labels, values, or steps

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

## 3) Domain context — Azure Migrate CSV Processing Agents

### 3.1 Architecture

This project documents **5 Copilot Studio agents** that process Azure Migrate CSV export files:

| # | Agent | Purpose |
|---|-------|---------|
| 1 | **File Upload Handler** | Accept CSV/XLSX uploads, GPT-4.1 sheet compliance verification, coordinate processing |
| 2 | **App Inventory Processor** | GPT-4.1 noise detection, deduplication, consolidation of ApplicationInventory sheet |
| 3 | **SQL Server Processor** | GPT-4.1 consolidation of SQL Server instances, version grouping |
| 4 | **Web App Processor** | GPT-4.1 consolidation of web applications by server type |
| 5 | **Report Generator** | Combine processed data, write Excel to Blob Storage, provide download link |

### 3.2 GPT-4.1-first design principle

The GPT-4.1 model (configured in Copilot Studio) handles **ALL** intelligence:
- Sheet compliance verification
- Data extraction and parsing
- Consolidation, deduplication, noise detection
- Structured output generation (JSON, CSV)

**Power Automate handles ONLY file I/O** — storing uploaded files to Azure Blob Storage and writing the final report. No Azure Functions, no custom code, no scripts.

### 3.3 In-memory data passing

Data flows between topics via **Global variables** (JSON strings). Persistent storage (Azure Blob) is used only for the final downloadable report.

Key global variables: `Global.sessionId`, `Global.uploadedFilePath`, `Global.consolidatedApplications`, `Global.consolidatedSQLInstances`, `Global.consolidatedWebApps`, `Global.applicationDedupCSV`, `Global.detectedSheets`, `Global.downloadUrl`, `Global.processingStatus`, `Global.uploadStatus`, `Global.errorMessage`

### 3.4 Classic orchestration

- Orchestration mode: **Classic** (not Generative)
- In 2026 UI: Settings → **Orchestration** section → Classic
- Generative answers and Boost are consolidated into Orchestration (selecting Classic disables both)
- Agent follows structured topic flows, not free-form responses

### 3.5 CSV schema (Azure Migrate exports)

| Sheet | Key Columns | Agent |
|-------|-------------|-------|
| **ApplicationInventory** | MachineName, Application, Version, Provider, MachineManagerFqdn | Agent 2 |
| **SQL Server** | MachineName, Instance Name, Edition, Service Pack, Version, Port, MachineManagerFqdn | Agent 3 |
| **WebApplications** | MachineName, WebServerType, WebAppName, VirtualDirectory, ApplicationPool, FrameworkVersion, MachineManagerFqdn | Agent 4 |
| **Database** (optional) | MachineName, Database Type, Version, MachineManagerFqdn | Not processed |

## 4) Documentation and naming conventions

- **Markdown**: Primary format for all documentation — clear headings, numbered steps, code blocks
- **JSON schemas**: For PA flow contracts, variable definitions, and trigger payloads
- **Topic YAML**: For Copilot Studio topics, author/maintain the topic as a `.yml` file under `Topics/` (see §14). Prefer generating or updating the YAML directly over click-by-click UI narration.
- **Topic naming**: `Verb + Object` (e.g., "Welcome and Upload Instructions", "Process Application Inventory")
- **Global variable naming**: `camelCase` (e.g., `consolidatedApplications`, `uploadedFilePath`)
- **PA flow naming**: `Verb + Object + Context` (e.g., "Read Application Inventory Data", "Generate Consolidated Report")
- **Placeholders**: Always label example values as **PLACEHOLDER — replace this** or **EXAMPLE only**

## 5) Copilot Studio & Power Automate constraints

### 5.1 Tools (not Actions)
Refer to Copilot Studio integrations as **Tools** (current UI terminology).

### 5.2 "Respond to the agent" — never null
Every Power Automate flow branch must end with "Respond to the agent" with non-null values.

### 5.3 File processing happens in Power Automate
The copilot does not parse CSV/Excel directly. File processing is delegated to Power Automate flows.

### 5.4 Storage must be explicit
Never assume Azure Blob vs SharePoint. Ask the user or document both paths.

### 5.5 GPT-4.1-first principle
The GPT-4.1 model handles all intelligence (validation, analysis, consolidation, noise detection, output generation). Power Automate handles only Blob Storage read/write. Do not suggest Azure Functions, custom scripts, or external compute — all processing logic lives in Copilot Studio topics.

### 5.6 Classic orchestration
This solution uses Classic orchestration mode with Generative answers OFF and Boost OFF. Do not suggest enabling generative features or free-form conversation patterns. All agent behavior is defined by structured topics.

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
- feat: Add assessment data processing guide
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

## 10) File relationships — source of truth

| File | Role | Version |
|------|------|---------|
| `AZURE_MIGRATE_AGENTS_GUIDE.md` | **Source of truth** — Full implementation guide (Agents 1-5, PA flows, testing, troubleshooting) | v1.3 |
| `COPILOT_STUDIO_INSTRUCTIONS.md` | System prompt to paste into Agent 1 in Copilot Studio | v1.1 |
| `COPILOT_STUDIO_GUIDE.md` | Generic Copilot Studio portal tutorial (reference) | — |
| `COPILOT_INSTRUCTIONS.md` | Parent repo coding conventions (reference only) | — |
| `AGENTS.md` | Development methodology (HVE Core) + agent quick-reference + lessons learned | v1.1 |
| `.github/copilot-instructions.md` | This file — repo-level Copilot behavior rules | v1.5 |

When referencing implementation details, cite `AZURE_MIGRATE_AGENTS_GUIDE.md` by section header (e.g., "Agent 1: File Upload Handler", "Step 5: Add Power Automate Flows as Tools"). Do not claim content exists in the guide unless it actually does.

## 11) Definition of Done (per agent component)

For each agent, topic, or PA flow documented, ensure:
1. **Expected input(s)** — what triggers it, what data it receives
2. **Expected output(s)** — what it produces, which variables it sets
3. **Minimum 3 test cases** — happy path, edge case, error case
4. **Where to check logs** — Flow run history (Power Automate) + Copilot test panel (Copilot Studio)

---
End of repo instructions.

## 12) Copilot Studio 2026 UI Mapping (Critical)

The `AZURE_MIGRATE_AGENTS_GUIDE.md` was updated to v1.3 for the 2026 Copilot Studio UI redesign. When building or reviewing, always use these mappings:

| Task | 2026 UI Location |
|------|-------------------|
| Set orchestration mode | Settings (gear icon) → **Orchestration** section |
| Paste system instructions | Agent main page → **Instructions** tab |
| Select model | Agent main page → **Select model** tab |
| Add PA flows as tools | Agent main page → **Tools** tab → **Add a tool** → Workflows |
| Create/edit topics | **Top navigation bar** → **Topics** |
| Create a new topic | Topics → **+ Add** → **Topic** → **From blank** |
| Map tool inputs | In topic node: `...` → **Custom** → select variable |
| Insert variable in message | Use `{x}` variable picker (still works in message nodes) |
| Test the agent | Bottom-right → **Test your agent** panel |
| View connected tools | Agent main page → **Tools** tab |
| Create new agent | Left nav → **Agents** → **+ New agent** |

### What no longer exists
- "Generative AI" settings tab (replaced by Orchestration section)
- "Agent details" tab for Instructions (replaced by Instructions tab on main page)
- "Call an action" menu in topics (replaced by "Add a tool")
- "Actions" page (replaced by "Tools" page/tab)
- Separate "Boost conversations" and "Generative answers" toggles (consolidated into Classic/Generative choice)

## 13) Lessons Learned — Pitfalls & Discoveries (from implementation sessions)

### 13.1 Copilot Studio UI Pitfalls

| # | Pitfall | What Happened | Resolution |
|---|---------|---------------|------------|
| 1 | **Model Responses not editable** | In Classic orchestration, the "Model Responses" field in Settings is greyed out / not editable | Expected behavior — Classic mode uses topic flows, not model responses |
| 2 | **Variable mapping in tool nodes** | The old `{x}` variable picker does NOT work for tool input mapping | Use `...` → **Custom** to select variables for tool inputs |
| 3 | **Output variables auto-generated** | When adding a tool to a topic, output variables get auto-generated names (e.g., `rawDataApp`) | Rename via the variable properties panel; or use the auto-generated names |
| 4 | **File type in Ask a Question** | The "File" option in Identify dropdown may not be available in all environments | If missing, use Text and handle file attachment via `System.Activity.Attachments` |
| 5 | **Power Fx formula for File inputs fails** | `{ contentBytes: Topic.uploadedFiles.Content }` → error: "The '.' operator cannot be used on Blob values" | Select the File variable directly from the picker — do NOT use Power Fx formulas for File-type inputs |
| 6 | **Table type error** | `[{ contentBytes: ... }]` → error: "incorrect type table" | Square brackets create Table type, not File. Use direct variable selection instead |
| 7 | **AI Builder prompt output binding** | Binding the whole `predictionOutput` record to a String variable → type mismatch | The analysis/report prompts (Agents 2–5) invoke as `InvokeAIBuilderModelAction`; in the topic bind the output key **`predictionOutput.text`** (the String) to your Global — binding `predictionOutput` (the record) fails. Read it as `Topic.predictionOutput.text`. |

### 13.2 Power Automate Pitfalls

| # | Pitfall | What Happened | Resolution |
|---|---------|---------------|------------|
| 1 | **Empty 'text' parameter error** | `FlowActionBadRequest: required parameter 'text' has a blank or empty value` when calling "Read Application Inventory Data" | Global variables `uploadedFilePath` and `sessionId` were empty — file upload topic wasn't wired yet |
| 2 | **Azure Blob Storage asks for "storage name"** | Get blob content (V2) asks for storage account name, not container or path | Select or create a connection with your storage account name first, then specify container and blob path separately |
| 3 | **Managed Identity not available** | Power Automate Blob connector didn't offer Managed Identity in some environments | Use **Microsoft Entra ID (Service Principal)** authentication instead; assign "Storage Blob Data Contributor" RBAC role |
| 4 | **Consent popup blocks testing** | First-time tool execution in Copilot Studio test panel shows consent dialog for storage connection | Grant consent once — subsequent runs proceed without prompt |
| 5 | **Action naming breaks expressions** | `outputs('Generate_Session_ID')` fails with "invalid reference" if the Compose action is still named "Compose" | Always rename actions exactly as instructed — internal names derive from display names with spaces → underscores |
| 6 | **coalesce() required for outputs** | Any null output in "Respond to the agent" causes `FlowActionException` | Wrap ALL dynamic output values in `coalesce(expression, '')` |
| 7 | **Both condition branches need response** | Flow with condition missing "Respond to the agent" in one branch → "output parameter missing from response data" | Every branch must have its own "Respond to the agent" action with ALL outputs populated |

### 13.3 Architecture & Design Lessons

| # | Lesson | Detail |
|---|--------|--------|
| 1 | **Wire file upload BEFORE testing processing** | Processing topics depend on `Global.uploadedFilePath` and `Global.sessionId` being populated. Test the Welcome & Upload topic first. |
| 2 | **Global variables must be populated before redirect** | Topic redirects happen immediately — the target topic sees globals as they were at redirect time. Set globals BEFORE redirecting. |
| 3 | **GPT-4.1 model cannot parse binary Excel** | The model can analyze text/CSV data passed as strings, but it cannot parse `.xlsx` binary format. PA flows must extract data first. |
| 4 | **Keep PA flows minimal** | PA flows should only do file I/O (read blob, write blob). All data analysis, consolidation, and formatting stays in Copilot Studio topics. |
| 5 | **Add condition guards in processing topics** | Each processing topic should check if `Global.uploadedFilePath` is empty before calling its PA tool. Show a helpful message redirecting to upload. |
| 6 | **Instructions tab replaces old system prompt location** | Don't waste time looking for "Agent details → Instructions" in Settings. It's now a main-page tab. |
| 7 | **Tools tab shows PA flows with agent triggers** | Only flows with "When an agent calls the flow" trigger appear in the Workflows list when adding tools. |
| 8 | **Extract CSV blob to text with `base64ToString`** | `Get blob content (V2)` returns base64 in `body(...)?['$content']`. Decode to readable rows with a Compose: `base64ToString(body('Get_blob_content_V2')?['$content'])`. Never feed the raw `.xlsx` binary to the model — process the sheet as `.csv` (Path A). |
| 9 | **Guard CSV size against the 128K-token limit** | GPT-4.1 context is 128K tokens (~350K chars input budget). In the Read flow add a `CsvLength` Compose = `length(outputs('Extract_csv_text'))`, then return `status = if(greater(outputs('CsvLength'), 350000), 'TooLarge', 'DataReady')` and blank `rawApplicationData` when too large. The processing topic checks `Topic.status = "TooLarge"` and shows trim/split guidance instead of calling the model. |
| 10 | **Large files (60K+ rows) need chunking, not single-pass** | Single-pass ceilings: ~300–400 full-column rows, ~2,000–2,500 trimmed-column rows. For 60K+ rows, trim columns at export first, then batch the CSV (~2,000–2,500 rows/batch), analyze each batch, and run a final cross-batch dedup/merge. See guide Step 1.8. |
| 11 | **Reuse the proven CSV+prompt pattern for every processor (Agents 2–5)** | Agents 3 (SQL), 4 (Web App) and 5 (Report) use the **same** shape as Agent 2: agent-called Read flow → `Extract csv text` (`base64ToString`) → `CsvLength` size guard → `Respond to the agent` (`raw<X>Data`/`rowCount`/`status`) → topic guard → AI Builder prompt (`predictionOutput.text`) → store `Global.consolidated<X>` → redirect to next topic. Do NOT reintroduce Excel-table `List rows`/Parse JSON reads or Excel-template report writes — both were proven to fail for Agent 2. |
| 12 | **Agent 5 report is CSV, not Excel; chain redirects App→SQL→Web→Report** | Report Generator formats the three in-memory globals into one **sectioned CSV** via the *Format Consolidated Report* prompt, then a single agent-called flow writes it to `reports/<sessionId>/…csv` and returns a 24h SAS `downloadUrl` (`coalesce`-wrapped). No HTTP trigger, no email, no Excel template, no Azure Functions. Topics redirect in sequence; the last (Report) presents the link. |
| 13 | **Replace PLACEHOLDER flowId GUIDs after creating flows** | Authored topic YAMLs (`Topics/Process_SQL_Server_Inventory.yaml`, `Process_Web_App_Inventory.yaml`, `Generate_Report.yaml`) use `PLACEHOLDER-…-flowId` values so they parse. After building each PA flow in Studio, replace the placeholder with the real flow GUID before pasting the topic back and testing. |

### 13.4 Development Process Lessons

| # | Lesson | Detail |
|---|--------|--------|
| 1 | **Pull source files BEFORE research** | Initial research produced incorrect findings because the full implementation guides weren't in the repo yet. Always pull all source files first. |
| 2 | **HVE RPI cycle catches fabrication** | The Research phase caught fabricated PowerShell cmdlets and fake script references in the original copilot-instructions.md. Always research before planning. |
| 3 | **Guide was written for wrong UI version** | The AZURE_MIGRATE_AGENTS_GUIDE.md was written for the 2024-2025 UI. Building interactively revealed the 2026 redesign. Guides must be validated against the actual current UI. |
| 4 | **Interactive walkthrough discovers real UI** | Step-by-step building with the user asking "what do you see?" is the only reliable way to document the current Copilot Studio UI. |
| 5 | **Track UI changes immediately** | Document UI discoveries in `.copilot-tracking/changes/` as they happen. This prevents losing critical mapping knowledge. |

## 14) Topic YAML Authoring (Agent-Native — preferred workflow)

Copilot Studio topics can be edited as YAML via the topic **code editor** (open a topic → **</> code** view). Copilot **must prefer generating or updating topic YAML directly** over narrating click-by-click UI steps, because it is faster, reviewable in git, and far less error-prone.

### 14.1 Workflow
1. Ask the user to export the topic YAML (topic → code editor → copy) into `Topics/<Topic-Name>.yml` in the repo — or update the existing file there.
2. Make surgical edits to the YAML.
3. **Validate it parses** before handing back (e.g., `python -c "import yaml; yaml.safe_load(open('Topics/<file>.yml',encoding='utf-8'))"`).
4. Have the user paste the corrected YAML back into the topic code editor and **Save**.
5. Then test in the **Test your agent** panel.

### 14.2 Topic YAML essentials (observed schema)
- Root: `kind: AdaptiveDialog` → `beginDialog: kind: OnRecognizedIntent` with `intent.triggerQueries` (trigger phrases) and `actions:`.
- Common action `kind`s: `Question` (with `variable:` and `entity: FilePrebuiltEntity` for file uploads), `SendActivity`, `InvokeFlowAction` (Power Automate tool call), `ConditionGroup` (with `conditions[].condition` Power Fx + `elseActions`), `SetVariable`, `BeginDialog` (topic redirect).
- `InvokeFlowAction` has `input.binding` (map tool inputs) and `output.binding` (capture tool outputs to `Topic.*`), plus `flowId`.
- Global variables are referenced as `Global.<name>`; topic-scoped as `Topic.<name>`.

### 14.3 Hard rules for topic YAML (from lessons — see §13)
- **File input mapping:** a PA flow File input surfaces as a Record; map it with `={contentBytes: Topic.uploadedFiles, name: "upload.xlsx"}` where `uploadedFiles` is a plain File var (Ask-a-question File response, **"Include file metadata" OFF**). Never use `First(System.Activity.Attachments)` (empty after the question turn) and never `.Content` on a Blob.
- **Guard order:** call a flow tool **inside** the "files uploaded" (`=!IsBlank(Topic.uploadedFiles)`) branch, never before the guard.
- **Every branch responds:** each `ConditionGroup` branch that ends the flow must send an appropriate message / redirect; every PA `Respond to the agent` must return all outputs non-null.
- **Redirect ordering:** set globals **before** `BeginDialog` redirects; don't emit "complete" messages before the work runs.
- Always validate the YAML parses before returning it.

## 15) Azure MCP Server & Copilot Skills (analysis/implementation tooling — NOT part of the solution)

Copilot working in this repo has access to the **Azure MCP server** and a set of **Azure-focused Copilot skills**. Use them to *analyze, verify, and implement* the documentation in this repo — they are development/authoring tools, **not** components of the shipped solution.

### 15.1 Hard boundary — tooling vs. solution
- The **solution architecture is unchanged**: GPT-4.1-first intelligence in Copilot Studio + Power Automate for file I/O only. See §3 and §5.5–5.6.
- Azure MCP tools and Azure skills are **for Copilot's own use** when researching Azure behavior, validating example values against a real environment, checking RBAC/storage config, or grounding documentation in fact.
- **Never** introduce Azure MCP calls, Azure CLI scripts, Azure Functions, or custom compute into the documented agent design, topic YAML, or Power Automate flows. Doing so violates the GPT-4.1-first principle (§5.5).
- **Never** paste real subscription IDs, resource names, connection strings, or secrets discovered via these tools into the repo. All example values stay labeled **PLACEHOLDER — replace this** / **EXAMPLE only** (§2.2).

### 15.2 When to use Azure MCP / Azure skills
Use them during **Research** and **Review** (HVE RPI, §9) — e.g. to:
- Verify Azure Migrate export schemas, product behavior, and current API/UI facts (avoids fabrication, §2.1).
- Confirm Blob Storage, container, RBAC ("Storage Blob Data Contributor"), and Entra ID (Service Principal) setup that the PA flows depend on (§6, §13.2).
- Ground best-practice guidance (Well-Architected, security) before writing it into a guide.

### 15.3 Relevant Azure MCP tool groups
The Azure MCP server exposes tools prefixed `azure-`. The most relevant for this repo:

| Tool group | Use it to (analysis/implementation only) |
|------------|------------------------------------------|
| `azure-documentation` | **Primary grounding tool** — search official Microsoft Learn docs (incl. Azure Migrate export schemas) to cite facts instead of fabricating (§2.1) |
| `azure-storage` | Inspect Blob containers/paths and storage-account config the PA flows read/write |
| `azure-role` | Confirm RBAC roles (e.g. Storage Blob Data Contributor) for the flow connections |
| `azure-keyvault` | Check secret/SAS handling patterns (never surface secret values into docs) |
| `azure-group_list`, `azure-group_resource_list`, `azure-subscription_list` | Discover resources when validating an environment |
| `azure-foundry` | Verify GPT-4.1 model availability/config facts |
| `azure-monitor`, `azure-applicationinsights` | Ground flow-diagnostics / log-location guidance (§11) |
| `azure-get_azure_bestpractices`, `azure-wellarchitectedframework` | Ground security/reliability recommendations before documenting them |

> Azure MCP tools are **deferred** — call the tool-search tool first to load a tool's schema before invoking it. Never guess a deferred tool's signature.

### 15.4 Relevant Azure Copilot skills
The following Copilot skills help analyze/implement this repo (invoke by name; they do not become part of the solution):

| Skill | Use for |
|-------|---------|
| `azure-storage` | Blob/container concepts, access tiers, storage I/O patterns the flows use |
| `azure-rbac` | Least-privilege role selection + assignment code for flow identities |
| `azure-resource-lookup` | Inventory/find storage accounts and related resources |
| `azure-diagnostics` | Troubleshoot deployed Azure resources referenced by the flows |
| `azure-compliance` | Security/compliance and Key Vault expiration checks on the Azure side |

### 15.5 Rules
1. Read-only / analysis first — prefer `list`/`get`/best-practice tools; never mutate a real environment without explicit user confirmation.
2. Keep findings out of the shipped artifacts except as **PLACEHOLDER**/**EXAMPLE** values.
3. Record verified facts (schemas, RBAC, UI) in `.copilot-tracking/` (gitignored, §9), not in committed docs unless they belong in the guide.
4. If a tool would change the documented **solution** design, stop — that's out of scope (§15.1).
