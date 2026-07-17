---
title: "Building an Agent to Sift Years of Messy Data Into a Plan"
date: 2026-07-17
excerpt: "A client hands you years of advertising data. Most of it is noise. You need a tool that reads all of it, keeps only what matters, and hands back a real plan. This is a walk through how I'd actually build that: the three approaches you'd weigh, why the obvious one is a trap, and the design I'd ship, with the honest trade-offs."
tags: [ai, agents, rag, architecture, use-case, explainer]
---

<style>
.sd-fig{margin:2.5rem 0;}
.sd-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* the problem: signal in noise */
.sd-noise{max-width:560px;margin:0 auto;display:flex;flex-wrap:wrap;gap:4px;justify-content:center;}
.sd-noise .d{width:13px;height:13px;border-radius:3px;background:var(--surface-2);border:1px solid var(--border);}
.sd-noise .d.sig{background:var(--grad);border-color:var(--accent);}

/* three approaches cards */
.sd-appr{display:flex;flex-direction:column;gap:.8rem;max-width:660px;margin:0 auto;}
.sd-a{border:1px solid var(--border);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.sd-appr.go .sd-a{opacity:1;transform:none;}
.sd-appr.go .sd-a:nth-child(1){transition-delay:.1s} .sd-appr.go .sd-a:nth-child(2){transition-delay:.3s} .sd-appr.go .sd-a:nth-child(3){transition-delay:.5s}
.sd-a .hd{display:flex;align-items:center;gap:.6rem;margin-bottom:.4rem;}
.sd-a .num{flex:none;width:26px;height:26px;border-radius:50%;background:var(--surface-2);border:1px solid var(--border-2);color:var(--text-2);font-family:var(--font-mono);font-weight:600;font-size:.8rem;display:flex;align-items:center;justify-content:center;}
.sd-a.win .num{background:var(--grad);color:var(--accent-ink);border-color:transparent;}
.sd-a .nm{font-family:var(--font-mono);font-size:.9rem;color:var(--text);font-weight:600;}
.sd-a.win .nm{color:var(--accent);}
.sd-a .verdict{font-family:var(--font-mono);font-size:.72rem;margin-left:auto;padding:.15rem .5rem;border-radius:999px;border:1px solid var(--border-2);color:var(--text-3);}
.sd-a.win .verdict{border-color:var(--accent);color:var(--accent);}
.sd-a p{font-size:.85rem;color:var(--text-2);margin:.2rem 0;line-height:1.5;}
.sd-a .pc{display:flex;gap:1.2rem;flex-wrap:wrap;margin-top:.5rem;font-size:.8rem;}
.sd-a .pro{color:var(--text-2);} .sd-a .pro b{color:var(--accent);}
.sd-a .con{color:var(--text-2);} .sd-a .con b{color:var(--text-3);}

/* dump-it-all failure viz */
.sd-dump{max-width:520px;margin:0 auto;}
.sd-dump .win{border:1px solid var(--border);border-radius:10px;padding:.6rem;background:var(--surface-2);display:flex;flex-direction:column;gap:3px;}
.sd-dump .row{height:12px;border-radius:2px;background:var(--surface);display:flex;align-items:center;padding-left:.4rem;font-family:var(--font-mono);font-size:.6rem;color:var(--text-3);}
.sd-dump .row.edge{background:var(--grad);color:var(--accent-ink);}
.sd-dump .row.mid{opacity:.5;}
.sd-dump .cap{text-align:center;font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);margin-top:.5rem;}

/* cost/latency bars */
.sd-cost{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:1rem;}
.sd-cost .row .lab{display:flex;justify-content:space-between;font-family:var(--font-mono);font-size:.78rem;margin-bottom:.3rem;}
.sd-cost .row .lab b{color:var(--text);} .sd-cost .row .lab span{color:var(--accent);}
.sd-cost .track{height:18px;border-radius:6px;background:var(--surface-2);border:1px solid var(--border);overflow:hidden;}
.sd-cost .fill{height:100%;width:0;transition:width 1.1s cubic-bezier(.2,.7,.2,1);}
.sd-cost.go .fill{width:var(--w);}
.sd-cost .fill.big{background:var(--surface-2);border-right:2px solid var(--text-3);}
.sd-cost .fill.small{background:var(--grad);}

