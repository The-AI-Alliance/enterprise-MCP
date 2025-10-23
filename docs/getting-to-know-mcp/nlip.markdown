---
layout: default
title: NLIP, Agents, and Protocols
nav_order: 220
parent: Getting to Know MCP and The Broader Ecosystem
has_children: false
---

# NLIP, Agents, and Protocols

<br/>
<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/nlip/NLIPlogo.png" alt="NLIP Logo" title="NLIP Logo" width="200">
</div>

_[Tom Sheffler](https://www.linkedin.com/in/tom-sheffler/){:target="linkedin"} is a computer engineer currently working towards interoperability of secure agentic systems as part of the NLIP project.  Previously, Tom built global web applications for managing and scaling genomics computations at Roche. As an entrepreneur, he founded the cloud-based video analytics company, Sensr.net, that provided a consumer solution for home security cameras.  Tom was an early employee at Rambus where he worked on the architecture and verification of high-speed memory devices._
<br/>
_Published: October 21, 2025_

{: .note}
> **Editor's Note:** 
> Tom is a member of the [NLIP project](https://github.com/nlip-project). NLIP complements MCP, as well as other protocols, in many ways, so I asked Tom to contribute this chapter about NLIP, which is likely to become an important tool for your AI projects. -- [Dean]({{site.baseurl}}/contributing/#contributors)


[NLIP (Natural Language Interaction Protocol)](https://github.com/nlip-project){:target="_blank"} defines a payload format for transferring natural language conversations augmented with multimodal media.  Its intended use is as a transport between a user-facing application and an agent, or between two agents.  NLIP has defined bindings for HTTP, WebSockets and AMPQ making it useable in a variety of scenarios.  NLIP joins a roster of protocols used in agent development.  These include MCP, A2A, and OpenAI API. 

This article will introduce NLIP and describe the motivation for its development.  To do so, the architecture of a hypothetical prototype agent will be introduced, and its components will be described.  As this initial monolith of an architecture is decomposed into separate pieces, the protocols enabling the externalization of the various components will be defined.  Finally, agent-to-agent interactions will be examined.

## A Hypothetical Prototype Agent

An agent is a software system that employs an LLM to perform tasks on natural language inputs, with natural language instructions.  In contrast to legacy systems in which the "business logic" of the software would be implemented in a programming language, an agent can implement equivalent functionality in natural language instructions called "prompts."

Both of these aspects bring exciting new possibilities.  Natural language inputs and outputs open up new interaction mechanisms with users.  Users can type messages or even speak directly to agents through a software "chat" application.  By eliminating traditional programming from some aspects of prompt engineering, people without formal programming language training can write methodical instructions for LLMs to follow to update databases, interact with users and perform a wide variety of tasks through tool calls.

Agents come in many forms, but all employ an LLM.  Some of the common elements of an agent architecture are shown in **Figure 1**.  The elements of this figure are described below.

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/nlip/Slide2.png" alt="A hypothetical monolith agent architecture" title="A hypothetical monolith agent architecture">
  <p>
    <strong>Figure 1:</strong> A hypothetical monolith agent architecture
  </p>
</div>

* **Input/Output** - an agent has an input/output channel for interacting with its primary user (a person or an upstream agent).  The channel accepts messages that contain natural language text and optional other data types like pictures, audio, and video.  The agent presents messages as output which again can contain text, pictures, audio, and other data types.
* **Conversation History** - the agent maintains the conversation history of a session.  The conversation history is a list of messages.  The list consists of input messages from a user, output messages from the LLM, and "instruction" messages derived from prompts. 
* **LLM** - the LLM (Large Language Model), strictly speaking, generates sequences of tokens given a sequence of tokens as input.  Less formally, the LLM generates a stochastic response that represents a likely next message given the conversation history reference resources.  Today's LLMs are taught how to invoke software "tools" (function calls) and the response of an LLM may, in fact, be a request to invoke a tool and add its result to the conversation history, and call the LLM again.  Each query/response iteration with the LLM is called a "turn."
* **Tool** - a tool is a traditional function call in a programming language.  A tool might make an HTTP request to an information server, or perform an operation on a database.  Tool calls can also be used to modify the execution environment of an agent.  Sometimes additional instructions (prompts) are necessary to "teach" the LLM how to use the tool correctly in the environment of the conversation.
* **Prompt** - a prompt is a message directed to the LLM from the "system" as opposed to a message from a "user."  Prompt instructions have very high priority.  For this reason, the introduction of prompts to the agent should be carefully controlled.
* **Resource** - a resource is a document or embedding that can be consulted by the LLM and adds context to the session.
* **Application Logic** - the application logic orchestrates processing of input messages, turns with the LLM and the incorporation of tool calls and results into the conversation history.


## NLIP standardizes the Input/Output channel

[NLIP](https://nlip-project.org/){:target="nlip"} (Natural Language Interaction Protocol) is an open specification that standardizes the messages on the input/output channel.  In addition to the message format, the NLIP project gives recommendations for secure message transport over HTTP, WebSockets, and AMQP.  In the hypothetical agent architecture of this paper, NLIP is used to serialize the messages on the front-end of the agent over a channel as shown in **Figure 2**.

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/nlip/Slide3.png" alt="NLIP many-turn conversation" title="NLIP many-turn conversation">
  <p>
    <strong>Figure 2:</strong> NLIP many-turn conversation
  </p>
</div>

The consumer of this channel is a software application employed by a user.  This may take the form of a "chat" application where a user can converse with the agent in text and include other media from the device.  It might also take the form of an audio chat agent where the user is conversing with the agent and the message stream consists of audio phrases.

An NLIP conversation is assumed to be many-turned.  That is, there may be many request/response pairs.  A user might ask a first question, receive a response, and then follow up with a second question.  In that case, the first question and answer provides context for the second question, etc.  The user's software application may keep a copy of the entire conversation, or it might not.  The NLIP channel simply transfers the series of messages of the conversation as they occur.


### The NLIP Project

The [NLIP project](https://nlip-project.org/){:target="nlip"} is open, and includes members from industry and academia.  To name just a few, it includes industrial representatives from [IBM](https://ibm.com){:target="ibm"}, [RedHat](https://redhat.com){:target="redhat"}, [Cisco](https://cisco.com){:target="cisco"}, [ServiceNow](https://servicenow.com){:target="service-now"}, as well as academic members from [Purdue](https://purdue.edu){:target="purdue"} and [Indiana](https://bloomington.iu.edu/){:target="indiana"}.  Working meetings are organized under the auspices of the [Enterprise Neurosystem](https://www.enterpriseneurosystem.org/) with formal documents submitted to Ecma [Technical Committee 56](https://ecma-international.org/technical-committees/tc56/).  In contrast to agent software development kits controlled by one company, the goal of the NLIP project is to define an interoperable protocol that is truly open and community governed.

At the present time (October 2025), five final documents have been submitted to Ecma with approval expected in December 2025.

- tc56-2025-026: Natural Language Interaction Protocol (NLIP)
- tc56-2025-027: Binding of Natural Language Interaction Protocol (NLIP) over HTTP/HTTPS
- tc56-2025-029: Binding of the Natural Language Interaction Protocol (NLIP) over WebSocket
- tc56-2025-030: Binding of the Natural Language Interaction Protocol (NLIP) over AMQP
- tc56-2025-031: Security profiles for Natural Language Interaction Protocol (NLIP)

### NLIP Message Stream

As stated earlier, an NLIP message consists of text and/or other media.  The NLIP project has defined encoding schemes for media such as audio, images and video, as well as documents and structured information, as shown in **Figure 3**.

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/nlip/Slide4.png" alt="What is in an NLIP Message Stream?" title="What is in an NLIP Message Stream?">
  <p>
    <strong>Figure 3:</strong> What is in an NLIP Message Stream?
  </p>
</div>

An NLIP message contains:
- **message type**: Identifies whether the message is conversation, control (a message about the transport) or error.
- **format**: A content type (text, image, audio, video or document).
- **subformat**: A refinement of the content type ("jpg", "png").
- **content**: The content itself, potentially encoded.
- **submessages**: A list of message parts, each with **format**, **subformat** and **content** fields.

Notably, the NLIP message does not define a "role" since that information is implied by the sender of the message.

The simplest NLIP message contains only a single text part with no other components.  An example is shown below.

```json
{
    "format": "text",
    "content": "Now is the time for all good men to come to the aid of their country."
}
```

In some ways the NLIP Message is similar in intent to the message types defined in some well-known AI and Agent Construction frameworks.  A few are listed below.

- [Agent Connect Protocol](https://spec.acp.agntcy.org/#model/message){:target="_blank"}
- [Autogen Messages](https://github.com/microsoft/autogen/blob/main/python/packages/autogen-agentchat/src/autogen_agentchat/messages.py){:target="_blank"}
- [LangChain `AIMessage`](https://python.langchain.com/api_reference/core/messages/langchain_core.messages.ai.AIMessage.html){:target="_blank"}
- [A2A Protocol `Message` object](https://a2aproject.github.io/A2A/latest/specification/#64-message-object){:target="_blank"}

The main difference being that NLIP is not tied to one particular toolkit and its definition is community driven.  NLIP is also intended to be serialized over a secure transport to a consumer software application and the NLIP project provides recommendations for what security entails in an agentic context.

### Why Standardize?

Standardization of the input/output channel of agents enable an open ecosystem of interoperable user software and agents.  The developers of user-facing software will be able to target one message back-end that will be compatible with many agents.  Providers of agents will likewise be able to expose one interface that can be consumed by both software applications and other agents.  NLIP will enable multi-turn conversations for agent-to-agent interactions.

Just as SMTP enabled any email client to communicate with any email server, NLIP aims to enable any chat application to communicate with any agent, or any agent to talk to any other agent.

Today's agent ecosystem is in a similar state to pre-standardization email.  Many agents exist, but most are deployed within enterprise applications.  Someday soon, people will enjoy using their own chat applications with a variety of webservices fronted by agents.  It is important that an interoperable, secure standard is defined today for this emerging agentic future, as illustrated in **Figure 4**.

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/nlip/Slide5.png" alt="Why Standardize?" title="Why Standardize?">
  <p>
    <strong>Figure 4:</strong> Why Standardize?
  </p>
</div>

## MCP

Let's discuss the implications for several popular protocols, starting with MCP.[^1]

[^1]: See also [MCP and Everyone Else - A Review of Inter-agent Communication Protocols](mcp-and-everyone-else/).

MCP (Model Context Protocol) is a serialization protocol for externalizing Tools, Prompts and Resources from the hypothetical agent monolith of the introduction.  MCP was released by Anthropic in November of 2024 and has been quickly adopted in a wide variety of applications --- even agent-to-agent scenarios. **Figure 5** illustrates MCP's use.

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/nlip/Slide6.png" alt="Model Context Protocol" title="Model Context Protocol">
  <p>
    <strong>Figure 5:</strong> Model Context Protocol
  </p>
</div>

By far, the most widely used ability provided by MCP is the externalization of tools into MCP servers.  For this purpose, MCP defines a very complete protocol for attaching a server to an LLM, querying the server for its capabilities and invoking the tools.  The main steps in tool utilization are the following.

1.  At system start-up time, the agent queries the tool server for the tools present and their function argument signature.  JSON Schema is used to define arguments and very rich types are supported.  It is possible to call tools whose arguments are "lists-of-structures" for instance.  At the time of this writing, MCP tool schemas focus on input parameters, while output schemas are not specified.  A future version should consider formalizing output schemas.

2. The list of tools and their signatures is delivered to the LLM.

3. When a tool is "invoked" by an LLM, its arguments are packaged up in a JSON serialization and delivered to the external server with a "tool-call" message.

4. The results of the tool-call are transferred back to the agent in a "tool-result" message.  These results are added to the conversation history for the LLM to consult on the next turn.

MCP defines transports over (local) sockets, HTTP (with optional Server-Sent Events) and WebSockets.

MCP servers are often providing tools that operate directly on critical data (enterprise databases, for example).  For this reason, the security of an MCP server should be considered carefully.

The MCP Project has produced [security guidelines](https://modelcontextprotocol.io/specification/draft/basic/security_best_practices){:target="_blank"} that identify best practices for securing MCP servers.

In practice, many providers of MCP servers protect their servers with OAuth2 flows.


## OpenAI API

Continuing the deconstruction of the monolith agent into components, it is almost always the case that the LLM is externalized from the agent and its interface is a serialization protocol, as shown in **Figure 6**.

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/nlip/Slide7.png" alt="OpenAI API" title="OpenAI API">
  <p>
    <strong>Figure 6:</strong> OpenAI API for Inference
  </p>
</div>

This serialization is an HTTP API with JSON payloads.  The [OpenAI API](https://openai.com/api/){:target="_blank"} is a de-facto standard that has been adopted by other companies.  For instance, Anthropic's Claude API is very similar, and the API exposed by Ollama is also very similar.  Because of the similarities in these LLM protocols, a number of software SDKs have been developed to unify their use.  One such SDK is [LiteLLM](https://https://www.litellm.ai/){:target="_blank"}.  It abstracts LLM APIs into one uniform interface so that agent application code can be written in such a way that it is unaware of whether the LLM is local, hosted on Azure or AWS, or whether it is accessed through an LLM aggregator gateway.

NLIP differs from OpenAI API in that it does not require the transfer of the conversation history: NLIP only transfers the messages of the conversation.  In addition, the NLIP project defines security profiles to help guide interoperable security assumptions when used as an Agent-to-Agent transport.


## A2A

[Agent2Agent (A2A)](https://a2a-protocol.org){:target="_blank"} is an open standard developed by Google.  It has been donated to the Linux Foundation, which will help promote it as an open-source standard.  A2A protocol has two main parts: discovery and interaction.

Discovery is the means by which a software or agent can determine the capabilities of an A2A agent.  Given a URL, A2A defines an [Agent Card](https://a2a-protocol.org/dev/specification/#55-agentcard-object-structure){:target="_blank"} that describes key features of the agent.  These include a description of the agent's specialties, its security schemes, and the modes in which input and output are transmitted.

The fundamental unit of work that an agent performs in the A2A protocol is the "task".  In an A2A interaction, one agent offloads a task to another agent and then waits for or polls for task completion.  The result of a task is called an "artifact." **Figure 7** illustrates A2A's task model.

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/nlip/Slide8.png" alt="Agent2Agent Task Model" title="Agent2Agent Task Model">
  <p>
    <strong>Figure 7:</strong> Agent2Agent Task Model
  </p>
</div>

The interaction model of A2A is slightly different than that provided by NLIP.  NLIP is inherently multi-turn, and does not necessarily have a conversation completion.  Once a user receives a response, they may want to continue with a new query or remark.  An A2A task may participate in a multi-turn conversation by asking for more information.  The two interaction models are subtly different. **Figure 8** illustrates this point.

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/nlip/Slide9.png" alt="NLIP is inherently multi-turn" title="NLIP is inherently multi-turn">
  <p>
    <strong>Figure 8:</strong> NLIP is inherently multi-turn
  </p>
</div>

### MCP for A2A

MCP is sometimes used to construct agent-to-agent configurations.  One way that this is done is shown in **Figure 9**.  Since MCP is useful for bringing additional context into an LLM and remote tool calls are easy to implement, this idea exposes a remote agent as a "tool" invoked over MCP protocol.

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/nlip/Slide10.png" alt="MCP configuration for Agent-to-Agent" title="MCP configuration for Agent-to-Agent">
  <p>
    <strong>Figure 9:</strong> MCP configuration for Agent-to-Agent
  </p>
</div>

The remote agent system consists of an MCP server and the agent itself.  This configuration replaces the input/output channel of the prototype agent of this discussion with a tool.  The signature of the tool might look like the following.

``` python
def remote_agent_tool(input: InputType) -> OutputType:
   ...
```

This architecture suffices to connect the two agents and conveniently uses MCP as the transport mechanism.  It does not, however, satisfy any sort of interoperability goal.  Without defining `InputType` and `OutputType` as a standard, each such agent pairing will remain a custom implementation.  The NLIP project is essentially standardizing `InputType` and `OutputType` as `NLIPMessage`, and is also defining a secure transport for NLIP messages in a conversation.

MCP is a very rich protocol.  Besides tool calls, there is tool discovery and dynamic resource discovery.  NLIP focuses only on the transfer of conversations and provides guidelines for its secure use with agents.

### Another Protocol of Note

A recent addition to the Agent-to-Agent is the [AMTP Protocol](https://amtp-protocol.org){:target="amtp"}.  This protocol uses the concept of an email-like address as the means to address an agent.  A special DNS record is used to identify AMTP agents and to learn where to query about their capabilities.

AMTP supports both interactive request/response interactions and also offline send-and-forget forwarding.  There are some interesting ideas in this project, even though adoption has not taken off.

## Summary

This chapter examined a number of protocols in the agent space by using the architecture of a hypothetical agent constructed as a monolith as a starting point.  Protocols exist to decompose this monolith by defining serialization formats and secure transports for different components of an agent.

When designing an agentic system, it is important to choose the right protocol for the right job. 

- Use NLIP for multi-turn multimodal conversations between
    - User-Agent and Agent
    - Agent and Agent
- Develop User-facing chat interfaces for NLIP rather than agent-specific interfaces, enabling one application to work with multiple agent providers
- Use MCP to externalize Tools, Resources and Prompts
- Use A2A to offload Tasks to specialized agents


The landscape of agentic frameworks and agent-to-agent protocols is evolving quickly.  Agents are shifting from simply responding to queries to autonomously pursuing goals, responding to stimuli, and making plans, sometimes with minimal human intervention.  Security in these systems is increasingly important and new methods of delegation of permissions will be required as people become more accustomed to offloading tasks to their agents.

The agentic approach to system development is a new computational paradigm that demands new security foundations.  The NLIP Project addresses these emerging challenges by providing a firm transport baseline with security profile definitions ensuring interoperability and trust.  We will examine the NLIP security profiles in a followup chapter.

