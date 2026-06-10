# mcp-manager

Monitor and recover MCP servers connected to Claude — without touching settings files or restarting Claude.

---

## Install

```sh
claude mcp add mcp-manager npx @waycraft/mcp-manager
```

Or add manually to `~/.claude/mcp.json` (global) or `.claude/mcp.json` (project-level):

```json
{
  "mcpServers": {
    "mcp-manager": {
      "command": "npx",
      "args": ["@waycraft/mcp-manager"]
    }
  }
}
```

---

## What it does

When an MCP server drops, flaps, or fails to restart, diagnosing it normally means leaving the conversation to check logs or edit config files. mcp-manager exposes the live state, disk config, restart control, and event history of every connected server as MCP tools Claude can call directly.

---

## Tools

### `mcp_status`

Returns live connection state for every MCP server — name, state (connected / dropped / degraded / failed), latency, last seen, restart count, and any current error. Use this first when something feels off.

### `mcp_config`

Returns how each MCP is configured on disk — the exact launch command, config file path, and config scope (project, user-settings, or global). **Distinct from `mcp_status`:** status is the live runtime state (is it running?); config is the setup on disk (how is it launched?). Use config when you need to verify a command path, check the node version in use, or understand which settings file controls a server.

### `mcp_restart`

Restarts a specific server by name. **Skips the server if it's already healthy** — use `force: true` to restart regardless of health state. Healthy servers are never disrupted by default.

**Parameters:**
- `name` — server name exactly as shown in `mcp_status`
- `force` — (optional) set to `true` to restart even if the server is currently healthy

### `mcp_restart_all`

Restarts all servers that are not currently healthy. Healthy servers are skipped. Returns a summary of what was restarted and what was left alone.

### `mcp_history`

Returns a timestamped event log for a specific server — connect, drop, and restart events. Drop events include the process exit details and the **last stderr output** from the server at the time of the crash, so you can see what it actually printed before it died.

**Parameters:**
- `name` — server name to inspect

---

## Typical workflow

```
Something feels off
  → mcp_status          # find which server is dropped or degraded
  → mcp_restart name    # try a restart
  → still failing?
  → mcp_history name    # read the stderr from the crash
  → mcp_config          # verify the launch command and file path are correct
  → fix the root cause, restart again
```

---

## Pairs with

**[session-continuity](https://www.npmjs.com/package/session-continuity)** — if session-continuity drops mid-session, mcp-manager restarts it without leaving the conversation. No lost context.

```bash
claude mcp add session-continuity npx session-continuity
```

**[@waycraft/waypoint-mcp](https://www.npmjs.com/package/@waycraft/waypoint-mcp)** — if waypoint drops during a build cycle, mcp-manager brings it back instantly.

```bash
claude mcp add waypoint npx @waycraft/waypoint-mcp
```

---

## Feedback & Discussion

Something not working, a server mcp-manager doesn't handle well, or a feature you'd find useful?

→ [GitHub Discussions](https://github.com/explorenav-dev/mcp-manager/discussions)

---

## License

[PolyForm Noncommercial License 1.0](https://polyformproject.org/licenses/noncommercial/1.0.0/) — free for personal and non-commercial use.
