# MCP Servers

Model Context Protocol (MCP) server configurations and custom server implementations.

## Structure

```
mcp-servers/
├── configs/     # Connection configs for third-party MCP servers
└── custom/      # Custom MCP server implementations
```

### configs/

JSON or YAML files that define how to connect to external MCP servers (e.g., filesystem, database, API integrations). These follow the standard MCP client configuration format:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@example/mcp-server"],
      "env": {}
    }
  }
}
```

### custom/

Custom MCP server implementations. Each server lives in its own subdirectory with:

- Source code
- `package.json` or `requirements.txt` (dependencies)
- A README explaining what tools the server exposes

## Conventions

- Use kebab-case for server names and directories.
- Document every tool a server exposes — name, description, parameters.
- Keep secrets out of config files — use environment variables and `.env`.
