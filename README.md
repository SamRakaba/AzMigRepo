# AzMigRepo — Azure Migrate Agents with HVE Core

Copilot-powered project for Azure Migrate workflows, leveraging the [HVE Core (Hypervelocity Engineering)](https://github.com/microsoft/hve-core) framework for structured AI-assisted development.

## Overview

This repository provides tools, documentation, and agent definitions for managing **Azure Migrate** workflows using GitHub Copilot and Microsoft Copilot Studio. It leverages the HVE Core framework for repeatable, standards-aligned AI workflows including research, planning, implementation, code review, and security assessment.

## Key Capabilities

| Capability | Description |
|-----------|-------------|
| **RPI Workflow** | Research → Plan → Implement → Review methodology for all tasks |
| **Security Assessment** | OWASP vulnerability scanning via security-reviewer agent |
| **Code Review** | Multi-perspective code review with human-gated orchestration |
| **Documentation** | Automated doc audit, drift detection, and authoring |
| **Architecture** | ADR creation, system architecture review |
| **Data Analysis** | Jupyter notebooks and Streamlit dashboards from data sources |
| **Backlog Management** | GitHub, Jira, and Azure DevOps integration |

## Project Structure

```
AzMigRepo/
├── .github/
│   └── copilot-instructions.md   # Copilot project instructions
├── AGENTS.md                      # Agent definitions and skills (HVE Core)
├── README.md                      # This file
├── .gitignore                     # Excludes .copilot-tracking/ and temps
└── docs/                          # Guides and references
```

## Quick Start

1. **Clone the repo**: `git clone https://github.com/SamRakaba/AzMigRepo.git`
2. **Install HVE Core**: Via [VS Code extension](https://marketplace.visualstudio.com/items?itemName=ise-hve-essentials.hve-core) or CLI (`copilot plugin marketplace add microsoft/hve-core`)
3. **Review agents**: See [AGENTS.md](AGENTS.md) for available agents, skills, and prompts
4. **Start working**: Use `/task-research <topic>` to begin the RPI workflow

## Resources

- [HVE Core — microsoft/hve-core](https://github.com/microsoft/hve-core)
- [Azure Migrate Documentation](https://learn.microsoft.com/en-us/azure/migrate/)
- [AZMrepo — Source project](https://github.com/SamRakaba/AZMrepo)
- [Microsoft Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/)

## License

This project is provided as-is for educational and reference purposes.

---

**Last Updated**: June 2026
