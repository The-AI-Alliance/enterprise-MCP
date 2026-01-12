---
layout: default
title: Home
nav_order: 10
has_children: false
---

# MCP (and Beyond) in the Enterprise: A User Guide

{: .tip }
> **TIP:** Use the search box at the top of this page to find specific content.

Welcome to the **AI Alliance** website, **MCP (and Beyond) in the Enterprise: A User Guide**. This guide provides guidance from leading experts on the [Model Context Protocol](https://modelcontextprotocol.io/introduction){:target="_blank"}, an emerging standard for AI agent and application communications, as well as alternatives and potential successor technologies. This guidance includes tips, best practices, and lessons learned building production deployments of distributed agents.

We are looking for more contributions! See our [contributing]({{site.baseurl}}/contributing) page for details.

## About MCP 

What is the Model Context Protocol? From the [MCP website](https://modelcontextprotocol.io/introduction){:target="_blank"}:

{: .highlight }
> MCP is an open protocol that standardizes how applications provide context to large language models (LLMs). Think of MCP like a USB-C port for AI applications. Just as USB-C provides a standardized way to connect your devices to various peripherals and accessories, MCP provides a standardized way to connect AI models to different data sources and tools. MCP enables you to build agents and complex workflows on top of LLMs and connects your models with the world.

[Anthropic](https://anthropic.com){:target="anthropic"} introduced MCP to make it easier for developers to integrate third-party web services with Anthropic's products. The simplicity and flexibility of MCP ignited wider interest. It is now a _de facto_ standard protocol supported by many AI service providers and open-source AI tools. Recently, the formation of the [Agentic AI Foundation](https://aaif.io/){:target="_blank"} under the [Linux Foundation](https://www.linuxfoundation.org/){:target="_blank"} was [announced](https://blog.modelcontextprotocol.io/posts/2025-12-09-mcp-joins-agentic-ai-foundation/){:target="_blank"} to promote the growth and standardization of MCP and related agent technologies.

{: .note }
> **NOTE:** This online guide is in the early stages of development. Check back regularly for new content. 
>
> We welcome feedback on our current content and suggestions for new additions. Actual submissions are even better! See [contributing]({{site.baseurl}}/contributing) or provide feedback and suggestions using our [discussion forum](https://github.com/The-AI-Alliance/enterprise-MCP/discussions){:target="discussions"}.
>
> In particular, the planned outline is being worked out in [this discussion topic](https://github.com/The-AI-Alliance/enterprise-MCP/discussions/4){:target="outline"}. It is summarized next. 

## Chapters in This User Guide

Here is the planned outline, with links to chapters that are already available.  

* [Getting to Know MCP and The Broader Ecosystem]({{site.baseurl}}/getting-to-know-mcp/)
  * A quick introduction to MCP
  * A dive into MCP under the hood
  * [MCP and Everyone Else: a Review of Inter-agent Communication Protocols]({{site.baseurl}}/getting-to-know-mcp/mcp-and-everyone-else/)
  * [NLIP, Agents, and Protocols]({{site.baseurl}}/getting-to-know-mcp/nlip/)
  * The MCP roadmap
* Managing MCP servers and how they are used in the enterprise.
  * How to evaluate third-party MCP servers for quality, security, and utility.
  * Tools for managing internal MCP server deployments.
  * Tools for controlling access to approved MCP servers.
  * …
* Security
  * OAuth for MCP
  * Securing your MCP servers
  * Known MCP vulnerabilities and what to do about them.
  * …
* [Developing MCP servers: tools, techniques, and design patterns]({{site.baseurl}}/developing-mcp-servers/)
  * [Building a Deep Research Agent Using MCP-Agent]({{site.baseurl}}/developing-mcp-servers/deep-research-mcp-agent/)
  * What makes a good MCP server?
  * Testing and debugging MCP servers.
  * Data stores as MCP servers: relational, documents, knowledge graphs, oh my!
  * …
* Technical Deep Dives
  * MCP internals
  * Optimizing performance of MCP servers
  * Future directions for MCP: extensions to support more use cases, etc.
  * …
* MCP in action: domain-specific examples
  * Lessons learned the hard way
  * Deep research agents in
    * Finance
    * Legal
    * …

## Other Resources on MCP

* [About MCP](https://modelcontextprotocol.io/introduction){:target="_blank"}
* The AI Alliance [glossary of terms](https://the-ai-alliance.github.io/glossary/glossary/){:target="_glossary"}.

## For More Information

* [Contributing]({{site.baseurl}}/contributing): We welcome your contributions! 
* [About Us]({{site.baseurl}}/about): More about the AI Alliance and this project.
* [The AI Alliance](https://aialliance.org){:target="aia"}: The AI Alliance website.
* This Project's [GitHub Repo](https://github.com/The-AI-Alliance/enterprise-MCP){:target="repo"}

| **Last Update** | {{site.last_version}}, {{site.last_modified_timestamp}} |
