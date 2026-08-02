# Changelog

All notable changes to this project are documented here. This project adheres
to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-08-02

Aligns the plugin to the canonical **18-command** set shared with the Postman
Claude Code and Cursor plugins (cross-plugin command parity).

### Changed
- Renamed `/postman:generate` → `/postman:generate-spec` and reconciled its
  intent with the canonical command: it now generates or updates an OpenAPI 3.0
  spec by analyzing the API routes in your codebase (and can optionally sync
  the spec to Postman), rather than only generating a collection from a spec.
- Renamed `/postman:run` → `/postman:run-collection` (same behavior: runs the
  project's collection and reports results by endpoint).

### Added
- Four Postman Flows commands: `/postman:deploy-flow`, `/postman:get-flow-run`,
  `/postman:list-flows`, and `/postman:trigger-flow`. These drive the Postman
  CLI (`postman flows ...`) via Bash and require the Postman CLI to be installed
  and `postman login` to be active — the same dependency as
  `/postman:send-request` (they are independent of the MCP endpoint/mode).
- `/postman:generate-client` — generate typed client **code** from a Postman
  collection (the inverse of `/postman:generate-spec`). MCP-tool-driven; the
  codegen tools (`getCodeGenerationInstructions` and the detailed
  `getCollection*` tools) require **Code** (`mcp.postman.com/code`) or **Full**
  (`mcp.postman.com/mcp`) mode — switch with `/postman:use-remote code`.

## [1.0.0] - 2026-08-02

First stable release of the Postman extension for Google Antigravity (`agy`).

### Added
- `plugin.json` manifest + `mcp_config.json` pointing at Postman's hosted MCP
  server (`https://mcp.postman.com/minimal`) with OAuth authentication.
- Thirteen `/postman:*` slash commands: `setup`, `sync`, `run`, `generate`,
  `test`, `search`, `mock`, `docs`, `security`, `learn`, `send-request`,
  `use-local`, and `use-remote`.
- `/postman:use-local` and `/postman:use-remote` transport-toggle commands that
  rewrite `mcp_config.json` in place while preserving the current toolset.
- Agent guidance in `skills/postman/SKILL.md`.
- Release pipeline: GitHub Actions workflows to validate the plugin layout on
  every push/PR and publish a GitHub release on `v*.*.*` tags.

[1.1.0]: https://github.com/postmanlabs/postman-antigravity-cli-extension/releases/tag/v1.1.0
[1.0.0]: https://github.com/postmanlabs/postman-antigravity-cli-extension/releases/tag/v1.0.0