/* the pipeline */
.sd-pipe{max-width:660px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.sd-pstep{display:flex;align-items:center;gap:.9rem;border:1px solid var(--border);border-radius:10px;padding:.65rem .9rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.sd-pipe.go .sd-pstep{opacity:1;transform:none;}
.sd-pipe.go .sd-pstep:nth-child(1){transition-delay:.1s} .sd-pipe.go .sd-pstep:nth-child(2){transition-delay:.28s}
.sd-pipe.go .sd-pstep:nth-child(3){transition-delay:.46s} .sd-pipe.go .sd-pstep:nth-child(4){transition-delay:.64s}
.sd-pipe.go .sd-pstep:nth-child(5){transition-delay:.82s}
.sd-pstep .n{flex:none;width:28px;height:28px;border-radius:50%;background:var(--grad);color:var(--accent-ink);font-family:var(--font-mono);font-weight:600;font-size:.8rem;display:flex;align-items:center;justify-content:center;}
.sd-pstep .t{font-size:.86rem;color:var(--text-2);} .sd-pstep .t b{color:var(--text);}
.sd-pstep .funnel{flex:none;font-family:var(--font-mono);font-size:.7rem;color:var(--accent);}

/* funnel shrink viz */
.sd-funnel{max-width:520px;margin:0 auto;display:flex;flex-direction:column;gap:.4rem;align-items:center;}
.sd-fbar{height:26px;border-radius:6px;background:var(--grad);display:flex;align-items:center;justify-content:center;font-family:var(--font-mono);font-size:.72rem;color:var(--accent-ink);width:0;transition:width 1s cubic-bezier(.2,.7,.2,1);}
.sd-funnel.go .sd-fbar{width:var(--w);}
.sd-funnel .lbl{font-family:var(--font-mono);font-size:.68rem;color:var(--text-3);}

/* table */
.sd-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.sd-tab th,.sd-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.sd-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.sd-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}

@media (prefers-reduced-motion: reduce){
  .sd-appr .sd-a,.sd-cost .fill,.sd-pipe .sd-pstep,.sd-funnel .sd-fbar{transition:none !important;}
  .sd-appr .sd-a,.sd-pipe .sd-pstep{opacity:1 !important;transform:none !important;}
  .sd-cost .fill{width:var(--w) !important;} .sd-funnel .sd-fbar{width:var(--w) !important;}
}
</style>

Here's a problem that lands on a lot of engineers' desks, in one shape or another. A client shows up with **years of data**: advertising campaigns, spend records, creative performance, audience reports, spreadsheets, exports, dashboards nobody has opened since 2022. It's a mountain. And the ask is deceptively simple: *"Go through all of it, figure out what actually matters, throw away the noise, and give us a plan."*

Most of that data is noise. Genuinely. A dead campaign from three years ago, a duplicated export, a metric nobody acts on. The signal, the stuff that should shape the next plan, is a small fraction buried in the pile.

<figure class="sd-fig">
  <div class="sd-noise" id="noise"></div>
  <figcaption>The shape of the problem: a sea of data, and the few highlighted squares are the parts that actually matter. Your job is to find those, discard the rest, and turn them into a plan. The hard part isn't the plan. It's the finding, at a scale no human wants to read through by hand.</figcaption>
</figure>

I want to walk through how I'd actually build an agentic tool for this. Not the buzzword version, the real engineering decisions. And the interesting part is the fork in the road early on: there are three obvious ways to approach it, the most tempting one is a trap, and understanding *why* it's a trap is what leads you to the design worth shipping. Let me think it through out loud.

## First, what does "good" even look like?

Before choosing an approach, pin down what success means, because it shapes everything:

- **Relevant, not just recent.** A great campaign from two years ago might matter more than a mediocre one from last month. "Relevant" is a judgment call, not a date filter.
- **Traceable.** When the plan says "double down on video," the client will ask *why*. Every conclusion must point back to the data that justified it. No unsourced claims.
- **Affordable and repeatable.** Years of data is a lot of tokens. If the tool costs a fortune or takes an hour per run, it won't get used.
- **Honest about gaps.** If the data doesn't support a recommendation, the tool should say so, not invent one.

Hold those four in mind. They're the scorecard we'll judge each approach against.

## The three approaches on the table

<figure class="sd-fig">
  <div class="sd-appr" id="appr">
    <div class="sd-a">
      <div class="hd"><div class="num">1</div><div class="nm">Dump it all into the LLM</div><div class="verdict">the trap</div></div>
      <p>Modern models have huge context windows. So just feed <em>everything</em> in and ask for a plan. Tempting because it's the least code.</p>
      <div class="pc"><span class="pro"><b>+</b> simplest to build</span><span class="con"><b>&minus;</b> expensive, unreliable, misses the middle</span></div>
    </div>
    <div class="sd-a">
      <div class="hd"><div class="num">2</div><div class="nm">Pure RAG retrieval</div><div class="verdict">close, not enough</div></div>
      <p>Index everything, and when you need a plan, retrieve the most relevant chunks and reason over just those. Cheap and focused.</p>
      <div class="pc"><span class="pro"><b>+</b> cheap, scalable, sourced</span><span class="con"><b>&minus;</b> retrieval alone can't judge or plan in passes</span></div>
    </div>
    <div class="sd-a win">
      <div class="hd"><div class="num">3</div><div class="nm">Agentic filter-then-plan pipeline</div><div class="verdict">what I'd ship</div></div>
      <p>An agent that retrieves in passes, <em>judges</em> what's relevant, discards the rest, and only then plans, checking its own work as it goes.</p>
      <div class="pc"><span class="pro"><b>+</b> handles judgment, multi-step, traceable</span><span class="con"><b>&minus;</b> more moving parts, higher cost than plain RAG</span></div>
    </div>
  </div>
  <figcaption>Three ways up the mountain. They look like "simple / medium / complex," but that framing is misleading. The right lens is: which one actually meets the four success criteria? Let me take them in order, because you learn the most from why the first one fails.</figcaption>
</figure>

## Approach 1: just dump it all in. Why it's a trap.

This is where most people start, and it's seductive. Context windows are enormous now, so why not pour in years of data and let the model sort it out? One prompt, no pipeline, done.

Here's why it falls apart, and it's not just cost.

**The "lost in the middle" problem.** This is the killer. Models don't read a giant context evenly. They pay strong attention to the *start* and *end* of what you give them, and they get noticeably worse at using information stuck in the *middle*. Research puts real numbers on it: when the fact you need sits in the middle of a long context, accuracy can drop by 15 to 20% versus when it's near the top.

<figure class="sd-fig">
  <div class="sd-dump">
    <div class="win">
      <div class="row edge">read well (start)</div>
      <div class="row mid">skimmed</div>
      <div class="row mid">skimmed</div>
      <div class="row mid">the key 2-year-old campaign... missed</div>
      <div class="row mid">skimmed</div>
      <div class="row mid">skimmed</div>
      <div class="row edge">read well (end)</div>
    </div>
    <div class="cap">years of data crammed in, the model attends to the edges, skims the middle</div>
  </div>
  <figcaption>Pour everything into one giant prompt and the model reads it like a bored student skimming a long chapter: sharp at the start and end, fuzzy in the middle. Your crucial two-year-old insight is probably somewhere in that fuzzy middle, and it gets missed. More context did not mean more understanding.</figcaption>
</figure>

**And the cost is brutal.** You pay per token, every single run. Stuffing years of data into every query is like rebuilding the whole library each time you want to look up one fact. Compared to retrieving only what's relevant, the full-dump approach can cost roughly *10 to 15 times more per query and take far longer*. Against our scorecard: it's not reliable (misses the middle), not cheap, and not repeatable. It fails three of four. Scratch it.

The honest exception: if the customer's data were *small*, say it comfortably fit in a prompt with room to spare, dumping it in is genuinely fine and you shouldn't over-engineer. But "years of ad data" is not that. This is the wrong tool for this size.

## Approach 2: pure RAG. Better, but it can't think in steps.

So if you can't feed everything in, feed in only what's *relevant*. That's Retrieval-Augmented Generation, RAG. You index all the data once (turning it into searchable vectors by meaning, using embeddings), and when you need a plan, you retrieve just the most relevant pieces and reason over those.

This is a big leap forward and hits most of our criteria: it's cheap (retrieving relevant chunks cuts token use by around 90% versus the full dump), it's scalable, and because every answer is tied to retrieved documents, it's *traceable*. Good.

But here's where plain RAG runs out of road for *this* task. RAG does **one retrieval, then answers.** Our problem isn't a single-answer question, it's a *judgment-and-planning* job with several phases:

- First decide *what topics even matter* across years of data.
- Then, for each, dig deeper and separate signal from noise.
- Then *judge* whether the evidence is strong enough to act on.
- Then synthesize all of that into a coherent plan.

One retrieval pass can't do that. It grabs some chunks and produces some text, but it can't *loop*, can't say "these results are thin, let me search differently," can't first filter and then plan. It has no agency. It retrieves; it doesn't reason across steps.

Plain RAG is the right *engine*. It's just missing a *driver*.

## Approach 3: the agentic pipeline. What I'd actually ship.

Here's the design. Take RAG as the engine, and put an **agent** in the driver's seat, something that can act in steps, judge results, and loop until the job's done. (If "agent" is fuzzy for you, it's an AI that directs its own process toward a goal, deciding its next move as it goes.) The whole thing becomes a funnel: start with the mountain, filter hard, and only plan on what survives.

