# obsidian + cortex code CLI: setup

how to connect Obsidian to Cortex Code CLI via MCP, so the agent can read and write your vault as part of its workflows.

---

## how it works

Cortex Code CLI supports MCP (Model Context Protocol) as an extensibility mechanism. MCP is an open standard for connecting AI agents to external tools and data sources. Obsidian, via the Local REST API community plugin, exposes your vault over a local HTTP endpoint. An MCP server (the `mcp-obsidian` package) bridges the two: it runs as a local subprocess, translates MCP calls into REST API requests, and returns results to Cortex Code.

once connected, Cortex Code can read notes, search the vault, create files, and update existing notes as part of any agentic workflow. you don't invoke tools manually. you describe what you want and the agent calls the right tools.

---

## comparison with claude code

the MCP setup in Cortex Code CLI is functionally identical to Claude Code. same protocol, same config structure, same transport types, same `${VAR}` syntax for environment variables.

| | claude code | cortex code CLI |
|---|---|---|
| config file | `~/.claude/mcp.json` | `~/.snowflake/cortex/mcp.json` |
| CLI command | `claude mcp add` | `cortex mcp add` |
| session status | `claude mcp list` | `/mcp-status` in session |
| permissions file | `~/.claude/permissions.json` | `~/.snowflake/cortex/permissions.json` |
| tool namespace | `mcp__server__tool` | `mcp__server__tool` |

if you already have this configured for Claude Code, you copy the same JSON to a different path.

---

## prerequisites

- Obsidian installed and running (the REST API plugin requires the app to be open)
- `uv` installed: `pip install uv` or `brew install uv`
- Cortex Code CLI installed and connected to Snowflake

---

## step 1: install the obsidian plugin

in Obsidian: **Settings > Community plugins > Browse**, search for **Local REST API**, install, enable.

after enabling, go to **Settings > Local REST API**:
- copy the API key
- note the port (default: `27123`)
- make sure the plugin is running (green indicator)

the plugin must be active whenever you want Cortex Code to access your vault. if Obsidian is closed, the MCP connection will fail.

---

## step 2: set the environment variable

don't hardcode the API key in the config file. use an env variable:

```bash
export OBSIDIAN_API_KEY="your_key_here"
```

add this to `~/.zshrc` or `~/.bashrc` to make it permanent:

```bash
echo 'export OBSIDIAN_API_KEY="your_key_here"' >> ~/.zshrc
source ~/.zshrc
```

---

## step 3: configure the MCP server

edit `~/.snowflake/cortex/mcp.json`. if the file doesn't exist, create it:

```json
{
  "mcpServers": {
    "obsidian": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-obsidian"],
      "env": {
        "OBSIDIAN_API_KEY": "${OBSIDIAN_API_KEY}",
        "OBSIDIAN_HOST": "127.0.0.1",
        "OBSIDIAN_PORT": "27123"
      }
    }
  }
}
```

`uvx` downloads and runs `mcp-obsidian` without a separate install step. if you already have it installed via pip or uv, you can use the installed binary directly instead.

---

## step 4: verify

```bash
cortex mcp list
```

should show `obsidian` in the list. start a session and run:

```
/mcp-status
```

the server should show as connected. if it shows disconnected, check that Obsidian is open and the plugin is active.

---

## step 5: configure permissions (optional)

MCP tool permissions are requested on first use. to pre-approve common read operations so you're not prompted every time, edit `~/.snowflake/cortex/permissions.json`:

```json
{
  "allow": [
    "mcp__obsidian__list_files",
    "mcp__obsidian__read_note",
    "mcp__obsidian__search"
  ],
  "deny": []
}
```

write operations (create, update) will still prompt unless you add them to `allow` too. it's fine to be conservative here and approve writes manually until you trust the workflows.

---

## troubleshooting

**server shows disconnected after `/mcp-status`**
- check that Obsidian is open
- check that the Local REST API plugin is enabled and running (green dot in plugin settings)
- verify the port matches what's in the plugin settings vs your mcp.json
- check `OBSIDIAN_API_KEY` is set in your current shell session: `echo $OBSIDIAN_API_KEY`

**`uvx` not found**
- `uv` is not installed. run `pip install uv` or `brew install uv`
- or replace `"command": "uvx"` with the full path to uvx: `which uvx`

**permission denied errors**
- the first time a tool is called, Cortex Code will ask for permission. this is expected. approve once and optionally add to `permissions.json` to avoid future prompts.

**connection works but Obsidian can't find files**
- file paths in MCP requests are relative to the vault root, not your filesystem. use `vault/folder/note.md` not `/Users/you/obsidian-vault/folder/note.md`.

---

## config directory overview

for reference, the full Cortex Code CLI config directory:

```
~/.snowflake/cortex/
├── settings.json       # main settings
├── mcp.json            # MCP server configs  ← this is what we edited
├── permissions.json    # saved tool permissions
├── conversations/      # session history
├── skills/             # global skills
├── commands/           # custom commands
├── hooks/              # hook scripts
├── memory/             # persistent memory
└── cache/              # temporary cache
```

---

## references

- Cortex Code CLI extensibility docs: https://docs.snowflake.com/en/user-guide/cortex-code/extensibility
- Cortex Code CLI reference: https://docs.snowflake.com/en/user-guide/cortex-code/cli-reference
- mcp-obsidian package: https://github.com/MarkusPfundstein/mcp-obsidian
- Obsidian Local REST API plugin: search "Local REST API" in Obsidian community plugins
