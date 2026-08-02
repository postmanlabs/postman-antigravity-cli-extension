# Postman Antigravity CLI Extension

The Postman extension for [Google Antigravity](https://antigravity.google) (`agy`) connects Postman to your Antigravity CLI and IDE agent, giving it the ability to access workspaces, manage collections and environments, evaluate APIs, and automate workflows through natural language.

This is the Antigravity plugin port of the [Postman Gemini CLI extension](https://github.com/postmanlabs/postman-gemini-cli-extension) — Google is transitioning Gemini CLI to Antigravity CLI, where extensions become native plugins.

## Prerequisites

Antigravity CLI (`agy`) 1.0.9 or later is recommended.

## Installation

Install the plugin directly from this repository:

```bash
agy plugin install https://github.com/postmanlabs/postman-antigravity-cli-extension
```

Or from a local checkout:

```bash
agy plugin install /path/to/postman-antigravity-cli-extension
```

Validate the plugin layout at any time with:

```bash
agy plugin validate /path/to/postman-antigravity-cli-extension
```

## Authentication

This extension connects to Postman's hosted MCP server and authenticates via **OAuth** — no API key required. Antigravity opens a browser window to complete the Postman login flow on first use.

### API key (alternative)

If you prefer API key authentication, add an `Authorization` header to the server entry in `mcp_config.json`, reading the key from your environment:

```json
{
  "mcpServers": {
    "postman": {
      "serverUrl": "https://mcp.postman.com/minimal",
      "headers": {
        "Authorization": "Bearer $POSTMAN_API_KEY"
      }
    }
  }
}
```

```bash
export POSTMAN_API_KEY=your-api-key-here
```

Add the `export` line to your `~/.zshrc` or `~/.bashrc` to persist it across sessions. Get your API key at [postman.postman.co/settings/me/api-keys](https://postman.postman.co/settings/me/api-keys).

> **Note**: OAuth is the recommended path. Never commit a real API key to this repo or to `mcp_config.json` — always read it from an environment variable.

## Slash commands

The plugin ships eighteen `/postman:*` commands (defined in `commands/postman/*.toml`) — the canonical command set shared with the Postman Claude Code and Cursor plugins:

| Command | What it does |
|---------|--------------|
| `/postman:setup` | Initialize a Postman workspace, collection, and environment for the current project |
| `/postman:sync` | Update the collection to reflect the current API code, then re-run tests |
| `/postman:run-collection` | Run the project's collection and show test results by endpoint |
| `/postman:generate-spec [path]` | Generate or update an OpenAPI 3.0 spec from your codebase, then optionally sync it to Postman |
| `/postman:generate-client [collection]` | Generate typed client code from a Postman collection (Code/Full mode) |
| `/postman:test [collection]` | Run collection tests, diagnose failures, and suggest fixes |
| `/postman:search <question>` | Discover APIs across your workspaces and the public network |
| `/postman:mock [source]` | Create a mock server from a collection or spec (auto-generates examples) |
| `/postman:docs [source]` | Generate, improve, and publish API documentation |
| `/postman:security [source]` | Audit an API against the OWASP API Top 10 |
| `/postman:learn <question>` | Search the Postman Learning Center for how-to guidance (Learn mode) |
| `/postman:send-request [req]` | Send an HTTP request via the Postman CLI |
| `/postman:list-flows <workspace>` | List Postman Flows in a workspace and resolve a flow name to its ID (Postman CLI) |
| `/postman:trigger-flow <flow>` | Trigger a deployed Flow with inputs; offers to deploy-then-trigger (Postman CLI) |
| `/postman:deploy-flow <flow>` | Deploy a Flow so it becomes triggerable, confirming a trigger path first (Postman CLI) |
| `/postman:get-flow-run <run-id>` | Inspect a Flow run — per-block logs, the failing block, and status (Postman CLI) |
| `/postman:use-local [mode]` | Switch this plugin's server to the local stdio package (`npx`) — see [Local vs. remote](#local-vs-remote-server) |
| `/postman:use-remote [mode]` | Switch this plugin's server back to the hosted server (OAuth) — see [Local vs. remote](#local-vs-remote-server) |

> `/postman:learn` needs the **Learn** endpoint (`mcp.postman.com/learn`) or the **Full** endpoint (`mcp.postman.com/mcp`) — `searchLearningCenter` isn't exposed in the default minimal mode. `/postman:send-request` and the four Flow commands (`/postman:list-flows`, `/postman:trigger-flow`, `/postman:deploy-flow`, `/postman:get-flow-run`) drive the **Postman CLI** and need it installed and `postman login` active — they work with any MCP mode. `/postman:generate-client` needs the **Code** endpoint (`mcp.postman.com/code`) or **Full** endpoint (`mcp.postman.com/mcp`) — its codegen tools (`getCodeGenerationInstructions` and the detailed `getCollection*` tools) aren't exposed in the default minimal mode. Switch with `/postman:use-remote code` (or `/postman:use-remote full`), or point `serverUrl` at `/code` or `/mcp`.

Agent guidance (collection-schema rules, workflow patterns, troubleshooting) is loaded on demand from `skills/postman/SKILL.md`.

## Tool Configuration

This extension uses the **minimal** toolset by default — fast, focused access to core Postman operations. To use a different mode, update the URL in `mcp_config.json`:

| Mode | URL | Use when |
|------|-----|----------|
| Minimal (default) | `https://mcp.postman.com/minimal` | Collections, workspaces, environments, specs |
| Full | `https://mcp.postman.com/mcp` | 100+ tools, advanced collaboration, Enterprise |
| Code | `https://mcp.postman.com/code` | API search and client code generation |
| Learn | `https://mcp.postman.com/learn` | Search Postman Docs / Learning Center for guides and concepts |

## Local vs. remote server

The same plugin can talk to either Postman's **hosted (remote)** MCP server or the **local** stdio package (`@postman/postman-mcp-server`, run through `npx`). You don't have to hand-edit `mcp_config.json` or install a second plugin to switch — two slash commands rewrite this plugin's own `mcp_config.json` in place:

| Command | Result |
|---------|--------|
| `/postman:use-remote [minimal\|code\|full\|learn]` | Points `serverUrl` at the hosted server (OAuth; US region), keeping your current toolset. The `[mode]` arg is an optional override. |
| `/postman:use-local  [minimal\|code\|full\|learn]` | Replaces the entry with a local `npx` stdio server reading `POSTMAN_API_KEY` from your environment, keeping your current toolset. The `[mode]` arg is an optional override. |

After running either command, **restart the `agy` session** (or reload servers via *Additional Options (…) → MCP Servers*) so Antigravity re-reads the config.

### When to use local

The local package runs the MCP server as a subprocess on your machine instead of calling Postman's hosted endpoint. Prefer it when you want to pin a specific server version, run fully offline against the package, or avoid the browser-based OAuth flow in headless/CI environments.

**Prerequisites**

- **Node.js / `npx`** available on your `PATH` (`npx` fetches `@postman/postman-mcp-server@latest` on first run).
- **`POSTMAN_API_KEY`** exported in your environment — the local package has no OAuth, so it authenticates with an API key:

  ```bash
  export POSTMAN_API_KEY=your-api-key-here   # add to ~/.zshrc or ~/.bashrc to persist
  ```

  Get a key at [postman.postman.co/settings/me/api-keys](https://postman.postman.co/settings/me/api-keys). The spawned stdio process inherits your shell environment; the `$POSTMAN_API_KEY` reference in the config is also expanded by Antigravity. Never hardcode the key into `mcp_config.json`.

Running `/postman:use-local` produces a config equivalent to:

```json
{
  "mcpServers": {
    "postman": {
      "command": "npx",
      "args": ["-y", "@postman/postman-mcp-server@latest"],
      "env": {
        "POSTMAN_API_KEY": "$POSTMAN_API_KEY"
      }
    }
  }
}
```

By default the command keeps whatever toolset is already configured — it only switches the transport. Passing the optional `[mode]` argument overrides the toolset, mapping to the server's flag: `code` → `--code`, `full` → `--full`, `learn` → `--learn`, `minimal` → no flag. Add `"--region", "eu"` to `args` for an EU account (`/postman:use-local` will do this when you tell it the account is EU).

### Shipping a local-only build

If you'd rather distribute a plugin that is local from the moment it's installed (no toggle needed), publish a variant of this repo whose `mcp_config.json` is the stdio form shown above. It installs with the same one-liner — the only extra requirement is that `POSTMAN_API_KEY` is exported before use:

```bash
export POSTMAN_API_KEY=your-api-key-here
agy plugin install https://github.com/<owner>/<local-variant-repo>
```

## EU Region

If your Postman account is in the EU region, use the EU endpoint in `mcp_config.json`:

```json
{
  "mcpServers": {
    "postman": {
      "serverUrl": "https://mcp.eu.postman.com/minimal",
      "headers": {
        "Authorization": "Bearer $POSTMAN_API_KEY"
      }
    }
  }
}
```

> **Note**: OAuth is not supported for the EU region — API key authentication is required.

## Use Cases

* **API testing** — Continuously test your API using Postman collections. Run test suites, view results by endpoint, and get fix suggestions without leaving your terminal.
* **Code synchronization** — Effortlessly keep your code up to date based on changes made to your [Postman Collections](https://learning.postman.com/docs/collections/collections-overview/) and [specs](https://learning.postman.com/docs/design-apis/specifications/overview/).
* **Collection management** — Create and tag collections, update collection and request documentation, add comments, or perform actions across multiple collections without leaving your editor.
* **Workspace and environment management** — Create workspaces and environments, plus manage your environment variables.
* **Automatic spec creation** — Create specs from your code and use them to generate collections.
* **Client code generation** — Generate production-ready client code from your API definitions using the `code` toolset.

## How this differs from the Gemini CLI extension

| | Gemini CLI extension | Antigravity plugin (this repo) |
|---|---|---|
| Manifest | `gemini-extension.json` | `plugin.json` + `mcp_config.json` |
| Agent guidance | `GEMINI.md` (`contextFileName`) | `skills/postman/SKILL.md` (`rules/` are not injected in `agy`) |
| Slash commands | `commands/postman/*.toml` | `commands/postman/*.toml` (unchanged) |
| Install | `gemini extensions install <url>` | `agy plugin install <url>` |

## License

[Apache 2.0](./LICENSE)
