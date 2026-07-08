---
title: "Fine-tuning vs RAG vs Prompting: How to Actually Make an AI Model Yours"
date: 2026-07-08
excerpt: "You want the AI to know your company's data, or sound like your brand, or stop getting one thing wrong. There are three ways to do it, and people constantly pick the wrong one, spending months fine-tuning when a good prompt would have done, or prompting forever at a problem only retrieval can fix. Here is the one mental model that tells you which lever to pull, and when."
tags: [ai, rag, fine-tuning, prompting, llm, explainer]
---

<style>
.ft-fig{margin:2.5rem 0;}
.ft-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* the three levers */
.ft-three{display:grid;grid-template-columns:repeat(3,1fr);gap:.7rem;max-width:720px;margin:0 auto;}
@media(max-width:640px){.ft-three{grid-template-columns:1fr;}}
.ft-three .c{border:1px solid var(--border-2);border-radius:12px;padding:1rem;background:var(--surface);text-align:center;}
.ft-three .c b{display:block;color:var(--text);font-size:.95rem;margin-bottom:.3rem;}
.ft-three .c .what{font-family:var(--font-mono);font-size:.72rem;color:var(--accent);margin-bottom:.5rem;}
.ft-three .c span{font-size:.83rem;color:var(--text-2);line-height:1.5;}

/* the core mental model: see vs behave */
.ft-model{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:.7rem;}
.ft-mrow{display:flex;align-items:center;gap:1rem;border:1px solid var(--border);border-radius:12px;padding:.9rem 1.1rem;background:var(--surface);}
.ft-mrow .k{flex:none;width:110px;font-family:var(--font-mono);font-size:.82rem;font-weight:600;}
.ft-mrow.see{border-color:var(--accent);} .ft-mrow.see .k{color:var(--accent);}
.ft-mrow.behave{border-color:var(--border-2);} .ft-mrow.behave .k{color:var(--text);}
.ft-mrow .d{font-size:.88rem;color:var(--text-2);} .ft-mrow .d b{color:var(--text);}

