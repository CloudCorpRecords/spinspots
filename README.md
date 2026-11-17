# spinspots

Outdoor table tennis lives or dies by the weather and by knowing where the
tables are. spinspots is a small local bridge to the League of Spin directory:
it answers three questions — which outdoor spots sit near a coordinate, what
does one particular spot look like (with its neighbours), and is it actually
playable outside right now, given the wind, temperature and rain?

The package holds no spot data itself. It is a forwarding client for the
hosted League of Spin MCP server. The tool list is served locally from a
verbatim capture of the upstream surface; every tool call is then forwarded
upstream over a streamable-HTTP JSON-RPC session and the upstream result is
handed back exactly as received.

## Install and run

```bash
npx -y @reneturcios/spinspots mcp        # stdio MCP server
npx -y @reneturcios/spinspots --version  # 0.1.0
```

One bin serves both the MCP server and a small CLI:

```bash
spinspots find-spots '{"lat": 37.7749, "lng": -122.4194, "radiusKm": 10}'
spinspots get-spot '{"slugOrId": "some-spot-slug"}'
spinspots playing-conditions '{"lat": 37.7749, "lng": -122.4194}'
spinspots mcp          # run the stdio MCP server
spinspots --version
```

Exit codes for tool subcommands: 0 on success, 1 when the forwarded result
is an error result, 2 on a usage error.

## MCP client configuration

```json
{
  "mcpServers": {
    "spinspots": {
      "command": "npx",
      "args": ["-y", "@reneturcios/spinspots", "mcp"]
    }
  }
}
```

## Docker

```bash
docker run -i ghcr.io/cloudcorprecords/spinspots          # stdio MCP server (default command: mcp)
docker run --rm ghcr.io/cloudcorprecords/spinspots --version
```

## Tools

| Tool | Arguments | What it returns |
|------|-----------|-----------------|
| `find-spots` | `lat`, `lng` (required); optional `radiusKm`, `limit` | The nearest outdoor spots to the coordinate, each with its distance and a citable url. |
| `get-spot` | `slugOrId` (required) | One spot's details plus nearby spots, addressed by the slug from a leagueofspin.com/spots/{slug} url (preferred) or by raw id. |
| `playing-conditions` | `lat`, `lng` (required) | Current wind, temperature and precipitation for the location. Wind is the decisive signal for outdoor table tennis. |

The upstream tool names carry hyphens and are served verbatim.

## How forwarding works

- Upstream endpoint: `https://leagueofspin.com/api/mcp`. It can be replaced
  with the `LEAGUEOFSPIN_MCP_URL` environment variable (a labelled
  reconstruction of this rebuild).
- Each call opens (once, lazily) a streamable-HTTP JSON-RPC session:
  `initialize`, then `notifications/initialized`, then `tools/call`. When the
  upstream issues an `Mcp-Session-Id`, later requests carry it. Plain-JSON
  and SSE-framed replies are both understood.
- The upstream registry record declares no headers, so no `Authorization`
  header (or any API-key header) is ever sent, under any environment.
- Upstream results pass through untouched, `isError` flag included. When the
  upstream itself fails — a JSON-RPC error, a non-2xx status, an unparseable
  body, a thrown or timed-out fetch — the result is an error result whose
  text begins with `spinspots error:`.

## Development

```bash
npm install
npm run build   # tsc -> dist/
npm test        # tsx --test test/*.test.ts (stubbed upstream, no network)
npm run dev     # run the CLI from source
```

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
