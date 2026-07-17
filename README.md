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
      "url": "https://mcp.postman.com/minimal",
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

The plugin ships four `/postman:*` commands (defined in `commands/postman/*.toml`):

| Command | What it does |
|---------|--------------|
| `/postman:setup` | Initialize a Postman workspace, collection, and environment for the current project |
| `/postman:sync` | Update the collection to reflect the current API code, then re-run tests |
| `/postman:run` | Run the project's collection and show test results by endpoint |
| `/postman:generate <spec>` | Generate a collection from an OpenAPI spec file |

Agent guidance (collection-schema rules, workflow patterns, troubleshooting) is loaded on demand from `skills/postman/SKILL.md`.

## Tool Configuration

This extension uses the **minimal** toolset by default — fast, focused access to core Postman operations. To use a different mode, update the URL in `mcp_config.json`:

| Mode | URL | Use when |
|------|-----|----------|
| Minimal (default) | `https://mcp.postman.com/minimal` | Collections, workspaces, environments, specs |
| Full | `https://mcp.postman.com/mcp` | 100+ tools, advanced collaboration, Enterprise |
| Code | `https://mcp.postman.com/code` | API search and client code generation |

## EU Region

If your Postman account is in the EU region, use the EU endpoint in `mcp_config.json`:

```json
{
  "mcpServers": {
    "postman": {
      "url": "https://mcp.eu.postman.com/minimal",
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
