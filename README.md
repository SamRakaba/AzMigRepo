# AzMigRepo — Azure Migrate Hyper-V Environment (HVE) Agents

Copilot-powered agents for Azure Migrate Hyper-V discovery, assessment, and migration planning.

## Overview

This repository provides tools, documentation, and agent definitions for managing **Azure Migrate Hyper-V Environment (HVE)** workflows using GitHub Copilot and Microsoft Copilot Studio. It builds on the patterns from the [AZMrepo](https://github.com/SamRakaba/AZMrepo) project, extending them to Hyper-V host discovery, VM inventory collection, and migration readiness assessment.

## Key Capabilities

| Capability | Description |
|-----------|-------------|
| **HVE Discovery** | Discover Hyper-V hosts, clusters, and guest VMs across environments |
| **VM Inventory** | Collect and consolidate VM metadata (CPU, memory, disk, network) |
| **Readiness Assessment** | Evaluate migration readiness for Azure targets (Azure VM, AVS, AKS) |
| **Dependency Mapping** | Map application dependencies across Hyper-V workloads |
| **CSV Processing** | Process Azure Migrate export files for Hyper-V assessments |
| **Report Generation** | Generate Excel reports with migration recommendations |
| **Cost Estimation** | Estimate Azure costs for migrated Hyper-V workloads |

## Project Structure

```
AzMigRepo/
├── .github/
│   └── copilot-instructions.md   # Copilot project instructions
├── AGENTS.md                      # HVE agent definitions and skills
├── README.md                      # This file
├── docs/                          # Guides and references
│   ├── hve-discovery-guide.md     # Hyper-V discovery walkthrough
│   ├── assessment-patterns.md     # Assessment and readiness patterns
│   └── migration-playbook.md      # End-to-end migration playbook
├── scripts/                       # Automation scripts
│   ├── discover_hyperv_hosts.ps1  # PowerShell: discover Hyper-V hosts
│   ├── collect_vm_inventory.ps1   # PowerShell: collect VM inventory
│   ├── export_assessment.py       # Python: export assessment data
│   └── generate_report.py         # Python: generate migration report
├── agents/                        # Agent configuration files
│   ├── hve_discovery_agent.json   # HVE Discovery agent definition
│   ├── assessment_agent.json      # Assessment agent definition
│   └── migration_planner.json     # Migration planner agent definition
├── templates/                     # Report and spreadsheet templates
│   └── hve_assessment_template.xlsx
└── tests/                         # Test files
    ├── test_discovery.py
    └── test_assessment.py
```

## Quick Start

1. **Clone the repo**: `git clone https://github.com/SamRakaba/AzMigRepo.git`
2. **Review HVE agents**: See [AGENTS.md](AGENTS.md) for agent skills and capabilities
3. **Set up discovery**: Follow [docs/hve-discovery-guide.md](docs/hve-discovery-guide.md)
4. **Run assessment**: Use the scripts in `scripts/` or invoke agents via Copilot

## Use Cases

- **Hyper-V Host Discovery**: Scan and catalog Hyper-V hosts across datacenters
- **Guest VM Inventory**: Collect detailed VM metadata for migration planning
- **Migration Readiness**: Assess workloads for Azure VM, AVS, or containerization
- **Dependency Analysis**: Map inter-VM dependencies before migration waves
- **Consolidation Reports**: Generate spreadsheets consolidating assessment data
- **Cost Planning**: Estimate Azure run costs for Hyper-V workloads

## Prerequisites

- Azure subscription with Azure Migrate project
- Hyper-V hosts running Windows Server 2012 R2 or later
- Azure Migrate appliance deployed for Hyper-V discovery
- Python 3.10+ and PowerShell 7+ for scripts
- Access to Microsoft Copilot Studio (for agent development)

## Resources

- [Azure Migrate: Hyper-V Assessment](https://learn.microsoft.com/en-us/azure/migrate/tutorial-assess-hyper-v)
- [Azure Migrate Appliance for Hyper-V](https://learn.microsoft.com/en-us/azure/migrate/how-to-set-up-appliance-hyper-v)
- [AZMrepo — Source project](https://github.com/SamRakaba/AZMrepo)
- [Microsoft Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/)

## License

This project is provided as-is for educational and reference purposes.

---

**Last Updated**: June 2026
