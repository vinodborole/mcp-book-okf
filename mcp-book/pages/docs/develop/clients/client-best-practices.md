---
type: Web Page
title: Client Best Practices - Model Context Protocol
description: Patterns for scaling MCP host applications across many servers and tools.
resource: https://modelcontextprotocol.io/docs/develop/clients/client-best-practices
timestamp: '2026-07-07T10:31:48.208319+00:00'
---

**progressive discovery**, which controls

*when*tool definitions enter context, and

**programmatic tool calling**, which controls

*how*tools are invoked.

## Progressive Tool Discovery

Naive MCP host implementations pass the tool definitions of every connected server directly to the model at the start of each conversation. For a handful of tools, this is perfectly reasonable. But when a host has access to dozens of servers exposing hundreds of tools, those definitions alone can consume the majority of the context window before the model has even read the user’s message. Progressive discovery avoids this:- The host fetches tool definitions via `tools/list`as normal, but defers injecting them into the model’s context.
- The host provides a lightweight `search_tools`meta-tool to the model.
- The host loads full definitions into context only as needed.

### When to Use Progressive Discovery

Progressive discovery is best used when tool definitions take large parts of the context window. For a small set of tools with tool definitions taking up a small part of the context window, loading all tools is fine. Once the tool definitions take up a significant part of the available context window, clients should switch to progressive discovery. We recommend that clients implement thresholds to determine when to switch:- Implement a threshold as a percentage of the context window. For example, 1%-5%.
- Load tool definitions. Once the threshold is reached, switch to progressive discovery.

### Choosing a Discovery Strategy

Once the model invokes the`search_tools` tool, we need to choose a search strategy:
- **Keyword-based**: Keyword matching (BM25, regex). Simple and effective, particularly for descriptive tool names and descriptions.
- **Embedding-based**: Vector-similarity retrieval over tool descriptions. Handles synonyms and semantic matching better.
- **Subagent-based**: A secondary model, often a small and fast model such as Claude Haiku or Gemini Flash, selects tools for the task. This usually works very well but can be more costly than embedding-based or keyword-based solutions.
- **Hybrid**: Combine approaches. For example, by scoring across keyword and embedding rankings, or choosing different strategies depending on use-case or query.

### Using Progressive Discovery

One common implementation for progressive discovery uses a search-based three-layer approach:**Layer 1: Catalog.**The host exposes a small set of meta-tools for searching available capabilities. A

`search_tools` tool accepts a natural-language query and returns matching tool names with brief descriptions.
**Layer 2: Inspect.**Once the model identifies a candidate, it fetches the full definition (input schema, output schema, documentation) for that tool only.

**Layer 3: Execute.**The model calls the tool with full knowledge of its interface, having loaded only the definitions it needed. This pattern reduces token usage dramatically and can improve tool selection accuracy: the model focuses on a few relevant tools rather than scanning hundreds of irrelevant ones. Other discovery strategies (embeddings, subagents, etc.) follow the same layered principle but substitute different retrieval mechanisms in the catalog layer.

### Dynamic Server Management

Progressive discovery extends beyond individual tools to entire servers. Rather than connecting to every configured server at startup, a host can:- Maintain a registry of available servers and their high-level descriptions.
- Connect to a server only when the model determines it needs that server’s capabilities.
- Disconnect servers that are no longer relevant to the current task, freeing context.

### Implementation Guidelines

When implementing progressive discovery:| Guideline | Rationale | 
|---|---|
| Offer multiple detail levels | Let the model choose between name-only, name-and-description, or full-schema responses. | 
| Cache tool definitions | Once fetched from a server, memoize the definition host-side so re-injecting it later doesn’t need another `tools/list`round trip. This is separate from what’s currently in the model’s context. | 
| Refresh on `list_changed` | Re-index the search catalog when a server sends `notifications/tools/list_changed`. | 
| Group tools by server | Present tools organized by their source server so the model can reason about related capabilities. | 

### Interaction with Prompt Caching

Most providers cache the prompt prefix, including the`tools` array. Adding or removing tool
definitions mid-conversation invalidates that cache, and the resulting miss can cost more tokens
than the definitions you removed. To preserve caching:
- Append newly discovered definitions after the cache breakpoint rather than re-sorting the
`tools`array, or route every call through a single stable`call_tool({name, args})`meta-tool so the array never changes.
- Treat server disconnection as a conversation-boundary operation rather than a per-turn one.
- Consult your provider’s caching documentation alongside the tool-search links above.

## Programmatic Tool Calling / Code Mode

With direct tool calling, every tool invocation is a round trip: the model generates a tool call, the client executes it, and the full result flows back into the model’s context. When a task requires chaining multiple tools (read a document, transform it, write it somewhere else), each intermediate result passes through the model, consuming tokens and adding latency even when it has nothing to do with them. Programmatic tool calling (sometimes called “code mode”) provides a way for clients to**compose tool calls**effectively. Instead of calling tools directly, the model writes code that calls tools. The code executes in a sandboxed environment, and only the final result returns to the model. Programmatic tool calling is powerful and allows for more efficient use of MCP tools and resources, but requires clients to implement a sandbox environment.

