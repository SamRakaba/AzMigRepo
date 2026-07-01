# AzMigRepo — Azure Migrate CSV Processing Agents

Implementation documentation and guides for building 5 Microsoft Copilot Studio agents that process Azure Migrate CSV export files. Uses a **GPT-4.1-first** approach where the model handles all intelligence and Power Automate handles only file I/O.

## Overview

This repository contains the complete implementation guide for building Azure Migrate CSV Processing agents in **Microsoft Copilot Studio** with **Power Automate** orchestration. The solution accepts Azure Migrate export files, uses GPT-4.1 to consolidate application, SQL Server, and web application inventories, and generates downloadable Excel reports.

Development follows the [HVE Core](https://github.com/microsoft/hve-core) methodology (Research → Plan → Implement → Review).

## Key Documents

| File | What It Contains | When To Use |
|------|-----------------|-------------|
| [`AZURE_MIGRATE_AGENTS_GUIDE.md`](AZURE_MIGRATE_AGENTS_GUIDE.md) | Full step-by-step implementation (Agents 1-5, PA flows, testing, troubleshooting) | **Start here** — this is the source of truth |
| [`COPILOT_STUDIO_INSTRUCTIONS.md`](COPILOT_STUDIO_INSTRUCTIONS.md) | System prompt to paste into Agent 1 in Copilot Studio | When configuring Agent 1's instructions |
| [`COPILOT_STUDIO_GUIDE.md`](COPILOT_STUDIO_GUIDE.md) | Generic Copilot Studio portal tutorial | When new to Copilot Studio |
| [`COPILOT_INSTRUCTIONS.md`](COPILOT_INSTRUCTIONS.md) | Copilot coding guidelines and patterns | Reference for coding conventions |
| [`AGENTS.md`](AGENTS.md) | Agent quick-reference + HVE Core development methodology | Quick lookup of agents, variables, flows |
| [`.github/copilot-instructions.md`](.github/copilot-instructions.md) | Copilot repo behavior rules | Automatically read by GitHub Copilot |

## Project Structure

```
AzMigRepo/
├── .github/
│   └── copilot-instructions.md        # Copilot repo behavior rules
├── AZURE_MIGRATE_AGENTS_GUIDE.md      # Source of truth (232KB implementation guide)
├── COPILOT_STUDIO_INSTRUCTIONS.md     # Agent 1 system prompt
├── COPILOT_STUDIO_GUIDE.md            # Generic Copilot Studio tutorial
├── COPILOT_INSTRUCTIONS.md            # Coding conventions (from parent repo)
├── AGENTS.md                          # Agent quick-reference + dev methodology
├── README.md                          # This file
└── .gitignore                         # Excludes .copilot-tracking/ and temps
```

## Prerequisites

- **Microsoft Copilot Studio** — agent creation and topic design
- **Power Automate** — flow creation for file I/O
- **Azure Blob Storage** — file storage (uploads, processing, reports containers)
- **Microsoft 365 License** with Copilot Studio and Power Automate access

## Getting Started

1. **Clone**: `git clone https://github.com/SamRakaba/AzMigRepo.git`
2. **Read**: Start with [`AZURE_MIGRATE_AGENTS_GUIDE.md`](AZURE_MIGRATE_AGENTS_GUIDE.md) for the full implementation
3. **Install HVE Core** (dev tooling): [VS Code extension](https://marketplace.visualstudio.com/items?itemName=ise-hve-essentials.hve-core) or `copilot plugin marketplace add microsoft/hve-core`
4. **Develop**: Use `/task-research <topic>` to research before building

## Resources

- [AZMrepo — Parent project](https://github.com/SamRakaba/AZMrepo)
- [Azure Migrate Documentation](https://learn.microsoft.com/en-us/azure/migrate/)
- [Microsoft Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/)
- [HVE Core — Dev methodology](https://github.com/microsoft/hve-core)

## License

This project is provided as-is for educational and reference purposes.

---

**Last Updated**: July 2026
