---
title: "One Agent or Many? How to Decide Between Single and Multi-Agent"
date: 2026-08-03
excerpt: "Multi-agent systems are having a moment, and most teams reaching for one don't need it. But some genuinely do. Two of the sharpest engineering teams around published opposite-sounding advice in the same month: one said don't build multi-agents, the other said we built one and it beat single-agent by 90%. They're both right, and the gap between them is exactly the decision you have to make. Here's what each architecture is, the multi-agent shapes, the honest costs, and a framework for choosing."
tags: [ai, agents, multi-agent, architecture, orchestration, llm]
---

<style>
.ma-fig{margin:2.5rem 0;}
.ma-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* single vs multi diagram */
.ma-sm{max-width:720px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:1rem;}
@media(max-width:600px){.ma-sm{grid-template-columns:1fr;}}
.ma-sm .box{border:1px solid var(--border);border-radius:14px;padding:1.2rem;background:var(--surface);opacity:0;transform:translateY(10px);transition:opacity .5s ease,transform .5s ease;}
.ma-sm.go .box{opacity:1;transform:none;} .ma-sm.go .box:nth-child(2){transition-delay:.18s;}
.ma-sm .box h4{margin:0 0 .8rem;font-size:.85rem;font-family:var(--font-mono);color:var(--accent);}
.ma-sm .box svg{width:100%;height:130px;overflow:visible;}
.ma-sm .box p{font-size:.83rem;color:var(--text-2);line-height:1.5;margin:.7rem 0 0;}
.ma-nd{fill:var(--surface-2);stroke:var(--border-2);stroke-width:1.5;}
.ma-nd.acc{stroke:var(--accent);}
.ma-ed{stroke:var(--accent);stroke-width:1.5;fill:none;opacity:.6;}
.ma-tx{fill:var(--text);font-family:var(--font-mono);font-size:9px;text-anchor:middle;}

/* topology cards */
.ma-topo{max-width:760px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:.8rem;}
@media(max-width:600px){.ma-topo{grid-template-columns:1fr;}}
.ma-tcard{border:1px solid var(--border-2);border-radius:12px;padding:1rem;background:var(--surface);opacity:0;transform:translateY(8px);transition:opacity .45s ease,transform .45s ease;}
.ma-topo.go .ma-tcard{opacity:1;transform:none;}
.ma-topo.go .ma-tcard:nth-child(1){transition-delay:.08s} .ma-topo.go .ma-tcard:nth-child(2){transition-delay:.18s}
.ma-topo.go .ma-tcard:nth-child(3){transition-delay:.28s} .ma-topo.go .ma-tcard:nth-child(4){transition-delay:.38s}
.ma-topo.go .ma-tcard:nth-child(5){transition-delay:.48s} .ma-topo.go .ma-tcard:nth-child(6){transition-delay:.58s}
.ma-tcard .nm{font-family:var(--font-mono);font-size:.8rem;color:var(--accent);font-weight:600;margin-bottom:.3rem;}
.ma-tcard svg{width:100%;height:64px;overflow:visible;margin:.4rem 0;}
.ma-tcard .d{font-size:.8rem;color:var(--text-2);line-height:1.45;} .ma-tcard .d b{color:var(--text);}
.ma-tn{fill:var(--surface-2);stroke:var(--border-2);stroke-width:1.3;} .ma-tn.a{fill:var(--accent);stroke:var(--accent);}
.ma-te{stroke:var(--accent);stroke-width:1.3;fill:none;opacity:.65;}

/* debate two-sides */
.ma-debate{max-width:720px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:1rem;}
@media(max-width:600px){.ma-debate{grid-template-columns:1fr;}}
.ma-side{border:1px solid var(--border);border-radius:14px;background:var(--surface);overflow:hidden;opacity:0;transform:translateY(10px);transition:opacity .5s ease,transform .5s ease;}
.ma-debate.go .ma-side{opacity:1;transform:none;} .ma-debate.go .ma-side:nth-child(2){transition-delay:.2s;}
.ma-side .hd{padding:.85rem 1.1rem;border-bottom:1px solid var(--border);font-family:var(--font-mono);font-size:.82rem;font-weight:600;}
.ma-side.no .hd{color:var(--text-2);} .ma-side.yes .hd{color:var(--accent);}
.ma-side .bd{padding:1rem 1.1rem;font-size:.85rem;color:var(--text-2);line-height:1.55;}
.ma-side .bd .who{font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);margin-bottom:.5rem;}
.ma-side .bd .pt{font-family:var(--font-mono);font-size:.92rem;color:var(--text);margin-bottom:.5rem;}

