---
type: Web Page
title: Versioning - Model Context Protocol
resource: https://modelcontextprotocol.io/docs/learn/versioning
timestamp: '2026-07-07T10:31:48.208319+00:00'
---

`YYYY-MM-DD`, to indicate the last date backwards incompatible changes were made.
The protocol version will 

*not*be incremented when the protocol is updated, as long as the changes maintain backwards compatibility. This allows for incremental improvements while preserving interoperability.## Revisions

Revisions may be marked as:- **Draft**: in-progress specifications, not yet ready for consumption.
- **Current**: the current protocol version, which is ready for use and may continue to receive backwards compatible changes.
- **Final**: past, complete specifications that will not be changed.

**current**protocol version is

**2025-11-25**.

## Feature States

Individual features of the specification may additionally be marked as**Deprecated**under the feature lifecycle and deprecation policy: the feature remains part of the specification, but is scheduled for removal. Deprecated features document a migration path (or state that none is required) and remain in the specification for at least twelve months, or at least ninety days under the policy’s expedited-removal exception, before they become eligible for removal, after which they may be

**Removed**in a future revision. Features that are currently Deprecated are listed in the deprecated features registry.

## Negotiation

Version negotiation happens during initialization. Clients and servers**MAY**support multiple protocol versions simultaneously, but they

**MUST**agree on a single version to use for the session. The protocol provides appropriate error handling if version negotiation fails, allowing clients to gracefully terminate connections when they cannot find a version compatible with the server.

# Citations

1. Source page: https://modelcontextprotocol.io/docs/learn/versioning
