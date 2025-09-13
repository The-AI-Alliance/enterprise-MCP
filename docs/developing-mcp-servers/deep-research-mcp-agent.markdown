---
layout: default
title: Building a Deep Research Agent Using MCP-Agent
nav_order: 210
parent: Developing MCP Servers
has_children: false
---

# Building a Deep Research Agent Using MCP-Agent

_[Sarmad Qadri](https://www.linkedin.com/in/sarmadqadri/){:target="_blank"}, Co-founder and CEO at LastMile AI, and creator of [mcp-agent](https://github.com/lastmile-ai/mcp-agent){:target="mcp-agent"}_

{: .note}
> **Note:** This chapter was originally published as an [AI Alliance blog post](https://thealliance.ai/blog/building-a-deep-research-agent-using-mcp-agent){:target="_blank"}

In this chapter, I document my journey building a general-purpose, deep research agent powered by MCP, and sharing the valuable (and sometimes painful) lessons learned along the way.

<div style="text-align: center;" id="figure-1">
  <img src="{{site.baseurl}}/assets/images/deep-research-agent-architecture.png" alt="Deep Research Agent Architecture" title="Deep Research Agent Architecture">
  <p>
    <strong>Figure 1:</strong> The <em>Final</em> Deep Research Agent Architecture
  </p>
</div>

## Background

My name is Sarmad Qadri and I'm the creator of the open source project, [mcp-agent](https://github.com/lastmile-ai/mcp-agent){:target="mcp-agent"}. My philosophy for agent development in 2025 can be summarized as this: **MCP is all you need**. Put more verbosely: _connect state-of-the-art LLMs to MCP servers, and leverage simple design patterns to let them make tool calls, gather context, and make decisions._

Over the past few months, I've been asked many times when mcp-agent would support deep research workflows. So I set out to build an open source general purpose agent that can work like [Claude Code](https://www.anthropic.com/claude-code){:target="_blank"} for complex tasks, including deep research, but also multi-step workflows that require making MCP tool calls.

It turns out this is a lot more complex than I expected, even if the architectural underpinnings are conceptually simple. This post is about lessons I wanted to share to help others build their own deep research agents.

You can find the open-source Deep Orchestrator agent here: [github.com/lastmile-ai/mcp-agent/src/mcp_agent/workflows/deep_orchestrator/](https://github.com/lastmile-ai/mcp-agent/src/mcp_agent/workflows/deep_orchestrator/){:target="do"}

## Objective: Deep Research, but with MCP

The first deep research agents started out with access to the Internet and filesystems only. The promise of MCP is to dramatically expand that list of tools while adhering to the same architecture. The goal is to be able to do deep research connected to an internal data warehouse, or any data source accessible via an MCP tool or resource. Plus, being able to mutate state by performing tool calls turns the agent from just a research agent to something much more powerful and general-purpose.

So following the Deep Research approach, I settled on the following requirements:

* **Core Functionality** - Be able to complete complex tasks (including deep research and multi-step workflows) by making multiple MCP tool calls.
* **Context Management** - Use the outputs from sequential steps to gather context and make informed decisions for later steps.
* **MCP Integration** - Use MCP servers for everything, i.e. the agent shouldn't need specialized tools beyond the ability to connect to tools via MCP servers.
* **General Purpose Design** - Be sufficiently general-purpose to work across multiple domain spaces.

With these goals in mind, my first instinct was to build an Orchestrator to manage complex queries.

## Take 1: Orchestrator (January 2025)

I implemented the Orchestrator pattern from Anthropic's [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents){:target="abea"} blog post.

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/take-1-orchestrator.png" alt="The First Orchestrator Design" title="The First Orchestrator Design">
  <p>
    <strong>Figure 2:</strong> The First Orchestrator Design
  </p>
</div>

The architecture components:

* **Planning Layer** - A planner LLM breaks down the user objective into sub-tasks and assigns them to predefined sub-agents.
* **Execution Layer** - The sub-agents execute, build up context progressively (since one sub-agent could chain into another).
* **Synthesis Layer** - A synthesizer LLM combines all the sub-agent results into a single final response.

The first attempt worked somewhat well. The Orchestrator usually did a good job defining and executing a _plan_, and the architecture was simple and elegant. It was particularly effective for tasks where a full plan could be determined upfront.

For example, for this objective, here's how the Orchestrator would break it down into a plan:

{: highlight}
> Load the student's short story from short_story.md, and generate a report with feedback across proofreading, factuality/logical consistency and style adherence. Use the style rules from [https://owl.purdue.edu/owl/research_and_citation/apa_style/apa_formatting_and_style_guide/general_format.html](https://owl.purdue.edu/owl/research_and_citation/apa_style/apa_formatting_and_style_guide/general_format.html){:target="_blank"}. Write the graded report to graded_report.md in the same directory as short_story.md

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/take-1-orchestrator-plan.png" alt="The Short Story Plan" title="First The Orchestrator Design">
  <p>
    <strong>Figure 3:</strong> The Story Plan
  </p>
</div>

However, the Orchestrator approach also uncovered a number of challenges:

* **Hallucination Issues** - Subagents had to be pre-defined, and the Orchestrator would sometimes hallucinate MCP server names or try to call agents that didn't exist.
* **Token Inefficiency** - The context from each step was aggregated and passed onto the next step. So by the last step, the context included everything from previous steps. This often meant that context windows were overwhelmed, and calls could be quite expensive without much improvement in the final result.
* **Fixed Planning** - For complex tasks where the Orchestrator couldn't plan everything upfront, the Orchestrator continued to push through to a final answer, not incorporating important context that should change sequential steps. The Orchestrator lacked a reflection step where it could reason about whether the current plan was sufficient to accomplish the task at hand.

To make plans more dynamic, I tried implementing an &ldquo;iterative&rdquo; plan mode, whereby the planner would only think of the immediate next step, then re-evaluate once that was completed, and keep going until the objective was accomplished.

The most counterintuitive learning was that asking for the Orchestrator to identify only the next immediate step worked substantially **worse** than expected. Instinctively, giving the LLM more granular reasoning tasks should provide high quality outputs. However, in practice, asking the Orchestrator for a high-level plan to begin with had dramatically improved the output.

I suspect the reason is that asking the LLM to think of all steps requires it to reason more deliberately.

However, this Orchestrator pattern is pretty useful for certain types of tasks, so it is [part of mcp-agent](https://github.com/lastmile-ai/mcp-agent/tree/main/src/mcp_agent/workflows/orchestrator){:target="orch"}, but it certainly isn't the general-purpose deep research agent that I was after.

## Take 2: Adaptive Workflow

Over the past few months, several AI companies came out with deep research agents. Some of them published detailed blogs on their approaches ([Anthropic](https://www.anthropic.com/engineering/multi-agent-research-system){:target="_blank"}, [Jina AI](https://jina.ai/news/a-practical-guide-to-implementing-deepsearch-deepresearch/){:target="_blank"}).

Armed with their learnings and my own from Take 1, I set out to build an &ldquo;Orchestrator&rdquo; workflow which would &ldquo;adapt&rdquo; its plan and subtasks based on the objective. I named it [`AdaptiveWorkflow`](https://github.com/lastmile-ai/mcp-agent/pull/346){:target="aw"}, which felt appropriate.

Compared to Take 1, the main architectural updates were the following:

* **Dynamic Subagents** - The planner LLM can now define subagents dynamically after analyzing the user's objective, instead of defining all the MCP servers and subagents upfront.
* **FIFO TODO Queue** - Similar to &ldquo;Steps in Plan,&rdquo; but explicitly defining a queue data structure. The theory is that new tasks (i.e., decomposed queries) are solved upfront to enable better context and performance for later stage tasks.
* **External Memory and Knowledge** - In the original orchestrator, the prompt context window would be accumulating ALL previous steps' results, so knowledge and memory were bound by context window. For the adaptive orchestrator, I created external knowledge/memory banks. Each task would lead to knowledge being extracted, which could then be injected into subsequent tasks more efficiently (by relevancy or recency).
* **Budget Management** - I added deep [token tracking](https://github.com/lastmile-ai/mcp-agent/blob/main/src/mcp_agent/tracing/token_counter.py){:target="tok"} throughout mcp-agent, where each agent, LLM and workflow could push/pop token context nodes, allowing you to see exactly how many tokens are being used at each substep. This let me set up a budget manager for the adaptive orchestrator – if we exceeded the time/cost/token budget, it would end the workflow.
* **Beast Mode** - Taking inspiration from [Jina AI](https://jina.ai/news/a-practical-guide-to-implementing-deepsearch-deepresearch/){:target="_blank"}, I added a &ldquo;beast mode&rdquo; agent that would trigger and generate a response based on accumulated knowledge once a constraint was hit (budget, timeout, etc). Something is better than nothing.
* **Mode Detection** - I wrote a lot of complex logic to determine which mode the workflow would operate in based on the task: simple, research or hybrid. The basic idea is that we can bring the right amount of &ldquo;firepower&rdquo; based on knowledge accumulation and planning for the task.

I was really excited about Adaptive Workflow to address some of the shortcomings that my original Orchestrator uncovered. With better workflow planning, budget management, external memory, more dynamic agent selection and mode detection, my deep research agent should be more versatile and efficient!

The hundreds of unit tests Claude and I wrote all passed, the individual components were all working correctly. Time to press the On button to try it out!

And lo and behold… it didn't work on real-world examples:

* **Navigation Problems** - Adaptive Workflow would get lost or sidetracked a lot.
* **Performance Issues** - Queries took way too long.
* **Complexity Overhead** - For simpler tasks, the base Orchestrator actually performed much better.

All of the ingredients for a deep research agent were there, matching the theory and architecture I had read, but for some reason, the whole wasn't greater than the sum of the parts.

## Take 3: Deep Orchestrator (August 2025) – Finally Getting Somewhere!

The key observation came from cases where the original Orchestrator was outperforming the Adaptive Workflow despite all its bells and whistles.

I started debugging the queries where the Orchestrator outperformed the Adaptive Workflow and unlocked a key insight:

{: .tip}
> **Tip:** _Simpler architecture consistently wins._

So I wiped the slate clean and started by running the original Orchestrator in a loop:

* **Input Processing** - Input user objective.
* **Plan Development** - Develop a full plan, with steps and subtasks.
* **Execution Phase** - Run through all steps.
* **New Verification Step** - Add an objective verification.
* If objective verification is not satisfied, replan and repeat.

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/take-3-orchestrator.png" alt="Iterative Deep Orchestrator" title="Iterative Deep Orchestrator">
  <p>
    <strong>Figure 4:</strong> Iterative Deep Orchestrator
  </p>
</div>

Next, I rewrote all the components I had built for Adaptive Workflow, but with new insights from what worked well in the base Orchestrator:

### TODO Queue

I instructed the LLM to generate a full plan upfront, so the queue is built with multiple steps (not just the next TODO step). Additionally, I added parallelism into sub-tasks for performance and sequential steps.

### Memory and Knowledge

The original Orchestrator was token-inefficient but context-rich – each task would get full context from previous steps.

I reused most of the architecture for external memory and knowledge extraction from Adaptive Workflow, but improved how tasks utilized memory. The planner now specifies dependencies between tasks when generating the plan, determining when context should be fed in and building a dependency graph that determines when to propagate memory.

Here are some highlighted additions to the `Task` and `Plan` models between Take 2 and Take 3 ([GitHub link](https://github.com/lastmile-ai/mcp-agent/blob/83cd876083660fabedae0718e2d0c6bfbc0f1b75/src/mcp_agent/workflows/deep_orchestrator/models.py#L87){:target="models_py"}):

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/code-changes.png" alt="Task and Plan Code Changes" title="Task and Plan Code Changes">
  <p>
    <strong>Figure 5:</strong> <code>Task</code> and <code>Plan</code> Code Changes
  </p>
</div>

I also added a &ldquo;full context propagation mode,&rdquo; but this is much less token efficient and usually not necessary.

### Plan, but Verify (Deterministically)

LLMs hallucinate. They're better than they were before, but they're still not perfect. So I added deterministic plan verification before the plan is executed. The verification validates:

* **Dependency Validation** - Task dependency graph
* **Server Verification** - Check to ensure that the MCP servers being called are real
* **Agent Verification** - Confirm that predefined agents that the plan will call actually exist

If the plan verification fails, Deep Orchestrator generates an error message prompt with all the issues identified and asks the Planner LLM to create a new plan to address the issues. (This is similar to the Evaluator-Optimizer pattern explained in [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents){:target="abea"}, but involves deterministic verification.) 

{: .tip}
> **Tips:** 
> 1. The combination of incorporating deterministic (code-based) validation in conjunction with LLM execution was a powerful improvement to the architecture.
> 1. There is more to agents than just LLMs. If we can check something deterministically with code, always prefer that over doing the same with an LLM.

### The Art and Science of Good Prompting

Good prompting really matters. Just take a look at the [reverse-engineered Claude Code system prompt](https://github.com/kn1026/cc/blob/main/claudecode.md){:target="cc_prompt"}. One thing I learned from [Roo Code](https://github.com/RooCodeInc/Roo-Code/tree/main/src/core/prompts){:target="roocode"} is to build up the prompts progressively in a functional/programmatic way instead of just long, giant strings.

So I added functions that can add sections to prompts, and to organize them all, I switched to XML tags (there are other approaches too).

You can see some of this in action in [mcp-agent/src/mcp_agent/workflows/deep_orchestrator/prompts.py](https://github.com/lastmile-ai/mcp-agent/blob/main/src/mcp_agent/workflows/deep_orchestrator/prompts.py){:target="prompts_py"}. For example, this is building up a prompt for synthesizing given different knowledge artifacts ([GitHub link](https://github.com/lastmile-ai/mcp-agent/blob/83cd876083660fabedae0718e2d0c6bfbc0f1b75/src/mcp_agent/workflows/deep_orchestrator/prompts.py#L466){:target="prompts_py"}):

```python
def get_synthesis_context(
    objective: str,
    execution_summary: dict,
    completed_steps: list,
    knowledge_by_category: dict,
    artifacts: dict,
) -> str:
    """Build comprehensive synthesis context."""
    context_parts = [
        "<synthesis_context>",
        f"  <original_objective>{objective}</original_objective>",
        "",
        "  <execution_summary>",
        f"    <iterations>{execution_summary.get('iterations', 0)}</iterations>",
        f"    <steps_completed>{execution_summary.get('steps_completed', 0)}</steps_completed>",
        f"    <tasks_completed>{execution_summary.get('tasks_completed', 0)}</tasks_completed>",
        f"    <tokens_used>{execution_summary.get('tokens_used', 0)}</tokens_used>",
        f"    <cost>${execution_summary.get('cost', 0):.2f}</cost>",
        "  </execution_summary>",
        "",
        "  <completed_work>",
    ]

    # Summarize completed steps and their results
    for step in completed_steps:
        context_parts.append(f'    <step name="{step.get("description", "Unknown")}">')

        for task_result in step.get("task_results", []):
            if task_result.get("success"):
                task_desc = task_result.get("description", "Unknown task")
                output_summary = task_result.get("output", "")[:300]
                if len(task_result.get("output", "")) > 300:
                    output_summary += "..."

                context_parts.append("      <task_result>")
                context_parts.append(f"        <task>{task_desc}</task>")
                context_parts.append(f"        <output>{output_summary}</output>")
                context_parts.append("      </task_result>")

        context_parts.append("    </step>")

    context_parts.append("  </completed_work>")

    # Add accumulated knowledge
    if knowledge_by_category:
        context_parts.append("")
        context_parts.append("  <accumulated_knowledge>")

        for category, items in knowledge_by_category.items():
            context_parts.append(f'    <category name="{category}">')
            for item in items[:5]:  # Limit per category
                context_parts.append(
                    f'      <knowledge confidence="{item.confidence:.2f}">'
                )
                context_parts.append(f"        <key>{item.key}</key>")
                value_str = (
                    str(item.value)[:200] + "..."
                    if len(str(item.value)) > 200
                    else str(item.value)
                )
                context_parts.append(f"        <value>{value_str}</value>")
                context_parts.append("      </knowledge>")
            context_parts.append("    </category>")

        context_parts.append("  </accumulated_knowledge>")

    # Add artifacts
    if artifacts:
        context_parts.append("")
        context_parts.append("  <artifacts_created>")
        for name, content in list(artifacts.items())[-10:]:  # Last 10 artifacts
            content_preview = content[:500] + "..." if len(content) > 500 else content
            context_parts.append(f'    <artifact name="{name}">')
            context_parts.append(f"      {content_preview}")
            context_parts.append("    </artifact>")
        context_parts.append("  </artifacts_created>")

    context_parts.append("</synthesis_context>")
    return "\n".join(context_parts)
```
<div style="text-align: center;">
  <p>
    <strong>Figure 6:</strong> <code>prompts.py</code> Code Section
  </p>
</div>

Using XML tags (or some other structured language within the string) to disambiguate sections really helped (See the same [prompts.py](https://github.com/lastmile-ai/mcp-agent/blob/83cd876083660fabedae0718e2d0c6bfbc0f1b75/src/mcp_agent/workflows/deep_orchestrator/prompts.py){:target="prompts_py"}file):

```xml
<planner_instruction>
  You are an expert strategic planner who creates comprehensive execution plans.

  <planning_process>
    <step>1. Deeply analyze the objective and any accumulated knowledge</step>
    <step>2. Identify major phases or milestones needed</step>
    <step>3. Break down into specific, actionable steps</step>
    <step>4. For each step, define parallel tasks with clear boundaries</step>
    <step>5. Order steps logically - later steps naturally depend on earlier ones</step>
    <step>6. Assign appropriate agents and tools to each task</step>
  </planning_process>

  <task_design_rules>
    <rule>Each task must have a single, clear deliverable</rule>
    <rule>Give each task a unique, descriptive name (e.g., "analyze_code", "check_grammar", "compile_report")</rule>
    <rule>Tasks should be specific enough to execute without ambiguity</rule>
    <rule>Parallel tasks within a step must not interfere with each other</rule>
    <rule>Leave agent field unset (not specified) to request dynamic agent creation</rule>
    <rule>CRITICAL: If you specify an agent name, it MUST be one of the available_agents - NEVER invent or hallucinate agent names</rule>
    <rule>CRITICAL: Only use MCP servers from the available_servers list - NEVER invent or hallucinate server names</rule>
    <rule>If no servers are needed for a task, use an empty list []</rule>
    <rule>Tasks run in parallel within a step, steps run sequentially</rule>
    <rule>Use requires_context_from to specify which previous task outputs this task needs</rule>
    <rule>requires_context_from can ONLY reference tasks from PREVIOUS steps, not the current step</rule>
    <rule>If a task needs output from another task in the same step, move it to a subsequent step</rule>
    <rule>Only set context_window_budget if task needs more than default (10000 tokens)</rule>
  </task_design_rules>

  <important_notes>
    <note>Do NOT recreate already completed steps - build on existing work</note>
    <note>If objective is already satisfied, set is_complete=true</note>
    <note>Consider resource constraints and prefer efficient approaches</note>
    <note>Think step by step about the best way to achieve the objective</note>
    <note>Tasks within a step run in parallel, steps run sequentially</note>
  </important_notes>

  <example_task_structure>
    Step 1: Analysis Phase
      - Task: name="check_grammar", description="Check grammar and spelling"
      - Task: name="analyze_style", description="Analyze writing style"
      - Task: name="assess_structure", description="Assess story structure"
    
    Step 2: Synthesis Phase  
      - Task: name="compile_report", description="Compile comprehensive grading report"
        requires_context_from=["check_grammar", "analyze_style", "assess_structure"]
        # Can reference tasks from Step 1, but NOT tasks from Step 2
  </example_task_structure>
</planner_instruction>
```

<div style="text-align: center;">
  <p>
    <strong>Figure 7:</strong> Using XML to Structure Prompts
  </p>
</div>

### Simple Policy Engine

Finally, I got rid of the complex Mode Detection from Adaptive Workflow and instead built a simple policy engine that would decide whether to:

* **Continue Executing** - Proceed with current plan
* **Trigger Replanning** - Revise approach based on new information
* **Emergency Stop** - Halt due to repeated failures
* **Force Completion** - Complete due to budget overruns

Having a module dedicated to decision-making helped simplify the architecture and showed me that my complex mode selection was a hack masquerading as a feature.

Putting all these new pieces together worked pretty well! **Figure 1** [above](#figure-1) shows the final architecture.


{: .tip}
> **Tip:** You can try it out in this [AI Alliance](https://thealliance.ai/){:target="aia"} example application, [Deep Research Agent for Finance](https://github.com/The-AI-Alliance/deep-research-agent-for-finance){:target="draf"}.

Here is the full Deep Orchestrator flow ([source code](https://github.com/lastmile-ai/mcp-agent/blob/main/src/mcp_agent/workflows/deep_orchestrator/README.md){:target="do"}):

<div style="text-align: center;">
  <img src="{{site.baseurl}}/assets/images/deep-orchestrator-flow.png" alt="Deep Orchestrator Flow" title="Deep Orchestrator Flow">
  <p>
    <strong>Figure 8:</strong> The Full Deep Orchestrator Flow
  </p>
</div>

## Key Learnings

### 1. Simple Architecture Wins

I over-complicated things with Adaptive Workflow. Even though the base components were correct (TODO queue, external memory/knowledge, lead planner, etc), they didn't work well together because each component was individually too complex.

### 2. MCP Is All You Need

I didn't implement anything specifically for deep research when building Deep Orchestrator. The fundamental building block of MCP-Agent is MCP servers, so the same agent that's performing general tasks can also be used for deep research. This shows the power of the generalizability of MCP.

### 3. The Little Details Matter

A little cliche, but the difference between an agent that works well and one that doesn't is in the many small decisions made along the way. The base components of Adaptive Workflow (attempt 2) and Deep Orchestrator (attempt 3) are really similar, but it took a lot of small tweaks to actually get Deep Orchestrator to work well.

## What's Next?

I'm really happy with the progress so far to bring us to Deep Orchestrator, but there are a few things that I'm excited about to take it to the next level.

* **Remote Execution** - MCP-Agent already supports temporal orchestration, and I want to expose it as an MCP server that you can call it to perform arbitrary operations when certain conditions are met.
* **Intelligent Tool Selection** - Right now, many MCP servers are relatively simple, but in the near future, we will have hundreds or thousands of possible MCP servers, each with dozens or hundreds of tools. Deep Orchestrator needs to be able to intelligently create subagents with access to the right servers/tools, and those subagents in turn need to intelligently filter tools they expose to the LLM (usually 20-30 tools per step work well).
* **Memory and Knowledge as MCP Resources** - MCP has an underutilized concept of [resources](https://modelcontextprotocol.io/specification/2025-06-18/server/resources){:target="_blank"} to reference data by URI. I want to see if I can represent all the knowledge and memory components of the Deep Orchestrator as MCP resources on a Memory Server. That way, they can be stored remotely and reused elsewhere as needed.
* **Dynamic Model Selection** - Core to the Deep Research Orchestrator is using a strong reasoning model for planning. For both latency and cost benefits, a faster, smaller LLM can be used for simple tool calls

For the latest improvements and projects with MCP-Agent, check out the open source repo: [mcp-agent](https://github.com/lastmile-ai/mcp-agent){:target="mcp-agent"}.

Also, check out the [AI Alliance's](https://thealliance.ai/){:target="aia"} example application, [Deep Research Agent for Finance](https://github.com/The-AI-Alliance/deep-research-agent-for-finance){:target="draf"}, which uses the Deep Research architecture for searching financial information and preparing reports.