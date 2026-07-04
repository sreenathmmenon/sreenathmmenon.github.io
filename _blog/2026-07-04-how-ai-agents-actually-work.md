---
title: "How AI Agents Actually Work"
date: 2026-07-04
excerpt: "A chatbot answers and stops. An agent keeps going: it thinks, does something, looks at what happened, and decides what to do next, over and over, until the job is done. That single loop is the whole idea. Here is how it works, where it came from (a 2022 paper called ReAct), and why the hardest part isn't making it smart. It's making it stop."
tags: [ai, agents, react, agent-loop, tool-use, explainer]
---

<style>
.ag-fig{margin:2.5rem 0;}
.ag-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* chatbot vs agent */
.ag-vs{display:grid;grid-template-columns:1fr 1fr;gap:.9rem;max-width:640px;margin:0 auto;}
@media(max-width:560px){.ag-vs{grid-template-columns:1fr;}}
.ag-vs .col{border:1px solid var(--border);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);}
.ag-vs .col h4{margin:0 0 .6rem;font-size:.9rem;}
.ag-vs .col.bot{border-color:var(--border-2);}
.ag-vs .col.bot h4{color:var(--text-2);}
.ag-vs .col.agent{border-color:var(--accent);}
.ag-vs .col.agent h4{color:var(--accent);}
.ag-vs .col p{font-size:.85rem;color:var(--text-2);margin:.3rem 0 0;line-height:1.55;}
.ag-vs .col .flow{font-family:var(--font-mono);font-size:.76rem;color:var(--text-3);margin-top:.6rem;}

/* the loop diagram */
.ag-loop{max-width:420px;margin:0 auto;position:relative;height:320px;}
.ag-loop svg{width:100%;height:100%;overflow:visible;}
.ag-node{fill:var(--surface);stroke:var(--border-2);stroke-width:1.5;}
.ag-node-txt{fill:var(--text);font-family:var(--font-mono);font-size:13px;font-weight:600;text-anchor:middle;}
.ag-node-sub{fill:var(--text-3);font-family:var(--font-mono);font-size:9.5px;text-anchor:middle;}
.ag-arrow{fill:none;stroke:var(--accent);stroke-width:2;marker-end:url(#agh);}
.ag-arrow-dash{stroke-dasharray:4 4;}

/* trajectory trace */
.ag-trace{max-width:600px;margin:0 auto;font-family:var(--font-mono);font-size:.82rem;display:flex;flex-direction:column;gap:.45rem;}
.ag-line{padding:.55rem .8rem;border-radius:8px;border:1px solid var(--border);background:var(--surface);opacity:0;transform:translateX(-6px);transition:opacity .45s ease,transform .45s ease;}
.ag-trace.go .ag-line{opacity:1;transform:none;}
.ag-trace.go .ag-line:nth-child(1){transition-delay:.1s}
.ag-trace.go .ag-line:nth-child(2){transition-delay:.5s}
.ag-trace.go .ag-line:nth-child(3){transition-delay:.9s}
.ag-trace.go .ag-line:nth-child(4){transition-delay:1.3s}
.ag-trace.go .ag-line:nth-child(5){transition-delay:1.7s}
.ag-trace.go .ag-line:nth-child(6){transition-delay:2.1s}
.ag-trace.go .ag-line:nth-child(7){transition-delay:2.5s}
.ag-line .tg{font-weight:600;}
.ag-line.think{border-color:var(--border-2);} .ag-line.think .tg{color:var(--text);}
.ag-line.act .tg{color:var(--accent);}
.ag-line.obs{color:var(--text-2);} .ag-line.obs .tg{color:var(--text-3);}
.ag-line.done{border-color:var(--accent);} .ag-line.done .tg{color:var(--accent);}

/* equation */
.ag-eq{max-width:600px;margin:0 auto;display:flex;flex-wrap:wrap;gap:.5rem;justify-content:center;align-items:stretch;}
.ag-eq .term{border:1px solid var(--border-2);border-radius:10px;padding:.7rem .9rem;background:var(--surface);text-align:center;min-width:110px;}
.ag-eq .term b{display:block;color:var(--accent);font-family:var(--font-mono);font-size:.9rem;}
.ag-eq .term span{font-size:.76rem;color:var(--text-2);}
.ag-eq .plus{align-self:center;font-family:var(--font-mono);color:var(--text-3);font-size:1.1rem;}

/* react vs function calling table + timeline */
.ag-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.ag-tab th,.ag-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.ag-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.ag-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}
.ag-tab code{font-size:.82rem;}

/* stopping / guardrail bars */
.ag-stop{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:.7rem;}
.ag-stop .row{display:flex;align-items:center;gap:.8rem;background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:.65rem .85rem;}
.ag-stop .row .ic{flex:none;width:26px;height:26px;border-radius:6px;background:var(--grad);color:var(--accent-ink);font-family:var(--font-mono);font-weight:600;font-size:.8rem;display:flex;align-items:center;justify-content:center;}
.ag-stop .row .t{font-size:.87rem;color:var(--text-2);}
.ag-stop .row .t b{color:var(--text);}

/* runaway meter */
.ag-run{max-width:560px;margin:0 auto;}
.ag-run .track{height:20px;border-radius:999px;background:var(--surface-2);border:1px solid var(--border);overflow:hidden;position:relative;}
.ag-run .fill{height:100%;width:0;background:var(--grad);transition:width 2s linear;}
.ag-run.go .fill{width:100%;}
.ag-run .cap{font-family:var(--font-mono);font-size:.74rem;color:var(--text-3);text-align:center;margin-top:.4rem;}
.ag-run .cap b{color:var(--accent);}

@media (prefers-reduced-motion: reduce){
  .ag-trace .ag-line{transition:none !important;opacity:1 !important;transform:none !important;}
  .ag-run .fill{transition:none !important;width:100% !important;}
}
</style>

Ask a normal chatbot to book you a table, and it will cheerfully tell you *how* to book a table. Ask an agent, and it will actually go and try to book the table: check the calendar, look up the restaurant, see it's full, pick another night, and come back with a confirmation. Same underlying model. Completely different behaviour. One talks. The other *acts, looks at what happened, and keeps going.*

That gap is the most important idea in AI right now, and underneath all the noise it is shockingly simple. An agent is a language model wrapped in a loop. That's the secret. Not a new kind of brain, a new kind of *plumbing* around the same brain, one that lets it take a step, see the result, and decide the next step, again and again, until the work is done.

I want to take you through that loop honestly: what it is, where it came from (a single 2022 paper that quietly started all of this), how modern agents changed it, and the part nobody warns you about, which is that the hard problem was never making an agent smart. It was making it *stop*. I've built a few of these, and that last part is where the real scars are.

## The difference in one picture

<figure class="ag-fig">
  <div class="ag-vs">
    <div class="col bot">
      <h4>A chatbot</h4>
      <p>Takes your message, produces one answer, and is finished. It cannot check whether the answer was right. It cannot do anything about it if it wasn't.</p>
      <div class="flow">ask → answer → stop</div>
    </div>
    <div class="col agent">
      <h4>An agent</h4>
      <p>Takes a goal, then works toward it in steps. Each step it can reach into the real world, see what came back, and adjust. It runs until the goal is met, not until it has said one thing.</p>
      <div class="flow">goal → think → act → observe → think → … → done</div>
    </div>
  </div>
  <figcaption>The whole distinction. A chatbot is a function that returns once. An agent is a loop that keeps returning to the same model with an updated picture of the world, asking "given what just happened, what now?"</figcaption>
</figure>

## The loop itself: Think, Act, Observe

Here is the engine. Strip an agent down to nothing and this is what's left, three moves in a circle:

- **Think.** The model reasons about the goal and the situation right now. What's the next best move?
- **Act.** It does one concrete thing in the world: calls a tool, runs code, queries a database, searches the web.
- **Observe.** It reads what came back, the result, the error, the data, and feeds that into the next Think.

Then it loops. The observation from this turn becomes part of the thinking for the next turn. That feedback, the fact that the agent *sees the consequences of its own action* before choosing again, is the entire source of an agent's power.

<figure class="ag-fig">
  <div class="ag-loop">
    <svg viewBox="0 0 400 320">
      <defs>
        <marker id="agh" markerWidth="9" markerHeight="9" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" fill="var(--accent)"/></marker>
      </defs>
      <!-- Think top -->
      <rect class="ag-node" x="140" y="20" width="120" height="58" rx="12"/>
      <text class="ag-node-txt" x="200" y="44">THINK</text>
      <text class="ag-node-sub" x="200" y="62">reason about next move</text>
      <!-- Act bottom-right -->
      <rect class="ag-node" x="250" y="200" width="120" height="58" rx="12"/>
      <text class="ag-node-txt" x="310" y="224">ACT</text>
      <text class="ag-node-sub" x="310" y="242">call a tool / run code</text>
      <!-- Observe bottom-left -->
      <rect class="ag-node" x="30" y="200" width="120" height="58" rx="12"/>
      <text class="ag-node-txt" x="90" y="224">OBSERVE</text>
      <text class="ag-node-sub" x="90" y="242">read the result</text>
      <!-- arrows around the circle -->
      <path class="ag-arrow" d="M258,58 Q340,110 320,196"/>
      <path class="ag-arrow" d="M250,240 Q200,270 152,240"/>
      <path class="ag-arrow" d="M82,196 Q60,110 142,60"/>
      <!-- done exit -->
      <path class="ag-arrow ag-arrow-dash" d="M200,78 L200,150"/>
      <text class="ag-node-sub" x="228" y="120" style="fill:var(--accent)">goal met? → stop</text>
    </svg>
  </div>
  <figcaption>Think leads to Act, Act produces something to Observe, Observe feeds back into Think. Round and round. The dashed line is the way out: at the top of every loop the agent checks "am I done?" and only then breaks the circle. Hold onto that exit. We'll come back to it, because it's where agents go wrong.</figcaption>
</figure>

## Where this came from: a paper called ReAct

This wasn't obvious. For a while, the clever trick was "chain of thought", letting a model reason step by step out loud before answering. It helped, but it had a rotten failure: the model was reasoning in a sealed room. If it *thought* something false, it had no way to check, and the mistake carried through to the end. Confident, well-reasoned, wrong.

In October 2022, a group of researchers (Shunyu Yao and colleagues, out of Princeton and Google) published a paper called **ReAct: Synergizing Reasoning and Acting in Language Models**. The idea in the title is the whole thing: don't make the model *only* reason, and don't make it *only* act. **Interleave them.** Let it write a thought, take an action based on that thought, observe the real result, and use that to correct its next thought.

That one move fixed the sealed-room problem. Now when the model reasoned toward something false, its very next action bumped into reality, a real Wikipedia lookup, a real tool result, and reality corrected it. Reasoning kept the actions purposeful; actions kept the reasoning honest. A ReAct trajectory reads almost like a person thinking out loud while working:

<figure class="ag-fig">
  <div class="ag-trace" id="trace">
    <div class="ag-line think"><span class="tg">Thought:</span> The user wants the population of the author's birth city. I don't know the city yet, so first find the author.</div>
    <div class="ag-line act"><span class="tg">Action:</span> search("author of The Left Hand of Darkness")</div>
    <div class="ag-line obs"><span class="tg">Observation:</span> Ursula K. Le Guin, born in Berkeley, California.</div>
    <div class="ag-line think"><span class="tg">Thought:</span> Good, birth city is Berkeley. Now I need Berkeley's population.</div>
    <div class="ag-line act"><span class="tg">Action:</span> search("population of Berkeley, California")</div>
    <div class="ag-line obs"><span class="tg">Observation:</span> About 120,000.</div>
    <div class="ag-line done"><span class="tg">Answer:</span> Around 120,000. → done</div>
  </div>
  <figcaption>A ReAct-style trace. Notice it never guessed the population from memory. It found the city, checked reality, then found the number, checking reality again. Each Observation grounds the next Thought. This interleaving is exactly what chain-of-thought alone couldn't do.</figcaption>
</figure>

And it worked, measurably. On interactive benchmarks, ReAct beat the previous best approaches by a wide margin, 34% better on a household-tasks benchmark called ALFWorld, 10% better on a web-shopping one called WebShop, using just one or two examples. On top of that, its step-by-step trajectories were far easier for a human to read and trust than a black-box answer. That combination, better results *and* more interpretable, is why this paper is widely treated as the seed of the whole agent era.

## An agent is four things bolted together

The loop is the runtime, but what's actually *running* inside it? A clean way to hold it, one that senior engineers use as a mental checklist:

<figure class="ag-fig">
  <div class="ag-eq">
    <div class="term"><b>LLM</b><span>the reasoning brain</span></div>
    <div class="plus">+</div>
    <div class="term"><b>Tools</b><span>hands to act in the world</span></div>
    <div class="plus">+</div>
    <div class="term"><b>Memory</b><span>what happened so far</span></div>
    <div class="plus">+</div>
    <div class="term"><b>Planning</b><span>breaking the goal into steps</span></div>
  </div>
  <figcaption>Agent = LLM + Tools + Memory + Planning, and the loop is what ties them together. Take away tools and it can only talk. Take away memory and it forgets last turn. Take away planning and it can't handle anything multi-step. The loop is where all four meet, turn after turn.</figcaption>
</figure>

If you've read my earlier posts, this is the moment they all click into one picture. **MCP** is how the *Act* step reaches real tools. **Skills** are packaged know-how the agent can pull in when a task matches. **Modes** (like Roo's Architect/Code/Debug) shape how the *Think* step behaves. **Embeddings** power the *Memory*, recalling what's relevant to now. None of those were separate topics. They were all pieces of this one loop, and now you can see where each one plugs in.

## How modern agents quietly changed the format

Here's a nuance worth knowing, because people conflate two things. The original ReAct had the model literally *write out* "Thought:", "Action:", "Observation:" as text, and the surrounding code parsed that text to figure out what to do. It worked, but parsing free-form text is fragile.

Modern models learned to do the *Act* step natively, through **tool calling** (also called function calling). Instead of writing "Action: search(...)" as prose to be parsed, the model emits a clean, structured request for a specific tool with typed arguments, and the runtime executes it directly. Same loop, sturdier joints. So today you'll meet two flavours, and both are the agent loop underneath:

<figure class="ag-fig">
<table class="ag-tab">
<thead><tr><th></th><th>Classic ReAct (text)</th><th>Native tool-calling</th></tr></thead>
<tbody>
<tr><td>How it acts</td><td>Writes "Action: ..." as text, code parses it</td><td>Emits a structured tool request directly</td></tr>
<tr><td>Reasoning</td><td>Fully visible in the trace</td><td>Often more implicit, less to read</td></tr>
<tr><td>Sturdiness</td><td>Fragile, parsing can break</td><td>Robust, no parsing guesswork</td></tr>
<tr><td>Best when</td><td>You need transparency, or the model has no tool-calling</td><td>Speed and reliability at scale</td></tr>
<tr><td>Still the loop?</td><td>Yes</td><td>Yes, exactly the same Think→Act→Observe</td></tr>
</tbody>
</table>
  <figcaption>ReAct is the founding idea; native tool-calling is the industrial version of the same idea. Transparency versus efficiency. Many production agents lean on tool-calling for speed but keep some visible reasoning for the hard, ambiguous steps. It isn't ReAct-or-nothing; it's a dial.</figcaption>
</figure>

So, to answer the question directly: **ReAct and "the agent loop" are not the same thing.** The agent loop is the general Think→Act→Observe cycle. ReAct is the specific, historic technique that made that loop practical for language models by interleaving written reasoning with actions. Every ReAct agent is running the loop; not every agent loop is literally ReAct.

## The part nobody warns you about: making it stop

Now the honest part. Building an agent that can *act* is the easy half. Building one that reliably *stops* is the half that keeps you up at night, and it's where most demos quietly fall apart in production.

Think back to that dashed "am I done?" exit on the loop diagram. An agent decides for itself whether to keep going. That's the whole point of autonomy, and it's also the whole danger. What happens when it's wrong about being done?

<figure class="ag-fig">
  <div class="ag-run" id="run">
    <div class="track"><div class="fill"></div></div>
    <div class="cap">iteration 1 … 50 … 400 … <b>still going, still spending</b></div>
  </div>
  <figcaption>A loop with no hard ceiling. The agent hits a subtle dead end, a tool that keeps returning empty, a goal it can't quite satisfy, and instead of stopping, it tries again. And again. Each turn is a real model call costing real money. This is not hypothetical: teams have reported single runaway sessions burning thousands of dollars before anyone noticed.</figcaption>
</figure>

The failure modes are famous enough that they have names now. The big ones:

