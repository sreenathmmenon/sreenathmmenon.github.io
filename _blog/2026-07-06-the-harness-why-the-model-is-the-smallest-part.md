---
title: "The Harness: Why the Model Is the Smallest Part of Your AI Agent"
date: 2026-07-06
excerpt: "Everyone obsesses over which model an agent uses. But teams keep making agents dramatically better without touching the model at all. One team deleted 80% of their agent's tools and success climbed from 80% to 100%. The thing they changed has a name: the harness. Here is what it is, why it suddenly matters more than the model, and how to use it to your advantage."
tags: [ai, agents, harness, harness-engineering, tool-use, explainer]
---

<style>
.hn-fig{margin:2.5rem 0;}
.hn-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* model+harness equation */
.hn-eq{max-width:600px;margin:0 auto;display:flex;flex-wrap:wrap;gap:.6rem;justify-content:center;align-items:stretch;}
.hn-eq .term{border:1px solid var(--border-2);border-radius:12px;padding:.9rem 1.1rem;background:var(--surface);text-align:center;min-width:120px;}
.hn-eq .term.model{border-color:var(--border-2);}
.hn-eq .term.harn{border-color:var(--accent);}
.hn-eq .term b{display:block;font-family:var(--font-mono);font-size:1rem;margin-bottom:.2rem;}
.hn-eq .term.model b{color:var(--text-2);} .hn-eq .term.harn b{color:var(--accent);}
.hn-eq .term span{font-size:.78rem;color:var(--text-2);}
.hn-eq .op{align-self:center;font-family:var(--font-mono);color:var(--text-3);font-size:1.3rem;}

/* the model is small, harness is big (proportion visual) */
.hn-prop{max-width:560px;margin:0 auto;}
.hn-prop .bar{display:flex;height:56px;border-radius:12px;overflow:hidden;border:1px solid var(--border);}
.hn-prop .model{flex:0 0 18%;background:var(--surface-2);display:flex;align-items:center;justify-content:center;font-family:var(--font-mono);font-size:.74rem;color:var(--text-2);}
.hn-prop .harness{flex:1;background:var(--grad);display:flex;align-items:center;justify-content:center;font-family:var(--font-mono);font-size:.78rem;color:var(--accent-ink);font-weight:600;}

/* five necessary parts */
.hn-parts{display:grid;grid-template-columns:repeat(5,1fr);gap:.55rem;max-width:700px;margin:0 auto;}
@media(max-width:640px){.hn-parts{grid-template-columns:repeat(2,1fr);}}
.hn-part{border:1px solid var(--accent);border-radius:12px;padding:.8rem .55rem;background:var(--surface);text-align:center;opacity:0;transform:translateY(8px);transition:opacity .4s ease,transform .4s ease;}
.hn-parts.go .hn-part{opacity:1;transform:none;}
.hn-parts.go .hn-part:nth-child(1){transition-delay:.1s}
.hn-parts.go .hn-part:nth-child(2){transition-delay:.25s}
.hn-parts.go .hn-part:nth-child(3){transition-delay:.4s}
.hn-parts.go .hn-part:nth-child(4){transition-delay:.55s}
.hn-parts.go .hn-part:nth-child(5){transition-delay:.7s}
.hn-part b{display:block;font-size:.82rem;color:var(--text);margin-bottom:.2rem;}
.hn-part span{font-size:.72rem;color:var(--text-2);line-height:1.4;}

/* the killer proof bars */
.hn-proof{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:1.2rem;}
.hn-proof .grp .cap{font-family:var(--font-mono);font-size:.78rem;color:var(--text-2);margin-bottom:.5rem;}
.hn-proof .grp .cap b{color:var(--text);}
.hn-proof .rows{display:flex;flex-direction:column;gap:.5rem;}
.hn-proof .r{display:flex;align-items:center;gap:.7rem;}
.hn-proof .r .lb{flex:none;width:110px;font-family:var(--font-mono);font-size:.72rem;color:var(--text-2);text-align:right;}
.hn-proof .r .track{flex:1;height:18px;border-radius:6px;background:var(--surface-2);border:1px solid var(--border);overflow:hidden;}
.hn-proof .r .fill{height:100%;width:0;transition:width 1s cubic-bezier(.2,.7,.2,1);}
.hn-proof.go .r .fill{width:var(--w);}
.hn-proof .r .fill.before{background:var(--surface-2);border-right:2px solid var(--text-3);}
.hn-proof .r .fill.after{background:var(--grad);}
.hn-proof .r .v{flex:none;width:44px;font-family:var(--font-mono);font-size:.72rem;color:var(--accent);}

