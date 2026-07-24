---
title: "The Three AI Systems That Are Secretly the Same Design"
date: 2026-07-24
excerpt: "RAG chatbots, agents, and recommendation engines look like three different problems. Build a few of them and you notice they're the same machine wearing different clothes: cast a cheap wide net, spend real compute narrowing it, then apply the rules a model won't learn on its own. Here's the shared shape, why it keeps showing up, and what it tells you about where these systems actually break."
tags: [ai, system-design, rag, agents, recommendations, architecture]
---

<style>
.fn-fig{margin:2.5rem 0;}
.fn-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* the funnel */
.fn-funnel{max-width:560px;margin:0 auto;}
.fn-funnel svg{width:100%;height:auto;overflow:visible;}
.fn-band{fill:var(--surface);stroke:var(--border-2);stroke-width:1;}
.fn-band.a{stroke:var(--accent);}
.fn-fill{fill:var(--accent);opacity:0;transition:opacity .7s ease;}
.fn-funnel.go .fn-fill{opacity:.14;}
.fn-funnel.go .fn-fill.f2{transition-delay:.2s;opacity:.24;}
.fn-funnel.go .fn-fill.f3{transition-delay:.4s;opacity:.36;}
.fn-lab{fill:var(--text);font-family:var(--font-mono);font-size:11px;font-weight:600;}
.fn-sub{fill:var(--text-3);font-family:var(--font-mono);font-size:8.5px;}
.fn-count{fill:var(--accent);font-family:var(--font-mono);font-size:10px;text-anchor:end;}

/* three stages */
.fn-stages{max-width:680px;margin:0 auto;display:grid;grid-template-columns:repeat(3,1fr);gap:.7rem;}
@media(max-width:620px){.fn-stages{grid-template-columns:1fr;}}
.fn-stages .c{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);position:relative;overflow:hidden;opacity:0;transform:translateY(8px);transition:opacity .45s ease,transform .45s ease;}
.fn-stages.go .c{opacity:1;transform:none;}
.fn-stages.go .c:nth-child(1){transition-delay:.1s} .fn-stages.go .c:nth-child(2){transition-delay:.3s} .fn-stages.go .c:nth-child(3){transition-delay:.5s}
.fn-stages .c::before{content:"";position:absolute;left:0;top:0;bottom:0;width:3px;background:var(--accent);opacity:.7;}
.fn-stages .c .k{font-family:var(--font-mono);font-size:.68rem;letter-spacing:.08em;text-transform:uppercase;color:var(--accent);}
.fn-stages .c b{display:block;color:var(--text);font-size:.95rem;margin:.4rem 0 .3rem;}
.fn-stages .c span{font-size:.82rem;color:var(--text-2);line-height:1.5;}

