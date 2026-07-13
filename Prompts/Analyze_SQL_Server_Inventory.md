# Prompt: Analyze SQL Server Inventory (Agent 3)

> **What this is**: The text to paste into a Copilot Studio **AI Builder / prompt** tool named
> **`Analyze SQL Server Inventory`**. The processing topic (`Topics/Process_SQL_Server_Inventory.yaml`)
> invokes it via `BeginDialog` to `…action.AnalyzeSQLServerInventory` and binds the output key
> `predictionOutput.text` (a String) to `Global.consolidatedSQLInstances`.
> Source of rules: `AZURE_MIGRATE_AGENTS_GUIDE.md` → "SQL Server Inventory Analysis".

## Definition of Done (Agent 3)
- **Input**: `rawSQLData` (CSV text of the `SQLServer` sheet) passed from the Read flow via `Topic.rawSQLData`.
- **Output**: JSON array of unique SQL Server instances → `Global.consolidatedSQLInstances` (via `predictionOutput.text`).
- **Test cases**:
  1. *Happy path* — mixed instances + updates/clients → updates/SSMS/drivers removed, instances grouped by version.
  2. *TooLarge* — Read flow returns `status = "TooLarge"`; topic shows trim/split guidance, prompt is **not** called.
  3. *Empty/missing sheet* — `rawSQLData` empty → prompt returns `[]`, topic shows "no results".
- **Logs**: Power Automate → Flow run history (*Read SQL Server Inventory Data*); Copilot Studio → Test your agent panel.

---

## Prompt input variable
Define one input on the prompt tool:
- **`rawSQLData`** (Text) — the raw CSV rows of the SQL Server sheet.

## Prompt body

```
You are consolidating SQL Server inventory data exported from Azure Migrate.

Analyze the raw CSV data below and return a clean, deduplicated list of unique SQL Server
instances grouped by version. Apply the rules exactly. Do not invent rows or values.

Raw SQL Server data (CSV):
{rawSQLData}

### Noise / Removal Rules — classify as NOISE and EXCLUDE
- SQL Server cumulative updates and update entries
- SQL Server service pack entries listed as separate rows
- Dependent client tools: SQL Server Management Studio (SSMS), SQL Server Native Client,
  SQL Server ODBC Driver, SQL Server OLE DB Driver, SQL Server Data Tools,
  Reporting Services / Integration Services when listed standalone,
  SQL Server command-line utilities
- SQL Server setup/installer entries
- SQL Server feature packs

### Consolidation Rules
- Consolidate by MachineName + Instance Name (case-insensitive).
- If Instance Name is empty/missing, default to "MSSQLSERVER".
- If Port is empty/missing, default to "1433".
- If the same instance appears with different editions, keep each as a separate entry.
- Preserve original Edition, ServicePack, and Version values.
- Group the unique entries by Version.

### Required Output Format
Return ONLY a JSON array (no prose before/after) of this shape:
[
  {
    "MachineName": "server name",
    "InstanceName": "instance name or MSSQLSERVER",
    "Edition": "SQL Server edition",
    "Version": "version string",
    "ServicePack": "service pack level",
    "Port": "port number or 1433",
    "MachineManagerFqdn": "FQDN"
  }
]
Sort by Version (descending, newest first), then by MachineName.
If no valid SQL Server rows exist, return [].

### External References Policy
Do NOT include links, URLs, or references to external resources unless explicitly asked.
```
