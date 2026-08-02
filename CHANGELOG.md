# Changelog

All notable changes to this project are documented here. This project adheres
to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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

[1.0.0]: https://github.com/postmanlabs/postman-antigravity-cli-extension/releases/tag/v1.0.0
