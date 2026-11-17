# BUILD REPORT — spinspots

Clean-room rebuild of `com.leagueofspin/spots` v1.0.0
(remote-only MCP server at `https://leagueofspin.com/api/mcp`), built
2026-10-28 as a forwarding client. Package: `@reneturcios/spinspots`
0.1.0, registry name `io.github.CloudCorpRecords/spinspots`.

## Surface built

Full surface: **3 of 3 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `find-spots` | `lat`, `lng` |
| `get-spot` | `slugOrId` |
| `playing-conditions` | `lat`, `lng` |

Each served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured (no `title` key on any tool — the
capture carries none — and `additionalProperties: false` on every
inputSchema, matching the fixture). The upstream tool names carry
HYPHENS (`find-spots`, `get-spot`, `playing-conditions`) and are served
verbatim. The tool literals were generated programmatically from
`harness/fixtures/leagueofspin-tools-original.json` by the script at
`/tmp/laneb4-spinspots-scratch/gen.py` (parse + re-serialise,
placeholder substitution into `src/forward.ts`) — never retyped — and
a post-generation check (the registration-integrity test suite, plus
the parity harness) confirmed the served literals deep-equal the
fixture's 3 tool objects, in order. Tools are registered via the MCP
SDK low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names,
descriptions, required fields (`lat`/`lng` for find-spots and
playing-conditions, `slugOrId` for get-spot) and property shapes.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://leagueofspin.com/api/mcp`.
- Env override `LEAGUEOFSPIN_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header is
  sent under any environment — including with key-like env vars
  (`LEAGUEOFSPIN_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and that
  no `X-API-Key` header is ever sent; upstream requests carry only
  `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `spinspots error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests, and no live call is
  claimed beyond the coordinator's tools/list capture.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3), then `npm ci --ignore-scripts --offline` to populate
  `node_modules`.
- Parses as JSON; `name` == `@reneturcios/spinspots`.
- Package entries: **128**.
- sha256: `05a56e58443690622d84a9f0e46f9b65daeca5c4d3b3adab64e5439107a5b0f4`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Outdoor table-tennis spots near a coordinate,
  spot details, and playing conditions." — **83 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/spinspots:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/spinspots"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/spinspots`
  and its smoke test asserts the server name `spinspots`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **53 tests, 53 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 3
   tools in order, plus session handling/reuse, env override, the
   no-Authorization-header assertions under several environments,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour including `--version` printing
   `0.1.0`).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/leagueofspin-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/spinspots/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/spinspots/parity-out` → **verdict PASS, parity 100.0**
   (3/3 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
