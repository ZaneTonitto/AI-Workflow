# AI-Workflow

Custom AI agents and skills for development on **CargoWise** — a curated collection of agent definitions and reusable skills used by GitHub Copilot.

## Directory Structure

```
AI-Workflow/
└── .github/
    ├── agents/      # Custom agent definitions
    └── skills/      # Reusable skill definitions for agents
```

## Agents

Agent definitions live in `.github/agents/`. Each agent has a focused purpose and a set of assigned skills.

| Agent | Description |
| ----- | ----------- |
| [issue-triager](.github/agents/issue-triager.agent.md) | Triage and analyze CargoWise work items (WI, CS, PRJ) — investigate issues, search related items, and produce resolution plans. |
| [issue-fixer](.github/agents/issue-fixer.agent.md) | Implement code fixes in the CargoWise repository using a resolution plan from the issue-triager agent. |

See [`.github/agents/README.md`](.github/agents/README.md) for conventions.

## Skills

Reusable skills live in `.github/skills/`. Skills encapsulate specific capabilities that can be composed into agents.

| Skill | Description |
| ----- | ----------- |
| [triage-issue](.github/skills/triage-issue/) | Investigate a work item, gather context, and produce a resolution plan. |
| [fix-issue](.github/skills/fix-issue/) | Implement a code fix and write unit tests based on a resolution plan. |
| [github-code-navigation](.github/skills/github-code-navigation/) | Locate files and understand code patterns via GitHub code search. |

See [`.github/skills/README.md`](.github/skills/README.md) for conventions.

## Contributing

1. Follow the README conventions in each subdirectory when adding new items.
2. Keep agent definitions focused — one agent per concern.
3. Skills should be composable — avoid coupling skills to specific agents.
