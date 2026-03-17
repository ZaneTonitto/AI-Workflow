# Skills

Reusable skill definitions that can be composed into agents for CargoWise development.

## Current Skills

| Skill | Description |
| ----- | ----------- |
| [investigate-issue](investigate-issue/) | Investigate a work item, gather context, and produce a resolution plan. |
| [fix-issue](fix-issue/) | Implement a code fix and write unit tests based on a resolution plan. |
| [github-code-navigation](github-code-navigation/) | Locate files and understand code patterns via GitHub code search. |

## What is a Skill?

A skill is a well-defined capability — a set of instructions, tool access, and context that lets an agent perform a specific task.

## Structure

Each skill lives in its own subdirectory containing a `SKILL.md` file:

| Field              | Description                                      |
| ------------------ | ------------------------------------------------ |
| **Name**           | Short skill name                                 |
| **Description**    | What the skill enables                           |
| **Instructions**   | Detailed instructions for executing the skill    |
| **Tools Required** | Any MCP tools or CLI tools the skill depends on  |

## Conventions

- One skill per directory.
- Use kebab-case directory names (e.g., `code-review/`).
- Skills should be composable — avoid coupling skills to specific agents.
