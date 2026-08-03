---
type: Web Page
title: Recipes - Model Context Protocol
description: Practical guides for transports, importing configs, reviewing MCP Apps,
  Docker, and network hosting
resource: https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector/recipes
timestamp: '2026-08-03T09:44:29.575770+00:00'
---

## Connecting stdio vs. HTTP servers

### stdio

A stdio server is a process the Inspector spawns. Everything positional is the command line:`--` before any arguments meant for your server. Without the separator, `--verbose` would be
parsed by the Inspector and never reach the server.
Give the process environment variables with `-e` and a working directory with `--cwd`:
`stderr` lands in the **Console**tab (web) or the Console tab (

`o`, TUI), which is where most stdio servers put their diagnostics, so check there first when a connection fails for no visible reason.
### HTTP and SSE

`--transport` accepts `http` (Streamable HTTP) and `sse`. If the server is protected, see [Authorization](/docs/2026-07-28/tools/inspector/authorization): no setup is needed in advance, because when the server answers

`401` the Inspector runs the OAuth flow described there and retries the connection.
For an HTTP server, also decide its [protocol era](/docs/2026-07-28/tools/inspector/protocol-eras). The default is

`legacy`; set `modern` or `auto` in Server Settings (or `protocolEra` in the catalog file) to exercise the 2026-07-28 behavior.
## Importing an existing client config

On the Servers screen,
**Add Servers**can import MCP servers you have already configured elsewhere instead of retyping them. It parses Claude Desktop, Cursor, Cline, and VS Code client configs directly, and it also reads a server’s own

[MCP Registry](/registry/about)

`server.json`.
Import merges into the active [catalog](/docs/2026-07-28/tools/inspector/configuration#choosing-servers)(the Inspector’s writable server list), so existing entries aren’t clobbered. If you’d rather not touch your catalog at all, launch against the foreign file read-only instead:

`--config` guarantees the file is served as-is and never written, seeded, or migrated.
## Reviewing an MCP App

[MCP Apps](/extensions/apps/overview)are tools that carry a UI widget. For an automated reviewer (CI or an agent), use the CLI for every check that returns JSON, and open a browser only to inspect the rendered widget.

1

Probe the security posture without calling the tool

`0` if the tool has an app, `2` if not, so an `&&` chain short-circuits:`csp` and `permissions` (and `domain`, when the resource declares one) live on the UI **resource**rather than the tool, so

`--app-info` reads that resource. The tool is never called.
2

Get the full result payload, still with no browser

3

Launch the web Inspector once, loopback-only

`MCP_SANDBOX_PORT` matters here: the app’s UI is served from a separate sandbox port that is dynamic by default, and your automation needs a fixed address to reach it.
4

Navigate one deep link to a rendered widget

`appArgs` is the tool’s arguments as base64url-encoded JSON, and every deep-link parameter is described under [Deep links](/docs/2026-07-28/tools/inspector/web#deep-links).

`autoConnect` and `autoOpen` must both equal the session token, since `autoOpen` fires a tool call straight from the URL and needs the same gate as `autoConnect`.
5

Wait on a deterministic signal instead of sleeping

The Apps screen exposes a stable automation contract. Poll these attributes instead of sleeping:

## Docker

A container image is published to GitHub Container Registry for`linux/amd64` and `linux/arm64`:
[session token](/docs/2026-07-28/tools/inspector/web#the-session-token)from the container logs, or pin it with

`-e MCP_INSPECTOR_API_TOKEN=<value>`.
The image defaults to `--web`, bound to `0.0.0.0:6274` with browser auto-open off, and runs as a non-root user. It sets `DANGEROUSLY_BIND_ALL_INTERFACES=true` because a container must bind the wildcard address to be reachable through `-p`.
Its `HEALTHCHECK` probes the web UI, so add `--no-healthcheck` when running `--cli` or `--tui` (neither has a web server). `<target>` below is an [ad-hoc target](/docs/2026-07-28/tools/inspector/configuration#ad-hoc-targets): a positional stdio command, or

`--server-url <url> --transport http`.
## Hosting on a network

The Inspector binds`localhost` by default and its backend spawns processes, so treat exposing it to a network as a deliberate decision.
The Inspector refuses to bind the **wildcard**all-interfaces addresses (

`0.0.0.0`, `::`, and every equivalent spelling) unless you set `DANGEROUSLY_BIND_ALL_INTERFACES=true`. Binding a **specific**address is allowed with no opt-in, because that’s one deliberate exposure rather than every interface at once, which is the shape DNS-rebinding attacks target.

Two further caveats when going off loopback:

- **MCP Apps need their sandbox port reachable too.** It’s a separate, dynamic-by-default port; pin it with`MCP_SANDBOX_PORT` and expose or forward it. The Docker image publishes only`6274` .
- **MCP Apps can’t render over TLS or at a bare IPv6 literal.** The sandbox URL is always plain`http` , so an`https://` page blocks the iframe as mixed content; and a bracketed IPv6 literal isn’t a valid CSP host-source, so browse at a name or an IPv4 address.

`DANGEROUSLY_OMIT_AUTH` on anything reachable by anyone but you.
## Development workflow

A loop that works well in practice:
1

Start with the CLI

`--method initialize` confirms the server starts, handshakes, and reports
the capabilities you expect, in one second, with a machine-readable answer.
Most “it doesn’t work” turns out to be here.
2

Move to the web client for exploration

Schema-driven forms, rendered results, and the Protocol tab beside them make
it fast to find the case where a tool misbehaves.

3

Test the edges

Invalid inputs, missing required prompt arguments, concurrent calls, and,
for HTTP servers, both protocol eras. Verify the 

*errors*are as intentional as the successes.
4

Lock it in with the CLI

Turn what you found into a CI assertion: pipe the CLI’s 

`--format json`
output to `jq -e` with `--stored-auth-only`, so a missing token fails fast
instead of starting interactive OAuth. See

# Citations

1. Source page: https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector/recipes
