---
layout: default
title: The MCP Standard
nav_order: 210
parent: Getting to Know MCP and The Broader Ecosystem
has_children: false
---

# The MCP Standard

_[Dean Wampler](https://www.linkedin.com/in/deanwampler/){:target="dw"} is IBM's Chief Technical Representative to the AI Alliance and the editor of this user guide._

<br/>
_Published: May 27, 2026_

{: .note}
> **Editor's Note:** 
> This chapter summarizes the highlights from the blog post [The 2026-07-28 MCP Specification Release Candidate](https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/){:target="spec-blog"}, which announced the draft standard for MCP.

MCP's popularity and rapid adoption quickly surfaced the need to standardize existing features of MCP, to make them unambiguous for implementers and users, as well as the need to identify and fill gaps that become potential blockers for even wider adoption.

The MCP community laid out a [2026 roadmap](https://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/){:target="roadmap"} for moving MCP forward. The [May 21, 2026 blog post](https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/){:target="spec-blog"} announced a major milestone on that roadmap, the [draft standard for MCP](https://modelcontextprotocol.io/specification/draft){:target="spec"}. The blog post also provided an overview of what's new. This chapter summarizes details in that post. 

## What's New?

In general, the changes clarify and standardize a number of behaviors, fill gaps, simplify client use and enhance scalability and security. The standard also clarifies how to support improving MCP going forward.

In brief, here are the significant updates to MCP introduced by the specification.

### MCP Is Now Stateless at the Protocol Level

Previously, a session id had to be acquired as part of session initialization and used for all subsequent interactions. Among the disadvantages is limiting scalability, as the same server has to serve all requests, with no ability to load balance if that server becomes heavily used while others have more capacity.

Now, the separate initialization step is gone and a single request for service is all that is required. Any available server can handle the request.

Applications can still have sessions, using application-level, ad-hoc mechanisms for generating and passing session tokens, which are widely used in HTTP APIs.

It used to be that a persistent connection was required for the server to ask for something of the client, such as an _elicitation_ prompt. A new mechanism is introduced where the server returns its request for information and the client reissues the initial invocation with all the information passed before as well as the new information needed. This enables the desired stateless behavior at the protocol level while supporting features like elicitation. 

Making request easier to route is provided by new required `Mcp-Method` and `Mcp-Name` headers, which enable routing, load balancing, and rate limiting to be done without inspection of the message body.

A cache control mechanism, similar to HTTP `Cache-Control`, lets clients know how long a tool or list response will be fresh and other benefits. A long-lived SSE stream is no longer necessary to learn of such changes.

Finally, distributed tracing is standardized with the adoption of W3C trace context propagation.

### Extensions to MCP

The ad-hoc process of adding proposed extensions to MCP has been standardized, providing a clean separation where needed in operational details, as well as providing a clear path for extensions to go from experimental to official.

Two official extensions are included in the release:

1. Server-rendered user interfaces for MCP apps enables servers to ship HTML interfaces that will be displayed in sandboxed `iframes.` Interactions are mediated using the same JSON-RPC base protocol used everywhere else in MCP, supporting the same audit, every UI-initiated action goes through the same security, auditing, and other processes used for other interactions.
* Tasks are now extensions. At first an experimental addition to the MCP core, support for the full lifecycle of tasks is now implemented as an extension with breaking API changes.

### Authorization Hardening

The authorization part of the specification is now aligned more closely with how OAuth 2.0 and OpenID Connect are deployed in practice.

### Deprecated Core Features

Three core features are deprecated under a new feature lifecycle policy:

* Roots are replaced with tool parameters, resource URIs, or server configuration.
* Sampling is replaced with direct integration with LLM provider APIs
* Logging is replaced with `stderr` for stdio transports and  OpenTelemetry for structured observability.

These are annotation-only deprecations. they will continue to work in this release and in every specification published within a year of it. Removing them will be done with individual SEPs (specification enhancement proposals).

### Full JSON Schema 2020-12 for Tools

Tool `inputSchema` and `outputSchema` now fully support the full JSON Schema 2020-12 (SEP-2106). This provides some additional flexibility where useful, but also tightens some behaviors and replaces some custom behaviors with JSON standard alternatives.

## What's Next?

The draft standard is currently under review. Feedback is welcome. If you find a problem, you can [open an issue](https://github.com/modelcontextprotocol/modelcontextprotocol/issues){:target="spec"} in the [specification repository](https://github.com/modelcontextprotocol/modelcontextprotocol/){:target="spec"}. The blog post mentions other avenues for providing feedback, too.

The plan is to publish the final standard July 28, 2026. We will update this chapter with final changes when that happens. 
