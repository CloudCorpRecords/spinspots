# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted League of Spin MCP server. The three captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `find-spots`, `get-spot`, `playing-conditions` — the full captured surface, with the upstream hyphenated names served verbatim.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `spinspots error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `LEAGUEOFSPIN_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