<figure class="sd-fig">
  <div class="sd-pipe" id="pipe">
    <div class="sd-pstep"><div class="n">1</div><div class="t"><b>Ingest and index.</b> All the ad data gets chunked and embedded into a searchable store, once. Now anything is findable by meaning, not just keywords. <span class="funnel">everything in</span></div></div>
    <div class="sd-pstep"><div class="n">2</div><div class="t"><b>Map the territory.</b> The agent surveys what's there: which campaigns, channels, time periods, metrics exist. It builds a shortlist of themes worth investigating, instead of reading blindly. <span class="funnel">↓ narrow</span></div></div>
    <div class="sd-pstep"><div class="n">3</div><div class="t"><b>Retrieve and judge, in passes.</b> For each theme, it pulls the relevant data and <em>decides</em>: is this signal or noise? Strong evidence or a fluke? Thin results? It searches again, differently. This looping judgment is the agentic core. <span class="funnel">↓ filter</span></div></div>
    <div class="sd-pstep"><div class="n">4</div><div class="t"><b>Keep only what survives.</b> What passed the judgment gets distilled into a compact, sourced set of findings. The noise is dropped. This is the "keep only relevant" the client asked for. <span class="funnel">↓ distill</span></div></div>
    <div class="sd-pstep"><div class="n">5</div><div class="t"><b>Plan from the survivors.</b> Only now does it write the plan, reasoning over the small, clean, high-signal set, with every recommendation citing the finding behind it. <span class="funnel">the plan out</span></div></div>
  </div>
  <figcaption>Five stages, one shape: a funnel. Everything goes in; a lot gets filtered by the agent's judgment; only the high-signal survivors reach the planning step. The plan is written over a small, clean set, exactly the situation where models are sharp and honest, not the giant messy context where they get lost.</figcaption>
