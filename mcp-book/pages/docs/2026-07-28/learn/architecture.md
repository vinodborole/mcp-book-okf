---
type: Web Page
title: Architecture overview - Model Context Protocol
resource: https://modelcontextprotocol.io/docs/2026-07-28/learn/architecture
timestamp: '2026-08-03T09:44:29.575770+00:00'
---

[scope](#scope)and

[core concepts](#concepts-of-mcp), and provides an

[example](#example)demonstrating each core concept. Because MCP SDKs abstract away many concerns, most developers will likely find the

[data layer protocol](#data-layer-protocol)section to be the most useful. It discusses how MCP servers can provide context to an AI application. For specific implementation details, please refer to the documentation for your

[language-specific SDK](/docs/2026-07-28/sdk).

## Scope

The Model Context Protocol includes the following projects:
- [MCP Specification](https://modelcontextprotocol.io/specification/latest) : A specification of MCP that outlines the implementation requirements for clients and servers.
- [MCP SDKs](/docs/2026-07-28/sdk) : SDKs for different programming languages that implement MCP.
- **MCP Development Tools** : Tools for developing MCP servers and clients, including the[MCP Inspector](https://github.com/modelcontextprotocol/inspector)
- [MCP Reference Server Implementations](https://github.com/modelcontextprotocol/servers) : Reference implementations of MCP servers.

MCP focuses solely on the protocol for context exchange—it does not dictate
how AI applications use LLMs or manage the provided context.

## Concepts of MCP

### Participants

MCP follows a client-server architecture where an MCP host — an AI application like
[Claude Code](https://www.anthropic.com/claude-code)or

[Claude Desktop](https://www.claude.ai/download)— establishes connections to one or more MCP servers. The MCP host accomplishes this by creating one MCP client for each MCP server. Each MCP client maintains a dedicated connection with its corresponding MCP server. Local MCP servers that use the STDIO transport typically serve a single MCP client, whereas remote MCP servers that use the Streamable HTTP transport will typically serve many MCP clients. The key participants in the MCP architecture are:

- **MCP Host** : The AI application that coordinates and manages one or multiple MCP clients
- **MCP Client** : A component that maintains a connection to an MCP server and obtains context from an MCP server for the MCP host to use
- **MCP Server** : A program that provides context to MCP clients

**For example**: Visual Studio Code acts as an MCP host. When Visual Studio Code establishes a connection to an MCP server, such as the

[Sentry MCP server](https://docs.sentry.io/product/sentry-mcp/), the Visual Studio Code runtime instantiates an MCP client object that maintains the connection to the Sentry MCP server. When Visual Studio Code subsequently connects to another MCP server, such as the

[local filesystem server](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem), the Visual Studio Code runtime instantiates an additional MCP client object to maintain this connection. Note that

**MCP server**refers to the program that serves context data, regardless of where it runs. MCP servers can execute locally or remotely. For example, when Claude Desktop launches the

[filesystem server](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem), the server runs locally on the same machine because it uses the STDIO transport. This is commonly referred to as a “local” MCP server. The official

[Sentry MCP server](https://docs.sentry.io/product/sentry-mcp/)runs on the Sentry platform, and uses the Streamable HTTP transport. This is commonly referred to as a “remote” MCP server.

### Layers

MCP consists of two layers:
- **Data layer** : Defines the JSON-RPC based protocol for client-server communication, including capability and version discovery, and core primitives, such as tools, resources, prompts and notifications.
- **Transport layer** : Defines the communication mechanisms and channels that enable data exchange between clients and servers, including transport-specific connection establishment, message framing, and authorization.

#### Data layer

The data layer implements a
[JSON-RPC 2.0](https://www.jsonrpc.org/)based exchange protocol that defines the message structure and semantics. This layer includes:

- **Discovery** : Lets clients query a server’s supported protocol versions, capabilities, and identity through the`server/discover` request
- **Server features** : Enables servers to provide core functionality including tools for AI actions, resources for context data, and prompts for interaction templates from and to the client
- **Client features** : Enables servers to elicit input from the user. Sampling is[deprecated](/specification/2026-07-28/deprecated) as of protocol version`2026-07-28` .
- **Utility features** : Supports additional capabilities like notifications for real-time updates and progress tracking for long-running operations

#### Transport layer

The transport layer manages communication channels and authentication between clients and servers. It handles connection establishment, message framing, and secure communication between MCP participants. MCP supports two transport mechanisms:
- **Stdio transport** : Uses standard input/output streams for direct process communication between local processes on the same machine, providing optimal performance with no network overhead.
- **Streamable HTTP transport** : Uses HTTP POST for client-to-server messages with optional Server-Sent Events for streaming capabilities. This transport enables remote server communication and supports standard HTTP authentication methods including bearer tokens, API keys, and custom headers. MCP recommends using OAuth to obtain authentication tokens.

### Data Layer Protocol

A core part of MCP is defining the schema and semantics between MCP clients and MCP servers. Developers will likely find the data layer — in particular, the set of
[primitives](#primitives)— to be the most interesting part of MCP. It is the part of MCP that defines the ways developers can share context from MCP servers to MCP clients. MCP uses

[JSON-RPC 2.0](https://www.jsonrpc.org/)as its underlying RPC protocol. Client and servers send requests to each other and respond accordingly. Notifications can be used when no response is required.

#### Statelessness and discovery

MCP is a . Every request carries the protocol version and the relevant to that request in its`_meta` field, so the server can process each request on its own. Clients should also identify themselves in the same field unless configured not to. Servers advertise their supported versions and capabilities through the mandatory [request, which clients may send before any other request. Detailed information can be found in the](/specification/2026-07-28/server/discover)

`server/discover`
[specification](/specification/2026-07-28/basic/index#statelessness), and the

[example](#example)showcases the per-request metadata and the discovery sequence.

#### Primitives

MCP primitives are the most important concept within MCP. They define what clients and servers can offer each other. These primitives specify the types of contextual information that can be shared with AI applications and the range of actions that can be performed. MCP defines three core primitives that
*servers*can expose:

- **Tools** : Executable functions that AI applications can invoke to perform actions (e.g., file operations, API calls, database queries)
- **Resources** : Data sources that provide contextual information to AI applications (e.g., file contents, database records, API responses)
- **Prompts** : Reusable templates that help structure interactions with language models (e.g., system prompts, few-shot examples)

`*/list`), retrieval (`*/get`), and in some cases, execution (`tools/call`).
MCP clients will use the `*/list` methods to discover available primitives. For example, a client can first list all available tools (`tools/list`) and then execute them. This design allows listings to be dynamic.
As a concrete example, consider an MCP server that provides context about a database. It can expose tools for querying the database, a resource that contains the schema of the database, and a prompt that includes few-shot examples for interacting with the tools.
For more details about server primitives see [server concepts](/docs/2026-07-28/learn/server-concepts). MCP also defines primitives that

*clients*can expose. These primitives allow MCP server authors to build richer interactions.

- **Elicitation** : Allows servers to request additional information from users. This is useful when server authors want to get more information from the user, or ask for confirmation of an action. Servers request user input with the`elicitation/create` method.

[Multi Round-Trip Requests](/specification/2026-07-28/basic/patterns/mrtr)pattern, explained in the

[elicitation overview](/docs/2026-07-28/learn/client-concepts#elicitation).

**Deprecated**: The following client primitives are deprecated as of protocol version

`2026-07-28`.
- **Sampling** : Allows servers to request language model completions from the client’s AI application. This is useful when server authors want access to a language model, but want to stay model-independent and not include a language model SDK in their MCP server. Servers request completions with the`sampling/createMessage` method, also delivered through the Multi Round-Trip Requests pattern. New implementations should integrate directly with LLM provider APIs.
- **Logging** : Enables servers to send log messages to clients for debugging and monitoring purposes. New implementations should log to`stderr` (stdio transport) or use OpenTelemetry.

[client concepts](/docs/2026-07-28/learn/client-concepts). Besides server and client primitives, the protocol supports optional

[extensions](/extensions/overview)that build on the core protocol. For example, the

[Tasks extension](/extensions/tasks/overview)lets servers return a durable handle for long-running requests, so clients can poll for status and retrieve the result later.

#### Notifications

The protocol supports real-time notifications to enable dynamic updates between servers and clients. For example, when a server’s available tools change (such as when new functionality becomes available or existing tools are modified), the server can send tool update notifications to inform connected clients about these changes. Notifications are sent as JSON-RPC 2.0 notification messages (without expecting a response). Change notifications are opt-in: the client opens a long-lived
[stream naming the notification types it wants to receive, and the server delivers matching notifications on that stream.](/specification/2026-07-28/basic/patterns/subscriptions)

`subscriptions/listen`
## Example

### Data Layer

This section provides a step-by-step walkthrough of an MCP client-server interaction, focusing on the data layer protocol. We’ll demonstrate discovery, tool operations, and notifications using JSON-RPC 2.0 messages.
1

Discovery

As described in the 

[statelessness and discovery](#statelessness-and-discovery)section, every MCP request carries the protocol version and client capabilities in its`_meta` field, and clients should also include their identity there. A client that wants to learn what a server supports before issuing other requests sends a `server/discover` request, which every server must implement. The discovery response is typically cacheable, meaning it can be re-used so the discovery flow does not need to be performed for every request.
#### Understanding the Discovery Exchange

The`_meta` fields and the discovery response together serve several purposes:
1. 
**Protocol Version Selection** : The`io.modelcontextprotocol/protocolVersion` field declares the version the client is speaking on this request, and`supportedVersions` in the response lists the versions the server accepts. If a server does not support the requested version, it rejects the request with an`UnsupportedProtocolVersionError` listing the versions it does support, and the client retries with a mutually supported version.
2. 
**Capability Discovery** : The client declares its capabilities in`io.modelcontextprotocol/clientCapabilities` on every request, and the server returns its own`capabilities` object from`server/discover` . This tells each party which[primitives](#primitives) the other can handle (tools, resources, prompts) and whether change[notifications](#notifications) are available, so unsupported operations are never attempted.
3. 
**Identity Exchange** : The`io.modelcontextprotocol/clientInfo` field in the request’s`_meta` and the`io.modelcontextprotocol/serverInfo` field in the result’s`_meta` provide identification and versioning information for debugging and compatibility purposes.

**Client Capabilities**:
- `"elicitation": {}` - The client declares it can gather additional input from the user when the server requests it

**Server Capabilities**:
- `"tools": {"listChanged": true}` - The server supports the tools primitive and can honor a`toolsListChanged` filter in[`subscriptions/listen`](/specification/2026-07-28/basic/patterns/subscriptions) . Clients that request this filter receive`notifications/tools/list_changed` when the tool list changes.
- `"resources": {}` - The server also supports the resources primitive (can handle`resources/list` and`resources/read` methods)

`server/discover` is optional. Because every request carries the same `_meta` fields, a client is free to send any request directly and handle a version error if one comes back. Discovery is a convenient way to fetch the server’s identity, capabilities, and supported versions in a single request.
#### How This Works in AI Applications

The AI application’s MCP client manager connects to configured servers and stores their discovered capabilities for later use. The application uses this information to determine which servers can provide specific types of functionality (tools, resources, prompts) and whether they support real-time updates. In the Python SDK, discovery happens while the client connects. The results are then available on the client object.
Pseudo-code for AI application discovery

2

Tool Discovery (Primitives)

The client can discover available tools by sending a Clients that federate many servers can use 

`tools/list` request. This request is fundamental to MCP’s tool discovery mechanism: it allows clients to understand what tools are available on the server before attempting to use them.
#### Understanding the Tool Discovery Request

The`tools/list` request requires no parameters beyond the standard `_meta` fields that accompany every MCP request. It also accepts an optional `cursor` parameter for [pagination](/specification/2026-07-28/server/utilities/pagination), which the example above omits.
#### Understanding the Tool Discovery Response

The response contains a`tools` array that provides comprehensive metadata about each available tool. This array-based structure allows servers to expose multiple tools simultaneously while maintaining clear boundaries between different functionalities.Each tool object in the response includes several key fields:
- **`name`** : A unique identifier for the tool within the server’s namespace. This serves as the primary key for tool execution and should follow a clear naming pattern (e.g.,`calculator_arithmetic` rather than just`calculate` )
- **`title`** : A human-readable display name for the tool that clients can show to users
- **`description`** : Detailed explanation of what the tool does and when to use it
- **`inputSchema`** : A JSON Schema that defines the expected input parameters, enabling type validation and providing clear documentation about required and optional parameters

`"resultType": "complete"` and carries two caching fields. `ttlMs` is a freshness hint in milliseconds, so this tool list can be cached for five minutes. `cacheScope` indicates who may reuse the response. The specification’s [caching utility](/specification/2026-07-28/server/utilities/caching)defines the full rules.
#### How This Works in AI Applications

The AI application fetches available tools from all connected MCP servers and combines them into a unified tool registry that the language model can access. This allows the LLM to understand what actions it can perform and automatically generates the appropriate tool calls during conversations.
Pseudo-code for AI application tool discovery

[progressive tool discovery](/docs/2026-07-28/develop/clients/client-best-practices#progressive-tool-discovery)rather than loading every tool upfront.
3

Tool Execution (Primitives)

The client can now execute a tool using the 

`tools/call` method. This demonstrates how MCP primitives are used in practice: after discovering available tools, the client can invoke them with appropriate arguments.
#### Understanding the Tool Execution Request

The`tools/call` request follows a structured format that ensures type safety and clear communication between client and server. Note that we’re using the proper tool name from the discovery response (`weather_current`) rather than a simplified name:
#### Key Elements of Tool Execution

The request structure includes several important components:
1. 
**`name`** : Must match exactly the tool name from the discovery response (`weather_current` ). This ensures the server can correctly identify which tool to execute.
2. 
**`arguments`** : Contains the input parameters as defined by the tool’s`inputSchema` . In this example:
  - `location` : “San Francisco” (required parameter)
  - `units` : “imperial” (optional parameter, defaults to “metric” if not specified)
3. 
**`_meta`** : Carries the standard per-request fields: the protocol version and client capabilities that every MCP request must include, plus the client’s identity, which clients should include unless configured not to.
4. 
**JSON-RPC Structure** : Uses standard JSON-RPC 2.0 format with unique`id` for request-response correlation.

#### Understanding the Tool Execution Response

The response demonstrates MCP’s flexible content system:
1. 
**`content` Array** : Tool responses return an array of content objects, allowing for rich, multi-format responses (text, images, resources, etc.)
2. 
**Content Types** : Each content object has a`type` field. In this example,`"type": "text"` indicates plain text content, but MCP supports various content types for different use cases.
3. 
**Structured Output** : The response provides actionable information that the AI application can use as context for language model interactions.

#### How This Works in AI Applications

When the language model decides to use a tool during a conversation, the AI application intercepts the tool call, routes it to the appropriate MCP server, executes it, and returns the results back to the LLM as part of the conversation flow. This enables the LLM to access real-time data and perform actions in the external world.
4

Real-time Updates (Notifications)

MCP supports real-time notifications that enable servers to inform clients about changes without being polled for them. This demonstrates the notification system, a key feature that keeps clients synchronized and responsive.Every client request carries the 

#### Subscribing to Changes

Change notifications are opt-in. To receive them, the client opens a long-lived notification stream by sending a[request with a](/specification/2026-07-28/basic/patterns/subscriptions)`subscriptions/listen``notifications` filter naming the event types it wants. Here the client asks for tool list changes:
Listen Request

`io.modelcontextprotocol/protocolVersion` and `io.modelcontextprotocol/clientCapabilities` fields in `_meta`, and normally `io.modelcontextprotocol/clientInfo` as well, so the server can identify the client without relying on connection state.The server acknowledges the subscription with `notifications/subscriptions/acknowledged`, which is the first message carrying that subscription’s ID in `_meta` (the server sends no other notification for that subscription before it). Its `notifications` field reflects the subset of the requested filter the server agreed to honor, with unsupported notification types omitted:
Acknowledgment

#### Understanding Tool List Change Notifications

After the acknowledgment, when the server’s available tools change (for example, when new functionality becomes available, existing tools are modified, or tools become temporarily unavailable), the server delivers a notification on that stream:
Notification

#### Key Features of MCP Notifications

1. 
**No Response Required** : Notice there’s no`id` field in the notification. This follows JSON-RPC 2.0 notification semantics where no response is expected or sent.
2. 
**Opt-In Based** : This notification is only sent to clients that requested`"toolsListChanged": true` in their`subscriptions/listen` filter, and it is only available from servers that declared`"listChanged": true` in their tools capability (as shown in Step 1).
3. 
**Subscription-ID Tagging** : Every notification on the stream carries`io.modelcontextprotocol/subscriptionId` in`_meta` . The value is the JSON-RPC ID of the`subscriptions/listen` request that opened the stream (`4` in this example), so clients can correlate each notification with the subscription that produced it.
4. 
**Event-Driven** : The server decides when to send notifications based on internal state changes, making MCP connections dynamic and responsive.
5. 
**Best Effort** : There are no guarantees that every notification will be sent or received, particularly across transport reconnects. Clients should also rely on polling to preserve freshness of results.

#### Client Response to Notifications

Upon receiving this notification, the client typically reacts by requesting the updated tool list. This creates a refresh cycle that keeps the client’s understanding of available tools current:
Request

#### Why Notifications Matter

This notification system is crucial for several reasons:
1. **Dynamic Environments** : Tools may come and go based on server state, external dependencies, or user permissions
2. **Efficiency** : Clients don’t need to poll for changes; they’re notified when updates occur
3. **Consistency** : Ensures clients always have accurate information about available server capabilities
4. **Real-time Collaboration** : Enables responsive AI applications that can adapt to changing contexts

# Citations

1. Source page: https://modelcontextprotocol.io/docs/2026-07-28/learn/architecture
