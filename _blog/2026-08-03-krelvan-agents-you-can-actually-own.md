---
title: "Krelvan: AI Agents You Can Actually Own, Inspect, and Trust"
date: 2026-08-03
excerpt: "A demo agent is easy. A dependable one is not. The moment you want an agent to run on a schedule, touch real tools, spend money, or email a customer, you need answers a prompt can't give: what program was actually built, what was it allowed to do, what did it really do, and can someone verify that without trusting your dashboard. Krelvan is my attempt at that. This is what I built, why it works the way it does, and where it's headed."
tags: [ai, agents, krelvan, architecture, self-hosted, project]
---

<style>
.kr-fig{margin:2.5rem 0;}
.kr-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* seven products */
.kr-seven{max-width:760px;margin:0 auto;display:grid;grid-template-columns:repeat(4,1fr);gap:.7rem;}
@media(max-width:680px){.kr-seven{grid-template-columns:repeat(2,1fr);}}
.kr-pc{border:1px solid var(--border-2);border-radius:12px;padding:.9rem;background:var(--surface);opacity:0;transform:translateY(10px);transition:opacity .45s ease,transform .45s ease;}
.kr-seven.go .kr-pc{opacity:1;transform:none;}
.kr-seven.go .kr-pc:nth-child(1){transition-delay:.06s} .kr-seven.go .kr-pc:nth-child(2){transition-delay:.14s}
.kr-seven.go .kr-pc:nth-child(3){transition-delay:.22s} .kr-seven.go .kr-pc:nth-child(4){transition-delay:.30s}
.kr-seven.go .kr-pc:nth-child(5){transition-delay:.38s} .kr-seven.go .kr-pc:nth-child(6){transition-delay:.46s}
.kr-seven.go .kr-pc:nth-child(7){transition-delay:.54s}
.kr-pc .ic{font-family:var(--font-mono);font-size:.66rem;color:var(--accent);border:1px solid var(--accent);border-radius:5px;padding:.1rem .35rem;}
.kr-pc b{display:block;color:var(--text);font-size:.86rem;margin:.4rem 0 .2rem;}
.kr-pc span{font-size:.76rem;color:var(--text-2);line-height:1.4;}

