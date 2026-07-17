---
title: "LangGraph: When Your AI Needs a Flowchart That Actually Runs"
date: 2026-07-17
excerpt: "A straight-line pipeline is fine until your AI needs to loop, branch, pause for a human, or survive a crash halfway through. That's the wall LangChain's simple agent hits, and exactly where LangGraph begins. This is what LangGraph is, why it uses graphs, its pieces, real use cases, the honest trade-offs, and how it differs from LangChain."
tags: [ai, langgraph, langchain, agents, orchestration, explainer]
---

<style>
.lg-fig{margin:2.5rem 0;}
.lg-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* line vs graph */
.lg-vs{display:grid;grid-template-columns:1fr 1fr;gap:.9rem;max-width:640px;margin:0 auto;}
@media(max-width:560px){.lg-vs{grid-template-columns:1fr;}}
.lg-vs .col{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.lg-vs .col h4{margin:0 0 .6rem;font-size:.85rem;font-family:var(--font-mono);}
.lg-vs .col.line h4{color:var(--text-2);} .lg-vs .col.graph h4{color:var(--accent);}
.lg-vs .col.graph{border-color:var(--accent);}
.lg-vs .col svg{width:100%;height:120px;}
.lg-vs .n{fill:var(--surface);stroke:var(--border-2);stroke-width:1.5;} .lg-vs .n.acc{stroke:var(--accent);}
.lg-vs .e{stroke:var(--accent);stroke-width:1.5;fill:none;marker-end:url(#lgh);}
.lg-vs .t{fill:var(--text-2);font-family:var(--font-mono);font-size:9px;text-anchor:middle;}

/* five limits */
.lg-limits{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.lg-lim{display:flex;align-items:center;gap:.8rem;border:1px solid var(--border);border-radius:9px;padding:.6rem .85rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.lg-limits.go .lg-lim{opacity:1;transform:none;}
.lg-limits.go .lg-lim:nth-child(1){transition-delay:.1s} .lg-limits.go .lg-lim:nth-child(2){transition-delay:.25s}
.lg-limits.go .lg-lim:nth-child(3){transition-delay:.4s} .lg-limits.go .lg-lim:nth-child(4){transition-delay:.55s}
.lg-limits.go .lg-lim:nth-child(5){transition-delay:.7s}
.lg-lim .no{flex:none;font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);border:1px solid var(--border-2);border-radius:5px;padding:.15rem .45rem;}
.lg-lim .can{flex:none;font-family:var(--font-mono);font-size:.72rem;color:var(--accent);}
.lg-lim .t{font-size:.84rem;color:var(--text-2);} .lg-lim .t b{color:var(--text);}

/* mental model */
.lg-anal{max-width:560px;margin:0 auto;text-align:center;border:1px solid var(--accent);border-radius:12px;padding:1.1rem 1.3rem;background:var(--surface);}
.lg-anal b{color:var(--accent);} .lg-anal p{font-size:.9rem;color:var(--text-2);line-height:1.6;margin:0;}

/* the three pieces */
.lg-parts{display:grid;grid-template-columns:repeat(3,1fr);gap:.7rem;max-width:700px;margin:0 auto;}
@media(max-width:520px){.lg-parts{grid-template-columns:1fr;}}
.lg-parts .c{border:1px solid var(--border-2);border-radius:12px;padding:.9rem;background:var(--surface);opacity:0;transform:translateY(8px);transition:opacity .45s ease,transform .45s ease;}
.lg-parts.go .c{opacity:1;transform:none;}
.lg-parts.go .c:nth-child(1){transition-delay:.1s} .lg-parts.go .c:nth-child(2){transition-delay:.3s} .lg-parts.go .c:nth-child(3){transition-delay:.5s}
.lg-parts .c .ic{font-family:var(--font-mono);font-size:.68rem;color:var(--accent);border:1px solid var(--accent);border-radius:5px;padding:.1rem .4rem;display:inline-block;margin-bottom:.5rem;}
.lg-parts .c b{display:block;color:var(--text);font-size:.9rem;margin-bottom:.25rem;}
.lg-parts .c span{font-size:.8rem;color:var(--text-2);line-height:1.45;}

/* worked graph example (SVG) */
.lg-graph{max-width:520px;margin:0 auto;}
.lg-graph svg{width:100%;height:250px;overflow:visible;}
.lg-gnode{fill:var(--surface);stroke:var(--border-2);stroke-width:1.5;}
.lg-gnode.acc{stroke:var(--accent);stroke-width:2;}
.lg-gtxt{fill:var(--text);font-family:var(--font-mono);font-size:10px;text-anchor:middle;}
.lg-gedge{stroke:var(--accent);stroke-width:1.5;fill:none;marker-end:url(#lgh2);}
.lg-gedge.dash{stroke-dasharray:4 3;}
.lg-glbl{fill:var(--text-3);font-family:var(--font-mono);font-size:8px;}

/* superpowers */
.lg-super{display:grid;grid-template-columns:1fr 1fr;gap:.7rem;max-width:680px;margin:0 auto;}
@media(max-width:520px){.lg-super{grid-template-columns:1fr;}}
.lg-super .c{border:1px solid var(--border-2);border-radius:12px;padding:.85rem 1rem;background:var(--surface);}
.lg-super .c .k{font-family:var(--font-mono);font-size:.68rem;color:var(--accent);margin-bottom:.25rem;}
.lg-super .c b{display:block;color:var(--text);font-size:.85rem;margin-bottom:.2rem;}
.lg-super .c span{font-size:.8rem;color:var(--text-2);line-height:1.45;}

/* pros cons */
.lg-pc{display:grid;grid-template-columns:1fr 1fr;gap:.8rem;max-width:680px;margin:0 auto;}
@media(max-width:520px){.lg-pc{grid-template-columns:1fr;}}
.lg-pc .side{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.lg-pc .side h4{margin:0 0 .6rem;font-size:.82rem;font-family:var(--font-mono);}
.lg-pc .side.pro{border-color:var(--accent);} .lg-pc .side.pro h4{color:var(--accent);}
.lg-pc .side.con h4{color:var(--text-2);}
.lg-pc .side .item{font-size:.83rem;color:var(--text-2);margin:.45rem 0;line-height:1.45;padding-left:1.1rem;position:relative;}
.lg-pc .side .item::before{position:absolute;left:0;font-family:var(--font-mono);}
.lg-pc .side.pro .item::before{content:"+";color:var(--accent);}
.lg-pc .side.con .item::before{content:"\2212";color:var(--text-3);}

/* table */
.lg-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.lg-tab th,.lg-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.lg-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.lg-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}

/* extra reveals */
.lg-super .c{opacity:0;transform:translateY(8px);transition:opacity .45s ease,transform .45s ease;}
.lg-super.go .c{opacity:1;transform:none;}
.lg-super.go .c:nth-child(1){transition-delay:.1s} .lg-super.go .c:nth-child(2){transition-delay:.28s}
.lg-super.go .c:nth-child(3){transition-delay:.46s} .lg-super.go .c:nth-child(4){transition-delay:.64s}

.lg-pc .side{opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.lg-pc.go .side{opacity:1;transform:none;}
.lg-pc.go .side:nth-child(1){transition-delay:.1s} .lg-pc.go .side:nth-child(2){transition-delay:.35s}

@media (prefers-reduced-motion: reduce){
  .lg-limits .lg-lim,.lg-parts .c,.lg-super .c,.lg-pc .side{transition:none !important;opacity:1 !important;transform:none !important;}
}
</style>

In my last post I described LangChain as a toolkit for building AI apps, and I mentioned it has a more powerful cousin for the genuinely complicated jobs, called LangGraph. This post is about that cousin, and about the exact moment you need it.

Here's that moment. You build an AI agent with a simple framework, and it works. Then the real world shows up. The agent needs to *loop*, keep trying until it succeeds. It needs to *branch*, go one way if the answer looks good, another way if it doesn't. It needs to *pause* and wait for a human to approve something before continuing. And it needs to *survive a crash*, if the server restarts halfway through a long job, it should pick up where it left off, not start over. A simple straight-line agent can do none of those things. It runs once, top to bottom, and that's it.

**LangGraph exists for exactly those four needs.** It lets you build an AI workflow as a *graph*, a flowchart that can actually run, with loops, branches, pause points, and a memory that survives failures. Let me walk through what it is, the handful of ideas that make it tick, what people build with it, and the honest trade-offs, in the same plain-language style as the LangChain post.

## The wall: why a straight line isn't enough

Picture the two shapes. A simple agent is a *straight line*: start, do steps, finish. LangGraph is a *graph*: a web of steps where the path can curve back, split, and rejoin.

<figure class="lg-fig">
  <div class="lg-vs">
    <div class="col line">
      <h4>Simple agent: a straight line</h4>
      <svg viewBox="0 0 260 120">
        <defs><marker id="lgh" markerWidth="8" markerHeight="8" refX="5" refY="3" orient="auto"><path d="M0,0 L5,3 L0,6 Z" fill="var(--accent)"/></marker></defs>
        <circle class="n" cx="40" cy="60" r="16"/><text class="t" x="40" y="63">A</text>
        <circle class="n" cx="130" cy="60" r="16"/><text class="t" x="130" y="63">B</text>
        <circle class="n" cx="220" cy="60" r="16"/><text class="t" x="220" y="63">C</text>
        <path class="e" d="M56,60 L114,60"/><path class="e" d="M146,60 L204,60"/>
      </svg>
    </div>
    <div class="col graph">
      <h4>LangGraph: a graph that loops &amp; branches</h4>
      <svg viewBox="0 0 260 120">
        <circle class="n acc" cx="40" cy="60" r="16"/><text class="t" x="40" y="63">A</text>
        <circle class="n acc" cx="130" cy="35" r="16"/><text class="t" x="130" y="38">B</text>
        <circle class="n acc" cx="130" cy="90" r="16"/><text class="t" x="130" y="93">C</text>
        <circle class="n acc" cx="220" cy="60" r="16"/><text class="t" x="220" y="63">D</text>
        <path class="e" d="M55,54 L116,40"/><path class="e" d="M55,66 L116,84"/>
        <path class="e" d="M144,40 L206,55"/>
        <path class="e" d="M138,76 Q100,60 122,48"/>
      </svg>
    </div>
  </div>
  <figcaption>Left: a simple agent marches A to B to C and stops. Right: LangGraph can split (A goes to B or C), loop back (C returns to B to retry), and rejoin. Real agent work looks like the right side far more often than the left. That's the whole reason LangGraph exists.</figcaption>
</figure>

To be concrete, a simple straight-line agent hits five hard walls. LangGraph knocks down every one of them:

<figure class="lg-fig">
  <div class="lg-limits" id="limits">
    <div class="lg-lim"><span class="no">can't</span><span class="can">→ can</span><div class="t"><b>Loop.</b> Keep retrying or refining until the result is good enough.</div></div>
    <div class="lg-lim"><span class="no">can't</span><span class="can">→ can</span><div class="t"><b>Branch.</b> Take different paths depending on what happened.</div></div>
    <div class="lg-lim"><span class="no">can't</span><span class="can">→ can</span><div class="t"><b>Run in parallel.</b> Do several things at once, then combine.</div></div>
    <div class="lg-lim"><span class="no">can't</span><span class="can">→ can</span><div class="t"><b>Pause for a human.</b> Wait for someone to approve, then resume.</div></div>
    <div class="lg-lim"><span class="no">can't</span><span class="can">→ can</span><div class="t"><b>Survive a crash.</b> Save progress and resume from where it stopped.</div></div>
  </div>
  <figcaption>The five things a simple agent cannot do, and LangGraph can. If your app never needs any of these, you don't need LangGraph, stick with the simpler tool. But the moment you need even one, this is the framework built for it.</figcaption>
</figure>

## The mental model: a flowchart you can run

You've drawn a flowchart before: boxes for steps, arrows for "what happens next," diamonds for decisions. LangGraph is that, but *executable*.

<figure class="lg-fig">
  <div class="lg-anal">
    <p><b>LangGraph is a flowchart that actually runs.</b> You draw the boxes (steps your AI takes) and the arrows (what happens next, including "go back and retry" or "if this, then that"). Then you hit go, and it runs your flowchart, remembering everything as it moves, and picking up where it left off if something breaks.</p>
  </div>
  <figcaption>Hold that image. A normal flowchart is a drawing on paper. LangGraph turns the drawing into a living program, one where arrows can loop backward, decisions route the flow, and the whole thing keeps a memory. If you can sketch how your agent should behave, you can build it.</figcaption>
</figure>

## The three pieces that make it work

LangGraph is built from just three core ideas. Get these and you get the whole framework.

<figure class="lg-fig">
  <div class="lg-parts" id="parts">
    <div class="c"><span class="ic">the boxes</span><b>Nodes</b><span>Each node is one step of work: call the AI, use a tool, run some logic. A node reads the shared memory, does its job, and writes back what it learned.</span></div>
    <div class="c"><span class="ic">the arrows</span><b>Edges</b><span>Edges connect the nodes and decide what runs next. Some are fixed ("always go here next"). Others are conditional ("if the answer is good, finish; if not, loop back").</span></div>
    <div class="c"><span class="ic">the memory</span><b>State</b><span>One shared object that flows through every node, the agent's working memory. Each step reads it and updates it, so the whole graph stays in sync.</span></div>
  </div>
  <figcaption>Nodes are the boxes (do work), edges are the arrows (decide the path), and state is the shared notebook that every box reads and writes. That's it. A LangGraph app is: define your memory, add your boxes, wire up your arrows, and run. The conditional arrows, the ones that pick the next box based on the current situation, are where the intelligence lives.</figcaption>
</figure>

Here's a real graph, so it's not abstract. Say you're building a research agent that must keep searching until it has enough to answer:

<figure class="lg-fig">
  <div class="lg-graph">
    <svg viewBox="0 0 400 250">
      <defs><marker id="lgh2" markerWidth="9" markerHeight="9" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" fill="var(--accent)"/></marker></defs>
      <rect class="lg-gnode acc" x="150" y="10" width="100" height="40" rx="10"/><text class="lg-gtxt" x="200" y="34">search</text>
      <rect class="lg-gnode acc" x="150" y="100" width="100" height="40" rx="10"/><text class="lg-gtxt" x="200" y="124">check: enough?</text>
      <rect class="lg-gnode" x="30" y="195" width="100" height="40" rx="10"/><text class="lg-gtxt" x="80" y="219">no: search more</text>
      <rect class="lg-gnode acc" x="270" y="195" width="100" height="40" rx="10"/><text class="lg-gtxt" x="320" y="219">yes: answer</text>
      <path class="lg-gedge" d="M200,50 L200,98"/>
      <path class="lg-gedge dash" d="M150,125 L100,193"/><text class="lg-glbl" x="105" y="165">not enough</text>
      <path class="lg-gedge dash" d="M250,125 L305,193"/><text class="lg-glbl" x="270" y="165">enough</text>
      <path class="lg-gedge" d="M80,195 Q120,120 155,55"/><text class="lg-glbl" x="70" y="110">loop back</text>
    </svg>
  </div>
  <figcaption>Search, then check "do I have enough?" If not, the conditional edge loops back to search again. If yes, it routes to answer. That loop-until-satisfied behaviour is trivial in LangGraph and impossible in a straight-line agent. The shared state carries what's been found so far through every pass.</figcaption>
</figure>

## The superpowers that come baked in

Beyond loops and branches, LangGraph includes two things that are genuinely hard to build yourself, and this is where it really earns its keep for serious apps.

<figure class="lg-fig">
  <div class="lg-super" id="super">
    <div class="c"><div class="k">save points</div><b>Checkpointing (durability)</b><span>It automatically saves the agent's state as it runs. If the server crashes or restarts, the agent resumes from the last save instead of starting over. You can even "rewind" to debug what happened, like a video game's save file.</span></div>
    <div class="c"><div class="k">the pause button</div><b>Human-in-the-loop</b><span>Built-in support to pause the agent, let a human review or approve or edit, then resume from exactly that point. Essential for anything high-stakes: spending money, sending emails, making changes.</span></div>
  </div>
  <figcaption>These two are the reason companies like Klarna, Uber, and J.P. Morgan build production agents on it. A demo doesn't need to survive crashes or wait for human approval; a real system running real tasks absolutely does. LangGraph makes both routine instead of a custom engineering project.</figcaption>
</figure>

## What people build with it

The pattern is always the same: a task that's too twisty for a straight line.

<figure class="lg-fig">
  <div class="lg-super" id="use">
    <div class="c"><div class="k">research</div><b>Autonomous research agents</b><span>Search, evaluate what came back, decide if it's enough, search again if not, loop until satisfied, then write it up. The loop is the whole point.</span></div>
    <div class="c"><div class="k">approval flows</div><b>Human-approval workflows</b><span>An agent drafts something risky (a refund, a contract change), pauses, a human approves, and it continues. The pause is first-class.</span></div>
    <div class="c"><div class="k">teamwork</div><b>Multi-agent systems</b><span>Several specialized agents coordinated as nodes in one graph, a manager routing work to workers and combining results.</span></div>
    <div class="c"><div class="k">long jobs</div><b>Long-running, resumable tasks</b><span>Jobs that take minutes or hours and must survive interruptions, picking up from the last checkpoint rather than restarting.</span></div>
  </div>
  <figcaption>Notice none of these is a simple "ask, answer, done." They all loop, branch, pause, or need to survive time. That's the signature of a LangGraph job. If your task is a simple pipeline, you're in LangChain (or plain API) territory instead.</figcaption>
</figure>

## The honest pros and cons

Same balanced treatment as always, because LangGraph's power comes with a real cost.

<figure class="lg-fig">
  <div class="lg-pc" id="pc">
    <div class="side pro">
      <h4>Why people reach for it</h4>
      <div class="item">Loops, branches, and parallel steps are natural, not hacks</div>
      <div class="item">Crash recovery and state-saving built in, huge for production</div>
      <div class="item">First-class human-in-the-loop pause and resume</div>
      <div class="item">Great for multi-agent coordination and long-running jobs</div>
      <div class="item">Trusted in production by serious companies</div>
    </div>
    <div class="side con">
      <h4>The honest downsides</h4>
      <div class="item">More concepts to learn: nodes, edges, state, checkpointers</div>
      <div class="item">Overkill for simple, linear tasks, more setup than you need</div>
      <div class="item">You think in graphs, which takes a mental shift at first</div>
      <div class="item">Lower-level: more control, but also more to wire up yourself</div>
    </div>
  </div>
  <figcaption>The trade in a nutshell: LangGraph asks you to learn a graph way of thinking and write more structure, and in return gives you power a straight-line tool simply can't. Worth it for complex, stateful, production agents. Not worth it for a simple chatbot. Match the tool to the job.</figcaption>
</figure>

## LangChain vs LangGraph: which, when?

This is the question everyone asks, so here's the clean answer, including a detail that surprises people.

<figure class="lg-fig">
<table class="lg-tab">
<thead><tr><th></th><th>LangChain</th><th>LangGraph</th></tr></thead>
<tbody>
<tr><td>Shape</td><td>Straight-line pipelines</td><td>Graphs: loops, branches</td></tr>
<tr><td>Level</td><td>Higher-level, quicker to start</td><td>Lower-level, more control</td></tr>
<tr><td>Pause for a human</td><td>Not really</td><td>Built in</td></tr>
<tr><td>Survive a crash</td><td>No</td><td>Yes, checkpointing</td></tr>
<tr><td>Best for</td><td>Chatbots, RAG, simple agents</td><td>Complex, stateful, multi-step agents</td></tr>
<tr><td>Start here if</td><td>You're prototyping or it's simple</td><td>You need loops, approval, or durability</td></tr>
</tbody>
</table>
  <figcaption>The surprising detail: they're not really rivals. As of recent versions, LangChain's agents are actually <em>built on top of</em> LangGraph underneath. So they're layers of one stack, not competitors. The recommended path: prototype fast in LangChain, and "drop down" to LangGraph when you hit the wall of needing loops, branching, human approval, or crash recovery.</figcaption>
</figure>

## The takeaway

A straight line is the right shape for a simple job: ask, answer, done. But real agent work loops, branches, waits for people, and has to survive the messiness of the real world, crashes, restarts, long waits. That's a graph, not a line, and LangGraph is the framework that lets you build that graph as a flowchart that actually runs, with memory that persists and save points that let it recover.

The mental model to keep: **nodes are the boxes, edges are the arrows, state is the shared notebook, and the whole thing is a flowchart you can execute.** Reach for it when your agent needs to loop, branch, pause for a human, or pick up after a crash. Skip it, and stay with something simpler, when your task is a straight line. As always, the skill isn't using the most powerful framework. It's recognizing the shape of your problem, and LangGraph is what you want the moment that shape stops being a line and starts being a graph.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['limits','parts','super','use','pc'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
