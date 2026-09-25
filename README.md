# Tabellum — MCP setup

Let your AI assistant work with your databases through
[Tabellum](https://tabellum.dudko.dev), a desktop database client for
PostgreSQL, MySQL/MariaDB, SQL Server and BigQuery. The MCP server is hosted **by
the running app** on your machine; this package is a small stdio shim that
forwards to it. Queries the assistant writes appear live in the editor, and
**nothing that changes data runs without your click** in the app.

- **Server:** `tabellum`
- **Transport:** stdio (local process)
- **Package:** [`@dudko.dev/tabellum-mcp`](https://www.npmjs.com/package/@dudko.dev/tabellum-mcp) — Node.js 24+
- **Auth:** a per-install token the app generates; the shim reads it for you from `~/.tabellum/mcp.json`.

## What you can do

With the Tabellum app open, the assistant can:

| Tool | What it does |
|---|---|
| `list_connections` | Connection names, drivers, read-only flags. |
| `describe_schema` | Object names per schema. |
| `describe_object` | Columns, indexes, constraints, DDL. |
| `list_users` | Database users and roles. |
| `sample_rows` | Up to N rows — only where the connection allows sharing rows. |
| `run_read` | Execute a statement the engine classifies as a read. |
| `propose_write` | Submit a write/DDL for **your approval in the app** (10-minute timeout). |
| `explain` | EXPLAIN in the connection's dialect. |
| `draft_to_editor` | Stream SQL live into an editor tab. |
| `open_editor_tab` | Open a tab bound to a connection. |

---

## Install in Claude Code (plugin)

```
/plugin marketplace add dudko-dev/tabellum-mcp-setup
/plugin install tabellum@tabellum
/reload-plugins
```

**Start the Tabellum app first** (macOS, Windows or Linux). The shim reads the
app's local port and token from `~/.tabellum/mcp.json`, written while the app
runs; with the app closed every call answers "Tabellum is not running".

> Prefer not to use the marketplace? Add the server directly:
> ```
> claude mcp add tabellum -- npx -y @dudko.dev/tabellum-mcp
> ```

---

## Connect from other clients

### Claude Desktop

`claude_desktop_config.json` (Settings → Developer → Edit Config):

```json
{
  "mcpServers": {
    "tabellum": {
      "command": "npx",
      "args": [
        "-y",
        "@dudko.dev/tabellum-mcp"
      ]
    }
  }
}
```

### Cursor

`~/.cursor/mcp.json` or `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "tabellum": {
      "command": "npx",
      "args": [
        "-y",
        "@dudko.dev/tabellum-mcp"
      ]
    }
  }
}
```

### VS Code (GitHub Copilot / MCP) and other clients

```json
{
  "servers": {
    "tabellum": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@dudko.dev/tabellum-mcp"
      ]
    }
  }
}
```

---

## Safety

- **The engine classifies, you approve.** Every statement is classified by the
  app's parser; anything that is not a read goes through `propose_write` and
  waits for your click. A label claimed by the model never overrides it.
- **Loopback only.** The app listens on `127.0.0.1` with a per-install bearer
  token (Settings → Agents shows it and can regenerate it).
- **Row data is opt-in per connection.** `sample_rows` returns rows only where
  you enabled sharing for that connection.

---

## Troubleshooting

- **"Tabellum is not running"** — open the app, then retry.
- **Unauthorized after regenerating the token** — restart the MCP server in your
  client so the shim re-reads `~/.tabellum/mcp.json`.
- **A write seems stuck** — it is waiting for approval in the app; after 10
  minutes it returns `declined: timeout`.

## Support

- Website: https://tabellum.dudko.dev
- Contact: siarhei@dudko.dev