/* architecture diagram (layered) */
.kr-arch{max-width:720px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.kr-lane{border:1px solid var(--border);border-radius:12px;padding:.85rem 1rem;background:var(--surface);opacity:0;transform:translateX(-10px);transition:opacity .5s ease,transform .5s ease;}
.kr-arch.go .kr-lane{opacity:1;transform:none;}
.kr-arch.go .kr-lane:nth-child(1){transition-delay:.08s} .kr-arch.go .kr-lane:nth-child(2){transition-delay:.2s}
.kr-arch.go .kr-lane:nth-child(3){transition-delay:.32s} .kr-arch.go .kr-lane:nth-child(4){transition-delay:.44s}
.kr-arch.go .kr-lane:nth-child(5){transition-delay:.56s}
.kr-lane .lh{font-family:var(--font-mono);font-size:.72rem;letter-spacing:.06em;text-transform:uppercase;color:var(--accent);margin-bottom:.5rem;}
.kr-lane.core{border-color:var(--accent);}
.kr-chips{display:flex;flex-wrap:wrap;gap:.4rem;}
.kr-chip{font-family:var(--font-mono);font-size:.73rem;color:var(--text-2);background:var(--surface-2);border:1px solid var(--border);border-radius:7px;padding:.25rem .55rem;}
.kr-chip.acc{color:var(--accent);border-color:var(--accent);}
.kr-arch .down{text-align:center;color:var(--text-3);font-family:var(--font-mono);font-size:.9rem;line-height:.5;}

/* create flow */
.kr-flow{max-width:820px;margin:0 auto;overflow-x:auto;}
.kr-frow{display:flex;flex-wrap:nowrap;gap:.4rem;align-items:stretch;min-width:660px;}
.kr-node{flex:1 1 0;min-width:112px;border:1px solid var(--border-2);border-radius:11px;background:var(--surface);padding:.72rem .68rem;opacity:0;transform:translateY(10px);transition:opacity .45s ease,transform .45s ease;}
.kr-frow.go .kr-node{opacity:1;transform:none;}
.kr-frow.go .kr-node:nth-child(1){transition-delay:.05s} .kr-frow.go .kr-node:nth-child(3){transition-delay:.18s}
.kr-frow.go .kr-node:nth-child(5){transition-delay:.31s} .kr-frow.go .kr-node:nth-child(7){transition-delay:.44s}
.kr-frow.go .kr-node:nth-child(9){transition-delay:.57s}
.kr-node.acc{border-color:var(--accent);}
.kr-node .st{font-family:var(--font-mono);font-size:.65rem;color:var(--accent);}
.kr-node b{display:block;color:var(--text);font-size:.8rem;margin:.2rem 0 .1rem;line-height:1.25;}
.kr-node span{font-size:.72rem;color:var(--text-3);line-height:1.35;}
.kr-arr{align-self:center;color:var(--text-3);font-family:var(--font-mono);flex:none;}

/* the loop */
.kr-loop{max-width:560px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.kr-lstep{display:flex;align-items:center;gap:.85rem;border:1px solid var(--border);border-radius:10px;padding:.65rem .9rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .45s ease,transform .45s ease;}
.kr-loop.go .kr-lstep{opacity:1;transform:none;}
.kr-loop.go .kr-lstep:nth-child(1){transition-delay:.1s} .kr-loop.go .kr-lstep:nth-child(2){transition-delay:.22s}
.kr-loop.go .kr-lstep:nth-child(3){transition-delay:.34s} .kr-loop.go .kr-lstep:nth-child(4){transition-delay:.46s}
.kr-loop.go .kr-lstep:nth-child(5){transition-delay:.58s}
.kr-lstep .n{flex:none;width:24px;height:24px;border-radius:50%;border:1px solid var(--accent);color:var(--accent);font-family:var(--font-mono);font-size:.72rem;display:flex;align-items:center;justify-content:center;}
.kr-lstep .t{font-size:.85rem;color:var(--text-2);} .kr-lstep .t b{color:var(--text);}

/* side-effect classes table */
.kr-tab-wrap{max-width:760px;margin:0 auto;overflow-x:auto;border:1px solid var(--border);border-radius:12px;background:var(--surface);}
.kr-tab{width:100%;border-collapse:collapse;font-size:.84rem;min-width:560px;}
.kr-tab th,.kr-tab td{text-align:left;padding:.6rem .8rem;border-bottom:1px solid var(--border);vertical-align:top;}
.kr-tab th{font-family:var(--font-mono);font-size:.68rem;text-transform:uppercase;letter-spacing:.04em;color:var(--text-3);font-weight:500;}
.kr-tab td:first-child,.kr-tab th:first-child{color:var(--text);font-weight:500;font-family:var(--font-mono);font-size:.76rem;}
.kr-tab tr:last-child td{border-bottom:none;}
.kr-tab .g{color:var(--green);font-family:var(--font-mono);font-size:.76rem;}
.kr-tab .w{color:var(--accent);font-family:var(--font-mono);font-size:.76rem;}
.kr-tab .r{color:var(--accent-2);font-family:var(--font-mono);font-size:.76rem;}

/* effect protocol three-part */
.kr-proto{max-width:680px;margin:0 auto;display:grid;grid-template-columns:repeat(3,1fr);gap:.7rem;}
@media(max-width:560px){.kr-proto{grid-template-columns:1fr;}}
.kr-ppanel{border:1px solid var(--border-2);border-radius:12px;padding:1rem;background:var(--surface);text-align:center;opacity:0;transform:translateY(8px);transition:opacity .45s ease,transform .45s ease;}
.kr-proto.go .kr-ppanel{opacity:1;transform:none;}
.kr-proto.go .kr-ppanel:nth-child(1){transition-delay:.1s} .kr-proto.go .kr-ppanel:nth-child(2){transition-delay:.28s} .kr-proto.go .kr-ppanel:nth-child(3){transition-delay:.46s}
.kr-ppanel .k{font-family:var(--font-mono);font-size:.7rem;color:var(--accent);margin-bottom:.4rem;}
.kr-ppanel b{display:block;color:var(--text);font-size:.86rem;margin-bottom:.25rem;} .kr-ppanel span{font-size:.78rem;color:var(--text-2);line-height:1.4;}

/* registry stats */
.kr-stats{max-width:640px;margin:0 auto;display:grid;grid-template-columns:repeat(4,1fr);gap:.7rem;}
@media(max-width:520px){.kr-stats{grid-template-columns:repeat(2,1fr);}}
.kr-stat{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);text-align:center;opacity:0;transform:scale(.94);transition:opacity .45s ease,transform .45s ease;}
.kr-stats.go .kr-stat{opacity:1;transform:none;}
.kr-stats.go .kr-stat:nth-child(1){transition-delay:.08s} .kr-stats.go .kr-stat:nth-child(2){transition-delay:.2s}
.kr-stats.go .kr-stat:nth-child(3){transition-delay:.32s} .kr-stats.go .kr-stat:nth-child(4){transition-delay:.44s}
.kr-stat .num{font-family:var(--font-mono);font-size:1.6rem;color:var(--accent);font-weight:600;}
.kr-stat .lb{font-size:.76rem;color:var(--text-2);margin-top:.2rem;}

@media (prefers-reduced-motion: reduce){
  .kr-seven,.kr-pc,.kr-lane,.kr-node,.kr-lstep,.kr-ppanel,.kr-stat{opacity:1!important;transform:none!important;}
}
</style>

Building an AI agent that looks impressive is easy. You write a good prompt, wire up a tool or two, and the demo lands. Building one you'd actually trust to run unattended is a completely different job, and it's the job most agent tooling quietly skips.

Because the moment you want an agent to run on a schedule, touch real tools, spend money, or email a customer, the questions change. What exact program got built from my request? Which tools was it allowed to use, and which did it actually call? What happens if it crashes between asking to send an email and recording that it sent one? Can I stop it before it does something irreversible? And can someone else verify the record without trusting my dashboard?

A prompt can't answer any of those. [Krelvan](https://krelvan.com) is my attempt to build the thing that can. It's a self-hosted operating environment for AI agents: you describe an outcome or install a pre-built agent, inspect the program it produces, run it through real tools and models, stop consequential actions for human approval, receive the output in an Inbox or an external channel, and keep a record you can replay and check. This post walks the whole product, what each part does, why it works the way it does, and where it's headed.

And the framing that runs through all of it: Krelvan is **self-hosted**. You run it, your data lives in your directory, your keys sign your records. You're not shipping your whole operating environment to a hosted black box and hoping.

## Seven products, one runtime

The easiest way to understand Krelvan is as seven cooperating products that share a single runtime. It's not only an agent builder, or only a canvas, or only an automation runner, or only an audit system. It's all of these, and each one is a real surface. Together they're what turns "a model in a loop" into something you can actually operate.

<figure class="kr-fig">
<div class="kr-seven">
  <div class="kr-pc"><span class="ic">1</span><b>Studio</b><span>Describe an outcome in plain language or install a template, then inspect the graph it built.</span></div>
  <div class="kr-pc"><span class="ic">2</span><b>Runtime</b><span>Deterministic fold, decide, append execution with bounded graphs and real recovery.</span></div>
  <div class="kr-pc"><span class="ic">3</span><b>Trust layer</b><span>Signed event history, independent verification, exportable proof.</span></div>
  <div class="kr-pc"><span class="ic">4</span><b>Safety layer</b><span>Deny-by-default authority, side-effect classes, human approval gates.</span></div>
  <div class="kr-pc"><span class="ic">5</span><b>Ecosystem</b><span>Built-ins, YAML and code capabilities, MCP connectors, packs, a Git registry.</span></div>
  <div class="kr-pc"><span class="ic">6</span><b>Operations</b><span>Schedules, triggers, chat, failure diagnosis, correction, replay, rehearsal.</span></div>
  <div class="kr-pc"><span class="ic">7</span><b>Consumption</b><span>An Inbox, external delivery, public agent pages, widgets, shareable outputs.</span></div>
</div>
<figcaption>Studio is the part everyone sees. The harder, more valuable product is everything underneath and after creation: authority, execution, evidence, recovery, and delivery.</figcaption>
</figure>

## The architecture, in lanes

Here's how those pieces stack. The important thing to read here is the direction of dependency: everything points inward toward a small, pure core. And one design choice ties it together: rather than keep a pretty UI history alongside a separate reality, Krelvan derives the canvas, run state, approvals, timeline, accounting, replay, and proof from one append-only signed record. The interface shows what the runtime actually recorded, which is what makes the trust claims real rather than decorative. It's one part of the design, not the whole story, but it's the part that makes the rest trustworthy.

<figure class="kr-fig">
<div class="kr-arch">
  <div class="kr-lane">
    <div class="lh">Customer surfaces</div>
    <div class="kr-chips"><span class="kr-chip">Next.js web app</span><span class="kr-chip">Launcher + verify CLI</span><span class="kr-chip">Webhook / widget / share</span></div>
  </div>
  <div class="down">&darr;</div>
  <div class="kr-lane">
    <div class="lh">Trust and transport boundary</div>
    <div class="kr-chips"><span class="kr-chip">Page middleware</span><span class="kr-chip">Same-origin proxy</span><span class="kr-chip">HTTP API + auth</span></div>
  </div>
  <div class="down">&darr;</div>
  <div class="kr-lane core">
    <div class="lh">Core model (the pure center)</div>
    <div class="kr-chips"><span class="kr-chip acc">Compiler + manifest validation</span><span class="kr-chip acc">Pure kernel + projection</span><span class="kr-chip acc">Engine shell</span><span class="kr-chip acc">Capability admission</span><span class="kr-chip acc">Signed ledger</span></div>
  </div>
  <div class="down">&darr;</div>
  <div class="kr-lane">
    <div class="lh">Extension plane</div>
    <div class="kr-chips"><span class="kr-chip">Built-in capabilities</span><span class="kr-chip">YAML capabilities</span><span class="kr-chip">Sandboxed code plugins</span><span class="kr-chip">MCP servers</span><span class="kr-chip">LLM providers</span></div>
  </div>
  <div class="down">&darr;</div>
  <div class="kr-lane">
    <div class="lh">Owned data directory</div>
    <div class="kr-chips"><span class="kr-chip">ledger.db</span><span class="kr-chip">agents / runs / schedules</span><span class="kr-chip">encrypted secrets + keys</span><span class="kr-chip">memory / RAG / wiki</span></div>
  </div>
</div>
<figcaption>The dependency direction is inward. The pure core (kernel and projection) has no I/O: it takes a program and a history and returns the next decision. The engine shell does the messy real-world work and appends signed facts. The UI only ever reads projections. It never decides what happened.</figcaption>
</figure>

That purity boundary is the trick that makes the rest possible. The core decides *what should happen next* using only the signed history and the agent's program. It can't reach out and do things. A separate impure shell does the actual reaching out, and then records what it observed, signed. Model output, by the way, is always treated as data: it's parsed, validated against a schema and the caller's authority, and signed. No generated text is ever executed as code.

## An agent is a signed program, not a prompt

When you describe an outcome, Krelvan doesn't just stash your prompt. It compiles your intent into a **manifest**: a declarative program with a name, an entry point, a bounded number of steps, nodes (each with a role, an autonomy level, and the exact capabilities it's allowed to use), and edges between them that can be guarded by typed conditions.

