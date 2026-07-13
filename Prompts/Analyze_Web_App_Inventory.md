# Prompt: Analyze Web App Inventory (Agent 4)

> **What this is**: The text to paste into a Copilot Studio **AI Builder / prompt** tool named
> **`Analyze Web App Inventory`**. The processing topic (`Topics/Process_Web_App_Inventory.yaml`)
> invokes it via `BeginDialog` to `…action.AnalyzeWebAppInventory` and binds the output key
> `predictionOutput.text` (a String) to `Global.consolidatedWebApps`.
> Source of rules: `AZURE_MIGRATE_AGENTS_GUIDE.md` → "Web App Inventory Analysis".

## Definition of Done (Agent 4)
- **Input**: `rawWebAppData` (CSV text of the `WebApplications` sheet) via `Topic.rawWebAppData`.
- **Output**: JSON array of unique web apps → `Global.consolidatedWebApps` (via `predictionOutput.text`).
- **Test cases**:
  1. *Happy path* — real apps + default sites → default/system apps removed, same app on N machines merged.
  2. *TooLarge* — Read flow returns `status = "TooLarge"`; topic shows guidance, prompt not called.
  3. *Empty/missing sheet* — `rawWebAppData` empty → prompt returns `[]`, topic shows "no results".
- **Logs**: Power Automate → Flow run history (*Read Web App Inventory Data*); Copilot Studio → Test panel.

---

## Prompt input variable
- **`rawWebAppData`** (Text) — the raw CSV rows of the WebApplications sheet.

## Prompt body

```
You are consolidating web application inventory data exported from Azure Migrate.

Analyze the raw CSV data below and return a clean list of unique web applications.
Apply the rules exactly. Do not invent rows or values.

Raw web application data (CSV):
{rawWebAppData}

### Noise / Removal Rules — classify as NOISE and EXCLUDE
- Default web server pages / placeholder sites (e.g., "Default Web Site", "IIS Start Page")
- Web server administration consoles and management interfaces
- Health check endpoints and monitoring agents
- Auto-generated or system-created application pools
- Web server modules and handlers (not actual applications)
- Test/sample applications (e.g., "iisstart", "aspnet_client")

### Consolidation Rules
- Identify unique web apps by WebAppName (case-insensitive, trimmed).
- Same web app on different WebServerType (IIS vs Apache) = SEPARATE entries.
- Same web app on multiple machines = ONE entry listing all machine names.
- Preserve original WebServerType, FrameworkVersion, VirtualDirectory, ApplicationPool.
- Group results by WebServerType.

### Required Output Format
Return ONLY a JSON array (no prose before/after) of this shape:
[
  {
    "WebAppName": "application name",
    "WebServerType": "IIS / Apache / Tomcat / Nginx / etc.",
    "VirtualDirectory": "virtual path",
    "ApplicationPool": "app pool name",
    "FrameworkVersion": "framework/runtime version",
    "Machines": ["machine1", "machine2"],
    "MachineCount": 2,
    "MachineManagerFqdn": "FQDN"
  }
]
Sort by WebServerType, then alphabetically by WebAppName.
If no valid web application rows exist, return [].

### External References Policy
Do NOT include links, URLs, or references to external resources unless explicitly asked.
```