/* eval vs agent harness */
.hn-two{display:grid;grid-template-columns:1fr 1fr;gap:.8rem;max-width:660px;margin:0 auto;}
@media(max-width:560px){.hn-two{grid-template-columns:1fr;}}
.hn-two .c{border:1px solid var(--border-2);border-radius:12px;padding:1rem;background:var(--surface);}
.hn-two .c .tag{font-family:var(--font-mono);font-size:.7rem;color:var(--accent);border:1px solid var(--accent);border-radius:5px;padding:.1rem .4rem;display:inline-block;margin-bottom:.5rem;}
.hn-two .c b{display:block;color:var(--text);font-size:.9rem;margin-bottom:.3rem;}
.hn-two .c span{font-size:.83rem;color:var(--text-2);line-height:1.5;}

/* layers: prompt -> context -> harness */
.hn-layers{max-width:560px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.hn-lay{border:1px solid var(--border);border-radius:10px;padding:.75rem 1rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.hn-layers.go .hn-lay{opacity:1;transform:none;}
.hn-layers.go .hn-lay:nth-child(1){transition-delay:.1s}
.hn-layers.go .hn-lay:nth-child(2){transition-delay:.35s}
.hn-layers.go .hn-lay:nth-child(3){transition-delay:.6s}
.hn-lay b{color:var(--text);font-size:.9rem;} .hn-lay.top b{color:var(--accent);}
.hn-lay span{display:block;font-size:.82rem;color:var(--text-2);margin-top:.15rem;}

/* table */
.hn-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.hn-tab th,.hn-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.hn-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.hn-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}

@media (prefers-reduced-motion: reduce){
  .hn-parts .hn-part,.hn-proof .r .fill,.hn-layers .hn-lay{transition:none !important;}
  .hn-parts .hn-part{opacity:1 !important;transform:none !important;}
  .hn-proof .r .fill{width:var(--w) !important;}
  .hn-layers .hn-lay{opacity:1 !important;transform:none !important;}
}
</style>

Here's a result that should stop you in your tracks. A team at Vercel took their AI agent, deleted **80% of its tools**, changed nothing about the underlying model, and watched its success rate climb from 80% to 100%. Around the same time, LangChain moved their coding agent from the *bottom* of a well-known benchmark to the *top five*, again without swapping the model. They changed something else entirely.

That "something else" is the quiet star of AI in 2026, and it has a slightly odd name: the **harness**.

We spend so much energy arguing about *which model* to use, as if the model were the whole story. It isn't. A raw language model, on its own, can only do one thing: read some text and predict more text. It can't click a button, run a test, remember what it did five minutes ago, or stop when it's finished. Everything that turns that text-predictor into a thing that *gets work done*, the loop, the tools, the memory, the guardrails, all of that is the harness. And it turns out the harness often matters *more* than the model. Let me show you why, in a way that'll make sense even if today is your first day thinking about agents.

## The one equation: Agent = Model + Harness

Start here, because everything flows from it. An agent is not just a model. It's a model *plus* the infrastructure wrapped around it.

<figure class="hn-fig">
  <div class="hn-eq">
    <div class="term model"><b>Model</b><span>the intelligence: reasons, predicts text</span></div>
    <div class="op">+</div>
    <div class="term harn"><b>Harness</b><span>the machinery: loop, tools, memory, guardrails</span></div>
    <div class="op">=</div>
    <div class="term harn" style="border-style:dashed"><b>Agent</b><span>something that actually does things</span></div>
  </div>
  <figcaption>The model supplies the brains. The harness supplies the hands, the memory, and the rules. Neither is an agent on its own. A brilliant model with no harness is just a chatbot; a great harness with no model is an empty shell. The agent lives in the combination.</figcaption>
</figure>

And here's the part that surprises people: by *volume of engineering*, the model is the small piece. Almost everything you build when you build an agent is harness.

<figure class="hn-fig">
  <div class="hn-prop">
    <div class="bar">
      <div class="model">Model</div>
      <div class="harness">Harness: loop · tools · memory · context · guardrails · tracing</div>
    </div>
  </div>
  <figcaption>"The LLM is the smallest part of your agent system." You don't build the model, you call it. What you actually design, debug, and sweat over is the harness around it. This is why two teams using the identical model can ship agents that are worlds apart in quality.</figcaption>
</figure>

## What a harness is actually made of

So what's inside this thing? Researchers have even worked out the *minimum* set of parts, the necessary-and-sufficient conditions for something to count as a harness (there's a 2026 paper on exactly this, arXiv 2606.10106). Boiled down, a harness needs these five:

