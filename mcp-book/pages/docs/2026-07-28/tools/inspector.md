---
type: Web Page
title: MCP Inspector - Model Context Protocol
description: Interactive developer tooling for testing and debugging MCP servers,
  in the browser, on the command line, and in the terminal
resource: https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector
timestamp: '2026-08-03T09:44:29.575770+00:00'
---

[MCP Inspector](https://github.com/modelcontextprotocol/inspector)is the reference developer tool for testing and debugging

[MCP servers](/docs/2026-07-28/learn/server-concepts). It ships as a single package,

`@modelcontextprotocol/inspector`, providing **three clients behind one binary**:

All three are built on the same shared core, so a connection behaves identically across them: the same transports, the same configuration files, the same OAuth state on disk, and the same 

[protocol-era](/docs/2026-07-28/tools/inspector/protocol-eras)negotiation (legacy vs. modern 2026-07-28).

## Quickstart

The Inspector requires
**Node 22.19.0 or newer**and runs directly through

`npx`. No installation is required:
- Web
- CLI
- TUI

[Web client](/docs/2026-07-28/tools/inspector/web).

### Inspecting published servers

Pass the command that launches the server as the Inspector’s arguments, or point it at a remote server with`--server-url`:
- npm package
- PyPI package
- Remote HTTP server

## Launcher flags vs. client flags

`mcp-inspector`, the binary that `npx @modelcontextprotocol/inspector` runs, is a thin launcher. It owns only two things:
1. **The mode flag:**`--web` (default),`--cli` , or`--tui` . At most one; passing two errors with`Specify at most one of --web, --cli, or --tui.`
2. **`-h` / `--help`.**

`--catalog`, `--config`, `--server-url`, `--transport`, `--method`, the OAuth flags) is defined by the *client*, not the launcher, and the clients do not all define the same set. The

[Configuration and flags](/docs/2026-07-28/tools/inspector/configuration)page is organized that way, by owner.

Mode flags are recognized only at the front of the command line: the first token that isn’t 

`--web` / `--cli` / `--tui` ends launcher parsing, and everything after it is forwarded to the client unchanged. That’s what lets a literal `--cli` appear later as one of your server’s own arguments:`--help` behaves differently with and without a mode flag. Bare `mcp-inspector   --help` prints the launcher’s help and exits. With a mode flag it is
forwarded, so `mcp-inspector --cli --help` prints the CLI’s full flag
reference instead.
## Where to go next

## Web client

A tab-by-tab walkthrough of the graphical inspector.

## CLI client

Method reference, output formats, exit codes, and CI recipes.

## TUI client

Terminal navigation and keyboard reference.

## Configuration and flags

Catalog vs. config files, the full per-client flag reference, and
environment variables.

## Authorization

The OAuth flow end to end, mid-session re-authorization, and loopback
callbacks.

## Protocol eras

Legacy vs. modern (2026-07-28) operation, and how every tab changes between
protocol eras.

## Recipes

Importing client configs, reviewing MCP Apps, Docker, and network hosting.

## Debugging guide

Broader debugging strategies beyond the Inspector.

# Citations

1. Source page: https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector
