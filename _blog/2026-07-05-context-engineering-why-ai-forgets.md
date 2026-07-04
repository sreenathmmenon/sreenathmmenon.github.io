---
title: "Context Engineering: Why AI Forgets, and How We Fight It"
date: 2026-07-05
excerpt: "Every AI model has a working memory with a hard edge. Fill it and the oldest things fall off, and even before that, it quietly stops paying attention to the middle. Context engineering is the craft of deciding what the model gets to see on every single call. Here is why forgetting happens, the research that proved it, and the real techniques teams use to fight it."
tags: [ai, context-engineering, llm, memory, agents, explainer]
---

<style>
.cx-fig{margin:2.5rem 0;}
.cx-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* context window as a fixed box filling and overflowing */
.cx-win{max-width:520px;margin:0 auto;}
.cx-win .box{border:2px solid var(--border-2);border-radius:12px;padding:.6rem;background:var(--surface);display:flex;flex-direction:column;gap:.35rem;position:relative;overflow:hidden;}
.cx-win .slot{font-family:var(--font-mono);font-size:.74rem;padding:.35rem .6rem;border-radius:6px;background:var(--surface-2);border:1px solid var(--border);color:var(--text-2);}
.cx-win .slot.sys{border-color:var(--accent);color:var(--accent);}
.cx-win .slot.gone{opacity:0;transform:translateY(-8px);transition:opacity .6s ease,transform .6s ease;}
.cx-win.go .slot.gone{opacity:.25;transform:none;}
.cx-win .cap{font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);text-align:center;margin-top:.5rem;}
.cx-win .limit{position:absolute;right:8px;top:6px;font-family:var(--font-mono);font-size:.66rem;color:var(--text-3);}

/* lost-in-the-middle U curve */
.cx-u{max-width:560px;margin:0 auto;}
.cx-u svg{width:100%;height:220px;overflow:visible;}
.cx-u .grid-l{stroke:var(--border);stroke-width:1;}
.cx-u .curve{fill:none;stroke:var(--accent);stroke-width:2.5;stroke-linecap:round;stroke-dasharray:600;stroke-dashoffset:600;transition:stroke-dashoffset 1.8s ease;}
.cx-u.go .curve{stroke-dashoffset:0;}
.cx-u .axl{fill:var(--text-3);font-family:var(--font-mono);font-size:10px;}
.cx-u .zone{fill:var(--text-2);font-family:var(--font-mono);font-size:10px;text-anchor:middle;}
.cx-u .dot{fill:var(--accent);opacity:0;transition:opacity .4s ease .8s;}
.cx-u.go .dot{opacity:1;}

/* four failures of a naive approach */
.cx-fail{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:.6rem;}
.cx-fail .row{display:flex;align-items:center;gap:.8rem;background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:.65rem .85rem;}
.cx-fail .row .ic{flex:none;width:26px;height:26px;border-radius:6px;background:var(--surface-2);border:1px solid var(--text-3);color:var(--text-3);font-family:var(--font-mono);font-weight:600;font-size:.8rem;display:flex;align-items:center;justify-content:center;}
.cx-fail .row .t{font-size:.87rem;color:var(--text-2);}
.cx-fail .row .t b{color:var(--text);}

/* techniques cards */
.cx-tech{display:grid;grid-template-columns:repeat(2,1fr);gap:.7rem;max-width:680px;margin:0 auto;}
@media(max-width:560px){.cx-tech{grid-template-columns:1fr;}}
.cx-tech .t{border:1px solid var(--border-2);border-radius:12px;padding:.9rem 1rem;background:var(--surface);}
.cx-tech .t .k{font-family:var(--font-mono);font-size:.7rem;color:var(--accent);border:1px solid var(--accent);border-radius:5px;padding:.1rem .4rem;display:inline-block;margin-bottom:.5rem;}
.cx-tech .t b{display:block;color:var(--text);font-size:.92rem;margin-bottom:.25rem;}
.cx-tech .t span{font-size:.83rem;color:var(--text-2);line-height:1.5;}