/* the three-way mapping table */
.fn-map{max-width:760px;margin:0 auto;overflow-x:auto;}
.fn-mtab{width:100%;border-collapse:collapse;font-size:.86rem;min-width:560px;}
.fn-mtab th,.fn-mtab td{text-align:left;padding:.65rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.fn-mtab th{font-family:var(--font-mono);font-size:.7rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.fn-mtab td:first-child,.fn-mtab th:first-child{color:var(--text);font-weight:500;font-family:var(--font-mono);font-size:.78rem;}
.fn-mtab .stage{color:var(--accent);}

/* mental model box */
.fn-model{max-width:600px;margin:0 auto;text-align:center;border:1px solid var(--accent);border-radius:12px;padding:1.1rem 1.3rem;background:var(--surface);}
.fn-model b{color:var(--accent);} .fn-model p{font-size:.92rem;color:var(--text-2);line-height:1.6;margin:0;}

@media (prefers-reduced-motion: reduce){
  .fn-fill{opacity:.22!important;}
  .fn-stages .c{opacity:1!important;transform:none!important;}
}
</style>

I've built a RAG chatbot, wired up a tool-calling agent, and worked on ranking. For a long time I filed them under three separate headings in my head. Different problems, different diagrams, different words.

Then one afternoon I was drawing the RAG pipeline on a whiteboard, retrieve a bunch of chunks, rerank them down, hand the good ones to the model, and I realised I'd drawn the recommendation system from the week before. Same funnel. Different labels.

Once you see it you can't unsee it. RAG, agents, and recommendations are the same machine wearing three outfits. And the shape of that machine tells you exactly where each one is going to break.

## The shape

Here's the thing all three do. You have far too many candidates to look at carefully, millions of documents, millions of products, an open-ended space of possible next actions. You can't run your expensive, smart step on all of them. So you don't.

Instead you build a funnel.

<figure class="fn-fig">
<div class="fn-funnel fn-anim">
<svg viewBox="0 0 480 220" role="img" aria-label="A funnel: millions of candidates narrow through three stages to a final few.">
  <polygon class="fn-band a" points="20,10 460,10 380,70 100,70"></polygon>
  <polygon class="fn-fill f1" points="20,10 460,10 380,70 100,70"></polygon>
  <text class="fn-lab" x="40" y="35">1 · Cheap wide recall</text>
  <text class="fn-sub" x="40" y="52">get the good stuff into the pool</text>
  <text class="fn-count" x="450" y="35">millions</text>
  <polygon class="fn-band a" points="100,80 380,80 320,140 160,140"></polygon>
  <polygon class="fn-fill f2" points="100,80 380,80 320,140 160,140"></polygon>
  <text class="fn-lab" x="120" y="105">2 · Expensive precision</text>
  <text class="fn-sub" x="120" y="122">spend real compute, small set only</text>
  <text class="fn-count" x="370" y="105">dozens</text>
  <polygon class="fn-band a" points="160,150 320,150 285,205 195,205"></polygon>
  <polygon class="fn-fill f3" points="160,150 320,150 285,205 195,205"></polygon>
  <text class="fn-lab" x="180" y="175">3 · Policy</text>
  <text class="fn-sub" x="180" y="192">the rules a model won't learn</text>
  <text class="fn-count" x="312" y="175">a few</text>
</svg>
</div>
<figcaption>Cast a cheap wide net, narrow it with something expensive, then apply the rules on what's left.</figcaption>
</figure>

Three stages, and the trick is that the cost per item goes *up* as the number of items goes *down*.

<div class="fn-stages fn-anim">
  <div class="c">
    <span class="k">Stage 1</span>
    <b>Cheap, wide recall</b>
    <span>Something fast and approximate that pulls a big pool of maybe-relevant candidates from a huge set. Its only job is to not miss the good ones. Precision comes later.</span>
  </div>
  <div class="c">
    <span class="k">Stage 2</span>
    <b>Expensive, narrow precision</b>
    <span>Something slow and accurate that reads the small pool carefully and picks the best. Too costly to run on everything, which is exactly why stage 1 exists.</span>
  </div>
  <div class="c">
    <span class="k">Stage 3</span>
    <b>Policy and guardrails</b>
    <span>The business rules an accuracy metric will never give you: safety, diversity, freshness, permissions, a human sign-off. Applied last, on the handful that survived.</span>
  </div>
</div>

That's it. That's the whole design. Now watch it show up three times.

## The same funnel, three times

<div class="fn-map">
<table class="fn-mtab">
<thead>
<tr><th>Stage</th><th>RAG chatbot</th><th>Agent</th><th>Recommendations</th></tr>
</thead>
<tbody>
<tr><td>Cheap wide recall</td><td>Hybrid vector + keyword search over chunks</td><td>The planning step: what could I do next?</td><td>Two-tower retrieval, millions to hundreds</td></tr>
<tr><td>Expensive precision</td><td>Cross-encoder rerank, then the LLM answers</td><td>The reasoning step: read results, decide</td><td>Heavy ranking model scores each candidate</td></tr>
<tr><td>Policy layer</td><td>Groundedness check, PII filter, citations</td><td>Guardrails, human-in-the-loop on risky actions</td><td>Diversity, freshness, ads, dedupe</td></tr>
</tbody>
</table>
</div>

Look at the middle column. A **RAG chatbot** casts a wide net with cheap vector search (you can search millions of chunks in milliseconds), narrows it with an expensive cross-encoder reranker that reads the query and each chunk together, then hands a tidy handful to the model and checks the answer is grounded before it ships.

An **agent** does the same thing in a loop. The planning step is cheap-ish recall over "things I could do next." The actual reasoning, deciding which tool to call and reading what came back, is the expensive part, and you don't want to do more of it than you have to. Guardrails and human approval gates are the policy layer sitting on top of every risky step.

A **recommendation system** is the textbook version. Candidate generation retrieves a few hundred items out of millions using cheap nearest-neighbour lookups. A ranking model then scores just those few hundred with rich features. Then a re-ranking pass handles diversity and business rules, so you don't show someone ten near-identical things or bury the sponsored slot.

Three systems. One funnel. Cheap-wide, then expensive-narrow, then policy.

## Why it keeps happening

This isn't a coincidence, and it isn't a fashion. It falls out of a hard constraint: **your best step is too expensive to run on everything.**

A cross-encoder that reads a query and a document together is far more accurate than comparing two embeddings, but it's also orders of magnitude slower, so you can't run it on ten million documents per query. A frontier model reasoning over tool results is powerful and pricey, so you want to call it as few times as possible. A deep ranking model with hundreds of features gives great scores but you can't score a whole catalogue with it in the eighty milliseconds you've got.

So every one of these systems arrives at the same compromise. Use something cheap and roughly-right to throw away 99% of the candidates, then unleash the expensive thing on what's left. The funnel isn't a design choice someone made. It's what you're forced into the moment quality and scale pull in opposite directions.

<div class="fn-model">
<p><b>The one-line version:</b> when your smartest step is too slow to run at scale, you build a funnel in front of it. RAG, agents, and recommendations are three funnels feeding three different expensive steps.</p>
</div>

## Why this is useful, not just tidy

Noticing the shared shape would be a fun trivia fact if it stopped there. It doesn't. The shape tells you where these systems fail, and it's the *same* failure every time.

**Most failures are recall failures, not precision failures.** Stage 2 can only pick from what stage 1 handed it. If the right document never made it into the retrieved pool, the world's best reranker and the smartest model will confidently answer from the wrong context. If the right product never got retrieved, no ranking model can surface it. If the agent never considered the right action, no amount of reasoning saves the plan.

So when a RAG bot gives a wrong answer, the instinct is to blame the model or tweak the prompt. Usually the model is fine. The retrieval missed. The single most useful thing you can do across all three systems is **measure the recall of stage 1 separately from the quality of stage 2.** How often does the good candidate actually make it into the pool? That number, not the eloquence of the final output, is where most of your quality lives.

The second shared lesson: **stage 3 is not optional, and it's not the model's job.** Diversity, safety, freshness, permissions, none of these are things your accuracy metric rewards, so your model will never learn them on its own. You bolt them on as an explicit policy layer, or they don't happen. A ranking model left to itself will happily show ten versions of the same item. A RAG model left to itself will happily quote a document the user isn't allowed to see. The policy layer exists precisely because being accurate and being *right for the product* are different things.

## Start with the cheapest funnel that works

There's a corollary worth saying out loud, because it saves a lot of pain. Not every problem needs the full three-stage machine.

If a single model call answers the question, that's your whole system. If a fixed sequence of steps does the job, build the sequence, a RAG pipeline is a fixed chain, not an agent, and that's a feature. You only reach for the elaborate version, the multi-source retrieval, the agent loop, the full candidate-generation-and-ranking stack, when the scale genuinely forces the funnel on you.

The mistake I see most often is people building stage 1 and stage 2 for a problem that had five hundred candidates, where you could just run the expensive step on all of them and skip the funnel entirely. The funnel is what you build when you *can't* do that. Know which situation you're in.

## The upshot

RAG, agents, and recommendations aren't three things to learn. They're one thing, a funnel that trades cheap breadth for expensive depth, pointed at three different expensive steps. Learn the shape once and each new system becomes "oh, it's the funnel again, feeding a different thing."

And when one of them misbehaves, you already know where to look first. Not the clever model at the narrow end. The wide, cheap net at the top, quietly dropping the answer before anything smart ever got to see it.

*The next posts in this series work each funnel out in full: the [RAG design](/blog/2026-07-24-designing-a-rag-system-that-actually-retrieves/), the [agent design](/blog/2026-07-24-designing-an-agent-that-doesnt-go-off-the-rails/), and the [recommendation design](/blog/2026-07-24-designing-a-recommendation-system-the-funnel/), box by box, with the cloud services that fill each one on AWS, GCP, and Azure.*

<script>
(function(){
  var els=document.querySelectorAll('.fn-anim');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.2});
  els.forEach(function(e){io.observe(e)});
})();
</script>
