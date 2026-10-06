# mcp-mail
Superhuman Mail MCP Server

Add to your MCP client configuration:

```json
{
  "mcpServers": {
    "Superhuman Mail": {
      "command": "npx",
      "args": ["-y", "@superhuman/mcp-mail"]
    }
  }
}
```

## Hosted MCP server

The official Superhuman Mail MCP server uses OAuth on first connection and exposes email and calendar tools for authenticated accounts.
