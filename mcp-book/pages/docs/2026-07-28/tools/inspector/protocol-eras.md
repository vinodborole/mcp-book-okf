---
type: Web Page
title: Protocol eras - Model Context Protocol
description: How the Inspector negotiates legacy vs. modern MCP, and how every feature
  is handled between protocol eras
resource: https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector/protocol-eras
timestamp: '2026-08-03T09:44:29.575770+00:00'
---

**protocol era**(legacy or modern, meaning before or as of that revision) as a first-class, per-server setting, orthogonal to the transport: the same HTTP URL can be inspected as a legacy server or as a modern one. Several tabs render meaningfully different UI and traffic depending on which era is in effect.

## The `Protocol Era` setting

Each server carries a `protocolEra` of `legacy`, `auto`, or `modern`. In the web client it lives in **Server Settings**; in a catalog or config file it is the

`protocolEra` field; in the CLI and TUI it comes from that same file.
**Why**A debugging tool must not auto-probe. A

`legacy` is the default, and not `auto`.`server/discover` probe stalls against silent legacy stdio
servers, and it pollutes the recorded transcript you came here to read. Opting
into `auto` or `modern` is a deliberate act, so what you see in the Protocol
tab is what your server would have seen from a client behaving the way you
configured.
**Connection Info**. On a modern connection,

`server/discover` also supplies `capabilities` (including `extensions`), `instructions`, and the list of `supportedVersions`. The server’s name and version arrive in the result `_meta` under `io.modelcontextprotocol/serverInfo`.
## Reproducing each era locally

Every section below ends with a
**Reproduce with …**pointer to a JSON config for one of the

**composable test servers**shipped in the Inspector repository. Clone the repo, build the test servers, then point the Inspector at the config the section names.

## Logging

- Legacy
- Modern

Logging is 

**session-scoped**. The client sends`logging/setLevel` once, and the server emits `notifications/message` at or above that level for the rest of the session.The **Logs**tab shows a**Set Active Level**selector plus a**Set**button. Choose a level, click Set, and subsequent server logs stream into the panel.Reproduce with`test-servers/configs/logging-legacy-http.json`.
## Resource subscriptions

- Legacy
- Modern

Clicking 

**Subscribe**on a resource sends`resources/subscribe`. The Subscriptions section lists the URI with no stream chrome. When the resource changes, the server emits `notifications/resources/updated` and the subscribed tile’s last-updated time is stamped.Reproduce with `test-servers/configs/subscriptions-legacy-http.json`, which also serves an `update_resource` tool so you can drive the notification round-trip yourself.
## Tasks

Tasks change the most between protocol eras, including
*how the Inspector UI tab is gated*.

- Legacy
- Modern

The 

**Tasks**tab appears when the server advertises`capabilities.tasks`. Run a tool with **Run as task**enabled and the tab lists it, populated by`tasks/list` and polled with `tasks/get`. The completed payload is fetched with a **blocking**, and`tasks/result`**Cancel**sends`tasks/cancel`.Reproduce with `test-servers/configs/tasks-legacy-http.json`.
## Multi-round tool results (MRTR)

On the modern era a tool can return`input_required` instead of a final result, embedding an [elicitation](/specification/draft/client/elicitation), a

[sampling](/specification/draft/client/sampling)request, or a

[request. The client answers that embedded request and retries the](/specification/draft/client/roots)

`roots/list``tools/call` under a fresh JSON-RPC id until the call reaches `complete`.
The Inspector drives MRTR **manually**, so each round pauses at the

**pending-request modal**, tagged

`input_required`, for you to answer. The Protocol view groups the whole exchange as one MRTR conversation rather than as unrelated calls.
`test-servers/configs/mrtr-showcase-http.json` bundles every shape in one modern server:
The legacy 

`collect_elicitation` pattern (a server calling
`server.elicitInput`) **errors**on a 2026-07-28 connection, because server-to-client requests aren’t allowed there. MRTR is its modern replacement.
## Tools: mirrored headers and excluded tools

[SEP-2243](/seps/2243-http-standardization)lets a tool annotate an argument with

`x-mcp-header`, asking a Streamable HTTP client to mirror that argument’s value into an `Mcp-Param-*` request header.
The Inspector surfaces both halves of that contract in the **Tools**tab:

- A tool with a **valid** annotation shows a**“Mirrored request headers (SEP-2243)”** section in its detail panel, for example`city -> Mcp-Param-City` .
- A tool with an **invalid** annotation (say, a header name of`"Bad Header"` , where the space makes it an invalid RFC 9110 token) appears struck through in the sidebar under an**“Excluded (SEP-2243)”** divider, with the reason on hover. A conforming client MUST drop such a tool from`tools/list` ; the Inspector shows you*why* it was dropped instead of silently hiding it.

`test-servers/configs/xmcpheader-modern-http.json`.
### `-32602` error panels

Under the modern era a `tools/call` that rejects with `-32602` renders as a distinct **error panel**:

- **Unknown Tool** : when the message names a tool the server does not list. Reproduce by calling any name absent from the server’s`tools/list` .
- **Invalid Parameters** : any other`-32602` . Reproduce with the`trigger_invalid_params` tool in the config above.

`-32602`; only the Inspector’s presentation changes. On a legacy connection you get one generic JSON-RPC failure and have to read the message to tell which case you hit.
## Network and Protocol: headers and the error taxonomy

The modern era standardizes a set of`Mcp-*` HTTP headers and introduces a richer JSON-RPC error taxonomy (
[SEP-2243](/seps/2243-http-standardization)/

[SEP-2575](/seps/2575-stateless-mcp)). The two monitoring tabs divide the work:

- The **Network** tab is the HTTP view: mirrored`Mcp-*` headers are highlighted and sentinel values decoded.
- The **Protocol** tab is the JSON-RPC view: each spec error renders distinctly rather than as a generic failure.

`test-servers/configs/modern-network-http.json` serves four tools that produce a real HTTP status plus a JSON-RPC error body, one per class:
## Sessions

A legacy Streamable HTTP connection may carry a server-assigned session id (`Mcp-Session-Id`), which the client tears down with an HTTP `DELETE`. A modern connection is **sessionless and per-request**: with no session id the client SDK sends no

`DELETE` to the server, so disconnect is purely local.
This has a practical consequence for your own test servers. A stateless modern handler constructed per request cannot hold state between calls, which is why `test-servers/configs/subscriptions-modern-http.json`, unlike its legacy counterpart, omits an `update_resource` tool: the mutation would run against a throwaway server instance and be invisible to the next read.

# Citations

1. Source page: https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector/protocol-eras
