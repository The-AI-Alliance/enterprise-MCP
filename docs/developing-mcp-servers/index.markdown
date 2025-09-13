---
layout: default
title: Developing MCP Servers
nav_order: 20
has_children: true
---

# Developing MCP Servers: Tools, Techniques, and Design Patterns

This section contains chapters about how to write effective MCP servers, including experience reports, tools to consider, and other guidance. Here is a summary of these chapters.

## Building a Deep Research Agent Using MCP-Agent

In [Building a Deep Research Agent Using MCP-Agent](deep-research-mcp-agent/), [Sarmad Qadri](https://www.linkedin.com/in/sarmadqadri/){:target="_blank"}, Co-founder and CEO at LastMile AI, and creator of [mcp-agent](https://github.com/lastmile-ai/mcp-agent){:target="mcp-agent"} documents the journey of building a **Deep Research Agent** with mcp-agent, highlighting the evolution from an initial _Orchestrator_ design, to an over-engineered _Adaptive Workflow_, and finally to the streamlined [Deep Orchestrator](https://github.com/lastmile-ai/mcp-agent/tree/main/src/mcp_agent/workflows/deep_orchestrator){:target="do"} now supported in mcp-agent. Sarmad emphasizes his view that &ldquo;MCP is all you need,&rdquo; and shows how connecting LLMs to MCP servers with simple design patterns enables agents to perform complex, multi-step research tasks. Key lessons include the importance of simplicity over complexity, leveraging deterministic, code-based verification alongside LLM reasoning, external memory for efficiency, and structured prompting for clarity. The resulting Deep Orchestrator balances performance, scalability, and adaptability, proving effective across domains like finance research. Future directions include remote execution, intelligent tool and model selection, and treating memory and knowledge as MCP resources. The open-source project, available on [GitHub](https://github.com/lastmile-ai/mcp-agent){:target="mcp-agent"}, offers developers a powerful foundation for creating general-purpose, AI research agents.

## Coming Soon

We plan to include chapters on the following topics:

* What makes a good MCP server?
* Testing and debugging MCP servers.
* Data stores as MCP servers: relational, documents, knowledge graphs, oh my!

{: .todo}
> **TODO:** We welcome feedback on our current content and suggestions for new additions. Actual submissions are even better! See [contributing]({{site.baseurl}}/contributing) or provide feedback and suggestions using our [discussion forum](https://github.com/The-AI-Alliance/enterprise-MCP/discussions){:target="discussions"}.

---