/* tiered memory animation */
.cx-tiers{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:.6rem;}
.cx-tier{display:flex;align-items:flex-start;gap:1rem;border:1px solid var(--border);border-radius:12px;padding:.85rem 1rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.cx-tiers.go .cx-tier{opacity:1;transform:none;}
.cx-tiers.go .cx-tier:nth-child(1){transition-delay:.1s}
.cx-tiers.go .cx-tier:nth-child(2){transition-delay:.4s}
.cx-tiers.go .cx-tier:nth-child(3){transition-delay:.7s}
.cx-tier .lvl{flex:none;width:34px;height:34px;border-radius:8px;background:var(--grad);color:var(--accent-ink);font-family:var(--font-mono);font-weight:600;font-size:.78rem;display:flex;align-items:center;justify-content:center;}
.cx-tier .b{flex:1;}
.cx-tier .b b{display:block;color:var(--text);font-size:.92rem;}
.cx-tier .b span{font-size:.84rem;color:var(--text-2);}
.cx-tier.t3{border-color:var(--accent);}

/* compaction before/after bars */
.cx-comp{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:1rem;}
.cx-comp .row .lab{display:flex;justify-content:space-between;font-family:var(--font-mono);font-size:.78rem;margin-bottom:.35rem;}
.cx-comp .row .lab b{color:var(--text);}
.cx-comp .row .lab span{color:var(--accent);}
.cx-comp .track{height:18px;border-radius:6px;background:var(--surface-2);border:1px solid var(--border);overflow:hidden;}
.cx-comp .fill{height:100%;width:0;transition:width 1.1s ease;}
.cx-comp.go .fill{width:var(--w);}
.cx-comp .fill.before{background:var(--surface-2);border-right:2px solid var(--text-3);}
.cx-comp .fill.after{background:var(--grad);}

/* table */
.cx-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.cx-tab th,.cx-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.cx-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.cx-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}

@media (prefers-reduced-motion: reduce){
  .cx-win .slot,.cx-u .curve,.cx-u .dot,.cx-tiers .cx-tier,.cx-comp .fill{transition:none !important;}
  .cx-win.go .slot.gone{opacity:.25 !important;transform:none !important;}
  .cx-u .curve{stroke-dashoffset:0 !important;} .cx-u .dot{opacity:1 !important;}
  .cx-tiers .cx-tier{opacity:1 !important;transform:none !important;}
  .cx-comp .fill{width:var(--w) !important;}
}
</style>

Talk to an AI assistant long enough and you will feel it happen. Something you told it near the start, your name, a constraint, a decision you both agreed on, quietly stops being true for it. It contradicts itself. It asks for a thing you already gave it. It forgets.

This is not the model being dumb. It is the model hitting a wall that every one of them has: a fixed-size working memory, and a hard edge to it. Everything the model can "see" at once, your message, its own past replies, any documents, the instructions, all of it has to fit inside one bounded space called the **context window**. Fill it up, and something has to go. And here is the twist that surprised even the researchers: the model starts failing *before* the window is full, because it doesn't even read the whole thing evenly.

The craft of managing that bounded space, deciding what the model gets to see on every single call so the right things are present and the noise is gone, is called **context engineering**. It has quietly become one of the most important skills in building with AI, to the point that people say it *replaced* prompt engineering. Let me show you why forgetting happens, the study that proved the sneaky part, and the concrete moves teams use to fight it, with both everyday and developer examples.

## The context window: a desk, not a filing cabinet

Picture the model's memory not as a filing cabinet that stores everything, but as a **desk of a fixed size**. Whatever it's working on has to be laid out on that desk right now. The system instructions, your question, the conversation so far, any files it's referencing, all of it competes for the same finite surface. There is no "and also remember this from an hour ago" drawer. If it's not on the desk, the model cannot see it. Full stop.