</figure>

The magic is stage 3. That's where "agentic" earns its name. A plain pipeline would retrieve once and hope. This agent *evaluates* what it found, notices when results are weak, and retrieves again with a better query, the same way a good analyst circles back when the data looks thin. It filters *before* it plans, so the planning step never drowns.

Watch what the funnel does to the volume, which is the whole point:

<figure class="sd-fig">
  <div class="sd-funnel" id="funnel">
    <div class="sd-fbar" style="--w:100%">years of raw data</div>
    <div class="lbl">↓ map the themes</div>
    <div class="sd-fbar" style="--w:55%">candidate themes</div>
    <div class="lbl">↓ retrieve &amp; judge</div>
    <div class="sd-fbar" style="--w:25%">relevant, verified findings</div>
    <div class="lbl">↓ distill</div>
    <div class="sd-fbar" style="--w:12%">the clean set the plan is built on</div>
  </div>
  <figcaption>From a mountain to a handful. By the time the model writes the plan, it's reasoning over a small, curated, fully-sourced set, not years of noise. This is why the output is both trustworthy and cheap: the expensive, error-prone "read everything" work was replaced by cheap retrieval plus targeted judgment.</figcaption>
</figure>

## Why this one wins, measured against the scorecard

Let's be disciplined and score all three against the four criteria we set at the start. This is the actual decision, not a vibe.

