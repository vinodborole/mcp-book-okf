---
type: Web Page
title: Understanding MCP clients - Model Context Protocol
resource: https://modelcontextprotocol.io/docs/2026-07-28/learn/client-concepts
timestamp: '2026-08-03T09:44:29.575770+00:00'
---

*host*is the application users interact with, while

*clients*are the protocol-level components that enable server connections.

## Core Client Features

In addition to making use of context provided by servers, clients may provide several features to servers. These client features allow server authors to build richer interactions.
### Elicitation

Elicitation enables servers to request specific information from users during interactions, creating more dynamic and responsive workflows.
#### Overview

Elicitation provides a structured way for servers to gather necessary information on demand. Instead of requiring all information up front or failing when data is missing, servers can pause their operations to request specific inputs from users. This creates more flexible interactions where servers adapt to user needs rather than following rigid patterns. Elicitation supports two modes:
- **Form mode** : The server asks the client to collect structured data from the user. The request includes a schema that the client uses to build an input form and validate the response.
- **URL mode** : The server provides a URL for the user to open. The interaction happens out of band and its data never passes through the client, which makes this mode suitable for sensitive flows such as credential entry or third-party OAuth authorization.

[Multi Round-Trip Requests](/specification/2026-07-28/basic/patterns/mrtr)(MRTR) pattern. When a server needs user input while processing a request such as

`tools/call`, it responds with an `InputRequiredResult` whose `inputRequests` field carries one or more `elicitation/create` requests. The client gathers the input and retries the original request, attaching the collected `inputResponses` and echoing back any `requestState` the server included.
**Elicitation flow:**The flow enables dynamic information gathering. Servers can request specific data when needed, users provide information through appropriate UI, and servers complete the retried request with the newly acquired context.

**Elicitation request example (delivered inside**

`InputRequiredResult.inputRequests`):
#### Example: Holiday Booking Approval

A travel booking server demonstrates elicitation’s power through the final booking confirmation process. When a user has selected their ideal vacation package to Barcelona, the server needs to gather final approval and any missing details before proceeding. The server elicits booking confirmation with a structured request that includes the trip summary (Barcelona flights June 15-22, beachfront hotel, total $3,000) and fields for any additional preferences—such as seat selection, room type, or travel insurance options. As the booking progresses, the server elicits contact information needed to complete the reservation. It might ask for traveler details for flight bookings, special requests for the hotel, or emergency contact information.
#### User Interaction Model

Elicitation interactions are designed to be clear, contextual, and respectful of user autonomy:
**Request presentation**: Clients display elicitation requests with clear context about which server is asking, why the information is needed, and how it will be used. The request message explains the purpose while the schema provides structure and validation.

**Response options**: Users can provide the requested information through appropriate UI controls (text fields, dropdowns, checkboxes), decline to provide information with optional explanation, or cancel the entire operation. Clients validate responses against the provided schema before returning them to servers.

**URL handling**: For URL mode, clients show the full URL and gather explicit consent before opening it, and never fetch the URL automatically. The client only learns whether the user consented. The interaction itself stays between the user and the target site.

**Privacy considerations**: Servers must not use form mode to request sensitive information such as passwords, API keys, access tokens, or payment credentials. Those interactions belong in URL mode, which keeps the data out of band so it never passes through the client or the LLM context. Clients warn about suspicious requests and let users review form data before sending.

### Roots

Roots define filesystem boundaries for server operations, allowing clients to specify which directories servers should focus on.
#### Overview

Roots are a mechanism for clients to communicate filesystem access boundaries to servers. They consist of file URIs that indicate directories where servers can operate, helping servers understand the scope of available files and folders. While roots communicate intended boundaries, they do not enforce security restrictions. Actual security must be enforced at the operating system level, via file permissions and/or sandboxing.
**Root structure:**

`file://` URI scheme. They help servers understand project boundaries, workspace organization, and accessible directories. The roots list can change as users work with different projects or folders. Servers pick up the updated boundaries the next time they request the roots list.
#### Example: Travel Planning Workspace

A travel agent working with multiple client trips benefits from roots to organize filesystem access. Consider a workspace with different directories for various aspects of travel planning. The client provides filesystem roots to the travel planning server:
- `file:///Users/agent/travel-planning` - Main workspace containing all travel files
- `file:///Users/agent/travel-templates` - Reusable itinerary templates and resources
- `file:///Users/agent/client-documents` - Client passports and travel documents

