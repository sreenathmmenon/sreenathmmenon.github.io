---
title: "Streaming Live Context Into LLMs and AI Agents"
date: 2026-07-31
excerpt: "Most agents wait to be asked. A human types a question, the agent answers, and nothing happens until the next question. The more useful pattern is the opposite: the agent is asleep until something in the world happens, and the event itself wakes it up. This post is about event-driven agents, where the event is the perception, and how you keep the context window fed with what just happened without letting it overflow."
tags: [ai, agents, streaming, event-driven, generative-ai, architecture]
---

<style>
.ev-fig{margin:2.5rem 0;}
.ev-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* poll vs event */
.ev-vs{display:grid;grid-template-columns:1fr 1fr;gap:.9rem;max-width:680px;margin:0 auto;}
@media(max-width:560px){.ev-vs{grid-template-columns:1fr;}}
.ev-vs .col{border:1px solid var(--border);border-radius:12px;padding:1.1rem;background:var(--surface);opacity:0;transform:translateY(10px);transition:opacity .5s ease,transform .5s ease;}
.ev-vs.go .col{opacity:1;transform:none;} .ev-vs.go .col:nth-child(2){transition-delay:.18s;}
.ev-vs .col h4{margin:0 0 .6rem;font-size:.85rem;font-family:var(--font-mono);}
.ev-vs .col.poll{border-color:var(--border-2);} .ev-vs .col.poll h4{color:var(--text-2);}
.ev-vs .col.push{border-color:var(--accent);} .ev-vs .col.push h4{color:var(--accent);}
.ev-vs .col p{font-size:.85rem;color:var(--text-2);line-height:1.55;margin:0;}
.ev-vs .col .tick{font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);margin-top:.7rem;display:flex;gap:.35rem;flex-wrap:wrap;}
.ev-vs .col .tick span{border:1px solid var(--border-2);border-radius:5px;padding:.1rem .4rem;}
.ev-vs .col.poll .tick span{opacity:.5;}
.ev-vs .col.push .tick span{color:var(--accent);border-color:var(--accent);}
.ev-vs .col.push .tick span.wake{background:var(--grad);color:var(--accent-ink);border-color:transparent;font-weight:600;}

/* architecture flow */
.ev-pipe{max-width:840px;margin:0 auto;overflow-x:auto;}
.ev-flow{display:flex;flex-wrap:nowrap;gap:.4rem;align-items:stretch;min-width:680px;}
.ev-node{flex:1 1 0;min-width:118px;border:1px solid var(--border-2);border-radius:11px;background:var(--surface);padding:.75rem .7rem;opacity:0;transform:translateY(10px);transition:opacity .45s ease,transform .45s ease;}
.ev-flow.go .ev-node{opacity:1;transform:none;}
.ev-flow.go .ev-node:nth-child(1){transition-delay:.05s} .ev-flow.go .ev-node:nth-child(3){transition-delay:.2s}
.ev-flow.go .ev-node:nth-child(5){transition-delay:.35s} .ev-flow.go .ev-node:nth-child(7){transition-delay:.5s}
.ev-flow.go .ev-node:nth-child(9){transition-delay:.65s}
.ev-node .svc{font-family:var(--font-mono);font-size:.64rem;color:var(--accent);}
.ev-node b{display:block;color:var(--text);font-size:.82rem;margin:.2rem 0 .12rem;line-height:1.25;}
.ev-node span.d{font-size:.72rem;color:var(--text-3);line-height:1.4;display:block;}
.ev-node .loop{font-family:var(--font-mono);font-size:.6rem;color:var(--accent-2);margin-top:.4rem;text-transform:uppercase;letter-spacing:.05em;}
.ev-arrow{align-self:center;color:var(--text-3);font-family:var(--font-mono);flex:none;font-size:.9rem;}

