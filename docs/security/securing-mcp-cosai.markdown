---
layout: default
title: "Securing the Model Context Protocol: What You Need to Know"
nav_order: 310
parent: MCP Security
has_children: false
---

# Securing the Model Context Protocol: What You Need to Know

_[Ian Molloy](https://www.linkedin.com/in/ian-molloy-8b951977/){:target="linkedin"}, IBM Research, and [Dean Wampler](https://www.linkedin.com/in/deanwampler/){:target="dw"}, The AI Alliance and IBM._
<br/>
_Published: September 16, 2025_

{: .note}
> **Editor's Note:** 
> This chapter summarizes the [CoSAI](https://www.coalitionforsecureai.org/){:target="cosai"} paper [Model Context Protocol (MCP) Security](https://github.com/cosai-oasis/ws4-secure-design-agentic-systems/blob/main/model-context-protocol-security.md){:target="paper"}. Ian is one of the workstream leads for this project, reflecting his deep expertise in security and what changes are required for secure MCP. -- [Dean]({{site.baseurl}}/contributing/#contributors)

The Model Context Protocol (MCP) is now a de facto standard for connecting AI agents and large language models (LLMs) to tools, data sources, and other services. A powerful, standard protocol simplifies the work developers have to do, but as adoption accelerates, so does the protocol’s security exposure.

The governing board for the Coalition for Secure AI, [CoSAI](https://www.coalitionforsecureai.org/){:target="cosai"}, has approved a new paper on MCP security, [Model Context Protocol (MCP) Security](https://github.com/cosai-oasis/ws4-secure-design-agentic-systems/blob/main/model-context-protocol-security.md){:target="paper"}. It provides the most comprehensive analysis to date of MCP’s threat landscape and offers concrete guidance for organizations deploying MCP in production.

Let's summarize the key points in this paper.

## Why MCP Security Is Different

Traditional application security frameworks assume deterministic software components operating behind well-defined trust boundaries. MCP breaks those assumptions. It places LLMs at the center of security-critical workflows, but LLMs are [Stochastic]({{site.glossaryurl}}/#stochastic){:target="_glossary"}, (i.e., probabilistic), meaning their outputs are not [Deterministic]({{site.glossaryurl}}/#determinism){:target="_glossary"}. Also, MCP servers dynamically advertise capabilities, negotiate access, and execute actions based on model-mediated decisions.

This architecture introduces new risks that cannot be addressed by conventional API or RPC security alone. Prompt injection, tool poisoning, over-privileged delegation, and invisible agent behavior are not edge cases; they are core concerns.

## A Real and Growing Threat Surface

The paper grounds its analysis in real incidents, including cross-tenant data leakage, privilege escalation via MCP-enabled plugins, and prompt injection attacks that exposed private databases. These are not theoretical vulnerabilities; they are already being exploited in early MCP deployments.

To bring clarity to this rapidly evolving space, the paper defines **twelve core MCP threat categories**, spanning identity, access control, input handling, data protection, transport security, supply chain risk, and operational governance.

For these categories, the framework articulates **34 specific threats** spread over three _tiers_ of threats:

* **Tier 1 \- MCP Specific Threats (7 Threats)**: Novel risks and threats due to MCP’s architecture and design decisions.   
* **Tier 2 \- MCP Contextualized Threats (8 Threats)**: known threats that manifest differently in MCP contexts or are amplified in MCP deployments.
* **Tier 3 \- Conventional Threats (19 Threats)**: security threats are are broadly applicable or derive from legacy, infrastructure, or transport implementation decisions.

Everything we know about "classic" deployment security still applies, but new concerns are introduced by LLMs and MCP.

## Key Risks Every MCP Deployment Faces

Among the most critical risks highlighted:

* **Identity and delegation failures**: Weak agent identity, confused-deputy OAuth flows, and excessive permissions allow silent privilege escalation.
* **Instruction–data boundary collapse**: MCP expands the attack surface for prompt injection by allowing tools, schemas, and resources themselves to influence model behavior.
* **Tool and resource poisoning**: Malicious manipulation of tool metadata or backend data can persistently hijack agent behavior.
* **Supply chain blind spots**: Unsigned MCP servers, unvetted community tools, and “shadow MCP servers” undermine trust and auditability.
* **Lack of observability**: Without comprehensive logging, organizations cannot trace agent actions or perform post-incident forensics.

## Defense in Depth for Agentic Systems

The paper advocates using the proven _defense-in-depth_ strategy, tailored to MCP’s unique architecture. Key recommendations include:

* **Strong agent identity and secure delegation**: Use modern OAuth patterns, token exchange, and least-privilege scopes.
* **Strict input validation and sanitization**: Treat all LLM-generated content as untrusted input.
* **Cryptographic integrity and verification**: This includes code signing, SBOMs, and (where feasible) remote attestation.
* **Sandboxing and isolation**: Go beyond basic containerization for MCP servers and LLM-generated code.
* **Transport-layer hardening**: Include TLS, mutual authentication, payload limits, and CSRF/CORS protections.
* **Lifecycle governance**: Use approved server _allow lists_, continuous vulnerability scanning, and automated detection of unauthorized deployments.
* **Comprehensive logging and auditability**: Across all hosts, clients, servers, and tools to ensure accountability.

## Deployment Context Matters

The paper also emphasizes that MCP security is highly deployment-dependent. An all-local developer setup has a radically different risk profile than a multi-tenant, cloud-hosted MCP server. Understanding trust boundaries, local vs. remote, single-tenant vs. multi-tenant, etc. is essential for applying the right controls.

## The Bottom Line

The [paper](https://github.com/cosai-oasis/ws4-secure-design-agentic-systems/blob/main/model-context-protocol-security.md){:target="paper"} explores all these topics in depth and offers guidance on best practices to follow.

MCP is becoming critical infrastructure for agentic AI systems. As with any infrastructure layer, insecure defaults and ad-hoc deployments will lead to systemic risk if left unaddressed.

[CoSAI's](https://www.coalitionforsecureai.org/){:target="cosai"} paper on [Model Context Protocol (MCP) Security](https://github.com/cosai-oasis/ws4-secure-design-agentic-systems/blob/main/model-context-protocol-security.md){:target="paper"} makes one message clear: **security cannot be an afterthought in MCP adoption**. Organizations that invest early in identity, isolation, supply chain integrity, and observability will be far better positioned as agentic AI moves from experimentation to enterprise-wide deployment.

For teams building or deploying MCP today, this guidance is not optional reading—it is a blueprint for responsible, scalable, and secure agentic systems.