`file:///Users/agent/archive/2023-trips`, the client adds it to the roots list, and the server sees the new boundary on its next `roots/list` request.
For a complete implementation of a server that respects roots, see the [filesystem server](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem)in the official servers repository.

#### Design Philosophy

Roots serve as a coordination mechanism between clients and servers, not a security boundary. The specification requires that servers “SHOULD respect root boundaries,” and not that they “MUST enforce” them, because servers run code the client cannot control. Roots work best when servers are trusted or vetted, users understand their advisory nature, and the goal is preventing accidents rather than stopping malicious behavior. They excel at context scoping (telling servers where to focus), accident prevention (helping well-behaved servers stay in bounds), and workflow organization (such as managing project boundaries automatically).
#### User Interaction Model

Roots are typically managed automatically by host applications based on user actions, though some applications may expose manual root management:
**Automatic root detection**: When users open folders, clients automatically expose them as roots. Opening a travel workspace allows the client to expose that directory as a root, helping servers understand which itineraries and documents are in scope for the current work.

**Manual root configuration**: Advanced users can specify roots through configuration. For example, adding

`/travel-templates` for reusable resources while excluding directories with financial records.
### Sampling

Sampling allows servers to request language model completions through the client, enabling agentic behaviors while maintaining security and user control.
#### Overview

Sampling enables servers to perform AI-dependent tasks without directly integrating with or paying for AI models. Instead, servers can request that the client—which already has AI model access—handle these tasks on their behalf. This approach puts the client in complete control of user permissions and security measures. Because sampling requests occur within the context of other operations—like a tool analyzing data—and are processed as separate model calls, they maintain clear boundaries between different contexts, allowing for more efficient use of the context window. Sampling follows the same
[Multi Round-Trip Requests](/specification/2026-07-28/basic/patterns/mrtr)flow described under

[elicitation](#elicitation), with the

`InputRequiredResult` carrying a `sampling/createMessage` request.
Servers can also request tool use during sampling by including a `tools` array and an optional `toolChoice` field in the request. The tool definitions are scoped to that sampling request and do not need to correspond to tools the server exposes. Clients declare support through the `sampling.tools` capability, and servers must not send tool-enabled sampling requests to clients that have not declared it. See [sampling](/specification/2026-07-28/client/sampling#tools-in-sampling)in the specification for details.

**Sampling flow:**The flow ensures security through multiple human-in-the-loop checkpoints. Users review and can modify both the initial request and the generated response before the client retries the original request with it.

**Request parameters example:**

#### Example: Flight Analysis Tool

Consider a travel booking server with a tool called`findBestFlight` that uses sampling to analyze available flights and recommend the optimal choice. When a user asks “Book me the best flight to Barcelona next month,” the tool needs AI assistance to evaluate complex trade-offs.
The tool queries airline APIs and gathers 47 flight options. It then requests AI assistance to analyze these options: “Analyze these flight options and recommend the best choice: [47 flights with prices, times, airlines, and layovers] User preferences: morning departure, max 1 layover.”
The client initiates the sampling request, allowing the AI to evaluate trade-offs—like cheaper red-eye flights versus convenient morning departures. The tool uses this analysis to present the top three recommendations.
#### User Interaction Model

While not a requirement, sampling is designed to allow human-in-the-loop control. Users can maintain oversight through several mechanisms:
**Approval controls**: Sampling requests may require explicit user consent. Clients can show what the server wants to analyze and why. Users can approve, deny, or modify requests.

**Transparency features**: Clients can display the exact prompt, model selection, and token limits, allowing users to review AI responses before they return to the server.

**Configuration options**: Users can set model preferences, configure auto-approval for trusted operations, or require approval for everything. Clients may provide options to redact sensitive information.

**Security considerations**: Both clients and servers must handle sensitive data appropriately during sampling. Clients should implement rate limiting and validate all message content. The human-in-the-loop design ensures that server-requested AI interactions cannot compromise security or access sensitive data without explicit user consent.

# Citations

1. Source page: https://modelcontextprotocol.io/docs/2026-07-28/learn/client-concepts