/* worked sequence */
.ev-seq{max-width:640px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;position:relative;}
.ev-step{display:flex;gap:.85rem;border:1px solid var(--border);border-radius:11px;background:var(--surface);padding:.75rem .95rem;align-items:flex-start;opacity:0;transform:translateX(-10px);transition:opacity .5s ease,transform .5s ease;}
.ev-seq.go .ev-step{opacity:1;transform:none;}
.ev-seq.go .ev-step:nth-child(1){transition-delay:.05s} .ev-seq.go .ev-step:nth-child(2){transition-delay:.2s}
.ev-seq.go .ev-step:nth-child(3){transition-delay:.35s} .ev-seq.go .ev-step:nth-child(4){transition-delay:.5s}
.ev-seq.go .ev-step:nth-child(5){transition-delay:.65s} .ev-seq.go .ev-step:nth-child(6){transition-delay:.8s}
.ev-step .t{flex:none;font-family:var(--font-mono);font-size:.68rem;color:var(--accent);border:1px solid var(--accent);border-radius:6px;padding:.15rem .45rem;margin-top:.05rem;min-width:52px;text-align:center;}
.ev-step .body b{color:var(--text);font-size:.86rem;}
.ev-step .body p{font-size:.82rem;color:var(--text-2);line-height:1.5;margin:.15rem 0 0;}
.ev-step.fire .t{background:var(--grad);color:var(--accent-ink);border-color:transparent;font-weight:600;}

/* context window feeding */
.ev-win{max-width:660px;margin:0 auto;}
.ev-window{border:2px solid var(--accent);border-radius:12px;background:var(--surface);padding:1rem;position:relative;}
.ev-window .cap{font-family:var(--font-mono);font-size:.68rem;color:var(--accent);position:absolute;top:-.65rem;left:1rem;background:var(--bg);padding:0 .4rem;}
.ev-slots{display:flex;flex-direction:column;gap:.4rem;}
.ev-slot{display:flex;align-items:center;gap:.6rem;border:1px solid var(--border-2);border-radius:8px;background:var(--surface-2);padding:.45rem .7rem;font-size:.8rem;color:var(--text-2);opacity:0;transform:translateX(12px);transition:opacity .5s ease,transform .5s ease,background .5s ease,border-color .5s ease;}
.ev-win.go .ev-slot{opacity:1;transform:none;}
.ev-win.go .ev-slot:nth-child(1){transition-delay:.1s} .ev-win.go .ev-slot:nth-child(2){transition-delay:.25s}
.ev-win.go .ev-slot:nth-child(3){transition-delay:.4s} .ev-win.go .ev-slot:nth-child(4){transition-delay:.55s}
.ev-slot .tag{font-family:var(--font-mono);font-size:.64rem;padding:.1rem .4rem;border-radius:5px;flex:none;}
.ev-slot.fresh .tag{background:var(--grad);color:var(--accent-ink);}
.ev-slot.stale{opacity:.4;text-decoration:line-through;border-style:dashed;}
.ev-slot.stale .tag{border:1px solid var(--text-3);color:var(--text-3);}
.ev-win.go .ev-slot.stale{opacity:.4;}
.ev-promote{font-family:var(--font-mono);font-size:.72rem;color:var(--accent-2);text-align:center;margin:.8rem 0 0;}
.ev-mem{border:1px dashed var(--accent-2);border-radius:10px;background:var(--surface-2);padding:.7rem .9rem;margin-top:.5rem;font-size:.8rem;color:var(--text-2);}
.ev-mem b{color:var(--accent-2);font-family:var(--font-mono);font-size:.7rem;}

/* cloud table */
.ev-tab-wrap{max-width:780px;margin:0 auto;overflow-x:auto;border:1px solid var(--border);border-radius:12px;background:var(--surface);}
.ev-tab{width:100%;border-collapse:collapse;font-size:.86rem;min-width:580px;}
.ev-tab th,.ev-tab td{text-align:left;padding:.65rem .8rem;border-bottom:1px solid var(--border);vertical-align:top;}
.ev-tab th{font-family:var(--font-mono);font-size:.7rem;text-transform:uppercase;letter-spacing:.05em;color:var(--text-3);font-weight:500;}
.ev-tab td:first-child,.ev-tab th:first-child{color:var(--text);font-weight:500;font-family:var(--font-mono);font-size:.76rem;}
.ev-tab tr:last-child td{border-bottom:none;}