Here's the creation flow, and note the loop in the middle: if the compiler doesn't like what the model proposed, it feeds the validation errors back and tries again, within bounds.

<figure class="kr-fig">
<div class="kr-flow">
<div class="kr-frow">
  <div class="kr-node"><span class="st">you</span><b>Describe outcome</b><span>Plain language, or pick a template</span></div>
  <div class="kr-arr">&rarr;</div>
  <div class="kr-node"><span class="st">model</span><b>Propose manifest</b><span>Given your intent + the live capability vocabulary</span></div>
  <div class="kr-arr">&rarr;</div>
  <div class="kr-node acc"><span class="st">compiler</span><b>Validate authority</b><span>Graph, edges, and can't grant more power than you have</span></div>
  <div class="kr-arr">&rarr;</div>
  <div class="kr-node acc"><span class="st">sign</span><b>Sign + save</b><span>The manifest and its provenance are signed</span></div>
  <div class="kr-arr">&rarr;</div>
  <div class="kr-node"><span class="st">you</span><b>Inspect graph</b><span>See the actual program before you run it</span></div>
</div>
</div>
<figcaption>Intent becomes a validated, signed program. The compiler enforces authority monotonicity: a proposal can't grant an agent more authority than the person or context asking for it already holds. If validation fails, the errors go back to the model for bounded self-correction.</figcaption>
</figure>

