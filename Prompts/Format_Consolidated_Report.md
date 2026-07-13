# Prompt: Format Consolidated Report (Agent 5)

> **What this is**: The text to paste into a Copilot Studio **AI Builder / prompt** tool named
> **`Format Consolidated Report`**. The Report topic (`Topics/Generate_Report.yaml`) invokes it via
> `BeginDialog` to `…action.FormatConsolidatedReport` and binds `predictionOutput.text` (a String)
> to `Topic.reportBody`, which is then passed to the *Generate Consolidated Report* flow to write to Blob.
>
> **Report format decision**: plain **CSV** (sectioned), model-generated — no Excel template, no Azure
> Functions, consistent with the GPT-4.1-first / PA-file-I/O-only design
> (`AZURE_MIGRATE_AGENTS_GUIDE.md` Agent 5, Step 6).

## Definition of Done (Agent 5)
- **Input**: `applicationsJson`, `sqlJson`, `webAppsJson` (the three consolidated globals).
- **Output**: a single CSV report body → `Topic.reportBody` → written to `reports/<sessionId>/…csv` by the flow;
  `Global.downloadUrl` set from the flow's SAS URL.
- **Test cases**:
  1. *Happy path* — all three datasets present → 3 labeled CSV sections produced.
  2. *Partial* — one dataset empty (`[]`) → its section renders headers + "(none)" row; others populated.
  3. *All empty* — all `[]` → report with headers only and a note; flow still returns a valid download link.
- **Logs**: Power Automate → Flow run history (*Generate Consolidated Report*); Copilot Studio → Test panel.

---

## Prompt input variables
- **`applicationsJson`** (Text) — `Global.consolidatedApplications`
- **`sqlJson`** (Text) — `Global.consolidatedSQLInstances`
- **`webAppsJson`** (Text) — `Global.consolidatedWebApps`

## Prompt body

```
You are formatting a consolidated Azure Migrate report as plain CSV text.

You are given three JSON arrays. Convert them into ONE CSV document with three clearly
labeled sections, in this exact order. Do not add commentary before or after the CSV.
Do not invent data. If an array is empty, still output its section header row followed by
a single line: (none).

Applications JSON:
{applicationsJson}

SQL Server instances JSON:
{sqlJson}

Web applications JSON:
{webAppsJson}

### Output (CSV) — produce exactly this structure

## Applications
MachineName,Application,Version,Provider,MachineManagerFqdn
<one row per unique application>

## SQL Server Instances
MachineName,InstanceName,Edition,Version,ServicePack,Port,MachineManagerFqdn
<one row per unique SQL instance>

## Web Applications
WebAppName,WebServerType,VirtualDirectory,ApplicationPool,FrameworkVersion,Machines,MachineCount,MachineManagerFqdn
<one row per unique web app; join the Machines array with ';' inside the Machines cell>

### Rules
- Quote any field that contains a comma with double quotes.
- Keep the section header lines (## Applications, etc.) exactly as shown.
- Output plain text only — no code fences, no JSON, no links.
```
