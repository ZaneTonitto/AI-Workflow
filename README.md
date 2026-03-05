# AI-Workflow

Custom AI workflow for software engineering — a curated collection of agents, skills, reference material, and MCP server configurations.

## Directory Structure

```
AI-Workflow/
├── .copilot/        # Copilot CLI config (MCP servers, etc.)
├── agents/          # Custom agent definitions and personas
├── skills/          # Reusable skill definitions for agents
└── references/      # Reference documentation agents can consult
```

## Quick Start

### Agents

Place agent definition files in `agents/`. Each agent should have a clear purpose, persona, and set of assigned skills. See [`agents/README.md`](agents/README.md) for the expected format.

### Skills

Define reusable skills in `skills/`. Skills encapsulate specific capabilities (e.g., code review, test generation) that can be composed into agents. See [`skills/README.md`](skills/README.md).

### References

Add reference material to `references/` — coding standards, architecture docs, API specs, style guides, or any context agents should have access to. See [`references/README.md`](references/README.md).

### MCP Servers

MCP server connections are configured in [`.copilot/mcp.json`](.copilot/mcp.json).

## Contributing

1. Follow the README conventions in each subdirectory when adding new items.
2. Keep agent definitions focused — one agent per concern.
3. Document any external dependencies in the relevant subdirectory.