/* gotchas */
.ev-got{max-width:660px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.ev-g{display:flex;gap:.85rem;border:1px solid var(--border);border-radius:10px;background:var(--surface-2);padding:.75rem .95rem;align-items:flex-start;opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.ev-got.go .ev-g{opacity:1;transform:none;}
.ev-got.go .ev-g:nth-child(1){transition-delay:.1s} .ev-got.go .ev-g:nth-child(2){transition-delay:.25s}
.ev-got.go .ev-g:nth-child(3){transition-delay:.4s} .ev-got.go .ev-g:nth-child(4){transition-delay:.55s}
.ev-g .no{flex:none;font-family:var(--font-mono);font-size:.72rem;color:var(--accent);border:1px solid var(--accent);border-radius:6px;padding:.12rem .45rem;margin-top:.05rem;}
.ev-g p{font-size:.85rem;color:var(--text-2);line-height:1.5;margin:0;}
.ev-g p b{color:var(--text);}

/* references */
.ev-refs{max-width:680px;margin:0 auto;border:1px solid var(--border);border-radius:12px;background:var(--surface-2);padding:.4rem 1.1rem;}
.ev-refs a{display:block;padding:.6rem 0;border-top:1px solid var(--border);font-size:.85rem;color:var(--text-2);line-height:1.4;}
.ev-refs a:first-child{border-top:none;}
.ev-refs a:hover{color:var(--accent);}
.ev-refs a .src{font-family:var(--font-mono);font-size:.68rem;color:var(--text-3);display:block;margin-bottom:.15rem;letter-spacing:.03em;}

@media (prefers-reduced-motion: reduce){
  .ev-vs .col,.ev-node,.ev-step,.ev-slot,.ev-g{opacity:1!important;transform:none!important;}
}
</style>

It's 3am. A latency alarm on the checkout service crosses its threshold and fires. Nobody's awake. By the time the on-call human blinks at their phone, there's already a thread waiting for them: the error rate over the last twenty minutes, the deploy that went out at 2:47, the three services downstream that started timing out right after, and a one-line hypothesis about which change is the likely culprit. A human didn't assemble that. An agent did, and it started the moment the alarm fired, because the alarm *was* the thing that woke it up.

That's the pattern this post is about. Most agents you've seen are built the other way round: a person asks, the agent answers, and between questions the agent doesn't exist. That's fine for a chatbot. It's the wrong shape for anything that's supposed to *notice* things.

## Waiting to be asked vs waking up

Here's the split, and it's more fundamental than it looks.

<figure class="ev-fig ev-anim">
<div class="ev-vs">
  <div class="col poll">
    <h4>Request-response (the default)</h4>
    <p>The agent sits idle until a human types a question. Or it polls: it wakes on a timer, checks "anything new?", finds nothing, sleeps, checks again. Most of its life is spent asking a question nobody answered yet.</p>
    <div class="tick"><span>tick</span><span>tick</span><span>tick</span><span>tick</span></div>
  </div>
  <div class="col push">
    <h4>Event-driven (the useful one)</h4>
    <p>The agent is asleep. Something happens in the world, a ticket, an alert, a webhook, a new file, and that event is delivered to the agent, which wakes, reacts, and sleeps again. No polling. The event is the trigger.</p>
    <div class="tick"><span>...</span><span class="wake">EVENT</span><span>wake</span></div>
  </div>
</div>
<figcaption>Polling asks "anything new?" forever. Event-driven gets told. The event does the waking.</figcaption>
</figure>

Polling is what you write when you don't have events. You loop, you check, you sleep, you check again, and you pay for every check that found nothing. Webhooks were the first fix (let the source call you), but a raw webhook still lands in your app and forces you to route it. The clean version is an **event backbone**: sources publish events, a bus fans them out, and an agent is one of the subscribers that gets woken. AWS's own guidance for serverless agents puts this bluntly, event-driven architecture is the backbone, and the events are the agent's *observations*.

That word, observations, is the whole idea. If you've read [what an agent really is](/blog/2026-07-14-what-is-an-agent-really/), you know the loop: **perceive, reason, act.** Textbook diagrams draw "perceive" as the agent going out and looking at the world. Event-driven flips it. The world comes to the agent. The event is the perception layer.

## The architecture, mapped to the loop

Draw it out and the shape is simple: a source emits an event, a bus carries it, and that delivery is the agent's *perceive* step. Everything after is the same reason-and-act loop you'd build for a chatbot.

<figure class="ev-fig ev-anim">
<div class="ev-pipe">
<div class="ev-flow">
  <div class="ev-node"><span class="svc">webhook / alert / ticket / log</span><b>Event source</b><span class="d">Something happened worth reacting to</span></div>
  <div class="ev-arrow">&rarr;</div>
  <div class="ev-node"><span class="svc">EventBridge / Pub/Sub / Event Grid</span><b>Event bus</b><span class="d">Durable, fans out to subscribers, routes by rule</span></div>
  <div class="ev-arrow">&rarr;</div>
  <div class="ev-node"><span class="svc">agent invocation</span><b>Agent wakes</b><span class="d">The event is delivered as the input</span><span class="loop">perceive</span></div>
  <div class="ev-arrow">&rarr;</div>
  <div class="ev-node"><span class="svc">LLM</span><b>Reason</b><span class="d">Read the event, plan what to check and do</span><span class="loop">reason</span></div>
  <div class="ev-arrow">&rarr;</div>
  <div class="ev-node"><span class="svc">tools / APIs</span><b>Act</b><span class="d">Pull data, call tools, post the result</span><span class="loop">act</span></div>
</div>
</div>
<figcaption>The event bus delivers the observation. From there it's the ordinary perceive to reason to act loop, just started by an event instead of a human.</figcaption>
</figure>

The sources are whatever your world produces: a webhook from GitHub or Stripe, a message on a queue, an object landing in S3, a row changing in a database, an alert from your monitoring stack. The **bus** is the load-bearing middle: EventBridge on AWS, Pub/Sub on Google Cloud, Event Grid on Azure. It's durable (the event survives if the agent is down), it fans out (multiple agents can subscribe to the same event), and it routes (rules decide which events reach which agent). AWS Lambda's docs split event sources into **push** (the source invokes you, like EventBridge or SNS) and **pull** (the runtime polls a queue like SQS or a Kinesis stream on your behalf), and either way the point is the same: you stop writing the polling loop yourself.

Then the agent wakes. On the clouds this is a serverless runtime: an [Azure Functions trigger](https://learn.microsoft.com/en-us/azure/azure-functions/functions-serverless-agents-runtime) that starts an agent on *any* event, a [Cloud Run service fired per Pub/Sub message](https://docs.cloud.google.com/run/docs/triggering/pubsub-triggers) through Eventarc, a Lambda invoked by EventBridge. Scale to zero when nothing's happening, spin up when an event lands. That economic shape (pay only when the world does something) is exactly why serverless and event-driven agents fit together so well.

## Deterministic or LLM-native? Pick per step

Once the agent is awake, "reason and act" isn't one choice. There are two orchestration styles, and the mistake is treating it as either-or.

A **deterministic** orchestrator (Step Functions on AWS, Workflows on Google Cloud, Logic Apps on Azure) runs a fixed graph: do this, then this, branch here. You know the steps in advance, so you hard-code them. An **LLM-native agent** (Bedrock AgentCore, Vertex agents, Azure AI Foundry agents) hands the model a set of tools and lets *it* decide the order at runtime. AWS's serverless guidance describes both, and both get triggered the same way, by an event off the bus.

The useful framing is per step, not per system. Deterministic where the path is known and you want it auditable and cheap. LLM-native where the input is messy and the right move genuinely depends on what the event says. A triage agent might use a deterministic wrapper to fetch a fixed set of signals, then hand those to an LLM step to actually reason about them. Same idea as picking the right tool inside [an agent that doesn't go off the rails](/blog/2026-07-24-designing-an-agent-that-doesnt-go-off-the-rails/): give the model latitude where judgment is needed, and none where it isn't.

## A worked example: the alert that triages itself

Back to 3am. Here's the sequence, which is roughly the shape of PagerDuty's SRE agent, an alert event triggers an agent that gathers context and posts a timeline, described generically.

<figure class="ev-fig ev-anim">
<div class="ev-seq">
  <div class="ev-step fire"><span class="t">03:00</span><div class="body"><b>Alert fires</b><p>Latency on checkout crosses threshold. The monitoring system publishes an incident event to the bus. No human involved yet.</p></div></div>
  <div class="ev-step"><span class="t">+0s</span><div class="body"><b>Agent wakes</b><p>A routing rule matches the event and invokes the triage agent, handing it the alert payload as its perception.</p></div></div>
  <div class="ev-step"><span class="t">+2s</span><div class="body"><b>Pulls context</b><p>It calls tools: recent logs, the error-rate metric, deploy history, related past incidents. This is the act phase feeding the reason phase.</p></div></div>
  <div class="ev-step"><span class="t">+8s</span><div class="body"><b>Reasons</b><p>The LLM correlates: error rate jumped right after the 2:47 deploy, three downstream services timed out in sequence. It forms a hypothesis.</p></div></div>
  <div class="ev-step"><span class="t">+12s</span><div class="body"><b>Posts a triage timeline</b><p>Into the incident channel: what changed, when, the likely cause, and what it already checked. A starting point, not a verdict.</p></div></div>
  <div class="ev-step"><span class="t">03:04</span><div class="body"><b>Human arrives to a head start</b><p>Instead of a blank alert, the responder opens a thread that's already done the boring first ten minutes of investigation.</p></div></div>
</div>
<figcaption>Nobody pasted the alert into a chat box. The alert triggered the whole run. That's the difference the event makes.</figcaption>
</figure>

The thing to notice: "investigate this alert" never waited for a human to *ask*. In a request-response world, someone gets paged, reads the alert, opens a chat window, pastes it in, and only then does the agent start. Event-driven collapses all of that. The alert is the request. It's the same reason GitHub's [Copilot coding agent](https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent) can be kicked off by assigning it an issue or applying a label, the webhook for that label *is* the "go do this" signal. No human types "please start."

## Keeping the window fed with what just happened

Here's where event-driven agents get genuinely hard, and it's a generative-AI problem, not a plumbing one.

The agent reasons over its **context window**, and the window is bounded. As events stream in (more alerts, more log lines, more tool results from the last three checks), they pile into the context. Left alone, that context overflows, and a full window is a slow, expensive, confused window. So you can't just keep appending "what just happened" forever. You have to decide, continuously, what stays in the window and what gets thrown out.

<figure class="ev-fig ev-anim">
<div class="ev-win">
<div class="ev-window">
  <div class="cap">context window (bounded)</div>
  <div class="ev-slots">
    <div class="ev-slot fresh"><span class="tag">now</span>latest alert + the metric it fired on</div>
    <div class="ev-slot fresh"><span class="tag">recent</span>tool results from the last two checks</div>
    <div class="ev-slot stale"><span class="tag">old</span>a resolved alert from 40 minutes ago</div>
    <div class="ev-slot stale"><span class="tag">old</span>verbose log dump the model already summarized</div>
  </div>
</div>
<div class="ev-promote">&darr; evict the stale, but don't lose the durable facts &darr;</div>
<div class="ev-mem"><b>memory store (outside the window)</b> &nbsp; "deploy 2:47 correlated with checkout latency" gets promoted here as a durable fact, retrievable later without eating window space.</div>
</div>
<figcaption>Fresh events stay in-window. Stale ones get evicted or summarized. The facts worth keeping get promoted to memory outside the window.</figcaption>
</figure>

Anthropic's [context management](https://claude.com/blog/context-management) work names both halves of this. **Context editing** automatically clears stale tool results as you approach the limit, so old, already-digested observations stop taking up room. The **memory tool** lets the agent write durable facts to a store *outside* the window and read them back on demand. Put together: the window holds what's happening *now*, and the things worth remembering get pushed to memory so they survive the eviction.

The real design question isn't "how do I stream events in." It's **what belongs in-window versus what gets promoted to memory.** Keep too much in the window and you overflow and pay for it. Promote too aggressively and the model loses the thread of what's happening right now. There's no universal answer, it's the same context-engineering judgment call from [why AI forgets](/blog/2026-07-04-context-engineering-why-ai-forgets/), just under a live stream instead of a static prompt.

And the payoff for getting it right is **real-time grounding**: the agent answers from the live state of the world, not a snapshot baked in at startup. If you've read the [companion post on streaming into RAG](/blog/2026-07-31-streaming-data-into-rag-keeping-the-index-live/), it's the same instinct from the retrieval side, don't let the model reason over a photograph of a world that's already moved on.

## Agents that talk through events, not HTTP

One more shift worth flagging. Once you've got an event backbone, multi-agent systems change shape. Instead of agent A calling agent B over HTTP and waiting, agent A *publishes* an event ("triage complete, cause identified") and whichever agents care about that (the remediation agent, the incident-comms agent) are subscribed and wake up on it. Publish and subscribe instead of point-to-point calls. It's looser, it's more durable (the event survives if a consumer is briefly down), and it's how streaming-agent platforms like [Confluent's](https://www.confluent.io/blog/introducing-streaming-agents/) frame agents that run directly on live event streams. Worth noting the vendor tilt there: Confluent sells Kafka, so of course the answer is a stream. The underlying idea holds regardless.

## The same shape on the three clouds

Every cloud has a full stack for this. Different names, identical roles.

<figure class="ev-fig">
<div class="ev-tab-wrap">
<table class="ev-tab">
<thead>
<tr><th>Role</th><th>AWS</th><th>GCP</th><th>Azure</th></tr>
</thead>
<tbody>
<tr><td>Event source</td><td>S3 / webhooks / CloudWatch alarms</td><td>Cloud Storage / webhooks / Cloud Monitoring</td><td>Blob / webhooks / Monitor alerts</td></tr>
<tr><td>Bus / trigger</td><td>EventBridge / SNS / SQS</td><td>Pub/Sub / Eventarc</td><td>Event Grid / Service Bus</td></tr>
<tr><td>Agent runtime</td><td>Lambda / Bedrock AgentCore</td><td>Cloud Run / Vertex agents</td><td>Azure Functions / Foundry agents</td></tr>
<tr><td>Short-term memory</td><td>In-window + DynamoDB / ElastiCache</td><td>In-window + Firestore / Memorystore</td><td>In-window + Cosmos DB / Redis</td></tr>
<tr><td>Long-term memory</td><td>AgentCore Memory / vector store</td><td>Vertex Memory Bank / vector store</td><td>Foundry memory / AI Search</td></tr>
</tbody>
</table>
</div>
<figcaption>Pick a row of names per cloud. The architecture underneath is the same one.</figcaption>
</figure>

## The gotchas nobody warns you about

Event-driven agents fail in ways request-response agents never do, because now the input arrives on its own schedule, possibly twice, possibly in the wrong order, possibly a thousand at once.

<figure class="ev-fig ev-anim">
<div class="ev-got">
  <div class="ev-g"><span class="no">1</span><p><b>Out-of-order events.</b> The event bus doesn't promise the order things happened is the order you receive them. An "incident resolved" can land before the "incident opened" it resolves. Carry a timestamp or sequence number on every event and let the agent reason about ordering itself, rather than trusting arrival order.</p></div>
  <div class="ev-g"><span class="no">2</span><p><b>Duplicate events.</b> Most buses deliver at-least-once, which is a polite way of saying "sometimes twice." If the same alert fires your agent twice, you get two triage threads, or worse, two remediation actions. Make the agent idempotent: dedupe on an event ID so a repeat delivery is a no-op, not a second run.</p></div>
  <div class="ev-g"><span class="no">3</span><p><b>Context overflow.</b> A long-running incident streams in more events than the window can hold. If you never evict, you overflow and the run degrades. Lean on context editing to clear stale tool results and a memory store to hold the durable facts, so the window stays about *now*.</p></div>
  <div class="ev-g"><span class="no">4</span><p><b>Event storms.</b> One outage can fire a thousand alerts in a minute, and a naive setup wakes a thousand agent runs, each burning tokens on the same incident. Debounce and rate-limit at the bus: collapse related events, batch them into one invocation, and cap how often the agent can be woken for the same source.</p></div>
</div>
</figure>

That last one bites hardest. The whole appeal of event-driven is that events trigger the agent automatically, which is exactly why an event storm is dangerous: automatic triggering with no throttle is a token bonfire waiting for a bad night. Debounce before you deploy, not after the bill.

## The takeaway

The default agent waits to be asked. It's a very good chatbot and a very bad colleague, because a colleague notices things without being prompted. Event-driven flips the polarity: the agent sleeps until the world does something, and the event itself is the perception that starts the perceive-reason-act loop. "Investigate this alert" stops being a sentence a tired human types at 3am and becomes the alert firing, directly.

The plumbing is the easy half, sources, a bus, a serverless runtime, and every cloud sells the whole stack. The hard half is generative-AI-shaped: keeping the context window fed with what just happened without drowning it, deciding what stays in-window and what gets promoted to memory, and surviving duplicate, out-of-order, storm-shaped events without doing the wrong thing twice. Get those right and you've got an agent that's already working by the time you wake up.

## References

I built the argument and the diagrams myself. The patterns aren't mine to claim, and the docs below are the real sources if you're implementing this. Nothing here is copied from them.

<div class="ev-refs">
<a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-serverless/event-driven-architecture.html"><span class="src">AWS Prescriptive Guidance</span>Event-driven architecture as the backbone of serverless agentic AI, with events framed as the agent's observations in a perceive-reason-act loop</a>
<a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-serverless/orchestration-models.html"><span class="src">AWS Prescriptive Guidance</span>Orchestration models, deterministic Step Functions vs LLM-native Bedrock agents, both triggered off the event bus</a>
<a href="https://docs.aws.amazon.com/lambda/latest/dg/concepts-event-driven-architectures.html"><span class="src">AWS Lambda docs</span>Event-driven concepts, push vs pull event sources, and replacing polling and webhooks with events</a>
<a href="https://learn.microsoft.com/en-us/azure/azure-functions/functions-serverless-agents-runtime"><span class="src">Microsoft Learn</span>Azure Functions serverless agents runtime, where any trigger starts an agent</a>
<a href="https://docs.cloud.google.com/run/docs/triggering/pubsub-triggers"><span class="src">Google Cloud docs</span>Cloud Run Pub/Sub triggers, an agent fired per message through Eventarc</a>
<a href="https://claude.com/blog/context-management"><span class="src">Anthropic</span>Context management: context editing auto-clears stale tool results near the limit, and the memory tool holds state outside the window</a>
<a href="https://www.pagerduty.com/blog/ai/meet-your-virtual-responder-pagerdutys-sre-agent-for-ai-driven-reliability/"><span class="src">PagerDuty</span>The SRE agent: an alert event triggers an agent that pulls logs, metrics, and history and posts a triage timeline</a>
<a href="https://www.pagerduty.com/eng/pagerduty-for-ai-how-the-sre-agent-triages-ai-incidents/"><span class="src">PagerDuty Engineering</span>How the SRE agent triages incidents, the engineering detail behind the triage flow</a>
<a href="https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent"><span class="src">GitHub docs</span>Copilot coding agent, where assigning an issue or a labeled-issue webhook triggers an autonomous agent</a>
<a href="https://www.confluent.io/blog/introducing-streaming-agents/"><span class="src">Confluent</span>Streaming agents that run on live event streams (note the vendor tilt: Confluent sells the stream)</a>
</div>

*This is one of a set of streaming posts. See the [streaming backbone under generative AI](/blog/2026-07-31-the-streaming-backbone-under-generative-ai/), [streaming LLM output tokens and tool calls](/blog/2026-07-31-streaming-llm-output-tokens-and-tool-calls/), and the companion [streaming data into RAG](/blog/2026-07-31-streaming-data-into-rag-keeping-the-index-live/). For the loop mechanics under all of it, [LangGraph](/blog/2026-07-17-langgraph-when-your-ai-needs-a-flowchart-that-runs/), and for the wider design series, [three AI systems, same design](/blog/2026-07-24-three-ai-systems-same-design/).*

<script>
(function(){
  var els=document.querySelectorAll('.ev-anim');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.2});
  els.forEach(function(e){io.observe(e)});
})();
</script>
