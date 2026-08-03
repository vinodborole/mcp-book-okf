---
type: Web Page
title: CLI client - Model Context Protocol
description: 'Scripting the MCP Inspector: methods, output formats, exit codes, and
  CI recipes'
resource: https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector/cli
timestamp: '2026-08-03T09:44:29.575770+00:00'
---

`--method`, prints the result, and exits. That makes it a good fit for CI pipelines, shell one-liners, and coding agents that need to verify a server change immediately.
`mcp-inspector` binary. Without a global install, prefix each command with `npx @modelcontextprotocol/inspector` instead, as above.
## Choosing a server

The CLI accepts a positional command (stdio), a`--server-url` (HTTP/SSE), or a named server out of a catalog or config file:
[protocol era](/docs/2026-07-28/tools/inspector/protocol-eras), and roots) apply to the connection, resolved exactly as the TUI and web client resolve them. A

`--header` flag overrides the file’s headers for that run while leaving its timeouts and OAuth in place.
Later examples abbreviate whichever of these forms you use, along with its `--transport` or `--config`/`--server` flags, as `<server>`.
**The config file is the only durable way to give a run its**there is no roots flag, and

[roots](/specification/draft/client/roots):`--method roots/set` applies only to that one short-lived connection. Roots
configured for a server are advertised at connect, so a server that calls
`roots/list` (as `@modelcontextprotocol/server-filesystem` does, to learn its
allowed directories) gets them.
[Configuration and flags](/docs/2026-07-28/tools/inspector/configuration)for

`--catalog` vs. `--config`, the `--` separator, and the shared server-selection flags.
## Methods

Stream- or session-only methods (

`logging/tail`, for example) are rejected, since a process that exits can’t hold a stream open.
### Passing arguments

`--tool-arg` takes `key=value` and **coerces**values by JSON-parsing them, so

`count=1` becomes a number and `"012"` becomes `12`:
`--tool-args-json` takes the whole argument object at once and passes it **verbatim**, with no coercion, so

`"012"` stays the string `012`. The two are mutually exclusive:
## Output

`--format text` (the default) pretty-prints for humans. `--format json` emits a single JSON object on stdout with no banners, so the whole output pipes cleanly:
## Probing MCP Apps

`--app-info` reports whether a tool ships an [MCP App](/extensions/apps/overview)UI (its

`ui://` resource, CSP, and permissions) **without calling the tool**, so a pipeline can decide whether it needs a browser before invoking anything:

`0`, one with no app exits `2`, and a missing tool exits `5`, so a typo isn’t mistaken for “no app”. A probe failure (unreadable UI resource, malformed `resourceUri`) is reported in a `resourceError` field rather than aborting, so one bad tool never kills a whole listing.
`tools/list --app-info` always emits NDJSON (one line per tool) regardless of
`--format`; `--format json` reshapes only the single-tool output of
`tools/call --app-info`.
## Exit codes and error envelopes

Every non-zero exit maps to a stable failure class, so a caller can branch on
*why*without scraping prose:

On any non-zero exit the CLI also writes a 

**single JSON line to stderr**:

`2>&1 | tail -1 | jq .error`.
A `tools/call` that returns `isError: true` still prints its payload, but exits `5`, so an `&&` chain doesn’t proceed on a failed call.
## Authorization in scripts

By default the CLI runs the same loopback OAuth flow as the TUI: it opens a browser and waits on a localhost callback that a CI job can’t complete. Two flags make non-interactive runs predictable:
- `--stored-auth-only` : never start interactive OAuth or step-up, and never auto-open a browser. Use tokens from the shared store if present, otherwise fail immediately with`auth_required` . This is the flag CI wants.
- `--use-stored-auth` : reuse a token that the web Inspector already obtained on this machine, refreshing it first when a refresh token is stored.

`auth_required` rather than hanging for fifteen minutes on a callback nobody will complete.
See [Authorization](/docs/2026-07-28/tools/inspector/authorization)for the full flow, the web-to-CLI handoff, and

`--print-handoff`.
## Recipes

### Verify a server in CI

### Branch on the failure class

### Smoke-test every tool that has a UI

### Inspect a catalog without connecting

## Proxies

Connections to remote HTTP/SSE servers honor the conventional proxy variables:`HTTPS_PROXY` / `HTTP_PROXY` (and their lowercase forms) select the proxy and `NO_PROXY` exempts hosts. No Inspector-specific flag is needed, and the proxy agent is loaded lazily, so runs without a proxy pay nothing. The same applies to the web client’s backend.

# Citations

1. Source page: https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector/cli
