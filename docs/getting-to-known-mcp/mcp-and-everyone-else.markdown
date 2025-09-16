---
layout: default
title: MCP and Everyone Else - A Review of Inter-agent Communication Protocols
nav_order: 310
parent: Getting to Know MCP and Its Ecosystem
has_children: false
---

# MCP and Everyone Else - A Review of Inter-agent Communication Protocols



_Laurie Voss is VP of Developer Relations at LlamaIndex, and former co-founder of npm Inc._
<br/>
_Published: September 16, 2025_

{: .note}
> **Editor's Note:** 
> This chapter is adapted from a [talk Laurie presented](https://www.youtube.com/watch?v=kqB_xML1SfA){:target="youtube"} ([slides](https://slides.com/seldo/comparing-agent-protocols){:target="slides"}) at the first [MCP Developers Summit](https://luma.com/mcpdevsummit2025){:target="mcp-dev-summit"}. I loved that talk and wanted Laurie to reprise what he learned in this user guide. I recommend that you watch the video for his witty commentary. -- [Dean]({{site.baseurl}}/contributing/#contributors)

It was promised that 2025 would be the year of agents and that seems to have broadly held true. Every company in the AI space seems to be building an agent or even an agent framework. This proliferation of agents has led to a natural next question: how, in a world of thousands or millions of useful, active agents, does one get those agents to inter-operate and ideally co-operate?

The emerging answer to that question has resulted in a secondary proliferation of inter-agent protocols. In my review I have looked at [13 different protocols](https://slides.com/seldo/comparing-agent-protocols#/5){:target="slides"} but there are surely more out there, and the space is moving very quickly indeed.

## **The Vision vs. Reality**

Nearly everybody working on these protocols is aiming at roughly the same goal: agents can do amazing things, and they should be able to do amazing things *together*. The vision is that agents, regardless of their underlying implementations, should be able to:

* Discover other agents wherever they exist on the internet  
* Authenticate and communicate with them  
* Form coalitions to solve problems  
* Collaborate, negotiate, debate, and respond  
* Resulting in a brave new world of autonomous software

The current reality is quite different. The more than a dozen protocols that exist are often in the early design or implementation stages, and vary widely in scope and completeness.

The closest thing to a standard in the agent communication space is MCP, which has been growing by leaps and bounds in adoption, but currently at least MCP does not have inter-agent communication as an explicit design goal. So, if you’re looking for a solution for inter-agent communication, should you look elsewhere, and if so where? That’s the question this post explores.

## **A Helpful Taxonomy**

Thanks to an incredibly valuable paper by Yang et al. (["A Survey of AI Agent Protocols"](https://arxiv.org/abs/2504.16736){:target="arxiv"}, 2025), we can organize these protocols into a useful taxonomy that splits them into two main categories:

**Context-Oriented Protocols**: The goal is to provide an LLM with context to achieve its goal. Basically, tool use. The LLM makes a request and gets a synchronous response. This category includes MCP, which as mentioned is not explicitly about inter-agent communication.

**Inter-Agent Protocols**: The goal is for agents to communicate with each other, going beyond request-response to discuss, negotiate, and debate before reaching agreements.

This taxonomy raises an interesting and important question (raised in the paper itself) that we’re going to explore further later in this post: what is the difference, philosophically and technically, between an agent calling a tool, and an agent calling another agent?

## **The Protocol Landscape**

Now let’s do a lightning review of the protocols themselves.

### **Context-Oriented Protocols**

[**MCP (Model Context Protocol)**](https://modelcontextprotocol.io/docs/getting-started/intro){:target="mcp-intro"}: If you’re reading this, I assume you already know what MCP is. It's got tools, discovery, auth, request-response architecture, streaming, and as of September 8th, [a registry](https://blog.modelcontextprotocol.io/posts/2025-09-08-mcp-registry-preview/){:target="registry"}, which didn’t exist at the time of the original review.

[**Agents.json**](https://docs.wild-card.ai/agentsjson/introduction){:target="agents"}: The only other major context-oriented protocol. It's a file that sits at `/.well-known/agents.json` on your domain, describing what APIs you expose and crucially, how an agent should use them. It is based on OpenAPI but goes further, telling agents how to chain API calls together to achieve goals.

### **Inter-Agent Protocols**

[**A2A (Agent to Agent Protocol)**](https://a2a-protocol.org/latest/){:target="a2a"}: If you've heard about any other protocol in this space, it's probably this one. Built by Google and recently donated to the Linux Foundation for ongoing development, it's expecting to call *agents* rather than tools, so everything is async. You make a call, get a status, then poll until you get a result. This async nature is a strength if the agent will take days to answer.

The A2A specification covers transport, authentication, authorization, and discovery via an “agent card” which works rather like agents.json does: if you know the domain of an agent you can discover its card at a well-known location, which details the agent and how it works. A2A’s launch came with a long list of high-profile enterprise backers.

[**ANP (Agent Network Protocol)**](https://agent-network-protocol.com/){:target="anp"}: Similar to A2A in many ways, ANP uses the W3C's DID standard for decentralized identity as a way of identifying agents reliably. The DID standard came originally out of the cryptocurrency community, but no longer relies on blockchain for its implementation. Those origins show up in the protocol’s keen interest in decentralization.

[**AITP (Agent Interaction & Transaction Protocol)**](https://aitp.dev/vision){:target="aitp"}: Another protocol with roots in the blockchain community, AITP is very concerned with interaction costs and allowing competing agents to bid on problems. This is a real problem: in a world of agents interacting with agents, who pays for the work being done and how?

[**ACP (Agent Connect Protocol)**](https://spec.acp.agntcy.org/){:target="acp1"} from Cisco: Has a nice API and SDK with adapters for LangChain and LlamaIndex. ACP is more substantially specified and implemented than other protocols in this list, but not much differentiation. Interestingly, it describes how to host and launch agents, so you can either call a discovered agent or deploy and run it yourself.

[**ACP (Agent Communication Protocol)**](https://agentcommunicationprotocol.dev/){:target="acp2"} from IBM: Yes, there are two ACPs. This one is a fork of MCP optimized for agents rather than tools. Solving a very similar set of problems to A2A using MCP as a jumping-off point, it begs the question of why these modifications would not just be part of MCP itself in the long term.

[**Agora**](https://agoraprotocol.org/){:target="agora"}: A fascinating, LLM-first approach where communication starts as natural-language queries. Agents can send "protocol documents"—natural-language descriptions of protocols—and upgrade their communication from ambiguous natural language to structured protocols on the fly. Very interesting idea, but still mostly at the idea stage.

There are also several domain-specific protocols (PXP, LOKA, CrowdES, Spatial Population Protocols) designed to handle things like robot communication, and efforts like [LMOS](https://eclipse.dev/lmos/){:target="lmos"} (from the well-respected [Eclipse foundation](https://eclipse.org){:target="eclipse"}) and [Agent Protocol](https://agentprotocol.ai/){:target="ap"}, but they're either too specialized or too similar to what we've already covered.

## **MCP's Position**

So where does MCP stand in all this? The A\* protocols (I call them that because they nearly all start with A) collectively want agents to get data from other agents. MCP wants models to get context for things. As we asked earlier: is there really a difference between those two things? If a model is getting context from a tool, doesn’t that make it an agent? And if the thing it gets the context from is a procedural tool or another model-powered agent, isn’t that more of a semantic difference than a technical one?

A2A argues there is a real differentiation here, because inter-agent communication involves long response times, multi-turn negotiation, authorization, and payment considerations. These are valid concerns for an inter-agent protocol to have, but MCP already has rapidly-developing stories for both authentication and authorization, and the beginnings of multi-turn negotiation.

What does MCP lack relative to the other protocols listed above? Not very much, and even less since this initial review in April 2025\. If you know where an MCP server is, you can discover its tools and resources. It arguably lacks a good async story (something A2A does better), but that’s a relatively small point of differentiation that a few tweaks to MCP could address.

## **MCP's Secret Weapon: Adoption**

What MCP has that none of these others have is adoption. MCP has clearly demonstrated utility and rapidly increasing momentum in terms of usage, contributors, and features. When it comes to forming a de-facto standard, it has the attention of the public, and that may be the most critical requirement for a successful internet protocol.

Can MCP work with other protocols? That's the A2A perspective: MCP features in their architecture diagrams as the tool-use solution while they handle inter-agent concerns. But I'm not convinced. MCP is highly duplicative of these other protocols. Adding another protocol seems to add complexity for minimal gain, while adding inter-agent features to MCP seems like a relatively small lift.

## **What's Missing (from all the options)**

So is MCP perfect? No. There are key things that all these protocols lack, including MCP:

### **1\. A Registry**

At the time of the review, MCP lacked a centralized registry to discover agents. As of a few days prior to publication of this chapter, that [has changed](https://registry.modelcontextprotocol.io/){:target="registry"}, but it remains to be seen how successful it will be or whether it will see wide adoption.

Regardless, a registry is essential, and solutions based on well-known files at domains are insufficient: you have to know a server exists before you can ask it to do anything. But how do you find those servers in the first place? How does an agent starting from nothing discover new tools or other agents? At some point, you need to be able to search, and a unified (even if distributed) data source is necessary for search to work.

### **2\. Payment**

Many protocols sketch authentication, but authorization is different. In a world full of agents advertising themselves on a central registry, how do you get authorization to ask them to do things? Agents are expensive, LLM-powered machinery. We can't assume everyone will run them for free. The blockchain protocols have a valid problem here, if not a solution.

### **3\. Reputation**

In a universe of discoverable agents, there will be many doing similar things. You need a way to identify which ones are good and which just say they are. None of the protocols have anything to say about reputation between agents, and this is going to be critical to discovery and automated adoption where humans are not necessarily going to be reviewing every decision made by their agents.

## **Where do we go from here?**

There are tons of open questions, but my conclusion is that MCP is doing great work and so far it's all we need. The protocol landscape is fragmented, but MCP's adoption advantage is significant.

Rather than fragmenting further with competing protocols, the community might be better served by extending MCP to handle the inter-agent use cases that other protocols are trying to solve. The missing pieces—payment and reputation chief among them—are problems that need solving regardless of which protocol wins.

It’s an exciting time to be in this space, and I look forward to what comes next.