### How It Works

The host converts MCP tool schemas into a typed API available inside a sandbox. When the model needs tools, it writes a script and executes it.**Step 1: Generate a programmatic API from MCP schemas.**The host reads each server’s tool definitions and produces typed functions based on each tool’s arguments and

`outputSchema`:
`outputSchema` for each tool. When an output schema is present, the host can produce precise return types (like `LogEntry` above).
When an output schema is absent, prefer the simple path:
- **Use a generic type and move on.**Accept- `any`or- `string`and handle the unstructured output downstream. The real fix is for server authors to provide- `outputSchema`.
- **Extract a typed result using a fast model**, for single-shot calls outside loops. Expose a host-brokered- `extract(value, ExpectedType)`helper through the same stub-interception path as MCP tool calls so the sandbox itself never opens a network connection. The helper routes to a small model (for example, Claude Haiku or Gemini Flash) to coerce the value into- `ExpectedType`. This adds per-call latency and can hallucinate or drop fields, so validate the result against- `ExpectedType`before use.

**Step 2: The model writes code against these APIs.**Rather than making separate tool calls with full results flowing through context between them, the model writes a single script. Consider a task like “find all error logs from the past hour and file a ticket for each unique error.” With direct tool calling, thousands of log entries would flow through the model’s context. With code, the model filters in the sandbox:

**Step 3: The sandbox executes the code.**Function calls inside the sandbox are intercepted and routed back to the appropriate MCP server through the host broker. The log data and ticket creation flow directly between servers without ever entering the model’s context. Only the

`console.log` output, a single summary line, returns to the model.
### Choosing a Sandbox

The right sandbox depends on the language you want the model to write, your host application’s language, and how much isolation you need. The table lists example runtimes rather than endorsements; evaluate maturity for your use case:| Sandboxed language | Runtime / Library | Host language | Approach | 
|---|---|---|---|
| JavaScript | Deno, `isolated-vm` | Rust / Node / CLI | V8-based runtimes with fine-grained permissions. Can disable all permissions for full lockdown. | 
| Python | Monty (experimental) | Rust | Minimal Python interpreter built for AI use cases. No I/O by default. | 
| TypeScript | pctx (early-stage) | Python / Rust | Incorporates code mode concepts as a library, with low-level Rust support. | 
| Any (via Wasm) | Wasmtime | Rust / C / Go | Compile any language to Wasm and run it with capability-based security. | 

`tools/call` requests to MCP servers.
### Execution Architecture

The implementation has three components:**The sandbox**runs model-generated code in an isolated environment with no direct network access. Its only interface to the outside world is through the generated function stubs, which route calls back to the host.

**The host**acts as a broker. It receives function calls from the sandbox, maps them to the correct MCP server, executes the tool call, and returns the result to the sandbox. Authorization tokens and credentials are held by the host and never exposed to the generated code.

**The model**sees only what the sandbox returns, typically the output of

`console.log` statements or a final return value. This gives the model (and the client developer) precise control over what enters the context window.
### Security Considerations

Programmatic tool calling introduces a code execution surface that requires careful sandboxing:- **Per-call authorization**: The broker is still the MCP host for spec purposes. Apply the same human-in-the-loop confirmation policy to sandbox-originated calls that you apply to direct calls (see Tools: Security). Approving the script does not grant blanket approval for every tool call it makes at runtime; hosts may grant categorical approval (for example, “allow- `ticketing_createIssue`for this script run”) rather than prompting per iteration, but the broker must still evaluate each call against that grant.
- **Cross-server data flow**: Tool results from one server are untrusted input to another. The broker should apply the same input-review policy to brokered calls as to direct ones; output truncation alone does not prevent exfiltration.
- **Network isolation**: The sandbox should have no direct network access. All external communication flows through the host broker, which enforces authorization and access control.
- **No credential exposure**: API keys and tokens are held by the host. The generated code calls typed functions; the host adds authentication when forwarding to servers.
- **Resource limits**: Set timeouts and memory limits on sandbox execution to prevent runaway scripts.
- **Output filtering**: Validate and truncate sandbox console output before feeding it back to the model.

### Error Handling

MCP tool errors arrive as a successful response with`isError: true` rather than a transport
failure. Generated wrappers should convert this into a thrown exception so model-authored code
can use `try`/`catch`. If an uncaught error terminates the script, surface it as the script’s
result so the model can self-correct; the model is responsible for reporting any partial side
effects already committed.
## Combining Both Patterns

Progressive discovery and programmatic tool calling work well together. The model uses discovery tools to identify which tools it needs, loads their schemas, and then writes a single script that calls multiple tools in one execution pass. This combination minimizes both the token cost of tool definitions*and*the token cost of tool results, keeping the model’s context focused on reasoning rather than passing data through it.

# Citations

1. Source page: https://modelcontextprotocol.io/docs/develop/clients/client-best-practices
