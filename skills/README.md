# Skills

Reusable skill definitions that can be composed into agents.

## What is a Skill?

A skill is a well-defined capability — a set of instructions, tool access, and context that lets an agent perform a specific task. Examples:

- **Code Review** — Analyze diffs for bugs, security issues, and style violations.
- **Test Generation** — Create unit/integration tests for a given module.
- **Refactoring** — Apply safe, incremental refactoring patterns.
- **Documentation** — Generate or update docs from code.

## Structure

Each skill file should include:

| Field              | Description                                      |
| ------------------ | ------------------------------------------------ |
| **Name**           | Short skill name                                 |
| **Description**    | What the skill enables                           |
| **Instructions**   | Detailed instructions for executing the skill    |
| **Tools Required** | Any MCP tools or CLI tools the skill depends on  |
| **References**     | Relevant reference material from `../references/`|

## Conventions

- One skill per file.
- Use kebab-case filenames (e.g., `code-review.md`).
- Skills should be composable — avoid coupling skills to specific agents.
