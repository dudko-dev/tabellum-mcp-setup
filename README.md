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

With the Tabellum app open, the assistant gets the same tools as the app's own
agent — the app serves them, so the list follows the app version. Reads run at
once; anything that writes waits for your click in the app.

| Tool | What it does |
|---|---|
| **Look around** | |
| `list_connections` | Connection names, drivers, read-only flags. No hosts, no users. |
| `describe_schema` | Every schema of a connection with its table, view, function, procedure and sequence names. |
| `describe_object` | Columns, indexes, constraints and the `CREATE` of one object. |
| `list_users` | Users and roles with their flags, where the database lists them. |
| `sample_rows` | Up to N rows of a table — only where the connection shares rows. |
| **Query** | |
| `run_read` | Runs a statement only if the engine classifies it as a read; anything else answers `needs_approval`. |
| `explain` | The estimated plan of a read; nothing runs. |
| `propose_write` | A write, DDL or admin statement for **your approval in the app** (10-minute timeout). |
| `open_editor_tab` | Open a query tab bound to a connection. |
| `draft_to_editor` | Type SQL into a tab while you watch, without running it. |
| **Compare schemas** | |
| `compare_schemas` | Diff a connection against another or a saved snapshot; with `migration: true`, the statements that close the gap. Nothing runs. |
| `save_schema_snapshot` / `list_schema_snapshots` | Save a connection's schema to the workspace; list the snapshots. |
| **Dashboards** | |
| `list_dashboards` / `get_dashboard` | The workspace's dashboards; one dashboard's file and panels. |
| `propose_dashboard` | A new or changed dashboard — you see the diff and apply it. |
| **Schema designs** | |
| `list_designs` / `get_design` | The workspace's schema designs; one design's file and objects. |
| `propose_design` | A new or changed design — you see what is added, changed and removed. Nothing runs against a database. |
| `describe_feature` | The file format and rules of `designer`, `dashboards` or `schema_compare`, read before proposing one. |

Resources, for clients that prefer them: `tabellum://connections`,
`tabellum://schema/<connection>`, `tabellum://snapshots`,
`tabellum://dashboards`, `tabellum://designs`. Every call is attributed in the
app — a badge on the tab and a line in the agent log.

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
