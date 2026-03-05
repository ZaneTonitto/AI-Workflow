# Agents

Custom agent definitions for software engineering workflows.

## Structure

Each agent is defined in its own file (Markdown or YAML) and should include:

| Field           | Description                                        |
| --------------- | -------------------------------------------------- |
| **Name**        | Short, descriptive agent name                      |
| **Description** | What the agent does and when to use it             |
| **Persona**     | System-level instructions / personality             |
| **Skills**      | List of skills from `../skills/` the agent can use |
| **References**  | Relevant docs from `../references/`                |
| **MCP Servers** | MCP servers the agent connects to                  |

## Conventions

- One agent per file.
- Use kebab-case filenames (e.g., `code-reviewer.md`).
- Keep the persona focused — an agent that tries to do everything does nothing well.
