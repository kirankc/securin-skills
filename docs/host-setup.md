# Host setup guides

Per-host instructions for connecting the Securin MCP server and loading skills. See the main [README](../README.md) for Claude Code and Claude Desktop setup.

---

## VS Code (GitHub Copilot)

> **One-click:** use the install badge from the [README](../README.md#install), or use the CLI:
> ```bash
> code --add-mcp '{"name":"securin","type":"http","url":"https://mcp.securin.io/mcp"}'
> ```

**Manual setup** — add to `.vscode/mcp.json`:

```json
{
  "servers": {
    "securin": {
      "type": "http",
      "url": "https://mcp.securin.io/mcp"
    }
  }
}
```

**Skills** — clone this repo and add the path to your VS Code settings (`settings.json`):

```json
{
  "chat.agentSkillsLocations": [
    { "path": "/path/to/securin-skills/skills" }
  ]
}
```

Or copy skill subdirectories into your project's `.agents/skills/`.

---

## Cursor and Grok Bot

The Cursor Marketplace plugin is `securin-platform` (manifest `.cursor-plugin/plugin.json`, MCP config `mcp.json`). Grok Bot uses that same plugin and the same OAuth callbacks. After Cursor approves the listing, install **Securin Platform** from the Marketplace. The plugin loads `skills/` and connects to `https://mcp.securin.io/mcp`. On first use, sign in with your own Securin account in the browser.

Publishers: <https://cursor.com/marketplace/publish>. Checklist: [submission.md](submission.md).

**Manual setup** (before the listing is live). Add to `~/.cursor/mcp.json` or `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "securin": {
      "url": "https://mcp.securin.io/mcp"
    }
  }
}
```

Cursor uses remote HTTP. Do not wrap this URL in `npx mcp-remote`.

**Skills.** The Marketplace plugin discovers `skills/*/SKILL.md`. For a manual install, copy each `skills/securin-*` directory into the project's `.agents/skills/`.

**First-run OAuth.** Desktop uses `http://localhost:8787/callback` (login page already works). Cursor Agents and Grok Bot use `https://www.cursor.com/agents/mcp/oauth/callback`, which Keycloak currently rejects. The URI list for identity is [oauth-redirects.md](oauth-redirects.md).

---

## Windsurf

Add to `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "securin": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.securin.io/mcp"]
    }
  }
}
```

**Skills** — clone this repo and copy skill subdirectories from `skills/` into your project's `.agents/skills/`.

---

## Gemini CLI

Add to `.gemini/settings.json`:

```json
{
  "mcpServers": {
    "securin": {
      "url": "https://mcp.securin.io/mcp"
    }
  }
}
```

**Skills** — clone this repo and copy skill subdirectories into `.gemini/skills/` or `.agents/skills/`.

---

## OpenAI Codex CLI

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.securin]
url = "https://mcp.securin.io/mcp"
```

**Skills** — clone this repo and copy skill subdirectories into `.agents/skills/`.

---

## Any other MCP-compatible host

The endpoint `https://mcp.securin.io/mcp` uses Streamable HTTP transport. Connect directly if your host supports it, or use the `mcp-remote` bridge for stdio-only hosts:

```json
{
  "command": "npx",
  "args": ["-y", "mcp-remote", "https://mcp.securin.io/mcp"]
}
```

For skills, clone this repo and point the host's skill loader at the `skills/` directory.