The conditions on edges use a small, bounded, typed expression language, comparisons and boolean logic over the run's state, nothing that can turn into arbitrary code execution. Your agent's control flow is legible and safe by construction.

## Running it: fold, decide, append

Execution is a small loop repeated until the run finishes, halts, or fails. It's deliberately boring, and boring is the point.

<figure class="kr-fig">
<div class="kr-loop">
  <div class="kr-lstep"><span class="n">1</span><span class="t"><b>Fold</b> the current run's events into a state projection.</span></div>
  <div class="kr-lstep"><span class="n">2</span><span class="t"><b>Decide</b>: the pure kernel looks at the program and the state and returns the next action.</span></div>
  <div class="kr-lstep"><span class="n">3</span><span class="t"><b>Append</b> that transition to the ledger.</span></div>
  <div class="kr-lstep"><span class="n">4</span><span class="t"><b>Act</b> if a capability is needed: check admission, maybe pause for approval, execute under supervision, record the result.</span></div>
  <div class="kr-lstep"><span class="n">5</span><span class="t"><b>Repeat</b> from step 1 until the graph reaches an end.</span></div>
</div>
<figcaption>Fold, decide, append. Because every transition is recorded before the next one, a crash is recoverable: the run resumes from what was written, not from a guess.</figcaption>
</figure>