So what happens when the desk fills? The oldest papers slide off the far edge to make room. That is the plain mechanical reason an assistant forgets what you said early in a long chat: those tokens literally scrolled off the desk to fit the newer ones.

<figure class="cx-fig">
  <div class="cx-win" id="win">
    <div class="box">
      <span class="limit">desk is full</span>
      <div class="slot sys">system instructions (pinned)</div>
      <div class="slot gone">turn 1: "my name is Sreenath, I use TypeScript"</div>
      <div class="slot gone">turn 2: "the budget is fixed, don't exceed it"</div>
      <div class="slot">turn 14: recent back-and-forth</div>
      <div class="slot">turn 15: your latest message</div>
    </div>
    <div class="cap">newest stays · oldest slides off to make room</div>
  </div>
  <figcaption>A context window is a desk of fixed size measured in tokens. When a long conversation overflows it, the earliest turns fall off the edge, which is exactly the stuff you set up at the start ("my name is...", "don't exceed the budget"). The model isn't ignoring you. That information is no longer on the desk.</figcaption>
</figure>

You might think: fine, just buy a bigger desk. Models *have* grown huge windows, hundreds of thousands of tokens, even millions. Problem solved? Not quite, and this is where it gets interesting.

## The sneaky part: lost in the middle

In 2023, a team of researchers (Nelson Liu and colleagues, from Stanford, Berkeley, and Samaya AI) ran a careful study with a title that says it all: **"Lost in the Middle."** They gave models a long context containing the one document that held the answer, and they slid that document around, sometimes near the front, sometimes buried in the middle, sometimes near the end, and measured how often the model found it.

The result is one of those findings you don't forget. Performance traced a **U-shape**. Models were sharp when the key fact sat near the *beginning* or the *end* of the context, and noticeably worse when it sat in the *middle*, even though the whole thing fit comfortably in the window. A bigger desk doesn't help if the model skims the middle of the page.

<figure class="cx-fig">
  <div class="cx-u" id="ucurve">
    <svg viewBox="0 0 480 220">
      <line class="grid-l" x1="50" y1="30" x2="50" y2="180"/>
      <line class="grid-l" x1="50" y1="180" x2="450" y2="180"/>
      <text class="axl" x="8" y="40">high</text>
      <text class="axl" x="10" y="178">low</text>
      <text class="axl" x="6" y="110" transform="rotate(-90 14,110)">accuracy</text>
      <path class="curve" d="M70,55 C150,70 190,165 250,165 C310,165 350,70 430,55"/>
      <circle class="dot" cx="70" cy="55" r="5"/>
      <circle class="dot" cx="250" cy="165" r="5"/>
      <circle class="dot" cx="430" cy="55" r="5"/>
      <text class="zone" x="80" y="200">beginning</text>
      <text class="zone" x="250" y="200">middle</text>
      <text class="zone" x="410" y="200">end</text>
    </svg>
  </div>
  <figcaption>The "lost in the middle" U-curve, redrawn. Put the crucial fact where the model actually reads best, the start or the end, and it thrives. Bury it dead-center among distractors and accuracy sags, no matter how large the window. Position is not neutral. Where a fact sits changes whether the model uses it.</figcaption>
</figure>

Put those two facts together and you have the whole motivation for context engineering:

1. The window is **finite**, so you cannot just pour everything in.
2. Even what fits is **not read evenly**, so more is often worse, extra noise in the middle actively hurts.

The goal, then, is not "give the model everything." It is "give the model the *right* things, in the *right* places, and nothing else." Curate the desk.

## What goes wrong if you don't

Before the fixes, feel the failure modes. Naively stuffing the window, or naively letting it overflow, produces predictable pain:

