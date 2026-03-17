# Agents

Custom agent definitions for CargoWise development workflows.

## Current Agents

| Agent | Description |
| ----- | ----------- |
| [issue-investigator](issue-investigator.agent.md) | Investigate and analyze work items (WI, CS, PRJ) — search related items, inspect code, and produce resolution plans. |
| [issue-fixer](issue-fixer.agent.md) | Implement code fixes in the CargoWise repository using a resolution plan. |

## Structure

Each agent is defined in its own `.agent.md` file and should include:

| Field           | Description                                        |
| --------------- | -------------------------------------------------- |
| **Name**        | Short, descriptive agent name                      |
| **Description** | What the agent does and when to use it             |
| **Persona**     | System-level instructions / personality             |
| **Skills**      | List of skills from `../skills/` the agent can use |
| **Tools**       | MCP tools and built-in tools the agent can access  |

## Conventions

- One agent per file.
- Use kebab-case filenames (e.g., `code-reviewer.agent.md`).
- Keep the persona focused — an agent that tries to do everything does nothing well.