That last part matters more than it looks. If the ledger shows an effect was *requested* but has no recorded *result*, Krelvan calls it a crash hole and halts rather than pretending it knows whether the outside world acted. It won't cheerfully assume the email went out. That honesty about the boundary between "what we did" and "what a remote system did" is rare, and it's deliberate.

## Every action is classified, and permission is deny-by-default

An agent node can only call capabilities its signed manifest declared. And every capability is tagged with what kind of consequence it has. This is how Krelvan knows when to pause for you.

<figure class="kr-fig">
<div class="kr-tab-wrap">
<table class="kr-tab">
<thead>
<tr><th>Side-effect class</th><th>Meaning</th><th>Example</th></tr>
</thead>
<tbody>
<tr><td class="g">read</td><td>Observes without changing anything external</td><td>Search, retrieval, a GET</td></tr>
<tr><td class="w">write-reversible</td><td>Changes something you can normally undo</td><td>Store a memory, update a draft</td></tr>
<tr><td class="r">write-irreversible</td><td>Can't safely be assumed reversible</td><td>A destructive or final action</td></tr>
<tr><td class="r">spend</td><td>Moves or commits money</td><td>A payment or refund</td></tr>
<tr><td class="w">message-human</td><td>Communicates with a person</td><td>Email, Slack, Telegram</td></tr>
<tr><td class="r">identity-mutation</td><td>Changes standing identity or authority</td><td>The agent's persistent soul or credentials</td></tr>
</tbody>
</table>
</div>
<figcaption>Six consequence classes. Whether an action pauses for you depends on the class plus the node's autonomy setting: "suggest" pauses every non-read action, "act-with-veto" lets reversible writes through but stops the consequential ones, and a delegated sub-agent always forces its consequential actions through approval. Admission is deny-by-default: if it wasn't declared and allowed, it doesn't run.</figcaption>
</figure>

And when an action does run, it goes through a three-part protocol that keeps permission, intent, and observed outcome separate. The key detail: a plugin never signs its own success. A trusted supervisor observes what happened and signs the result.

<figure class="kr-fig">
<div class="kr-proto">
  <div class="kr-ppanel"><div class="k">1 · admit</div><b>Permission</b><span>Check the declaration, availability, and internal allowance. Deny by default.</span></div>
  <div class="kr-ppanel"><div class="k">2 · request</div><b>Intent</b><span>Record exactly what's about to happen, with a deterministic idempotency key.</span></div>
  <div class="kr-ppanel"><div class="k">3 · observe</div><b>Outcome</b><span>The supervisor watches the tool run and signs the result. The tool can't fake its own success.</span></div>
</div>
<figcaption>Permission, intention, and observed outcome are three separate signed facts. An untrusted connector or model can't quietly expand its own authority or claim an effect it didn't achieve.</figcaption>
</figure>

