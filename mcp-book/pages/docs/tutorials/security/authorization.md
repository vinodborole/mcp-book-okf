---
type: Web Page
title: Understanding Authorization in MCP - Model Context Protocol
description: Learn how to implement secure authorization for MCP servers using OAuth
  2.1 to protect sensitive resources and operations
resource: https://modelcontextprotocol.io/docs/tutorials/security/authorization
timestamp: '2026-07-09T12:16:39.468634+00:00'
---

[OAuth 2.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-13). For detailed information, see the

[Authorization specification](/specification/latest/basic/authorization).

## When Should You Use Authorization?

While authorization for MCP servers is**optional**, it is strongly recommended when:

- Your server accesses user-specific data (emails, documents, databases)
- You need to audit who performed which actions
- Your server grants access to its APIs that require user consent
- You’re building for enterprise environments with strict access controls
- You want to implement rate limiting or usage tracking per user

## The Authorization Flow: Step by Step

Let’s walk through what happens when a client wants to connect to your protected MCP server:Initial Handshake

When your MCP client first tries to connect, your server responds with a This tells the client that authorization is required for the MCP server and where to get the necessary information to kickstart the authorization flow.

`401 Unauthorized` and tells the client where to find authorization information, captured in a [Protected Resource Metadata (PRM) document](https://datatracker.ietf.org/doc/html/rfc9728). The document is hosted by the MCP server, follows a predictable path pattern, and is provided to the client in the`resource_metadata` parameter within the `WWW-Authenticate` header.Protected Resource Metadata Discovery

With the URI pointer to the PRM document, the client will fetch the metadata to learn about the authorization server, supported scopes, and other resource information. The data is typically encapsulated in a JSON blob, similar to the one below.You can see a more comprehensive example in 

[RFC 9728 Section 3.2](https://datatracker.ietf.org/doc/html/rfc9728#name-protected-resource-metadata-r).Authorization Server Discovery

Next, the client discovers what the authorization server can do by fetching its metadata. If the PRM document lists more than one authorization server, the client can decide which one to use.With an authorization server selected, the client will then construct a standard metadata URI and issue a request to the 

[OpenID Connect (OIDC) Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html)or[OAuth 2.0 Auth Server Metadata](https://datatracker.ietf.org/doc/html/rfc8414)endpoints (depending on authorization server support) and retrieve another set of metadata properties that will allow it to know the endpoints it needs to complete the authorization flow.Client Registration

With all the metadata out of the way, the client now needs to make sure that it’s registered with the authorization server. This can be done in two ways.First, the client can be If the registration succeeds, the authorization server will return a JSON blob with client registration information.

**pre-registered**with a given authorization server, in which case it can have embedded client registration information that it uses to complete the authorization flow.Alternatively, the client can use**Dynamic Client Registration**(DCR) to dynamically register itself with the authorization server. The latter scenario requires the authorization server to support DCR. If the authorization server does support DCR, the client will send a request to the`registration_endpoint` with its information:User Authorization

The client will now need to open a browser to the The access token is what the client will use to authenticate requests to the MCP server. This step follows standard 

`/authorize` endpoint, where the user can log in and grant the required permissions. The authorization server will then redirect back to the client with an authorization code that the client exchanges for tokens:[OAuth 2.1 authorization code with PKCE](https://oauth.net/2/grant-types/authorization-code/)conventions.## Implementation Example

To get started with a practical implementation, we will use a[Keycloak](https://www.keycloak.org/)authorization server hosted in a Docker container. Keycloak is an open-source authorization server that can be easily deployed locally for testing and experimentation. Make sure that you download and install

[Docker Desktop](https://www.docker.com/products/docker-desktop/). We will need it to deploy Keycloak on our development machine.

### Keycloak Setup

From your terminal application, run the following command to start the Keycloak container:`8080` and have an `admin` user with `admin` password.
You will be able to access the Keycloak authorization server from your browser at `http://localhost:8080`.
**Client scopes**in the Keycloak dashboard and create a new

`mcp:tools` scope. We will use this to access all of the tools on our MCP server.
**Default**and have flipped the

**Include in token scope**switch, as this will be needed for token validation. Let’s now also set up an

**audience**for our Keycloak-issued tokens. An audience is important to configure because it embeds the intended destination directly into the issued access token. This helps your MCP server to verify that the token it got was actually meant for it rather than some other API. This is key to help avoid token passthrough scenarios. To do this, open your

`mcp:tools` client scope and click on **Mappers**, followed by

**Configure a new mapper**. Select

**Audience**.

**Name**, use

`audience-config`. Add a value for **Included Custom Audience**, set to

`http://localhost:3000`. This will be the URI of our test server.
Now, navigate to **Clients**, then

**Client registration**, and then

**Trusted Hosts**. Disable the

**Client URIs Must Match**setting and add the hosts from which you’re testing. You can get your current host IP by running the

`ifconfig` command on Linux or macOS, or `ipconfig` on Windows. You can see the IP address you need to add by looking at the keycloak logs for a line that looks like `Failed to verify remote host : 192.168.215.1`. Check that the IP address is associated with your host. This may be for a bridge network depending on your docker setup.
**MCP server itself**to talk to Keycloak for things like

[token introspection](https://oauth.net/2/token-introspection/). To do that:

- Go to **Clients**.
- Click **Create client**.
- Give your client a unique **Client ID**and click**Next**.
- Enable **Client authentication**and click**Next**.
- Click **Save**.

*one of*the available approaches to validate tokens. This can also be done with the help of standalone libraries, specific to each language and platform. When you open the client details, go to

**Credentials**and take note of the

**Client Secret**.

### MCP Server Setup

We will now set up our MCP server to use the locally-running Keycloak authorization server. Depending on your programming language preference, you can use one of the supported[MCP SDKs](/docs/sdk). For our testing purposes, we will create an extremely simple MCP server that exposes two tools - one for addition and another for multiplication. The server will require authorization to access these.

- TypeScript
- Python
- C#

You can see the complete TypeScript project in the When you run the server, you can add it to your MCP client, such as Visual Studio Code, by providing the MCP server endpoint.For more details about implementing MCP servers in TypeScript, refer to the 

[sample repository](https://github.com/localden/min-ts-mcp-auth).Prior to running the code below, ensure that you have a`.env` file with the following content:`OAUTH_CLIENT_ID` and `OAUTH_CLIENT_SECRET` are associated with the MCP server client we created earlier.In addition to implementing the MCP authorization specification, the server below also does token introspection via Keycloak to make sure that the token it receives from the client is valid. It also implements basic logging to allow you to easily diagnose any issues.[TypeScript SDK documentation](https://github.com/modelcontextprotocol/typescript-sdk).## Testing the MCP Server

For testing purposes, we will be using[Visual Studio Code](https://code.visualstudio.com), but any client that supports MCP and the new authorization specification will fit. Press

`Cmd`+

`Shift`+

`P`and select

**MCP: Add server…**. Select

**HTTP**and enter

`http://localhost:3000`. Give the server a unique name to be used inside Visual Studio Code. In `mcp.json` you should now see an entry like this:
`mcp:tools` scope.
`mcp.json`.
`#` sign in the chat view.
## Common Pitfalls and How to Avoid Them

For comprehensive security guidance, including attack vectors, mitigation strategies, and implementation best practices, make sure to read through[Security Best Practices](/specification/draft/basic/security_best_practices). A few key issues are called out below.

- **Do not implement token validation or authorization logic by yourself**. Use off-the-shelf, well-tested, and secure libraries for things like token validation or authorization decisions. Doing everything from scratch means that you’re more likely to implement things incorrectly unless you are a security expert.
- **Use short-lived access tokens**. Depending on the authorization server used, this setting might be customizable. We recommend to not use long-lived tokens - if a malicious actor steals them, they will be able to maintain their access for longer periods.
- **Always validate tokens**. Just because your server received a token does not mean that the token is valid or that it’s meant for your server. Always verify that what your MCP server is getting from the client matches the required constraints.
- **Store tokens in secure, encrypted storage**. In certain scenarios, you might need to cache tokens server-side. If that is the case, ensure that the storage has the right access controls and cannot be easily exfiltrated by malicious parties with access to your server. You should also implement robust cache eviction policies to ensure that your MCP server is not re-using expired or otherwise invalid tokens.
- **Enforce HTTPS in production**. Do not accept tokens or redirect callbacks over plain HTTP except for- `localhost`during development.
- **Least-privilege scopes**. Don’t use catch‑all scopes. Split access per tool or capability where possible and verify required scopes per route/tool on the resource server.
- **Don’t log credentials**. Never log- `Authorization`headers, tokens, codes, or secrets. Scrub query strings and headers. Redact sensitive fields in structured logs.
- **Separate app vs. resource server credentials**. Don’t reuse your MCP server’s client secret for end‑user flows. Store all secrets in a proper secret manager, not in source control.
- **Return proper challenges**. On 401, include- `WWW-Authenticate`with- `Bearer`,- `realm`, and- `resource_metadata`so clients can discover how to authenticate.
- **DCR (Dynamic Client Registration) controls**. If enabled, be aware of constraints specific to your organization, such as trusted hosts, required vetting, and audited registrations. Unauthenticated DCR means that anyone can register any client with your authorization server.
- **Multi‑tenant/realm mix-ups**. Pin to a single issuer/tenant unless explicitly multi‑tenant. Reject tokens from other realms even if signed by the same authorization server.
- **Audience/resource indicator misuse**. Don’t configure or accept generic audiences (like- `api`) or unrelated resources. Require the audience/resource to match your configured server.
- **Error detail leakage**. Return generic messages to clients, but log detailed reasons with correlation IDs internally to aid troubleshooting without exposing internals.
- **Session identifier hardening**. Treat- `Mcp-Session-Id`as untrusted input; never tie authorization to it. Regenerate on auth changes and validate lifecycle server‑side.

## Related Standards and Documentation

MCP authorization builds on these well-established standards:- [OAuth 2.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-13)
- [RFC 8414](https://datatracker.ietf.org/doc/html/rfc8414)
- [RFC 7591](https://datatracker.ietf.org/doc/html/rfc7591)
- [RFC 9728](https://datatracker.ietf.org/doc/html/rfc9728)
- [RFC 8707](https://datatracker.ietf.org/doc/html/rfc8707)

# Citations

1. Source page: https://modelcontextprotocol.io/docs/tutorials/security/authorization
