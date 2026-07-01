# AzMigRepo — Azure Migrate Agents

Tools and documentation for building Microsoft Copilot Studio agents focused on Azure Migrate workflows — assessment data processing, inventory consolidation, and migration reporting.

## Overview

This repository provides documentation, scripts, and agent definitions for managing **Azure Migrate** tasks using Microsoft Copilot Studio and Power Automate. It builds on the patterns from the [AZMrepo](https://github.com/SamRakaba/AZMrepo) project.

Development in this repo follows the [HVE Core](https://github.com/microsoft/hve-core) methodology (Research → Plan → Implement → Review) for structured, repeatable AI-assisted workflows.

## Project Scope

- **CSV Processing**: Process and consolidate Azure Migrate export files
- **Application Inventory**: Consolidate application inventories with duplicate removal
- **SQL Server Assessment**: Process SQL Server inventory for migration assessment
- **Database Consolidation**: Aggregate database inventory across environments
- **Automated Reporting**: Generate Excel reports from assessment data
- **Copilot Studio Agents**: Agent definitions with Power Automate tool integrations

## Project Structure

```
AzMigRepo/
├── .github/
│   └── copilot-instructions.md   # Copilot project conventions
├── AGENTS.md                      # Development guide (HVE Core methodology)
├── README.md                      # This file
├── .gitignore                     # Excludes .copilot-tracking/ and temps
└── docs/                          # Guides and references
```

## Getting Started

1. **Clone**: `git clone https://github.com/SamRakaba/AzMigRepo.git`
2. **Install HVE Core** (dev tooling): [VS Code extension](https://marketplace.visualstudio.com/items?itemName=ise-hve-essentials.hve-core) or `copilot plugin marketplace add microsoft/hve-core`
3. **Read AGENTS.md**: Development methodology and how Copilot should work here
4. **Start**: Use `/task-research <topic>` to research before building

## Resources

- [AZMrepo — Parent project](https://github.com/SamRakaba/AZMrepo)
- [Azure Migrate Documentation](https://learn.microsoft.com/en-us/azure/migrate/)
- [Microsoft Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/)
- [HVE Core — Dev methodology](https://github.com/microsoft/hve-core)

## License

This project is provided as-is for educational and reference purposes.

---

**Last Updated**: June 2026