/* cost bars */
.ma-cost{max-width:640px;margin:0 auto;display:flex;flex-direction:column;gap:.8rem;}
.ma-crow{display:grid;grid-template-columns:130px 1fr;gap:.9rem;align-items:center;}
@media(max-width:520px){.ma-crow{grid-template-columns:95px 1fr;}}
.ma-crow .lab{font-family:var(--font-mono);font-size:.78rem;color:var(--text-2);}
.ma-ctrack{height:26px;background:var(--surface-2);border-radius:6px;overflow:hidden;position:relative;}
.ma-cfill{height:100%;width:0;border-radius:6px;transition:width 1s ease;display:flex;align-items:center;justify-content:flex-end;padding-right:.55rem;}
.ma-cost.go .ma-cfill{width:var(--w);}
.ma-cfill.a{background:color-mix(in srgb,var(--green) 55%,var(--surface));}
.ma-cfill.b{background:color-mix(in srgb,var(--accent) 55%,var(--surface));}
.ma-cfill.c{background:linear-gradient(90deg,var(--accent-2),var(--accent));}
.ma-cfill .v{font-family:var(--font-mono);font-size:.72rem;color:var(--accent-ink);font-weight:600;}
.ma-cfill.a .v{color:var(--text);}
.ma-cost .foot{font-family:var(--font-mono);font-size:.74rem;color:var(--text-3);text-align:center;margin-top:.4rem;}