<figure class="sd-fig">
<table class="sd-tab">
<thead><tr><th>Criterion</th><th>Dump it all</th><th>Pure RAG</th><th>Agentic pipeline</th></tr></thead>
<tbody>
<tr><td>Relevant, not just recent</td><td>Misses the middle</td><td>Retrieves relevant, but one-shot</td><td>Judges relevance in passes ✓</td></tr>
<tr><td>Traceable to sources</td><td>Hard, it's a blob</td><td>Yes ✓</td><td>Yes, per finding ✓</td></tr>
<tr><td>Affordable / repeatable</td><td>No, 10x+ cost</td><td>Very ✓</td><td>Costs more than RAG, still fine ✓</td></tr>
<tr><td>Honest about gaps</td><td>Tends to hallucinate</td><td>Better</td><td>Can flag thin evidence ✓</td></tr>
<tr><td>Multi-step judgment</td><td>No</td><td>No</td><td>Yes, the whole point ✓</td></tr>
</tbody>
</table>
  <figcaption>The dump-it-all approach fails on cost, reliability, and traceability. Pure RAG is genuinely good and wins on simplicity, if the task were a single lookup, I'd stop there. But our task needs multi-step judgment, so the agentic pipeline is the only one that clears every bar. Note it isn't "best" in the abstract; it's best <em>for this shape of problem.</em></figcaption>
</figure>

## The honest cost of the choice

I'm not going to pretend the agentic approach is free. It's the right call here, but it comes with real trade-offs, and a good engineer names them:

<figure class="sd-fig">
  <div class="sd-cost" id="cost">
    <div class="row"><div class="lab"><b>Complexity to build and maintain</b><span>higher</span></div><div class="track"><div class="fill big" style="--w:85%"></div></div></div>
    <div class="row"><div class="lab"><b>Cost per run vs plain RAG</b><span>~10x of RAG (but tiny vs dump-it-all)</span></div><div class="track"><div class="fill big" style="--w:40%"></div></div></div>
    <div class="row"><div class="lab"><b>Reliability &amp; traceability of the output</b><span>much higher</span></div><div class="track"><div class="fill small" style="--w:95%"></div></div></div>
  </div>
  <figcaption>The trade you're making: more moving parts and more cost-per-run than plain RAG, in exchange for the judgment and traceability the task actually demands. It's still a fraction of the dump-it-all cost. The rule of thumb: don't reach for the agent unless the task genuinely needs multi-step judgment. This one does. A simpler task wouldn't, and then plain RAG (or even the dump) would be the smarter, cheaper call.</figcaption>
</figure>

There are also the unglamorous realities you'd plan for: **guardrails** so the agent can't loop forever running up a bill (a hard cap on retrieval passes), **a human check** on the final plan before it reaches the client (it's a draft from a sharp analyst, not gospel), and **observability** so when the client asks "why did it recommend this?", you can show the exact path from data to conclusion. Those aren't optional extras; they're what makes it trustworthy enough to actually deploy.

## The takeaway

The instinct with a mountain of data is to throw the whole mountain at a big model and hope. That's the one approach almost guaranteed to disappoint: expensive, and blind to whatever's buried in the middle. The better instinct is a funnel. Index everything so it's findable, then put an agent in charge of *retrieving in passes, judging what's relevant, discarding the noise, and only then planning* over the clean set that survives.

And the real lesson underneath the ad-data example is a way of thinking, not a recipe. When you face any "make sense of a huge pile" problem, don't ask "which fancy technique should I use." Ask "what does good actually look like here," write down the criteria, and let the *task* pick the approach. Sometimes that's a humble one-shot RAG. Sometimes, when real judgment across many steps is needed, it's an agent. The skill isn't reaching for the most powerful tool. It's reaching for the one the problem actually calls for, and being able to say exactly why.

<script>
(function(){
  // paint the noise/signal grid
  var grid = document.getElementById('noise');
  if(grid){
    var signal = {7:1,23:1,41:1,58:1,66:1};
    for(var i=0;i<84;i++){
      var d = document.createElement('span');
      d.className = 'd' + (signal[i] ? ' sig' : '');
      grid.appendChild(d);
    }
  }
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['appr','pipe','funnel','cost'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
