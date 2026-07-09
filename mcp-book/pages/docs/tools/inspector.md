---
type: Web Page
title: MCP Inspector - Model Context Protocol
description: In-depth guide to using the MCP Inspector for testing and debugging Model
  Context Protocol servers
resource: https://modelcontextprotocol.io/docs/tools/inspector
timestamp: '2026-07-09T12:16:39.468634+00:00'
---

[MCP Inspector](https://github.com/modelcontextprotocol/inspector)is an interactive developer tool for testing and debugging MCP servers. While the

[Debugging Guide](/docs/tools/debugging)covers the Inspector as part of the overall debugging toolkit, this document provides a detailed exploration of the Inspector’s features and capabilities.

## Getting started

### Installation and basic usage

The Inspector runs directly through`npx` without requiring installation:
#### Inspecting servers from npm or PyPI

A common way to start server packages from[npm](https://npmjs.com)or

[PyPI](https://pypi.org).

- npm package
- PyPI package

#### Inspecting locally developed servers

To inspect servers locally developed or downloaded as a repository, the most common way is:- TypeScript
- Python

## Feature overview

### Server connection pane

- Allows selecting the [transport](/specification/latest/basic/transports)for connecting to the server
- For local servers, supports customizing the command-line arguments and environment

### Resources tab

- Lists all available resources
- Shows resource metadata (MIME types, descriptions)
- Allows resource content inspection
- Supports subscription testing

### Prompts tab

- Displays available prompt templates
- Shows prompt arguments and descriptions
- Enables prompt testing with custom arguments
- Previews generated messages

### Tools tab

- Lists available tools
- Shows tool schemas and descriptions
- Enables tool testing with custom inputs
- Displays tool execution results

### Notifications pane

- Presents all logs recorded from the server
- Shows notifications received from the server

## Best practices

### Development workflow

- 
Start Development
- Launch Inspector with your server
- Verify basic connectivity
- Check capability negotiation
 
- 
Iterative testing
- Make server changes
- Rebuild the server
- Reconnect the Inspector
- Test affected features
- Monitor messages
 
- 
Test edge cases
- Invalid inputs
- Missing prompt arguments
- Concurrent operations
- Verify error handling and error responses
 

## Next steps

## Inspector Repository

Check out the MCP Inspector source code

## Debugging Guide

Learn about broader debugging strategies

# Citations

1. Source page: https://modelcontextprotocol.io/docs/tools/inspector