- **The infinite loop.** No stopping condition, so it never ends. The most common production failure, full stop.
- **The retry spiral.** A tool returns nothing useful, the model assumes it just phrased it wrong, and retries the same doomed action forever.
- **Cost blowup.** Every iteration is a paid model call plus paid tool calls. Uncapped loops turn a bug into a bill.
- **Wrong-tool drift.** Vague tool descriptions lead the agent to keep reaching for the wrong tool, never making progress.

The fix is not "make the model smarter." A smarter model can rationalize continuing just as easily. The fix is **guardrails that don't depend on the agent's own judgment**, an escape hatch it can't argue its way out of:

<figure class="ag-fig">
  <div class="ag-stop">
    <div class="row"><div class="ic">1</div><div class="t"><b>A hard iteration cap.</b> "You get at most 25 turns." Non-negotiable, enforced by the outer code, not the model.</div></div>
    <div class="row"><div class="ic">2</div><div class="t"><b>A cost / token budget.</b> Stop when spend crosses a line, no matter what the agent thinks it needs.</div></div>
    <div class="row"><div class="ic">3</div><div class="t"><b>No-progress detection.</b> If several turns produce nothing new, break out. Repetition is the tell.</div></div>
    <div class="row"><div class="ic">4</div><div class="t"><b>Human-in-the-loop checkpoints.</b> For anything irreversible (spending money, deleting data), pause and ask a person first.</div></div>
  </div>
  <figcaption>The line between a slick demo and a production agent is almost entirely here. The industry even has a name for this discipline now: loop engineering. The clever prompt gets you the demo; the guardrails get you something you can actually deploy.</figcaption>
</figure>

This is exactly the design principle behind the safer-agent work I've been building. When an agent can spend money or take irreversible actions, the sane default is deny-by-default with a human checkpoint, and a budget it physically cannot exceed. Autonomy is wonderful right up to the moment it isn't, and the guardrails are what let you sleep while it runs.

## The whole thing, in one breath

An agent is a language model in a loop. It **thinks** about the goal, **acts** on the world, **observes** what happened, and repeats, until it decides it's done. That loop was made practical by **ReAct** in 2022, which interleaved reasoning with real actions so the model's mistakes could be corrected by reality instead of compounding in a vacuum. Modern agents run the same loop with sturdier, native tool-calling. And the four pieces, LLM, tools, memory, planning, are what turn inside it.

The magic, when you finally see it, isn't magic at all. It's a very smart thing that learned to check its work by doing, one honest step at a time. And the craft of building one well is mostly the unglamorous discipline of knowing when to make it stop.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['trace','run'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