<figure class="hn-fig">
  <div class="hn-parts" id="parts">
    <div class="hn-part"><b>Loop</b><span>call, act, observe, repeat</span></div>
    <div class="hn-part"><b>Tools</b><span>reach the real world</span></div>
    <div class="hn-part"><b>Context</b><span>remember across steps</span></div>
    <div class="hn-part"><b>Actions + observations</b><span>do a thing, read the result</span></div>
    <div class="hn-part"><b>Stopping</b><span>know when it's done</span></div>
  </div>
  <figcaption>The five essentials. A prompt alone has none of this, it's just text. A model alone can't loop, use tools, or stop itself. The harness is what binds these five into a system where the model can act, see what happened, and keep going toward a goal. If you read my agent-loop post, you'll recognise this: the loop is the beating heart of the harness.</figcaption>
</figure>

In practice, a serious harness grows a few more organs on top of those five: **system prompts** (the standing rules), **sandboxing** (a safe place to run code), **persistent storage** (so it can pick up where it left off), **memory management** (compacting context so long tasks don't degrade, exactly the context-engineering problem), **verification loops** (checking its own work), **guardrails** (permission limits and human approval for risky actions), and **observability** (logs so you can see what it did and why). That's the full anatomy: a small model in the middle, a lot of carefully built machinery around it.

## Why this suddenly matters: the models are converging

For a while, the way to get a better agent was obvious: wait for a smarter model. And that worked, because models were leaping ahead every few months. But something shifted. The top models are now *close* to each other in raw ability. When everyone has access to roughly equally-brilliant brains, the brain stops being the differentiator.

So what decides the winner? The machinery around the brain. **As models converge, the harness increasingly determines performance.** This is the whole reason "harness engineering" became the phrase on everyone's lips in 2026. And the proof isn't theory, it's measured, repeatable, and honestly a little shocking:

<figure class="hn-fig">
  <div class="hn-proof" id="proof">
    <div class="grp">
      <div class="cap"><b>Vercel:</b> removed 80% of the agent's tools (same model)</div>
      <div class="rows">
        <div class="r"><div class="lb">before</div><div class="track"><div class="fill before" style="--w:80%"></div></div><div class="v">80%</div></div>
        <div class="r"><div class="lb">after</div><div class="track"><div class="fill after" style="--w:100%"></div></div><div class="v">100%</div></div>
      </div>
    </div>
    <div class="grp">
      <div class="cap"><b>Databricks:</b> paired the model with a better task-specific harness</div>
      <div class="rows">
        <div class="r"><div class="lb">weak harness</div><div class="track"><div class="fill before" style="--w:36%"></div></div><div class="v">36%</div></div>
        <div class="r"><div class="lb">strong harness</div><div class="track"><div class="fill after" style="--w:53%"></div></div><div class="v">53%</div></div>
      </div>
    </div>
  </div>
  <figcaption>Same model, different harness, dramatically different results. Vercel's counterintuitive win, <em>fewer</em> tools made it <em>better</em>, because a bloated toolset confuses the model about which tool to reach for. Databricks nearly doubled document-task accuracy purely through harness design. LangChain, similarly, jumped from bottom to top-five on a coding benchmark by changing only the harness. The model was never the bottleneck.</figcaption>
</figure>

Sit with the Vercel result especially, because it flips an instinct. More tools *feels* like more capability. But every extra tool is another choice the model can get wrong, more noise, more ways to reach for the wrong thing. Cutting the toolset down to the sharp essentials made the agent more reliable. In harness engineering, subtraction is often the upgrade. (If that echoes the Pi post's "do less" philosophy, it should, same lesson, different level.)

## Two things called "harness" (don't mix them up)

Quick but important clarification, because you'll hear the word in two different rooms and they mean different things:

<figure class="hn-fig">
  <div class="hn-two">
    <div class="c">
      <div class="tag">eval harness</div>
      <b>For testing models</b>
      <span>Standardized infrastructure that runs a model against benchmarks and scores it. Think EleutherAI's lm-evaluation-harness or Stanford's HELM. Its job: "load a model, run the tests, compare." It measures.</span>
    </div>
    <div class="c">
      <div class="tag">agent harness</div>
      <b>For running agents</b>
      <span>The runtime machinery that wraps a model so it can act: loop, tools, memory, guardrails. Its job: "turn a text-predictor into a worker." It operates. This is the one everyone's discussing in 2026.</span>
    </div>
  </div>
  <figcaption>Same word, two jobs. An eval harness <em>judges</em> a model; an agent harness <em>empowers</em> one. They're cousins (a good agent harness has evals baked in to check the agent's work), but if someone says "harness" in an agent conversation, they almost always mean the second.</figcaption>
</figure>

## Where harness sits above prompt and context

If you've followed my earlier posts, here's how the whole toolkit stacks up, from smallest lever to biggest:

<figure class="hn-fig">
  <div class="hn-layers" id="layers">
    <div class="hn-lay"><b>Prompt engineering</b><span>Optimize the words you send in. The smallest, most local lever.</span></div>
    <div class="hn-lay"><b>Context engineering</b><span>Curate everything the model sees on each call: what to keep, fetch, trim, and where to place it.</span></div>
    <div class="hn-lay top"><b>Harness engineering</b><span>Design the entire system: the loop, tools, memory, guardrails, evals. The biggest lever of all.</span></div>
  </div>
  <figcaption>A ladder of leverage. Prompt engineering tunes the input. Context engineering (from my earlier post) curates the information flow. Harness engineering designs the whole machine around the model. Each level up gives you more control, and in 2026, the top of that ladder is where the real gains live.</figcaption>
</figure>

## How to use this to your advantage

This isn't just theory to nod at. It's genuinely actionable, whether you're building agents or just choosing tools. The practical takeaways:

<figure class="hn-fig">
<table class="hn-tab">
<thead><tr><th>If you want to...</th><th>Work on the harness, not the model</th></tr></thead>
<tbody>
<tr><td>Make an agent more reliable</td><td>Trim the toolset to the essentials (fewer, sharper tools beat many). Add verification loops so it checks its own work.</td></tr>
<tr><td>Stop it running away or costing too much</td><td>Add guardrails: hard step limits, cost budgets, human approval for risky actions. This is harness work, not prompt work.</td></tr>
<tr><td>Keep it sane on long tasks</td><td>Invest in memory management: compaction and retrieval, so context doesn't degrade. (The context-engineering lever.)</td></tr>
<tr><td>Debug why it fails</td><td>Add observability: log every action and decision. You can't improve a harness you can't see.</td></tr>
<tr><td>Get more without a bigger model</td><td>Improve the harness first. The Vercel and LangChain results show the ceiling is usually the harness, not the model.</td></tr>
<tr><td>Avoid "agent sprawl"</td><td>Build shared harness infrastructure with common governance and evals, rather than every team hand-rolling their own.</td></tr>
</tbody>
</table>
  <figcaption>The unifying advice: before you reach for a more expensive model, ask whether your harness is the real bottleneck. Most of the time, it is, and fixing it is cheaper, faster, and more within your control than waiting for the next model.</figcaption>
</figure>

## The takeaway

We've been staring at the wrong part. The model is the brain, yes, and brains matter, but a brain with no body, no memory, and no rules doesn't get anything done. The harness is the body: the loop that lets it act, the tools that let it reach the world, the memory that lets it stay coherent, the guardrails that keep it safe, the evals that keep it honest. And as models grow more alike, that surrounding machinery is what separates an agent that dazzles in a demo from one you'd actually trust with real work.

So the next time someone asks "which model does your agent use?", the more interesting question is the one underneath it: *what's the harness?* Because a team that deleted 80% of their tools and doubled from good to perfect already told you where the real leverage lives. It was never only the brain. It was the harness all along.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['parts','proof','layers'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
