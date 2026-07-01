# AGENTS.md — AzMigRepo HVE Agent Definitions

This file defines the agents, skills, and orchestration patterns for Azure Migrate Hyper-V Environment (HVE) workflows in this project.

---

## Agent Overview

| Agent | Purpose | Trigger Examples |
|-------|---------|-----------------|
| **HVE Discovery Agent** | Discover Hyper-V hosts, clusters, and guest VMs | "discover Hyper-V hosts", "scan datacenter", "list HVE inventory" |
| **Assessment Agent** | Evaluate migration readiness for Azure targets | "assess readiness", "run HVE assessment", "check migration eligibility" |
| **CSV Processor Agent** | Process Azure Migrate export CSV files | "process assessment export", "consolidate CSV data", "clean inventory file" |
| **Migration Planner Agent** | Plan migration waves and estimate costs | "plan migration waves", "estimate Azure costs", "group workloads" |
| **Dependency Mapper Agent** | Map application dependencies across VMs | "map dependencies", "show VM connections", "analyze app topology" |
| **Report Generator Agent** | Generate Excel reports with findings | "generate report", "create assessment spreadsheet", "export summary" |

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

## Agent Orchestration

Agents can be chained for end-to-end workflows:

```
1. HVE Discovery Agent → discovers hosts and VMs
2. Assessment Agent → evaluates readiness
3. Dependency Mapper Agent → maps app dependencies
4. CSV Processor Agent → cleans and consolidates data
5. Migration Planner Agent → plans waves and estimates costs
6. Report Generator Agent → produces final Excel deliverable
```

### Orchestration via Copilot Studio Topics

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

---

## Skill Reference Quick Table

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

---

**Last Updated**: June 2026