/* sequential flow */
.ft-seq{max-width:640px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.ft-sstep{display:flex;align-items:center;gap:.9rem;border:1px solid var(--border);border-radius:10px;padding:.7rem .9rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.ft-seq.go .ft-sstep{opacity:1;transform:none;}
.ft-seq.go .ft-sstep:nth-child(1){transition-delay:.1s}
.ft-seq.go .ft-sstep:nth-child(2){transition-delay:.45s}
.ft-seq.go .ft-sstep:nth-child(3){transition-delay:.8s}
.ft-sstep .n{flex:none;width:28px;height:28px;border-radius:50%;background:var(--grad);color:var(--accent-ink);font-family:var(--font-mono);font-weight:600;font-size:.82rem;display:flex;align-items:center;justify-content:center;}
.ft-sstep .t{font-size:.87rem;color:var(--text-2);} .ft-sstep .t b{color:var(--text);}
.ft-sstep.last{border-color:var(--accent);}

/* decision tree */
.ft-tree{max-width:620px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.ft-q{border:1px solid var(--border);border-radius:10px;padding:.7rem .9rem;background:var(--surface);opacity:0;transform:translateY(6px);transition:opacity .45s ease,transform .45s ease;}
.ft-tree.go .ft-q{opacity:1;transform:none;}
.ft-tree.go .ft-q:nth-child(1){transition-delay:.1s} .ft-tree.go .ft-q:nth-child(2){transition-delay:.3s}
.ft-tree.go .ft-q:nth-child(3){transition-delay:.5s} .ft-tree.go .ft-q:nth-child(4){transition-delay:.7s}
.ft-q .ask{font-size:.88rem;color:var(--text-2);} .ft-q .ask b{color:var(--text);}
.ft-q .ans{font-family:var(--font-mono);font-size:.76rem;color:var(--accent);margin-top:.25rem;}

/* cost bars (LoRA vs full) */
.ft-cost{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:1rem;}
.ft-cost .row .lab{display:flex;justify-content:space-between;font-family:var(--font-mono);font-size:.78rem;margin-bottom:.3rem;}
.ft-cost .row .lab b{color:var(--text);} .ft-cost .row .lab span{color:var(--accent);}
.ft-cost .track{height:18px;border-radius:6px;background:var(--surface-2);border:1px solid var(--border);overflow:hidden;}
.ft-cost .fill{height:100%;width:0;transition:width 1.1s cubic-bezier(.2,.7,.2,1);}
.ft-cost.go .fill{width:var(--w);}
.ft-cost .fill.full{background:var(--surface-2);border-right:2px solid var(--text-3);}
.ft-cost .fill.lora{background:var(--grad);}

/* lora illustration */
.ft-lora{max-width:560px;margin:0 auto;display:flex;align-items:center;justify-content:center;gap:1rem;flex-wrap:wrap;}
.ft-lora .big{border:1px solid var(--border-2);border-radius:12px;padding:1.2rem 1.4rem;text-align:center;background:var(--surface-2);}
.ft-lora .big b{display:block;color:var(--text-2);font-family:var(--font-mono);font-size:.85rem;}
.ft-lora .big span{font-size:.72rem;color:var(--text-3);}
.ft-lora .plus{font-family:var(--font-mono);color:var(--text-3);font-size:1.3rem;}
.ft-lora .small{border:1px solid var(--accent);border-radius:12px;padding:1.2rem 1.4rem;text-align:center;background:var(--surface);}
.ft-lora .small b{display:block;color:var(--accent);font-family:var(--font-mono);font-size:.85rem;}
.ft-lora .small span{font-size:.72rem;color:var(--text-2);}

/* hybrid stack */
.ft-hybrid{max-width:460px;margin:0 auto;display:flex;flex-direction:column;gap:0;}
.ft-hlayer{border:1px solid var(--border-2);padding:.75rem 1rem;text-align:center;font-family:var(--font-mono);font-size:.8rem;color:var(--text-2);background:var(--surface);}
.ft-hlayer:first-child{border-radius:12px 12px 0 0;}
.ft-hlayer:last-child{border-radius:0 0 12px 12px;}
.ft-hlayer.acc{border-color:var(--accent);color:var(--accent);}
.ft-hlayer small{display:block;color:var(--text-3);font-size:.68rem;margin-top:.15rem;}

/* table */
.ft-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.ft-tab th,.ft-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.ft-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.ft-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}

@media (prefers-reduced-motion: reduce){
  .ft-seq .ft-sstep,.ft-tree .ft-q,.ft-cost .fill{transition:none !important;}
  .ft-seq .ft-sstep{opacity:1 !important;transform:none !important;}
  .ft-tree .ft-q{opacity:1 !important;transform:none !important;}
  .ft-cost .fill{width:var(--w) !important;}
}
</style>

Say you want an AI to work for *your* situation. Maybe it should know your company's internal docs. Maybe it should always answer in your brand's voice. Maybe it just keeps getting one specific thing wrong and you want that fixed. You've got a real goal, and now you hit the fork that trips up almost everyone: **do you prompt it better, bolt on retrieval (RAG), or fine-tune the model?**

People get this wrong constantly, and expensively. I've watched teams spend months and serious money fine-tuning a model to "know" their data, when a bit of retrieval would have done it in an afternoon, and done it *better*. I've watched others prompt endlessly at a problem that no prompt could ever solve. The three tools look interchangeable from a distance. They are not. Each fixes a different *kind* of problem, and once you can tell which kind you have, the choice becomes obvious.

This post is the map. If you've read my earlier ones on RAG and on prompting methods, this is where they click into a single decision. Let me give you the one mental model that settles it, then the exact signals that point to each tool.

## The three levers, quickly

<figure class="ft-fig">
  <div class="ft-three">
    <div class="c">
      <div class="what">cheapest, first</div>
      <b>Prompting</b>
      <span>Ask better. Use the model's existing knowledge, shape the request with instructions, examples, and format. Nothing about the model changes.</span>
    </div>
    <div class="c">
      <div class="what">for facts &amp; data</div>
      <b>RAG</b>
      <span>Retrieve relevant documents at question-time and hand them to the model along with the question. It reads your data fresh, every time.</span>
    </div>
    <div class="c">
      <div class="what">for behavior</div>
      <b>Fine-tuning</b>
      <span>Actually retrain the model's weights on your examples, so a new tendency is baked in permanently. Changes the model itself.</span>
    </div>
  </div>
  <figcaption>Three different tools for three different jobs. The trap is treating them as "small, medium, large versions of the same thing." They're not a size ladder, they're a toolbox. Using the wrong one doesn't just cost more; it often can't solve your problem at all.</figcaption>
</figure>

## The one mental model that decides everything

Here it is, the single sentence that untangles the whole thing. Tattoo it on your brain:

**RAG changes what the model can *see*. Fine-tuning changes how the model *behaves*. Prompting works with what it already *knows*.**

<figure class="ft-fig">
  <div class="ft-model">
    <div class="ft-mrow see">
      <div class="k">RAG</div>
      <div class="d"><b>Changes what it SEES.</b> Like handing someone the exact reference document right before you ask them a question. Their knowledge is now current, sourced, swappable.</div>
    </div>
    <div class="ft-mrow behave">
      <div class="k">Fine-tuning</div>
      <div class="d"><b>Changes how it BEHAVES.</b> Like sending someone to months of training so a new habit becomes automatic. It's now part of who they are, every time.</div>
    </div>
    <div class="ft-mrow behave" style="border-style:dashed">
      <div class="k">Prompting</div>
      <div class="d"><b>Works with what it KNOWS.</b> Like asking a knowledgeable colleague a clear, well-framed question. No new info, no retraining, just a better ask.</div>
    </div>
  </div>
  <figcaption>The whole decision lives here. Got a knowledge or facts problem (it doesn't know your data, or the data keeps changing)? That's a "what it sees" problem, RAG. Got a behavior problem (wrong tone, wrong format, a consistent bad habit)? That's a "how it behaves" problem, fine-tuning. Neither? Just prompt.</figcaption>
</figure>

This immediately kills the most common mistake. People try to **fine-tune a model to teach it facts** ("train it on our docs so it knows them"). But facts change, and baking them into weights makes them stale the moment a doc updates, plus the model can't tell you *which* document an answer came from. Facts are a "what it sees" problem. That's RAG's job, not fine-tuning's. Fine-tuning is for *behavior*, not *knowledge*.

## The rule: prompt, then RAG, then fine-tune

Because the three cost wildly different amounts of effort, there's a natural order. Always climb it in sequence, and stop the moment your problem is solved:

<figure class="ft-fig">
  <div class="ft-seq" id="seq">
    <div class="ft-sstep"><div class="n">1</div><div class="t"><b>Prompt first.</b> It's free and instant. A surprising amount of "we need to fine-tune" turns out to be "we needed a clearer prompt." Try this before anything else.</div></div>
    <div class="ft-sstep"><div class="n">2</div><div class="t"><b>Then RAG.</b> If the gap is that the model doesn't know your data (or the data changes), add retrieval. This is the default for most real production systems in 2026: prompting + RAG.</div></div>
    <div class="ft-sstep last"><div class="n">3</div><div class="t"><b>Fine-tune only if neither worked.</b> After prompting and RAG, if there's a <em>specific behavior</em> they still can't fix, then fine-tune, for that behavior, not for facts.</div></div>
  </div>
  <figcaption>The sequential rule. Each step up costs more time, money, and expertise, so you only climb when the cheaper tool genuinely can't solve it. Most teams never need step 3. The ones who jump straight to it usually regret it.</figcaption>
</figure>

## The decision tree, in four questions

Want it even more concrete? Ask yourself these, in order, and stop at the first "yes":

<figure class="ft-fig">
  <div class="ft-tree" id="tree">
    <div class="ft-q"><div class="ask"><b>Does the model already know this?</b> (general knowledge, common tasks)</div><div class="ans">yes → just PROMPT. Done.</div></div>
    <div class="ft-q"><div class="ask"><b>Is it about facts / data, especially data that changes or needs sources?</b></div><div class="ans">yes → RAG. (Never fine-tune for this.)</div></div>
    <div class="ft-q"><div class="ask"><b>Is it about consistent style, tone, or a rigid output format that prompting keeps drifting on?</b></div><div class="ans">yes → FINE-TUNE (after trying strong few-shot prompting first).</div></div>
    <div class="ft-q"><div class="ask"><b>Do you need a small, cheap model to match a big expensive one on one narrow task?</b></div><div class="ans">yes → FINE-TUNE (distillation).</div></div>
  </div>
  <figcaption>Four questions, in order. Notice fine-tuning only wins in two narrow cases: locking in a stubborn behavior, or making a small model punch above its weight on one task. Everything about <em>knowledge</em> routes to RAG or prompting. This is the whole framework on one screen.</figcaption>
</figure>

## Wait, isn't fine-tuning insanely expensive?

It used to be, and this is why people fear it. Fully retraining even a 7-billion-parameter model means updating *every* weight, which needs enormous GPU memory, think tens of thousands of dollars for a single training run. That's the scary version, and for most people it's overkill.

The thing that changed the game is a technique called **LoRA** (and its memory-frugal cousin QLoRA). The idea is clever: instead of retraining all the model's millions of weights, you *freeze* them and train only a tiny pair of add-on matrices that nudge the model's behavior. You're adjusting a small steering attachment, not rebuilding the engine.

<figure class="ft-fig">
  <div class="ft-lora">
    <div class="big"><b>Frozen base model</b><span>millions of weights, untouched</span></div>
    <div class="plus">+</div>
    <div class="small"><b>LoRA adapter</b><span>tiny, the only part you train</span></div>
  </div>
  <figcaption>LoRA keeps the giant base model frozen and trains only a small add-on. To give a real sense of scale: on one layer of a 7B model, full fine-tuning adjusts about 45 million parameters; LoRA adjusts around 60 thousand, hundreds of times fewer. You get most of the benefit at a tiny fraction of the cost.</figcaption>
</figure>

The cost difference is not subtle. It's the difference between renting a data center and using a gaming GPU:

<figure class="ft-fig">
  <div class="ft-cost" id="cost">
    <div class="row"><div class="lab"><b>Full fine-tuning (7B model)</b><span>~$50,000 of H100 GPUs</span></div><div class="track"><div class="fill full" style="--w:100%"></div></div></div>
    <div class="row"><div class="lab"><b>QLoRA on the same model</b><span>~$1,500 gaming GPU (RTX 4090)</span></div><div class="track"><div class="fill lora" style="--w:3%"></div></div></div>
  </div>
  <figcaption>Same model, same goal, wildly different bill. LoRA-style methods cut memory needs by 10 to 20 times while keeping roughly 90 to 95% of full fine-tuning's quality. This is why fine-tuning went from "only big labs can afford it" to "a solo developer can do it on their desktop." It's real, and it's accessible now.</figcaption>
</figure>

Full fine-tuning still wins for the hardest, deepest cases (complex domains, very large training sets). But for most "teach it our house style" jobs, LoRA is the sensible, affordable default.

## The best answer is usually "all of the above"

Here's the grown-up truth: in real production systems, it's rarely one tool. The strongest setups **stack** them, each doing the job it's best at.

<figure class="ft-fig">
  <div class="ft-hybrid">
    <div class="ft-hlayer acc">A good prompt<small>clear instructions + format</small></div>
    <div class="ft-hlayer acc">wrapped around RAG<small>fresh facts, cited from your documents</small></div>
    <div class="ft-hlayer">running on a lightly fine-tuned model<small>locked-in house voice + cheaper to run</small></div>
  </div>
  <figcaption>The hybrid pattern most mature systems land on: RAG supplies the current, sourced facts; a light fine-tune supplies the consistent voice and format; a clear prompt ties it together. Each layer does what it's uniquely good at. Facts from retrieval, behavior from fine-tuning, framing from prompting.</figcaption>
</figure>

## The full comparison

<figure class="ft-fig">
<table class="ft-tab">
<thead><tr><th></th><th>Prompting</th><th>RAG</th><th>Fine-tuning</th></tr></thead>
<tbody>
<tr><td>Changes</td><td>Nothing (just the ask)</td><td>What the model sees</td><td>How the model behaves</td></tr>
<tr><td>Best for</td><td>What it already knows</td><td>Facts, your data, fresh info</td><td>Style, tone, format, narrow skills</td></tr>
<tr><td>Freshness</td><td>Frozen at training</td><td>Instant, swap a document</td><td>Goes stale, needs retraining</td></tr>
<tr><td>Cite sources?</td><td>No</td><td>Yes, points at the document</td><td>No, weights are opaque</td></tr>
<tr><td>Cost / effort</td><td>Lowest</td><td>Medium (build + iterate)</td><td>Highest (data + training)</td></tr>
<tr><td>Try it...</td><td>First</td><td>Second</td><td>Last, only if needed</td></tr>
</tbody>
</table>
  <figcaption>The whole decision on one card. The two rows that resolve most arguments: "cite sources?" (only RAG can, which is why audited/regulated use cases need it) and "freshness" (fine-tuning quietly rots as your data moves on). When in doubt, these two columns usually point the way.</figcaption>
</figure>

## The takeaway

Stop asking "which is best." None is best; each is best *at its own job*. The question that actually matters is: **what kind of problem do I have?**

If the model just needs to be asked better, that's prompting. If it needs to *see* information it doesn't have, especially changing information you need to trust and source, that's RAG. If it needs to *behave* differently in a stable, repeatable way, tone, format, a narrow skill, that's fine-tuning, and thanks to LoRA it's finally affordable. And when you build something real, you'll probably reach for all three, each doing the one thing it does best.

The teams that ship great AI aren't the ones who fine-tune the most. They're the ones who correctly diagnose which lever the problem actually needs, and reach for the cheapest one that solves it. Prompt first. RAG next. Fine-tune last, and only on purpose. That order will save you more time and money than any single technique ever could.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['seq','tree','cost'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
