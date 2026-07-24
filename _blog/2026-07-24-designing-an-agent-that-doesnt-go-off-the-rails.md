---
title: "Designing an Agent That Doesn't Go Off the Rails"
date: 2026-07-24
excerpt: "Some tasks can't be one prompt: investigate a failing customer, pull their tickets, check billing, guess the cause, draft a reply. That needs planning, several tool calls, and reacting to what comes back. This is a full walk through how I'd design an agentic system end to end, the loop, the tools, the memory, the guardrails, and the cloud services that fill each box on AWS, GCP, and Azure, with the honest reasons you should reach for an agent as late as you possibly can."
tags: [ai, agents, orchestration, system-design, langgraph, architecture]
---

<style>
.ag-fig{margin:2.5rem 0;}
.ag-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* chain vs agent */
.ag-vs{display:grid;grid-template-columns:1fr 1fr;gap:.9rem;max-width:680px;margin:0 auto;}
@media(max-width:600px){.ag-vs{grid-template-columns:1fr;}}
.ag-vs .col{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.ag-vs .col h4{margin:0 0 .6rem;font-size:.85rem;font-family:var(--font-mono);}
.ag-vs .col.chain h4{color:var(--text-2);} .ag-vs .col.agent h4{color:var(--accent);}
.ag-vs .col.agent{border-color:var(--accent);}
.ag-vs .flow{display:flex;flex-direction:column;gap:.4rem;}
.ag-vs .node{font-family:var(--font-mono);font-size:.76rem;color:var(--text-2);background:var(--surface-2);border:1px solid var(--border-2);border-radius:8px;padding:.4rem .6rem;text-align:center;}
.ag-vs .agent .node.loop{border-color:var(--accent);color:var(--accent);}
.ag-vs .arr{text-align:center;font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);}
.ag-vs .arr.back{color:var(--accent);}
.ag-vs .tag{font-family:var(--font-mono);font-size:.72rem;margin-top:.6rem;}
.ag-vs .chain .tag{color:var(--text-3);} .ag-vs .agent .tag{color:var(--accent);}
.ag-vs .col{opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.ag-vs.go .col{opacity:1;transform:none;}
.ag-vs.go .col:nth-child(1){transition-delay:.1s} .ag-vs.go .col:nth-child(2){transition-delay:.35s}

/* the loop cycle */
.ag-loop{max-width:520px;margin:0 auto;}
.ag-loop svg{width:100%;height:auto;overflow:visible;}
.ag-loop .ring{fill:none;stroke:var(--border-2);stroke-width:2;stroke-dasharray:6 6;}
.ag-loop.go .ring{stroke:var(--accent);opacity:.5;}
.ag-loop .dot{fill:var(--surface);stroke:var(--border-2);stroke-width:1.5;}
.ag-loop .dot.g{stroke:var(--accent);}
.ag-loop .lab{fill:var(--text);font-family:var(--font-mono);font-size:9.5px;font-weight:600;text-anchor:middle;}
.ag-loop .sub{fill:var(--text-3);font-family:var(--font-mono);font-size:7.5px;text-anchor:middle;}
.ag-loop .step{opacity:0;transition:opacity .4s ease;}
.ag-loop.go .step{opacity:1;}
.ag-loop.go .s1{transition-delay:.1s}.ag-loop.go .s2{transition-delay:.3s}.ag-loop.go .s3{transition-delay:.5s}
.ag-loop.go .s4{transition-delay:.7s}.ag-loop.go .s5{transition-delay:.9s}.ag-loop.go .s6{transition-delay:1.1s}
.ag-loop .center{fill:var(--accent);font-family:var(--font-mono);font-size:10px;font-weight:600;text-anchor:middle;}
.ag-loop .center2{fill:var(--text-3);font-family:var(--font-mono);font-size:7.5px;text-anchor:middle;}

/* decision ladder */
.ag-ladder{max-width:620px;margin:0 auto;display:flex;flex-direction:column;gap:.6rem;}
.ag-rung{display:flex;align-items:center;gap:.9rem;border:1px solid var(--border);border-radius:10px;padding:.75rem 1rem;background:var(--surface);opacity:0;transform:translateX(-10px);transition:opacity .5s ease,transform .5s ease;}
.ag-ladder.go .ag-rung{opacity:1;transform:none;}
.ag-ladder.go .ag-rung:nth-child(1){transition-delay:.1s}.ag-ladder.go .ag-rung:nth-child(2){transition-delay:.35s}.ag-ladder.go .ag-rung:nth-child(3){transition-delay:.6s}
.ag-rung .lv{flex:none;width:26px;height:26px;border-radius:50%;background:var(--surface-2);border:1px solid var(--border-2);color:var(--text-2);font-family:var(--font-mono);font-weight:600;font-size:.8rem;display:flex;align-items:center;justify-content:center;}
.ag-rung.top .lv{background:var(--grad);color:var(--accent-ink);border-color:transparent;}
.ag-rung .nm{font-family:var(--font-mono);font-size:.85rem;color:var(--text);font-weight:600;}
.ag-rung.top .nm{color:var(--accent);}
.ag-rung .desc{font-size:.8rem;color:var(--text-2);line-height:1.45;}
.ag-rung .cost{font-family:var(--font-mono);font-size:.68rem;margin-left:auto;padding:.15rem .5rem;border-radius:999px;border:1px solid var(--border-2);color:var(--text-3);white-space:nowrap;}
.ag-rung.top .cost{border-color:var(--accent);color:var(--accent);}
@media(max-width:560px){.ag-rung .cost{display:none;}}

/* memory tiers */
.ag-mem{display:grid;grid-template-columns:1fr 1fr;gap:.9rem;max-width:680px;margin:0 auto;}
@media(max-width:600px){.ag-mem{grid-template-columns:1fr;}}
.ag-mem .c{border:1px solid var(--border);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.ag-mem.go .c{opacity:1;transform:none;}
.ag-mem.go .c:nth-child(1){transition-delay:.1s}.ag-mem.go .c:nth-child(2){transition-delay:.35s}
.ag-mem .c.short{border-left:3px solid var(--border-2);}
.ag-mem .c.long{border-left:3px solid var(--accent);}
.ag-mem .c .k{font-family:var(--font-mono);font-size:.68rem;letter-spacing:.06em;text-transform:uppercase;margin-bottom:.3rem;}
.ag-mem .c.short .k{color:var(--text-3);} .ag-mem .c.long .k{color:var(--accent);}
.ag-mem .c b{display:block;color:var(--text);font-size:.9rem;margin-bottom:.35rem;}
.ag-mem .c p{font-size:.8rem;color:var(--text-2);line-height:1.5;margin:0 0 .5rem;}
.ag-mem .c .trait{font-family:var(--font-mono);font-size:.72rem;color:var(--text-2);padding:.15rem 0;}
.ag-mem .c .trait::before{content:"› ";color:var(--accent);}

/* cloud table */
.ag-tab-wrap{max-width:820px;margin:0 auto;overflow-x:auto;}
.ag-tab{width:100%;border-collapse:collapse;font-size:.85rem;min-width:620px;}
.ag-tab th,.ag-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.ag-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.ag-tab td:first-child,.ag-tab th:first-child{color:var(--text);font-weight:500;font-family:var(--font-mono);font-size:.78rem;white-space:nowrap;}
.ag-tab tbody td{color:var(--text-2);}
.ag-tab .cloud{color:var(--accent);}

/* gotchas */
.ag-got{max-width:680px;margin:0 auto;display:flex;flex-direction:column;gap:.8rem;}
.ag-g{border:1px solid var(--border);border-left:3px solid var(--accent);border-radius:10px;padding:.9rem 1.1rem;background:var(--surface);opacity:0;transform:translateX(-10px);transition:opacity .5s ease,transform .5s ease;}
.ag-got.go .ag-g{opacity:1;transform:none;}
.ag-got.go .ag-g:nth-child(1){transition-delay:.1s}.ag-got.go .ag-g:nth-child(2){transition-delay:.35s}.ag-got.go .ag-g:nth-child(3){transition-delay:.6s}
.ag-g .hd{display:flex;align-items:baseline;gap:.6rem;margin-bottom:.35rem;flex-wrap:wrap;}
.ag-g .nm{font-family:var(--font-mono);font-size:.85rem;color:var(--accent);font-weight:600;}
.ag-g .fix{font-family:var(--font-mono);font-size:.68rem;color:var(--text);background:var(--surface-2);border:1px solid var(--border-2);border-radius:999px;padding:.12rem .5rem;}
.ag-g p{font-size:.83rem;color:var(--text-2);line-height:1.5;margin:0;}

@media (prefers-reduced-motion: reduce){
  .ag-vs .col,.ag-loop .step,.ag-loop .ring,.ag-ladder .ag-rung,.ag-mem .c,.ag-got .ag-g{transition:none !important;opacity:1 !important;transform:none !important;}
}
</style>

Here's a ticket that lands on your desk. "Customer X is unhappy and threatening to churn. Figure out what's going on and draft a reply." To actually do that, you'd pull their recent support tickets, check their billing history for a failed charge or a plan downgrade, cross-reference an outage window, form a theory about the likely cause, and only then write something. You don't know in advance how many of those steps you'll need, or in what order, until you see what the first ones turn up. If billing looks clean, you go dig in the tickets. If the tickets are quiet, you look harder at billing.

You can't write that as one prompt. And you can't write it as a fixed script either, because the *path* depends on what you find along the way. This is the class of problem where an agent earns its keep: an LLM in a loop, holding some memory, with a set of tools, and the authority to decide what to do next.

This is post three of the AI System Design series. The [first post](/blog/2026-07-24-three-ai-systems-same-design/) made the case that these systems share one shape; the [second](/blog/2026-07-24-designing-a-rag-system-that-actually-retrieves/) worked out a RAG chatbot box by box. Same spine each time: cheap wide recall, expensive narrow precision, then policy. An agent is that spine turned into a loop, and it's the most powerful and the most dangerous shape of the three. Let me design one end to end, and be honest about when you shouldn't.

## A chain is a straight line. An agent is a loop.

Start with the single most important distinction, because getting it wrong is the number one way these projects go sideways. RAG is a *chain*: retrieve, then answer. The steps are fixed and always run in that order. An agent *loops*: it decides, acts, looks at the result, and decides again, and it might go around two times or twelve.

<figure class="ag-fig">
  <div class="ag-vs ag-anim" id="vs">
    <div class="col chain">
      <h4>Chain (RAG, a workflow)</h4>
      <div class="flow">
        <div class="node">retrieve chunks</div>
        <div class="arr">↓</div>
        <div class="node">rerank</div>
        <div class="arr">↓</div>
        <div class="node">answer</div>
      </div>
      <div class="tag">fixed steps · deterministic · easy to test</div>
    </div>
    <div class="col agent">
      <h4>Agent (a loop)</h4>
      <div class="flow">
        <div class="node loop">decide next step</div>
        <div class="arr">↓</div>
        <div class="node loop">call a tool</div>
        <div class="arr">↓</div>
        <div class="node loop">read the result</div>
        <div class="arr back">↑ loop back, or finish</div>
      </div>
      <div class="tag">dynamic path · non-deterministic · loops until done</div>
    </div>
  </div>
  <figcaption>The whole difference in one picture. A chain runs a known sequence and stops. An agent runs a loop and decides for itself how many times to go around and which tool to reach for next. That freedom is exactly what makes it capable of the "investigate this customer" task, and exactly what makes it harder to test, more expensive, and able to run off a cliff if you let it.</figcaption>
</figure>

The chain is boring, and boring is a compliment. You can unit-test every step, the cost per run is predictable, and it never surprises you at 3am. The agent is the opposite on every axis. Keep that asymmetry in your head for the whole design, because the guiding principle falls right out of it.

## The agent loop, up close

When people say "an LLM in a loop," this is the loop. Six moves, and it goes around until the task is done or a budget stops it.

<figure class="ag-fig">
  <div class="ag-loop ag-anim" id="loop">
  <svg viewBox="0 0 360 300" role="img" aria-label="The agent loop as a cycle: load context, plan, guard, execute, observe, reflect, then back to plan.">
    <circle class="ring" cx="180" cy="150" r="112"></circle>
    <text class="center" x="180" y="146">the loop</text>
    <text class="center2" x="180" y="160">until done or budget hit</text>
    <g class="step s1"><circle class="dot" cx="180" cy="38" r="26"></circle><text class="lab" x="180" y="36">load</text><text class="sub" x="180" y="47">context</text></g>
    <g class="step s2"><circle class="dot g" cx="277" cy="94" r="26"></circle><text class="lab" x="277" y="92">plan</text><text class="sub" x="277" y="103">next step</text></g>
    <g class="step s3"><circle class="dot g" cx="277" cy="206" r="26"></circle><text class="lab" x="277" y="204">guard</text><text class="sub" x="277" y="215">check first</text></g>
    <g class="step s4"><circle class="dot" cx="180" cy="262" r="26"></circle><text class="lab" x="180" y="260">execute</text><text class="sub" x="180" y="271">a tool</text></g>
    <g class="step s5"><circle class="dot" cx="83" cy="206" r="26"></circle><text class="lab" x="83" y="204">observe</text><text class="sub" x="83" y="215">the result</text></g>
    <g class="step s6"><circle class="dot g" cx="83" cy="94" r="26"></circle><text class="lab" x="83" y="92">reflect</text><text class="sub" x="83" y="103">done?</text></g>
  </svg>
  </div>
  <figcaption>Load the working context. Plan the next single step. Guard it (is this allowed, is it safe, are the arguments valid?). Execute the tool. Observe what came back. Reflect on whether the task is finished, then loop or stop. The <span style="color:var(--accent)">accented nodes</span>, plan, guard, reflect, are where the LLM reasons; execute and observe are plain code. Most of your engineering effort goes into guard and reflect, because that's where an agent either stays sane or spirals.</figcaption>
</figure>

Notice that plan, guard, and reflect are the three points where model judgment happens, and they map cleanly onto the series funnel: planning is cheap recall (what *could* I do next), reflecting is expensive precision (was that actually good enough to stop), and guarding is policy (the rules the model won't enforce on itself). The loop is a funnel that runs over and over.

## Use the cheapest thing that works

Before you build any of that, the real question: do you even need an agent? Almost always, you need less than you think. This is a ladder, and you climb it only as far as the problem forces you.

<figure class="ag-fig">
  <div class="ag-ladder ag-anim" id="ladder">
    <div class="ag-rung">
      <div class="lv">1</div>
      <div>
        <div class="nm">Single LLM call</div>
        <div class="desc">One prompt in, one answer out. Summarize this, classify that, rewrite this email. If a prompt does the job, you're done.</div>
      </div>
      <div class="cost">cheapest · fully testable</div>
    </div>
    <div class="ag-rung">
      <div class="lv">2</div>
      <div>
        <div class="nm">Fixed chain / workflow</div>
        <div class="desc">Known steps in a known order. RAG lives here: retrieve, rerank, answer. Deterministic, cheap, you can test every hop. Most "AI features" are this and nothing more.</div>
      </div>
      <div class="cost">cheap · deterministic</div>
    </div>
    <div class="ag-rung top">
      <div class="lv">3</div>
      <div>
        <div class="nm">Agent</div>
        <div class="desc">The number and order of steps depend on intermediate results, and you genuinely can't hard-code the graph in advance. Only now do you reach for the loop.</div>
      </div>
      <div class="cost">expensive · non-deterministic</div>
    </div>
  </div>
  <figcaption>Climb the ladder, don't jump to the top. Every rung up costs more money, more latency, and more ways to fail. An agent makes several LLM calls per task instead of one, its output isn't reproducible, and it can loop. Reach for it only when the path is genuinely dynamic. If you can draw the flowchart ahead of time, you don't have an agent problem, you have a workflow, and you should build the workflow.</figcaption>
</figure>

Here's the discipline that separates production systems from demos: **push as much as you possibly can into deterministic workflow, and reserve agentic autonomy for the small part that truly needs it.** For our customer-investigation task, "pull the tickets" and "fetch the billing record" are fixed workflow steps. The only genuinely agentic decision is *"given what I just found, do I dig deeper into billing or pivot to the tickets?"* A workflow graph with a couple of LLM decision nodes beats a fully autonomous "figure it all out" agent in almost every real deployment. It's more debuggable, cheaper, and it fails in ways you can predict.

## Model the graph, don't let the model wing it

That last point deserves its own section, because it's a real fork in the design. You can build the agent two ways. One: hand the model all the tools and say "go, decide everything." Two: model an explicit **state graph**, nodes are steps, edges are the transitions you allow, and let the model make decisions only at the nodes where a decision is genuinely needed.

The graph version is more work up front and a little less flexible. It's also enormously more debuggable and testable, because at any moment you know which state you're in and which transitions are even possible. When something breaks, you can point at the node. A free-form "the LLM decides everything" agent gives you a transcript and a shrug. For production, the trade almost always favors the graph. This is exactly what [LangGraph](/blog/2026-07-17-langgraph-when-your-ai-needs-a-flowchart-that-runs/) is for, and I wrote a whole post on it, so I won't relitigate the details here. The point for *this* design is: prefer the modeled graph.

## Tool design is the actual product

An agent is only as good as the tools you give it, and tool design is where most of the quality (and most of the bugs) live. A few principles that matter more than they sound:

- **Fewer, well-described tools beat many overlapping ones.** The single biggest cause of an agent calling the wrong tool is two tools whose descriptions blur together. If `get_billing` and `get_invoices` sound similar, the model will coin-flip between them. Give each tool one crisp job and one crisp description.
- **Strict, typed schemas.** Every argument typed and constrained. This is your first line of defense against hallucinated arguments, a `date` field that only accepts a real date can't receive a made-up string.
- **Validate arguments before you execute.** The model *will* occasionally produce a plausible-looking but wrong argument. Check it in code before the tool runs, and if it's bad, return a structured error the model can read and recover from, not a stack trace.
- **Make tools idempotent.** Agents retry. If "issue refund" runs twice because of a retry, you've double-refunded. Design write tools so calling them twice with the same request is safe, an idempotency key on every mutating call.

Get this layer right and the loop mostly takes care of itself. Get it wrong, ambiguous descriptions, loose schemas, tools that double-charge on retry, and no amount of clever prompting saves you.

## Memory: a small scratchpad and a long shelf

An agent needs to remember things, but "remember" splits into two very different jobs with two very different designs.

<figure class="ag-fig">
  <div class="ag-mem ag-anim" id="mem">
    <div class="c short">
      <div class="k">short-term</div>
      <b>Working scratchpad</b>
      <p>What the agent has done and seen <em>during this task</em>: the steps so far, the tool results, the running plan. It lives in the context window.</p>
      <div class="trait">bounded, it can't grow forever</div>
      <div class="trait">trim or summarize as it fills</div>
      <div class="trait">gone when the task ends</div>
    </div>
    <div class="c long">
      <div class="k">long-term</div>
      <b>Durable memory store</b>
      <p>Facts worth keeping <em>across tasks</em>: this customer's preferences, a resolution that worked before, a learned pattern. Lives in a vector store you retrieve from.</p>
      <div class="trait">durable, survives restarts</div>
      <div class="trait">written selectively, not everything</div>
      <div class="trait">what to promote is a real design call</div>
    </div>
  </div>
  <figcaption>Short-term memory is a bounded scratchpad. Let it grow unchecked and you blow past the context window and watch your cost per task climb every loop, so you trim old steps or summarize them as it fills. Long-term memory is a durable vector store you write to <em>selectively</em>, and deciding what deserves to be promoted from "happened this once" to "worth remembering forever" is one of the genuinely hard design questions in agent building. Save everything and you drown in noise; save nothing and the agent never learns.</figcaption>
</figure>

The short-term side is a context-engineering problem (I've got a whole post on why AI forgets, if you want the mechanics). The long-term side is closer to the RAG design from post one: it's a vector store, written to and retrieved from, just with the agent deciding what goes in.

## Durable state, so a crash resumes instead of restarts

Our investigation task might be eight steps deep when the process dies. If your agent holds its state only in memory, that crash means starting over: re-pulling every ticket, re-running every tool call, re-paying for every LLM call. Unacceptable.

So persist the state after *every* step. Write down where you are, what you've done, and what you've observed, to durable storage, each time around the loop. Now a crash at step eight resumes at step eight. This single decision does double duty: it also enables **human-in-the-loop**. If you can pause, persist, and later resume, then "wait here for a human to approve" is just a pause state you can sit in for an hour or a day without holding a process open.

## Human-in-the-loop, gated by risk

Not every action is equal, and the design should treat them differently. Sort actions by risk tier.

Read-only and reversible actions, look up a ticket, read the billing record, draft (but don't send) a reply, can run fully autonomously. Nothing bad happens if the agent gets one wrong; you just redo it. But irreversible or costly actions, issuing a refund, sending a message to the customer, deleting a record, moving money, need a human to approve before they fire. And crucially, that approval gate is not an afterthought you bolt on. Design it as a **first-class pause state** in the graph: the agent reaches the "send refund" node, halts, surfaces its proposed action and its reasoning to a human, and waits. Approve and it proceeds; reject and it re-plans. Because you persisted state, waiting is free.

## Observability isn't optional, and evals score the whole trajectory

You cannot operate an agent you can't see into. Every loop, every plan, every tool call and its arguments and its result, has to be traced. When an agent does something baffling, and it will, the trace is the only thing that tells you *why*. This is non-negotiable in a way it simply isn't for a plain chain, because with a non-deterministic loop, "run it again and watch" doesn't reliably reproduce the bug.

Evaluation is where agents break the habits you brought from other systems. For a RAG bot you can score the final answer. For an agent, the final answer being right isn't enough, you have to score the **trajectory**: did it take sensible steps, call the right tools, in a reasonable order, without a wasteful detour? An agent can stumble to the right answer through five wrong turns, and that's a system one bad break away from disaster, not a success. And because the loop is non-deterministic, you don't run each eval case once. You run it many times and track a **success rate**, because "it worked when I tried it" tells you almost nothing about a system that behaves differently every run.

## The cloud boxes, on AWS, GCP, and Azure

Every piece above maps to a managed service on each major cloud. Here's the whole system, box by box, as of mid-2026.

<figure class="ag-fig">
<div class="ag-tab-wrap">
<table class="ag-tab">
<thead>
<tr><th>What you need</th><th class="cloud">AWS</th><th class="cloud">GCP</th><th class="cloud">Azure</th></tr>
</thead>
<tbody>
<tr><td>Agent runtime</td><td>Bedrock AgentCore</td><td>Vertex AI Agent Engine</td><td>Foundry Agent Service</td></tr>
<tr><td>The reasoner (LLM)</td><td>Bedrock (Claude)</td><td>Vertex (Gemini)</td><td>Foundry (GPT / Claude)</td></tr>
<tr><td>Tools / functions</td><td>Lambda + API Gateway</td><td>Cloud Run or Functions</td><td>Azure Functions</td></tr>
<tr><td>Durable state</td><td>Step Functions + DynamoDB</td><td>Workflows + Firestore</td><td>Durable Functions + Cosmos DB</td></tr>
<tr><td>Long-term memory</td><td>OpenSearch or Bedrock KB</td><td>Vertex Vector Search</td><td>Azure AI Search</td></tr>
<tr><td>Tracing</td><td>CloudWatch + OpenTelemetry</td><td>Cloud Trace</td><td>Azure Monitor + App Insights</td></tr>
</tbody>
</table>
</div>
  <figcaption>The same six boxes on all three clouds. Note the agent-runtime row: these managed runtimes (AWS Bedrock AgentCore, GCP Vertex AI Agent Engine, Azure Foundry Agent Service) handle hosting, identity, and memory plumbing so you don't build it from scratch. The framework-agnostic ones matter, AgentCore lets you bring a LangGraph agent and get the managed hosting, identity, and memory around it, so choosing a runtime doesn't mean giving up the graph framework you'd design in anyway.</figcaption>
</figure>

On the framework side: **LangGraph** is the controllable default, an explicit graph with durable checkpoints, which is precisely the "model the graph, persist every step" design above. **CrewAI** and **AutoGen** lean into multi-agent role setups (a researcher agent, a writer agent, and so on), useful when a task really decomposes into distinct roles, though multi-agent adds its own coordination headaches, so don't reach for it by default. And the managed runtimes above will host whichever you pick.

## The gotchas that bite in production

Three failure modes show up again and again. None are exotic; all are avoidable with design you put in *before* launch, not after the incident.

<figure class="ag-fig">
  <div class="ag-got ag-anim" id="got">
    <div class="ag-g">
      <div class="hd"><span class="nm">1 · The infinite loop</span><span class="fix">budgets + loop detection + a hard stop</span></div>
      <p>An agent that never decides it's done will happily loop forever, burning money every turn. Guard it with hard budgets: a max step count, a wall-clock timeout, a cost ceiling. Add loop detection (is it calling the same tool with the same args again?), and a forced-termination fallback that returns a graceful "I couldn't complete this" instead of spinning.</p>
    </div>
    <div class="ag-g">
      <div class="hd"><span class="nm">2 · Wrong tool, hallucinated arguments</span><span class="fix">crisp schemas + validation + fewer tools per state</span></div>
      <p>The model picks the wrong tool or invents a bad argument, and it's worst when you've given it many similar-sounding tools. Fight it three ways: crisp, non-overlapping tool descriptions; strict argument validation before execution; and limiting the toolset available at each state so the model chooses from a short, relevant menu instead of a sprawling one.</p>
    </div>
    <div class="ag-g">
      <div class="hd"><span class="nm">3 · Compounding errors</span><span class="fix">trace everything + trajectory evals + scope credentials per tool</span></div>
      <p>A small mistake early, the wrong customer ID, a misread result, propagates through every later step and comes out as a confidently wrong final answer. This is exactly why tracing is non-negotiable and why evals must score the trajectory, not just the ending. And scope credentials tightly, per tool, least privilege: a bug in an autonomous agent that holds broad write access doesn't make a typo, it does real damage to real data.</p>
    </div>
  </div>
  <figcaption>Loop budgets, tight tool schemas, and per-tool least-privilege credentials. The theme underneath all three: an agent's autonomy is a loaded tool. The guardrails aren't there to make it smarter, they're there to bound the blast radius when it's wrong, and over enough runs, it will be wrong.</figcaption>
</figure>

## The takeaway

An agent is the most capable shape in this series and the one to reach for last. The whole craft is restraint: climb the ladder only as far as the problem forces you, push everything you can into deterministic workflow, and hand the loop only the genuinely dynamic decisions. When you do build the loop, model it as an explicit graph, give it a handful of crisp idempotent tools, split memory into a trimmed scratchpad and a selectively-written store, persist state after every step so a crash resumes and a human can approve the risky moves, and trace and eval the whole trajectory because the thing behaves differently every time you run it.

Do that and you get an agent that can chase down a churning customer across tickets and billing and come back with a grounded draft. Skip it and you get a very expensive random number generator with write access to production. The difference is entirely in the guardrails, and you build them before the launch, not after the postmortem.

<script>
(function(){
  var els=document.querySelectorAll('.ag-anim');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.2});
  els.forEach(function(e){io.observe(e)});
})();
</script>