/* decision flow */
.ma-dec{max-width:640px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.ma-q{display:flex;gap:.85rem;align-items:flex-start;border:1px solid var(--border);border-radius:10px;padding:.75rem .95rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.ma-dec.go .ma-q{opacity:1;transform:none;}
.ma-dec.go .ma-q:nth-child(1){transition-delay:.1s} .ma-dec.go .ma-q:nth-child(2){transition-delay:.24s}
.ma-dec.go .ma-q:nth-child(3){transition-delay:.38s} .ma-dec.go .ma-q:nth-child(4){transition-delay:.52s}
.ma-dec.go .ma-q:nth-child(5){transition-delay:.66s}
.ma-q .ans{flex:none;font-family:var(--font-mono);font-size:.72rem;border-radius:6px;padding:.15rem .5rem;margin-top:.05rem;}
.ma-q .ans.multi{color:var(--accent);border:1px solid var(--accent);}
.ma-q .ans.single{color:var(--green);border:1px solid var(--green);}
.ma-q .t{font-size:.85rem;color:var(--text-2);line-height:1.45;} .ma-q .t b{color:var(--text);}

/* pros cons table */
.ma-tab-wrap{max-width:820px;margin:0 auto;overflow-x:auto;border:1px solid var(--border);border-radius:12px;background:var(--surface);}
.ma-tab{width:100%;border-collapse:collapse;font-size:.84rem;min-width:640px;}
.ma-tab th,.ma-tab td{text-align:left;padding:.65rem .8rem;border-bottom:1px solid var(--border);vertical-align:top;}
.ma-tab th{font-family:var(--font-mono);font-size:.68rem;text-transform:uppercase;letter-spacing:.04em;color:var(--text-3);font-weight:500;}
.ma-tab td:first-child,.ma-tab th:first-child{color:var(--text);font-weight:500;font-family:var(--font-mono);font-size:.76rem;}
.ma-tab tr:last-child td{border-bottom:none;}
.ma-tab .s{color:var(--green);} .ma-tab .m{color:var(--accent);}

@media (prefers-reduced-motion: reduce){
  .ma-sm .box,.ma-tcard,.ma-side,.ma-q{opacity:1!important;transform:none!important;}
  .ma-cost .ma-cfill{transition:none!important;width:var(--w)!important;}
}
</style>

In June 2025, two of the sharpest engineering teams in AI published advice that sounds like a flat contradiction, in the same month.

Cognition put out a post titled, bluntly, "Don't Build Multi-Agents." Anthropic put out one titled "How we built our multi-agent research system," in which their multi-agent setup beat a single agent by over 90% on their internal research benchmark.

So which is it? Build multi-agents or don't?

The answer, and the reason this is worth a whole post, is that they're both right. They're describing different tasks. The space between their two conclusions is exactly the decision you have to make every time you design an agentic system: **one agent, or several?** Get it right and you ship something fast, cheap, and debuggable. Get it wrong in the fashionable direction and you ship something that costs ten times as much and breaks in ways you can't trace.

Let me lay out what each architecture actually is, the shapes multi-agent comes in, the honest costs, and a framework for choosing.

## What they actually are

If you've read [what an agent really is](/blog/2026-07-14-what-is-an-agent-really/), the single-agent picture is familiar: one model in a loop. It gets a task, reasons, calls a tool, reads the result, and repeats until done. One context window holding everything it's seen, one set of tools, one thread of control. Every decision is made by the same model with full view of everything.

A multi-agent system splits that up. Several LLM instances, usually specialized, each often with its own context window, its own role, its own subset of tools, coordinated by some mechanism: a lead agent that delegates, a fixed pipeline, a shared message channel, or direct handoffs between peers.

<figure class="ma-fig">
<div class="ma-sm">
  <div class="box">
    <h4>Single agent</h4>
    <svg viewBox="0 0 200 120" role="img" aria-label="One agent looping over tools with one context.">
      <circle class="ma-nd acc" cx="100" cy="45" r="26"></circle>
      <text class="ma-tx" x="100" y="48">agent</text>
      <path class="ma-ed" d="M100 71 v18" marker-end=""></path>
      <rect class="ma-nd" x="45" y="90" width="42" height="22" rx="5"></rect><text class="ma-tx" x="66" y="104">tool</text>
      <rect class="ma-nd" x="113" y="90" width="42" height="22" rx="5"></rect><text class="ma-tx" x="134" y="104">tool</text>
      <path class="ma-ed" d="M100 71 q-40 5 -34 19"></path>
      <path class="ma-ed" d="M100 71 q40 5 34 19"></path>
    </svg>
    <p>One model, one context, one control loop. It sees everything it has done. Simple, cheap, deterministic-ish, easy to debug.</p>
  </div>
  <div class="box">
    <h4>Multi-agent</h4>
    <svg viewBox="0 0 200 120" role="img" aria-label="A lead agent delegating to three worker agents.">
      <circle class="ma-nd acc" cx="100" cy="26" r="20"></circle><text class="ma-tx" x="100" y="29">lead</text>
      <circle class="ma-nd" cx="45" cy="92" r="18"></circle><text class="ma-tx" x="45" y="95">a1</text>
      <circle class="ma-nd" cx="100" cy="92" r="18"></circle><text class="ma-tx" x="100" y="95">a2</text>
      <circle class="ma-nd" cx="155" cy="92" r="18"></circle><text class="ma-tx" x="155" y="95">a3</text>
      <path class="ma-ed" d="M88 40 L52 76"></path>
      <path class="ma-ed" d="M100 46 L100 74"></path>
      <path class="ma-ed" d="M112 40 L148 76"></path>
    </svg>
    <p>Several specialized models, each with its own context and tools, coordinated. More capable on the right task, and far more moving parts.</p>
  </div>
</div>
<figcaption>The difference isn't "smarter." It's how many context windows, how many decision-makers, and how many LLM calls. That shift is what buys you capability and what costs you money and simplicity.</figcaption>
</figure>

One thing to note before going further: this is generative-AI, LLM-agent territory. It has nothing to do with classical multi-agent reinforcement learning, which shares the name and none of the design questions.

## The shapes multi-agent comes in

"Multi-agent" isn't one thing. When you do decide to use several agents, you're picking a topology, and they're not interchangeable. These names are broadly consistent across LangGraph, Google's ADK, Microsoft's Agent Framework, and AWS Bedrock.

<figure class="ma-fig">
<div class="ma-topo">
  <div class="ma-tcard">
    <div class="nm">Supervisor</div>
    <svg viewBox="0 0 120 44"><circle class="ma-tn a" cx="60" cy="10" r="7"></circle><circle class="ma-tn" cx="25" cy="36" r="6"></circle><circle class="ma-tn" cx="60" cy="36" r="6"></circle><circle class="ma-tn" cx="95" cy="36" r="6"></circle><path class="ma-te" d="M55 15 L28 31"></path><path class="ma-te" d="M60 17 L60 30"></path><path class="ma-te" d="M65 15 L92 31"></path></svg>
    <div class="d">A lead agent breaks up the task, delegates to workers, and synthesizes their answers. <b>The most common production pattern.</b></div>
  </div>
  <div class="ma-tcard">
    <div class="nm">Pipeline</div>
    <svg viewBox="0 0 120 44"><circle class="ma-tn" cx="20" cy="22" r="7"></circle><circle class="ma-tn" cx="60" cy="22" r="7"></circle><circle class="ma-tn" cx="100" cy="22" r="7"></circle><path class="ma-te" d="M27 22 L53 22"></path><path class="ma-te" d="M67 22 L93 22"></path></svg>
    <div class="d">Agents run in a fixed order, each consuming the last one's output. <b>For known, ordered stages.</b> The order is code, not a guess.</div>
  </div>
  <div class="ma-tcard">
    <div class="nm">Parallel</div>
    <svg viewBox="0 0 120 44"><circle class="ma-tn" cx="60" cy="8" r="6"></circle><circle class="ma-tn" cx="20" cy="30" r="6"></circle><circle class="ma-tn" cx="60" cy="36" r="6"></circle><circle class="ma-tn" cx="100" cy="30" r="6"></circle><circle class="ma-tn a" cx="60" cy="22" r="0"></circle><path class="ma-te" d="M56 12 L24 26"></path><path class="ma-te" d="M60 14 L60 30"></path><path class="ma-te" d="M64 12 L96 26"></path></svg>
    <div class="d">Agents work concurrently on independent pieces, results merged. <b>For genuinely decomposable tasks,</b> cuts wall-clock time.</div>
  </div>
  <div class="ma-tcard">
    <div class="nm">Network / swarm</div>
    <svg viewBox="0 0 120 44"><circle class="ma-tn" cx="25" cy="14" r="6"></circle><circle class="ma-tn" cx="95" cy="14" r="6"></circle><circle class="ma-tn" cx="60" cy="36" r="6"></circle><path class="ma-te" d="M31 15 L89 15"></path><path class="ma-te" d="M28 19 L55 32"></path><path class="ma-te" d="M92 19 L65 32"></path></svg>
    <div class="d">Peers hand control directly to each other, no boss. <b>For flexible flows</b> where the next specialist depends on what just happened.</div>
  </div>
  <div class="ma-tcard">
    <div class="nm">Hierarchical</div>
    <svg viewBox="0 0 120 44"><circle class="ma-tn a" cx="60" cy="8" r="6"></circle><circle class="ma-tn" cx="32" cy="26" r="5"></circle><circle class="ma-tn" cx="88" cy="26" r="5"></circle><circle class="ma-tn" cx="20" cy="40" r="4"></circle><circle class="ma-tn" cx="44" cy="40" r="4"></circle><circle class="ma-tn" cx="76" cy="40" r="4"></circle><circle class="ma-tn" cx="100" cy="40" r="4"></circle><path class="ma-te" d="M56 12 L35 22"></path><path class="ma-te" d="M64 12 L85 22"></path><path class="ma-te" d="M30 30 L22 36"></path><path class="ma-te" d="M34 30 L42 36"></path><path class="ma-te" d="M86 30 L78 36"></path><path class="ma-te" d="M90 30 L98 36"></path></svg>
    <div class="d">Supervisors managing supervisors. <b>For big problems</b> that split into sub-domains, each needing its own decomposition.</div>
  </div>
  <div class="ma-tcard">
    <div class="nm">Reflection / debate</div>
    <svg viewBox="0 0 120 44"><circle class="ma-tn a" cx="40" cy="22" r="7"></circle><circle class="ma-tn" cx="90" cy="22" r="7"></circle><path class="ma-te" d="M47 19 Q68 8 83 19"></path><path class="ma-te" d="M83 25 Q68 36 47 25"></path></svg>
    <div class="d">One agent drafts, another critiques, they loop until it's good. <b>For quality over speed:</b> code review, drafting with a validator.</div>
  </div>
</div>
<figcaption>Six patterns. Most production multi-agent systems are a supervisor delegating to workers. Pipeline and parallel are often better built as deterministic code (the order is yours, not the model's) rather than "true" agents making routing decisions.</figcaption>
</figure>

## The debate, and why both sides are right

Now back to those two posts, because reading them together is the whole lesson.

<figure class="ma-fig">
<div class="ma-debate">
  <div class="ma-side no">
    <div class="hd">"Don't build multi-agents"</div>
    <div class="bd">
      <div class="who">Cognition (Walden Yan), June 2025</div>
      <div class="pt">Default to a single, linear agent.</div>
      When parallel agents can't see what the others are doing, they make conflicting assumptions and the whole thing gets fragile. Their case: coding. Two subagents building different parts of one app produce pieces that don't fit together, because each guessed at the shared decisions the other made.
    </div>
  </div>
  <div class="ma-side yes">
    <div class="hd">"We built one, it won by 90%"</div>
    <div class="bd">
      <div class="who">Anthropic, June 2025</div>
      <div class="pt">Multi-agent, for the right task.</div>
      Their orchestrator-worker research system (a lead plus subagents) beat single-agent by 90.2% on their internal eval. Their case: research. Breadth-first questions where independent subagents each chase a separate lead in parallel, then the lead synthesizes.
    </div>
  </div>
</div>
<figcaption>Not a contradiction. Cognition's task is tightly-coupled writing with shared state (coding). Anthropic's is parallel, read-heavy exploration (research). The deciding variable is whether your task actually splits into independent pieces.</figcaption>
</figure>

That's the reconciliation, and it's the single most useful idea here. Anthropic themselves say it plainly: their approach fits "heavy parallelization" and work that "exceeds single context windows," and it does *not* fit "domains that require all agents to share the same context or involve many dependencies between agents." They even note most coding tasks have fewer truly parallel pieces than research. Cognition and Anthropic aren't arguing. They're pointing at different tasks and giving the correct answer for each.

## The part nobody puts on the slide: cost

Here's why the default should be single-agent, and why "let's use agents for everything" is an expensive habit. Multi-agent buys capability with tokens, a lot of them. Anthropic measured it.

<figure class="ma-fig">
<div class="ma-cost">
  <div class="ma-crow"><span class="lab">Plain chat</span><div class="ma-ctrack"><div class="ma-cfill a" style="--w:8%"><span class="v">1x</span></div></div></div>
  <div class="ma-crow"><span class="lab">Single agent</span><div class="ma-ctrack"><div class="ma-cfill b" style="--w:27%"><span class="v">~4x tokens</span></div></div></div>
  <div class="ma-crow"><span class="lab">Multi-agent</span><div class="ma-ctrack"><div class="ma-cfill c" style="--w:100%"><span class="v">~15x tokens</span></div></div></div>
</div>
<div class="foot">token use vs a plain chat interaction, per Anthropic's engineering post</div>
</figure>

An agent already uses roughly 4x the tokens of a plain chat, because it loops and accumulates context. A multi-agent system uses about 15x. And in their evaluation, token usage alone explained about 80% of the performance variance, which means the capability gain is largely something you're *paying for* in tokens, not getting for free. Their own honest caveat: multi-agent only pays off "when the value of the task is high enough to pay for the increased performance."

On top of the token bill, you get coordination latency (the lead has to plan, delegate, and synthesize, each an extra call), compounding errors across agents, and a system that's genuinely harder to debug because the failure might be in any of several models or in how they talked to each other.

## Single vs multi, side by side

<figure class="ma-fig">
<div class="ma-tab-wrap">
<table class="ma-tab">
<thead>
<tr><th>Dimension</th><th class="s">Single agent</th><th class="m">Multi-agent</th></tr>
</thead>
<tbody>
<tr><td>Cost</td><td class="s">~4x chat</td><td class="m">~15x chat</td></tr>
<tr><td>Latency</td><td class="s">Lower</td><td class="m">Higher (coordination)</td></tr>
<tr><td>Debuggability</td><td class="s">One trace to read</td><td class="m">Many traces, plus their interaction</td></tr>
<tr><td>Shared context</td><td class="s">Total, sees everything</td><td class="m">Split, agents can conflict</td></tr>
<tr><td>Parallelism</td><td class="s">None</td><td class="m">Real, on decomposable tasks</td></tr>
<tr><td>Context limit</td><td class="s">One window</td><td class="m">Scales past one window</td></tr>
<tr><td>Best fit</td><td class="s">Sequential, coupled, coding</td><td class="m">Parallel, read-heavy, research</td></tr>
</tbody>
</table>
</div>
<figcaption>Single-agent wins on almost every operational axis. Multi-agent wins on capability for a specific task shape. That asymmetry is why single should be your default and multi should be a decision you justify.</figcaption>
</figure>

## The decision, as questions

You don't choose by vibe or by what's trending. You choose by answering a few honest questions about the task. If you're saying "single" to most of these, build one good agent with good tools and stop there.

<figure class="ma-fig">
<div class="ma-dec">
  <div class="ma-q"><span class="ans multi">multi</span><span class="t"><b>Does the task split into genuinely independent subtasks?</b> If pieces can run without knowing what the others are doing, parallel agents pay off. If they're tightly coupled, they'll conflict.</span></div>
  <div class="ma-q"><span class="ans multi">multi</span><span class="t"><b>Do subtasks need different tools or different specialized context?</b> A researcher and a code-runner wanting totally different context is a real reason to separate them.</span></div>
  <div class="ma-q"><span class="ans multi">multi</span><span class="t"><b>Does the work exceed one context window?</b> If no single agent can hold it all, splitting the context across agents is a legitimate fix.</span></div>
  <div class="ma-q"><span class="ans single">single</span><span class="t"><b>Do later steps depend heavily on earlier ones?</b> Tight coupling and shared state mean one agent that sees everything beats several that each see a slice.</span></div>
  <div class="ma-q"><span class="ans single">single</span><span class="t"><b>Do cost, latency, or debuggability matter more than raw capability?</b> Then eat a little less capability for a system you can afford and actually fix.</span></div>
</div>
<figcaption>A rough rule: start single. Go multi only when the task is genuinely parallel, high-value enough to justify 15x tokens, and would otherwise overflow one agent's context. When in doubt, better tools on one agent beats more agents.</figcaption>
</figure>

## The takeaway

Multi-agent is not the advanced version of single-agent. It's a different trade: you spend an order of magnitude more tokens and a lot of simplicity to buy parallelism and specialized context, and that trade is only worth it when the task actually decomposes and is valuable enough to justify the bill.

The reason so many multi-agent projects disappoint is that they were built for tasks that were really sequential and coupled, where a single agent with a good toolset would have been cheaper, faster, and easier to debug. The reason Anthropic's worked is that research genuinely fans out into independent threads. The reason Cognition warned against it is that coding genuinely doesn't.

So the honest default is one agent. Give it good tools, good context, and the discipline from the [agent design post](/blog/2026-07-24-designing-an-agent-that-doesnt-go-off-the-rails/). Reach for many only when you can look at your task and say, truthfully, "this splits into parallel pieces that don't need to watch each other." If you can't say that, you don't have a multi-agent problem. You have a single agent that needs better tools.

*Foundations for this post: [what an agent really is](/blog/2026-07-14-what-is-an-agent-really/), [how AI agents actually work](/blog/2026-07-04-how-ai-agents-actually-work/), and [LangGraph](/blog/2026-07-17-langgraph-when-your-ai-needs-a-flowchart-that-runs/) for building the graphs these topologies run on.*

## References

Written from scratch. These are the primary, verified sources, including the two posts that frame the debate and the cost numbers. Nothing here is copied from them.

- Anthropic Engineering, how we built our multi-agent research system (source of the 90.2% and token figures): <https://www.anthropic.com/engineering/multi-agent-research-system>
- Cognition (Walden Yan), Don't Build Multi-Agents: <https://cognition.com/blog/dont-build-multi-agents>
- LangGraph multi-agent supervisor (reference): <https://reference.langchain.com/python/langgraph-supervisor>
- LangChain multi-agent / subagents guide: <https://docs.langchain.com/oss/python/langchain/multi-agent/subagents-personal-assistant>
- LangChain blog, LangGraph multi-agent workflows: <https://www.langchain.com/blog/langgraph-multi-agent-workflows>
- Google Developers, guide to multi-agent patterns in ADK: <https://developers.googleblog.com/developers-guide-to-multi-agent-patterns-in-adk/>
- AWS Bedrock, create multi-agent collaboration: <https://docs.aws.amazon.com/bedrock/latest/userguide/create-multi-agent-collaboration.html>
- AWS, introducing multi-agent collaboration for Bedrock: <https://aws.amazon.com/blogs/aws/introducing-multi-agent-collaboration-capability-for-amazon-bedrock/>
- Microsoft Agent Framework overview: <https://learn.microsoft.com/en-us/agent-framework/overview/>
- OpenAI Swarm (educational multi-agent framework): <https://github.com/openai/swarm>

<script>
(function(){
  var els=document.querySelectorAll('.ma-sm,.ma-topo,.ma-debate,.ma-cost,.ma-dec');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.18});
  els.forEach(function(e){io.observe(e)});
})();
</script>
