---
type: Web Page
title: Debugging - Model Context Protocol
description: A comprehensive guide to debugging Model Context Protocol (MCP) integrations
resource: https://modelcontextprotocol.io/docs/tools/debugging
timestamp: '2026-07-09T12:16:39.468634+00:00'
---

## Debugging tools overview

MCP provides several tools for debugging at different levels:- [MCP Inspector](/docs/tools/inspector)- [tools](/specification/latest/server/tools),- [prompts](/specification/latest/server/prompts), and- [resources](/specification/latest/server/resources), and watch the notification stream. This should be your first stop.
- **Server logging**: structured logs to stderr (stdio transport) or via- `notifications/message`
- **Client developer tools**: most MCP clients expose logs and connection state. See- [Debugging in Claude Desktop](#debugging-in-claude-desktop)below for one example, or consult your client’s documentation.

## Implementing logging

### Server-side logging

When building a server that uses the local[stdio transport](/specification/latest/basic/transports#stdio), all messages logged to stderr (standard error) will be captured by the host application automatically. For servers using the

[Streamable HTTP transport](/specification/latest/basic/transports#streamable-http), stderr is not captured by the client. Use the log message notifications below, your own server-side log aggregation, or standard HTTP tooling (curl, browser DevTools Network panel) to inspect requests,

[, and SSE streams. For all](/specification/latest/basic/transports#session-management)

`Mcp-Session-Id` headers[transports](/specification/latest/basic/transports), you can also provide logging to the client by sending a log message notification:

[RFC 5424 severity levels](/specification/latest/server/utilities/logging#log-levels)(

`debug` through `emergency`). Clients can adjust the minimum level at runtime
via the
[request. Important events to log:](/specification/latest/server/utilities/logging#setting-log-level)

`logging/setLevel`- Initialization steps
- Resource access
- Tool execution
- Error conditions
- Performance metrics

## Common issues

The examples below use Claude Desktop’s[; the same principles apply to any stdio-based MCP client.](/docs/develop/connect-local-servers)

`claude_desktop_config.json`### Working directory

When an MCP client launches a stdio server:- The working directory for servers launched via the client’s config may be
undefined (like `/`on macOS) since the client could be started from anywhere
- Always use absolute paths in your configuration and `.env`files to ensure reliable operation
- For testing servers directly via command line, the working directory will be where you run the command

`claude_desktop_config.json`, use:
`./data`
### Environment variables

MCP servers launched over stdio inherit only a limited subset of environment variables automatically (the exact set is platform-dependent). To override the default variables or provide your own, you can specify an`env` key in `claude_desktop_config.json`:
### Server initialization

Common initialization problems:- 
**Path Issues**- Incorrect server executable path
- Missing required files
- Permission problems
- Try using an absolute path for `command`
 
- 
**Configuration Errors**- Invalid JSON syntax
- Missing required fields
- Type mismatches
 
- 
**Environment Problems**- Missing environment variables
- Incorrect variable values
- Permission restrictions
 

### Connection problems

When servers fail to connect:- Check client logs
- Verify server process is running
- Test standalone with [Inspector](/docs/tools/inspector)
- Verify
[protocol compatibility](/specification/latest/basic/lifecycle#version-negotiation)
- Check
[capability negotiation](/specification/latest/basic/lifecycle#capability-negotiation): error`-32602`[sampling](/specification/latest/client/sampling)or[elicitation](/specification/latest/client/elicitation)requests to a client that hasn’t declared that capability. Inspect the`initialize`exchange

## Debugging in Claude Desktop

Claude Desktop is one of many MCP clients. It is available on macOS and Windows.### Checking server status

Click the “Add files, connectors, and more” plus icon in the chat input, then hover over the**Connectors**menu to see connected servers and available tools.

### Viewing logs

Log files are written to:- macOS: `~/Library/Logs/Claude`
- Windows: `%APPDATA%\Claude\logs`

- Server connection events
- Configuration issues
- Runtime errors
- Message exchanges

### Using Chrome DevTools

Access Chrome’s developer tools inside Claude Desktop to investigate client-side errors:- Create a `developer_settings.json`file with`allowDevTools`set to true:

- Open DevTools: `Command-Option-I`(macOS) or`Ctrl+Alt+I`(Windows)

- Main content window
- App title bar window

- Message payloads
- Connection timing

## Debugging workflow

### Development cycle

- 
Initial Development
- Use [Inspector](/docs/tools/inspector)for basic testing
- Implement core functionality
- Add logging points
 
- Use 
- 
Integration Testing
- Test in your target MCP client
- Monitor logs
- Check error handling
 

### Testing changes

To test changes efficiently:- **Configuration changes**: Restart the MCP client
- **Server code changes**: Restart the client (for Claude Desktop, fully quit and reopen; closing the window is not enough)
- **Quick iteration**: Use- [Inspector](/docs/tools/inspector)during development

## Best practices

### Logging strategy

- 
**Structured Logging**- Use consistent formats
- Include context
- Add timestamps
- Track request IDs
 
- 
**Error Handling**- Log stack traces
- Include error context
- Track error patterns
- Monitor recovery
 
- 
**Performance Tracking**- Log operation timing
- Monitor resource usage
- Track message sizes
- Measure latency
 

### Security considerations

When debugging:- 
**Sensitive Data**- Sanitize logs
- Protect credentials
- Mask personal information
 
- 
**Access Control**- Verify permissions
- Check authentication
- Monitor access patterns
 

[Security Best Practices](/docs/tutorials/security/security_best_practices).

## Getting help

When encountering issues:- 
**First Steps**- Check server logs
- Test with [Inspector](/docs/tools/inspector)
- Review configuration
- Verify environment
 
- 
**Support Channels**
- 
**Providing Information**- Log excerpts
- Configuration files
- Steps to reproduce
- Environment details
 

## Next steps

## MCP Inspector

Learn to use the MCP Inspector

## Build an MCP server

Walk through building a server from scratch

## Connect local servers

Full claude_desktop_config.json reference and troubleshooting

# Citations

1. Source page: https://modelcontextprotocol.io/docs/tools/debugging
