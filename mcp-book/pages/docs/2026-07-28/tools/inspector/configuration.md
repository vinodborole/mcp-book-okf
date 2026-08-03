---
type: Web Page
title: Configuration and flags - Model Context Protocol
description: Catalog vs. config files, which client owns which flag, and every environment
  variable
resource: https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector/configuration
timestamp: '2026-08-03T09:44:29.575770+00:00'
---

`mcp-inspector` binary is a launcher: it reads two flags of its own and forwards every other argument to one of three clients (web, CLI, or TUI). Each client defines its own flags, so a flag that works in one can be unknown to another (`--method`, for example, is CLI-only). This page groups flags and environment variables by the client that owns them.
## The launcher owns exactly two things

Everything below belongs to a client.

## Choosing servers

### `--catalog` vs. `--config`

All three clients resolve `--catalog` and `--config` through the same shared code, so each flag behaves the same in the web app, the CLI, and the TUI. Where the two differ from each other is the table below.
The two are 

**mutually exclusive**, and neither combines with an ad-hoc target. Passing both is rejected identically by all three clients.

**What a freshly seeded catalog contains depends on the client.**The web backend seeds two sample servers, so a first launch has something to connect to immediately:

`{ "mcpServers": {} }` instead: they are non-interactive or list-driven, so sample entries would be noise rather than a starting point.Either way, seeding happens only when the file does not exist yet, and a read-only `--config` is never seeded at all.`--config` is what you want when pointing the Inspector at a config file you
didn’t write: a coworker’s, a client application’s, or one checked into a
repo. It guarantees the Inspector will not touch the file.
### Ad-hoc targets

Instead of a file you can name one server directly, either as a positional command (stdio) or a URL:
### Shared server-selection flags

Defined
**separately by each of web, CLI, and TUI**, so they’re available in all three, with the divergences noted:

### The `--` separator

The **web and CLI**clients split their arguments at a bare

`--` and pass everything after it to the target command as its own arguments. This is how you pass a flag that the Inspector would otherwise eat:
`--config` would be read as the Inspector’s own read-only-session flag.
## Web-only flags

## CLI and TUI: OAuth client flags

These five are defined by the
**CLI and TUI**only. The web client obtains the same settings through its Client Settings dialog.

## CLI-only flags

The whole scripting surface belongs to the CLI. See
[CLI client](/docs/2026-07-28/tools/inspector/cli)for usage.

## Environment variables

Environment variables split the same way as flags: two are read by the launcher itself, and the rest belong to the CLI and TUI or to the web backend.
### Read by the launcher

### CLI and TUI

### Web backend environment variables

## Catalog file format

A catalog or config file is the familiar MCP client config shape (a`mcpServers` object) with per-server Inspector settings alongside:
`protocolEra` (see [Protocol eras](/docs/2026-07-28/tools/inspector/protocol-eras)) defaults to

`legacy` and `modernLogLevel` to `debug`.
You do not have to hand-write these; the web client can [import an existing client config](/docs/2026-07-28/tools/inspector/recipes#importing-an-existing-client-config)from Claude Desktop, Cursor, Cline, or VS Code, or a registry

`server.json`.

# Citations

1. Source page: https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector/configuration