<figure class="cx-fig">
  <div class="cx-fail">
    <div class="row"><div class="ic">1</div><div class="t"><b>It forgets the setup.</b> Early constraints scroll off the desk. The assistant cheerfully violates a rule you gave it in turn two.</div></div>
    <div class="row"><div class="ic">2</div><div class="t"><b>It drowns in noise.</b> Twenty half-relevant documents crammed in, the one that mattered sits in the middle, and gets skimmed right past.</div></div>
    <div class="row"><div class="ic">3</div><div class="t"><b>It gets slow and expensive.</b> Every token on the desk is paid for and processed on every single call. A bloated context is a bigger bill and a slower reply, every turn.</div></div>
    <div class="row"><div class="ic">4</div><div class="t"><b>It distracts itself.</b> Old, stale, or contradictory turns still on the desk pull the model toward outdated answers. More context can mean worse answers.</div></div>
  </div>
  <figcaption>The counterintuitive lesson underneath all four: context is not free, and more of it is not better. Every token competes for attention and costs money. The skill is subtraction as much as addition.</figcaption>
</figure>

## The techniques: how we fight forgetting

Here are the real moves. None is exotic; together they're the toolkit. I'll give each a plain-language and a developer angle.

<figure class="cx-fig">
  <div class="cx-tech">
    <div class="t">
      <span class="k">summarize</span>
      <b>Compaction / summarization</b>
      <span>When history grows long, replace old turns with a tight summary that keeps the decisions and drops the chatter. The desk clears; the gist stays.</span>
    </div>
    <div class="t">
      <span class="k">retrieve</span>
      <b>Retrieval (bring back only what's needed)</b>
      <span>Instead of keeping everything on the desk, store it outside and fetch just the few relevant facts when a turn needs them. This is where embeddings earn their keep.</span>
    </div>
    <div class="t">
      <span class="k">prune</span>
      <b>Pruning / trimming</b>
      <span>Drop tool outputs, raw logs, and dead turns once they've served their purpose. Keep the conclusion, throw away the transcript that produced it.</span>
    </div>
    <div class="t">
      <span class="k">order</span>
      <b>Ordering (beat the U-curve)</b>
      <span>Put the most important material at the start and the end, never buried in the middle. Position it where the model actually reads well.</span>
    </div>
  </div>
  <figcaption>Four everyday moves: shrink it, fetch it, trim it, place it well. Summarization and pruning fight the finite-desk problem; retrieval sidesteps it entirely; ordering fights the lost-in-the-middle problem. Most real systems use all four together.</figcaption>
</figure>

Two of these deserve a real example, one general and one for the developers, because that's where it stops being theory.

**The general example, a long support chat.** You've been troubleshooting your internet for forty messages. A well-engineered assistant doesn't keep all forty on the desk. It quietly *compacts*: "User's router is model X, already tried restarting and cable-swapping, both failed, ISP confirmed no outage." Three lines now stand in for forty turns. The desk has room, the key facts survive, and it stops asking you to restart the router for the third time.

**The developer example, a coding agent on a big repo.** The agent cannot fit your whole codebase on the desk, not even close. So it doesn't try. It *retrieves*: embeds your files, and when a task touches authentication, it pulls in just `auth.ts` and its tests, leaving the other 900 files out on the shelf at zero cost. When it finishes a sub-task, it *prunes* the noisy tool output and keeps only the outcome ("tests pass"). And it *orders* the prompt so the task and the key file sit at the edges, not lost among boilerplate. Same finite desk, used with discipline.

## The production pattern: three tiers of memory

Put it all together and the shape that most serious 2026 systems land on is a **three-tier memory**, which is really just "the desk, plus two kinds of storage behind it."

<figure class="cx-fig">
  <div class="cx-tiers" id="tiers">
    <div class="cx-tier">
      <div class="lvl">1</div>
      <div class="b"><b>Working memory (on the desk)</b><span>The live context window: the current turn, full and lossless. Fast, precise, and small. This is the only thing the model can directly see.</span></div>
    </div>
    <div class="cx-tier">
      <div class="lvl">2</div>
      <div class="b"><b>Compressed session memory</b><span>As the session grows, older stretches get summarized into compact form. Continuity within the session, without keeping every raw turn on the desk.</span></div>
    </div>
    <div class="cx-tier t3">
      <div class="lvl">3</div>
      <div class="b"><b>External persistent store</b><span>Cross-session, long-term knowledge saved outside entirely (a vector store, a database). Retrieved back onto the desk only when relevant, at the start of a session or mid-task.</span></div>
    </div>
  </div>
  <figcaption>Tier 1 is the desk. Tier 2 is the notepad of summaries beside it. Tier 3 is the filing cabinet in the other room. The art is moving information between them at the right moments: compress before the desk overflows, save the durable facts before they're lost, and fetch them back only when they matter.</figcaption>
</figure>

The single hardest judgment in that whole picture is *when* to compact. Do it too early, mid-derivation, and you throw away detail the model still needs. Do it too late and the desk overflows and things fall off before you saved them. Good systems compact at natural seams, when a sub-task just finished, when the reasoning has clearly converged, not in the middle of a hard step. Compaction is a decision about timing, not just a size threshold.

## How much this actually saves

The payoff is not subtle. Compacting forty rambling turns into a three-line summary, or retrieving three relevant chunks instead of loading a whole knowledge base, cuts the tokens on the desk dramatically, which means faster replies, lower cost, *and* better answers, because the noise the model would have skimmed past in the middle is simply gone.

<figure class="cx-fig">
  <div class="cx-comp" id="comp">
    <div class="row">
      <div class="lab"><b>Naive: whole history + every doc on the desk</b><span>slow, costly, noisy</span></div>
      <div class="track"><div class="fill before" style="--w:92%"></div></div>
    </div>
    <div class="row">
      <div class="lab"><b>Engineered: summary + retrieved facts only</b><span>lean, cheap, sharp</span></div>
      <div class="track"><div class="fill after" style="--w:26%"></div></div>
    </div>
  </div>
  <figcaption>Tokens sitting on the desk, before and after context engineering, for the same task. The engineered version isn't just cheaper and faster. It's often *more accurate*, because it removed the middle-of-the-context noise the model would have gotten lost in. Less, placed well, beats more.</figcaption>
</figure>

## When to reach for which

A quick map, so you know which tool the situation calls for:

<figure class="cx-fig">
<table class="cx-tab">
<thead><tr><th>Situation</th><th>Reach for</th><th>Why</th></tr></thead>
<tbody>
<tr><td>Long conversation drifting</td><td>Compaction / summarization</td><td>Keep decisions, drop the chatter, free the desk</td></tr>
<tr><td>Huge knowledge base or codebase</td><td>Retrieval (embeddings)</td><td>Fetch only the few relevant pieces, leave the rest outside</td></tr>
<tr><td>Noisy tool outputs / logs piling up</td><td>Pruning</td><td>Keep the conclusion, discard the raw transcript</td></tr>
<tr><td>One critical instruction must land</td><td>Ordering</td><td>Put it at the start or end, never the middle</td></tr>
<tr><td>Facts must survive across sessions</td><td>External store (tier 3)</td><td>The desk is wiped each session; durable memory lives outside</td></tr>
</tbody>
</table>
  <figcaption>Most real systems combine several of these. A coding agent retrieves the right files, prunes stale output, compacts long sessions, and orders the prompt carefully, all at once. Context engineering is choosing the right mix for the moment.</figcaption>
</figure>

## The one idea to carry

An AI model forgets for two honest reasons: its working memory has a hard edge, and even inside that edge it reads the middle poorly. So the job was never to hand it *everything*. It was to hand it the *right* things, placed where it reads best, and to keep doing that as the conversation grows, compressing the past, fetching what's relevant, trimming the noise.

That is context engineering, and once you see it, you understand why the same model can feel brilliant in one product and forgetful in another. The model didn't change. The care taken with what sits on its desk did. The intelligence was never only in the model. A good share of it lives in the quiet discipline of deciding what it gets to see.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['win','ucurve','tiers','comp'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
