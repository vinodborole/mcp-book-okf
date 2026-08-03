---
type: Web Page
title: TUI client - Model Context Protocol
description: 'The terminal MCP Inspector: navigation, tabs, and keyboard reference'
resource: https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector/tui
timestamp: '2026-08-03T09:44:29.575770+00:00'
---

## Choosing servers

Unlike the CLI, the TUI has no`--server <name>` flag for picking one entry: it reads its servers from a catalog or config file, loads every server in it, and lets you pick from an on-screen list:
`--catalog` nor `--config`, and no [ad-hoc target](/docs/2026-07-28/tools/inspector/configuration#ad-hoc-targets), it uses the default writable catalog

`~/.mcp-inspector/mcp.json`. See [Configuration and flags](/docs/2026-07-28/tools/inspector/configuration).

## Tabs

The accelerators avoid collisions rather than always taking the first letter: 

**P**rotocol takes

`p` so Pro
**m**pts takes

`m`, and **C**onsole takes

`o` because `c` is the global Connect action.
## Navigation

## Authorizing an HTTP server

1. Select an HTTP or SSE server and press **`c`** to connect.
2. If the server requires authorization, the TUI starts OAuth automatically and opens the authorization URL in a browser.
3. When the browser redirect lands on the TUI’s loopback listener, the connection finishes on its own, with no second **`c`** .
4. Use the **Auth** tab to inspect the resulting OAuth state, or to clear it.

`http://127.0.0.1:6276/oauth/callback`. The port is fixed on purpose: a pre-registered (static) OAuth client, a [Client ID Metadata Document (CIMD)](/specification/latest/basic/authorization/client-registration#client-id-metadata-documents), or an enterprise-managed IdP all need a redirect URI known in advance. Register that URI once and it works across sessions. On a remote host where your browser is on another machine, forward the callback port so the redirect reaches this listener; see

[Callback URLs](/docs/2026-07-28/tools/inspector/authorization#callback-urls). The trade-off is that only one TUI OAuth flow can hold the port at a time; a second concurrent flow fails with

`EADDRINUSE`. To override it, pass `--callback-url` or set `MCP_OAUTH_CALLBACK_URL`: use a different fixed port per instance, or `http://127.0.0.1:0/oauth/callback` for an OS-assigned ephemeral port when your authorization server registers redirect URIs dynamically.
Per-server OAuth fields in the catalog (static client id/secret, scopes, the enterprise-managed flag) are applied automatically. Install-wide settings (CIMD, enterprise IdP) come from `~/.mcp-inspector/storage/client.json`, the same file the web client’s **Client Settings**dialog writes. Point at a different one with

`--client-config` or `MCP_CLIENT_CONFIG_PATH`.
See [Authorization](/docs/2026-07-28/tools/inspector/authorization)for the full picture.

## Requirements

The TUI needs a real TTY with raw-mode support. It will not run usefully in a headless CI job; use the
[CLI](/docs/2026-07-28/tools/inspector/cli)there.

# Citations

1. Source page: https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector/tui
