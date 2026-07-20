---
type: Web Page
title: Build with Agent Skills - Model Context Protocol
description: Use agent skills to guide AI coding assistants through MCP server design
  and implementation
resource: https://modelcontextprotocol.io/docs/develop/build-with-agent-skills
timestamp: '2026-07-20T08:58:19.184996+00:00'
---

[Agent skills](https://agentskills.io/home)are portable instruction sets that give AI coding assistants domain knowledge for a task. For MCP development, they encode the design decisions (deployment model, tool patterns, auth) so your agent can interrogate your use case and scaffold a server that fits.

## Available skills

A reference set of MCP development skills is available as the[. It provides three composing skills:](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/mcp-server-dev)

`mcp-server-dev` plugin
Each skill ships a 

`SKILL.md` file plus a `references/` folder of supporting
material (auth flows, tool-design patterns, widget templates, manifest schemas)
that the agent reads on demand. The files follow the open format and work with
any agent that implements the standard. For example, to install them in Claude
Code:
[skill directories](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/mcp-server-dev/skills)(

`SKILL.md` plus `references/`) into your agent’s skills location.
## Start a build

With the skills installed, ask your agent to help you build an MCP server. The entry skill triggers on natural-language requests, or you can invoke it directly using your agent’s skill-invocation syntax. The skill runs a short discovery phase before writing any code. Expect questions about:- **What it connects to**— a cloud API, a local process, the filesystem, hardware
- **Who will use it**— just you, your team, or anyone who installs it
- **Action surface size**— a handful of operations versus wrapping a large API
- **User interaction needs**— plain text results, structured input via- [elicitation](/specification/draft/client/elicitation), or rich UI widgets
- **Upstream auth**— API keys, OAuth 2.0, or none

## Deployment paths

Based on discovery, the skill recommends one of four paths and scaffolds accordingly:**Remote**is the default for anything wrapping a cloud API. Zero install friction, one deployment serves all users, and OAuth flows work properly because the server can handle redirects and token storage. The reference skill includes scaffolds for Cloudflare Workers and portable Express/FastMCP setups.

[Streamable HTTP](/specification/draft/basic/transports/streamable-http)**extend a server with interactive widgets rendered in chat, such as searchable pickers, charts, and live dashboards. The skill hands off to**

[MCP apps](/extensions/apps/overview)`build-mcp-app` when
[elicitation’s](/specification/draft/client/elicitation)flat-form constraints don’t fit.

**package a local server together with its runtime as a single**

[MCP Bundles (MCPB)](https://github.com/modelcontextprotocol/mcpb)`.mcpb` archive, so users
can install it without setting up Node or Python. Use this path when the server
must touch the user’s machine: reading local files, driving desktop apps, or
talking to localhost services. The skill hands off to `build-mcpb`.
**Local**remains available for prototyping, with a noted upgrade path to MCPB when you’re ready to distribute.

[stdio](/specification/draft/basic/transports/stdio)## Next steps

Once your agent scaffolds the server, iterate on tool descriptions and error handling, then test and ship:## MCP Inspector

Test your server’s tools, resources, and prompts interactively

## Connect to a client

Wire your server into an MCP client via local or remote configuration

## Publish to the Registry

Make your server discoverable in the MCP Registry

# Citations

1. Source page: https://modelcontextprotocol.io/docs/develop/build-with-agent-skills