## An ecosystem, not a walled garden

The core is useful alone, but the point is that it grows. Capabilities are the unit of behavior, and there are three ways to add one: a YAML definition for a simple HTTP wrapper, a sandboxed TypeScript plugin for custom local behavior, or an MCP connection to plug in an existing tool server. MCP isn't a second runtime bolted on: discovered MCP tools become normal Krelvan capabilities, subject to the same admission, approval, and signing.

All of it is discoverable through a Git-backed registry, the same shape as a mature open software platform: browsable, forkable, installable.

<figure class="kr-fig">
<div class="kr-stats">
  <div class="kr-stat"><div class="num">27</div><div class="lb">agent templates</div></div>
  <div class="kr-stat"><div class="num">31</div><div class="lb">MCP connectors</div></div>
  <div class="kr-stat"><div class="num">10</div><div class="lb">YAML capabilities</div></div>
  <div class="kr-stat"><div class="num">4</div><div class="lb">connector packs</div></div>
</div>
<figcaption>The current registry: 72 validated entries. Templates range from a 3-node personal advisor to a 12-node support resolution agent with grounded answer tiers and fail-safe escalation. Registry entries are connection definitions, an installable connector still needs its upstream account and your trust to actually run.</figcaption>
</figure>

Connectors carry secret *references*, not secrets. Code plugins install disabled and require an explicit trust choice, then run in a locked-down subprocess with scrubbed environment and brokered network access. The ecosystem is open, but it's open with the safety model intact.

## What's under the "trust" claim, precisely

I try hard not to overclaim here, because trust language is easy to inflate. So, precisely: every event binds its type, causal parents, previous event, payload, timestamp, and author into a canonical form, content-addressed with SHA-256 and signed. Fresh installs get per-install Ed25519 keys, and a standalone, dependency-free verifier can recompute and check the whole chain offline, against a keyring you obtain independently.

What that proves: that the events form the signed chain they claim to, and, with an independently trusted key, who issued them. What it does *not* prove: that an external API told the truth, that a model's answer is factually correct, or that a compromised host observed honestly. The ledger proves what Krelvan recorded and how it linked it. Factual accuracy still needs grounding, capable models, and independent checks, exactly the [hallucination toolkit](/blog/2026-08-02-how-to-actually-prevent-ai-hallucinations/) I wrote about separately.

## Where this is headed

Krelvan today is a strong single-install product. You own the runtime, build and run agents, connect tools, gate consequential actions, and export proof. Version 0.1.2 ships as npm, a container, and a signed release artifact.

The honest edges, and the direction: it's single-install today, so multi-tenant databases, roles and SSO, and distributed workers are a direction, not a current claim. The next engineering work I care about is closing the remaining seams so the design's strongest statements become fully true: making operational memory a rebuildable projection of the ledger rather than a parallel store, giving remote connectors the one complete network policy the built-in paths already have, adding an owner-level "always gate these classes" switch, and supporting independently signed registry publishing so the ecosystem can safely accept untrusted contributors.

## The takeaway

Most agent tools are betting that the model gets good enough that you won't need to look under the hood. I'm betting the opposite: that as agents do more consequential things, the ability to see exactly what was built, control what it's allowed to do, recover honestly when it breaks, and hand someone a signed record they can verify without trusting you, becomes the whole game.

That's Krelvan's defensible claim. Not that the model is always right, nobody can promise that. But that agent creation and operation are made inspectable, bounded, and genuinely yours. You can try it at [krelvan.com](https://krelvan.com).

*If you want the concepts underneath this: [what an agent really is](/blog/2026-07-14-what-is-an-agent-really/), [designing an agent that doesn't go off the rails](/blog/2026-07-24-designing-an-agent-that-doesnt-go-off-the-rails/) for the authority and approval thinking, and [single vs multi-agent](/blog/2026-08-03-one-agent-or-many-single-vs-multi-agent/) for the delegation model Krelvan's sub-agents use.*

<script>
(function(){
  var els=document.querySelectorAll('.kr-seven,.kr-arch,.kr-frow,.kr-loop,.kr-proto,.kr-stats');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.18});
  els.forEach(function(e){io.observe(e)});
})();
</script>
