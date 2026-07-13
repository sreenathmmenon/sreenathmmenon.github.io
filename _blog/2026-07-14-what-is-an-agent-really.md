---
title: "What Is an Agent, Really?"
date: 2026-07-14
excerpt: "Everyone throws the word around now. Agents, subagents, multi-agent systems, spawning agents. But strip away the buzz and ask the simple question: what actually makes something an agent, and not just a chatbot with extra steps? Here is the clear answer, the one idea that separates an agent from everything else, and the vocabulary people use loosely, explained properly."
tags: [ai, agents, concepts, autonomy, llm, explainer]
---

<style>
.wa-fig{margin:2.5rem 0;}
.wa-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* the spectrum: chatbot -> workflow -> agent */
.wa-spec{max-width:660px;margin:0 auto;display:flex;gap:.5rem;align-items:stretch;}
@media(max-width:560px){.wa-spec{flex-direction:column;}}
.wa-spec .stage{flex:1;border:1px solid var(--border-2);border-radius:12px;padding:.9rem;background:var(--surface);text-align:center;opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.wa-spec.go .stage{opacity:1;transform:none;}
.wa-spec.go .stage:nth-child(1){transition-delay:.1s} .wa-spec.go .stage:nth-child(2){transition-delay:.35s} .wa-spec.go .stage:nth-child(3){transition-delay:.6s}
.wa-spec .stage.agent{border-color:var(--accent);}
.wa-spec .stage b{display:block;color:var(--text);font-size:.9rem;margin-bottom:.2rem;}
.wa-spec .stage.agent b{color:var(--accent);}
.wa-spec .stage .who{font-family:var(--font-mono);font-size:.68rem;color:var(--accent);margin-bottom:.4rem;}
.wa-spec .stage span{font-size:.79rem;color:var(--text-2);line-height:1.45;}

/* the one question that decides */
.wa-decide{max-width:560px;margin:0 auto;text-align:center;}
.wa-decide .q{font-family:var(--font-mono);font-size:.9rem;color:var(--accent);border:1px solid var(--accent);border-radius:12px;padding:.9rem;margin-bottom:.9rem;}
.wa-decide .split{display:grid;grid-template-columns:1fr 1fr;gap:.7rem;}
@media(max-width:480px){.wa-decide .split{grid-template-columns:1fr;}}
.wa-decide .opt{border:1px solid var(--border-2);border-radius:10px;padding:.8rem;background:var(--surface);}
.wa-decide .opt .lab{font-family:var(--font-mono);font-size:.74rem;margin-bottom:.3rem;}
.wa-decide .opt.no .lab{color:var(--text-3);} .wa-decide .opt.yes .lab{color:var(--accent);}
.wa-decide .opt span{font-size:.8rem;color:var(--text-2);line-height:1.4;}
.wa-decide .opt.yes{border-color:var(--accent);}

/* who decides the path - fixed vs dynamic */
.wa-path{max-width:620px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:.8rem;}
@media(max-width:560px){.wa-path{grid-template-columns:1fr;}}
.wa-path .col{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.wa-path .col h4{margin:0 0 .5rem;font-size:.85rem;font-family:var(--font-mono);}
.wa-path .col.fixed h4{color:var(--text-2);} .wa-path .col.dyn h4{color:var(--accent);}
.wa-path .col.dyn{border-color:var(--accent);}
.wa-path .col .rail{font-family:var(--font-mono);font-size:.76rem;color:var(--text-2);padding:.25rem 0;}
.wa-path .col .rail .a{color:var(--accent);}

/* vocabulary tree */
.wa-vocab{max-width:640px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.wa-vrow{display:flex;align-items:flex-start;gap:.9rem;border:1px solid var(--border);border-radius:10px;padding:.7rem .9rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.wa-vocab.go .wa-vrow{opacity:1;transform:none;}
.wa-vocab.go .wa-vrow:nth-child(1){transition-delay:.1s} .wa-vocab.go .wa-vrow:nth-child(2){transition-delay:.3s}
.wa-vocab.go .wa-vrow:nth-child(3){transition-delay:.5s} .wa-vocab.go .wa-vrow:nth-child(4){transition-delay:.7s}
.wa-vrow .term{flex:none;width:120px;font-family:var(--font-mono);font-size:.8rem;font-weight:600;color:var(--accent);}
.wa-vrow .def{font-size:.85rem;color:var(--text-2);} .wa-vrow .def b{color:var(--text);}

/* orchestrator diagram */
.wa-orch{max-width:520px;margin:0 auto;}
.wa-orch svg{width:100%;height:230px;overflow:visible;}
.wa-onode{fill:var(--surface);stroke:var(--border-2);stroke-width:1.5;}
.wa-onode.lead{stroke:var(--accent);stroke-width:2;}
.wa-otxt{fill:var(--text);font-family:var(--font-mono);font-size:11px;text-anchor:middle;}
.wa-oleadtxt{fill:var(--accent);font-family:var(--font-mono);font-size:12px;font-weight:600;text-anchor:middle;}
.wa-osub{fill:var(--text-3);font-family:var(--font-mono);font-size:8.5px;text-anchor:middle;}
.wa-oline{stroke:var(--accent);stroke-width:1.5;fill:none;}

/* real examples */
.wa-ex{display:grid;grid-template-columns:1fr 1fr;gap:.7rem;max-width:660px;margin:0 auto;}
@media(max-width:520px){.wa-ex{grid-template-columns:1fr;}}
.wa-ex .c{border:1px solid var(--border-2);border-radius:12px;padding:.85rem 1rem;background:var(--surface);}
.wa-ex .c .k{font-family:var(--font-mono);font-size:.7rem;margin-bottom:.25rem;}
.wa-ex .c.not .k{color:var(--text-3);} .wa-ex .c.is .k{color:var(--accent);}
.wa-ex .c.is{border-color:var(--accent);}
.wa-ex .c b{display:block;color:var(--text);font-size:.85rem;margin-bottom:.2rem;}
.wa-ex .c span{font-size:.8rem;color:var(--text-2);line-height:1.45;}

/* table */
.wa-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.wa-tab th,.wa-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.wa-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.wa-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}

/* extra reveals */
.wa-decide .q{opacity:0;transition:opacity .5s ease;}
.wa-decide.go .q{opacity:1;}
.wa-decide .opt{opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.wa-decide.go .opt{opacity:1;transform:none;}
.wa-decide.go .opt:nth-child(1){transition-delay:.3s} .wa-decide.go .opt:nth-child(2){transition-delay:.55s}

.wa-path .col{opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.wa-path.go .col{opacity:1;transform:none;}
.wa-path.go .col:nth-child(1){transition-delay:.1s} .wa-path.go .col:nth-child(2){transition-delay:.4s}

.wa-ex .c{opacity:0;transform:translateY(8px);transition:opacity .45s ease,transform .45s ease;}
.wa-ex.go .c{opacity:1;transform:none;}
.wa-ex.go .c:nth-child(1){transition-delay:.1s} .wa-ex.go .c:nth-child(2){transition-delay:.25s}
.wa-ex.go .c:nth-child(3){transition-delay:.4s} .wa-ex.go .c:nth-child(4){transition-delay:.55s}

@media (prefers-reduced-motion: reduce){
  .wa-spec .stage,.wa-vocab .wa-vrow,.wa-decide .q,.wa-decide .opt,.wa-path .col,.wa-ex .c{transition:none !important;opacity:1 !important;transform:none !important;}
}
</style>

The word "agent" is everywhere now. People say they're building an agent, spawning subagents, running a multi-agent system, orchestrating a swarm of agents. It sounds impressive, and half the time even the people saying it couldn't give you a clean definition if you asked. So let me ask the plain question, the one underneath all the buzz: **what actually makes something an agent?**

Because here's the thing. You could wrap a language model in some code, have it call a tool, and print an answer, and someone will call that an "agent." You could also just have a chatbot that answers questions, and someone will call *that* an agent too, because it sounds better in a pitch. The word has been stretched until it means almost nothing. That's a shame, because there *is* a real, sharp idea hiding in there, and once you see it, you'll never be confused by the word again.

This post is about that one idea. What separates an agent from a chatbot, from a script, from a "workflow." And then the vocabulary everyone uses loosely, subagent, multi-agent, spawning, orchestrator, explained so it actually makes sense. No prior expertise needed.

## The one idea: who decides what happens next?

Here is the single question that settles whether something is an agent. Forget everything else and just ask this: **when a step finishes, who decides what the next step is, you (the programmer, in advance) or the AI (in the moment)?**

That's it. That's the whole distinction. Let me show you the three cases it produces.

<figure class="wa-fig">
  <div class="wa-spec" id="spec">
    <div class="stage">
      <div class="who">waits for you</div>
      <b>Chatbot</b>
      <span>You ask, it answers, it stops. It never decides to <em>do</em> anything on its own. Every move is yours.</span>
    </div>
    <div class="stage">
      <div class="who">you set the path</div>
      <b>Workflow</b>
      <span>It takes several steps automatically, but <em>you</em> wrote the path in advance: do A, then B, then C. Fixed rails.</span>
    </div>
    <div class="stage agent">
      <div class="who">it sets the path</div>
      <b>Agent</b>
      <span>You give it a goal, and <em>it</em> decides the steps, in the moment, based on what it sees. No fixed rails.</span>
    </div>
  </div>
  <figcaption>Three levels of "who's driving." A chatbot waits for you. A workflow follows the path you laid down ahead of time. An agent decides its own path as it goes. That last part, the AI choosing its own next step based on what just happened, is the thing that makes an agent an agent.</figcaption>
</figure>

## The definition, in one sentence

Put it plainly:

**An agent is an AI that directs its own process. You give it a goal, and it decides for itself what steps to take to get there, adjusting as it goes.**

That word *decides* is doing all the work. A chatbot doesn't decide anything beyond its next sentence. A workflow follows decisions *you* already made. An agent makes the decisions itself, at runtime, reacting to what it finds. This is the definition the serious labs use, and it's worth memorizing, because it cuts through every fuzzy use of the word.

<figure class="wa-fig">
  <div class="wa-decide" id="decide">
    <div class="q">Does the AI decide its own next steps, or did you script them in advance?</div>
    <div class="split">
      <div class="opt no">
        <div class="lab">you scripted them</div>
        <span>It's a workflow (or a chatbot). Useful, reliable, but not an agent.</span>
      </div>
      <div class="opt yes">
        <div class="lab">the AI decides</div>
        <span>Now it's an agent. It's steering itself toward the goal.</span>
      </div>
    </div>
  </div>
  <figcaption>The one test. Ask it of anything calling itself an agent. If a human wrote the sequence of steps ahead of time, it's a workflow wearing an agent costume. If the model is genuinely choosing its own steps as it goes, it's the real thing.</figcaption>
</figure>

## Fixed rails vs choosing the road

Here's the same idea pictured a different way, because it's the crux of everything. Think of a task as a journey from start to goal.

<figure class="wa-fig">
  <div class="wa-path" id="path">
    <div class="col fixed">
      <h4>Workflow: fixed rails</h4>
      <div class="rail">start</div>
      <div class="rail">↓ (you wrote this)</div>
      <div class="rail">step A</div>
      <div class="rail">↓ (you wrote this)</div>
      <div class="rail">step B</div>
      <div class="rail">↓</div>
      <div class="rail">goal</div>
    </div>
    <div class="col dyn">
      <h4>Agent: chooses the road</h4>
      <div class="rail">start</div>
      <div class="rail">↓ <span class="a">(it decides)</span></div>
      <div class="rail">"hmm, let me check X first"</div>
      <div class="rail">↓ <span class="a">(it reacts to X)</span></div>
      <div class="rail">"that failed, try Y instead"</div>
      <div class="rail">↓ <span class="a">(it adapts)</span></div>
      <div class="rail">goal</div>
    </div>
  </div>
  <figcaption>A workflow rides rails you laid down; it can only go where you built track. An agent looks at the road, picks a direction, hits a dead end, and reroutes, all by itself. The agent can handle surprises the workflow never anticipated, because it wasn't following a script. That flexibility is the whole point, and, as we'll see, the whole risk.</figcaption>
</figure>

## What an agent is made of

So what does an agent actually need, to be able to steer itself? Three things beyond the raw model. (I've gone deep on how these fit together in my other posts; here's the short version so the concept is complete.)

- **A goal**, not a script. You tell it *what* you want, not *how* to do it.
- **Tools**, so it can actually act in the world (read a file, search the web, call an API), not just talk.
- **A loop**, so it can take a step, see the result, and decide the next step, over and over, until the goal is met.

Take away the loop and it can't adapt. Take away the tools and it can only talk. Take away the goal-not-script freedom and it's just a workflow. All three together are what let it *direct its own process*, which is our definition.

## Now the vocabulary everyone uses loosely

Once you've got "an agent steers itself," the rest of the jargon falls into place easily. It's all just agents arranged in different ways.

<figure class="wa-fig">
  <div class="wa-vocab" id="vocab">
    <div class="wa-vrow"><div class="term">agent</div><div class="def"><b>One AI that steers itself toward a goal.</b> The building block. Everything below is made of these.</div></div>
    <div class="wa-vrow"><div class="term">subagent</div><div class="def"><b>An agent working under another agent.</b> Not weaker, just given a focused job by a boss agent. "You, go research flights; report back."</div></div>
    <div class="wa-vrow"><div class="term">multi-agent</div><div class="def"><b>Several agents working together</b> on one big task, each handling a specialized part.</div></div>
    <div class="wa-vrow"><div class="term">orchestrator</div><div class="def"><b>The lead agent that coordinates the others.</b> It breaks the big task into pieces, hands them out, and combines the results.</div></div>
  </div>
  <figcaption>The whole vocabulary, demystified. An <em>agent</em> is the unit. A <em>subagent</em> is an agent taking orders from another. <em>Multi-agent</em> just means several cooperating. The <em>orchestrator</em> is the one in charge. And "<em>spawning</em>" a subagent simply means the orchestrator starting up a new helper agent to handle a piece of the work.</figcaption>
</figure>

Here's how they fit together in a real multi-agent setup. A lead agent takes your request, splits it up, and spawns helpers:

<figure class="wa-fig">
  <div class="wa-orch">
    <svg viewBox="0 0 400 230">
      <rect class="wa-onode lead" x="130" y="15" width="140" height="50" rx="12"/>
      <text class="wa-oleadtxt" x="200" y="36">ORCHESTRATOR</text>
      <text class="wa-osub" x="200" y="52">splits the task, delegates</text>
      <rect class="wa-onode" x="20" y="150" width="110" height="50" rx="12"/>
      <text class="wa-otxt" x="75" y="171">subagent</text>
      <text class="wa-osub" x="75" y="187">research flights</text>
      <rect class="wa-onode" x="145" y="150" width="110" height="50" rx="12"/>
      <text class="wa-otxt" x="200" y="171">subagent</text>
      <text class="wa-osub" x="200" y="187">check hotels</text>
      <rect class="wa-onode" x="270" y="150" width="110" height="50" rx="12"/>
      <text class="wa-otxt" x="325" y="171">subagent</text>
      <text class="wa-osub" x="325" y="187">compare prices</text>
      <path class="wa-oline" d="M180,65 L75,150"/>
      <path class="wa-oline" d="M200,65 L200,150"/>
      <path class="wa-oline" d="M220,65 L325,150"/>
    </svg>
  </div>
  <figcaption>"Plan a trip" arrives at the orchestrator. It spawns three subagents, each an agent in its own right, to work in parallel on flights, hotels, and prices. Each steers itself on its piece; the orchestrator combines their answers. That's a multi-agent system. It's agents all the way down, just organized like a team with a manager.</figcaption>
</figure>

## Is it an agent? A few honest gut-checks

Let's test the definition on real things, because the label gets slapped on a lot that doesn't earn it.

<figure class="wa-fig">
  <div class="wa-ex" id="ex">
    <div class="c not">
      <div class="k">not really an agent</div>
      <b>A customer-service chatbot</b>
      <span>Answers questions from a script or knowledge base. Waits for each message. Decides nothing on its own. It's a chatbot.</span>
    </div>
    <div class="c not">
      <div class="k">not really an agent</div>
      <b>"Summarize this, then email it"</b>
      <span>Two automated steps, but <em>you</em> fixed the order. Nothing was decided at runtime. It's a workflow.</span>
    </div>
    <div class="c is">
      <div class="k">genuinely an agent</div>
      <b>"Fix the failing test in my repo"</b>
      <span>It decides to read files, run tests, try a fix, see it fail, try another, all self-directed. It's an agent.</span>
    </div>
    <div class="c is">
      <div class="k">genuinely an agent</div>
      <b>"Research this topic and write me a brief"</b>
      <span>It chooses what to search, what to read next based on what it found, when it has enough. Self-directed. An agent.</span>
    </div>
  </div>
  <figcaption>The test in action. The two on the left never decide their own steps, so despite the hype, they're a chatbot and a workflow. The two on the right genuinely choose their path as they go. Same underlying model in all four; the difference is entirely about <em>who decides the next move.</em></figcaption>
</figure>

## The honest caveat: more agent is not always better

One last thing, because it's the mistake everyone makes. The word "agent" sounds advanced, so people reach for it by default, building a fully autonomous agent when a simple workflow would have done the job better.

That's usually a mistake. The freedom that makes an agent powerful, deciding its own steps, also makes it less predictable and harder to control. A workflow that follows fixed rails is boring, but it's reliable and you always know what it'll do. Reach for a real agent only when the task is genuinely unpredictable enough that you *can't* script the path in advance. Start simple; add autonomy only when you truly need it.

<figure class="wa-fig">
<table class="wa-tab">
<thead><tr><th></th><th>Chatbot</th><th>Workflow</th><th>Agent</th></tr></thead>
<tbody>
<tr><td>Who decides steps</td><td>You, each turn</td><td>You, in advance</td><td>The AI, at runtime</td></tr>
<tr><td>Handles surprises</td><td>No</td><td>Only ones you foresaw</td><td>Yes, it adapts</td></tr>
<tr><td>Predictable</td><td>Very</td><td>Very</td><td>Less so</td></tr>
<tr><td>Best for</td><td>Q&A, conversation</td><td>Known, repeatable tasks</td><td>Open-ended, many-step tasks</td></tr>
<tr><td>Use it when</td><td>You just need answers</td><td>You can script the path</td><td>You can't script the path</td></tr>
</tbody>
</table>
  <figcaption>The full picture. None of these is "best," each fits a different kind of job. The skill isn't using the fanciest one; it's correctly spotting which one your task actually needs, and not reaching for an agent just because the word sounds good.</figcaption>
</figure>

## The takeaway

Strip away all the noise and an agent is a beautifully simple idea: **an AI that you point at a goal and let decide its own way there.** Not a chatbot waiting for your next message. Not a workflow riding rails you built in advance. Something that looks at the situation, picks a step, sees what happens, and picks the next one itself, until it's done.

Everything else is just arrangement. A subagent is an agent with a boss. A multi-agent system is a team of them. An orchestrator is the manager. Spawning is hiring a helper. It's agents all the way down, and now the whole vocabulary should read like plain English to you. The next time someone says they're "building an agent," you'll know exactly the right question to ask: *who decides what happens next?* Their answer tells you whether they've really built one, or just given a nice name to a script.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['spec','vocab','decide','path','ex'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
