---
title: "How to Actually Prevent AI Hallucinations: The Methods, Ranked"
date: 2026-08-02
excerpt: "I've already written about why models hallucinate. This is the other half: the actual toolkit. There are about ten distinct methods to stop an AI making things up, and they are not interchangeable. Some are nearly free, some multiply your bill by ten, some fix the problem at the root and some just hide it. Here's each one, what it really costs, what it can't do, and when to reach for it."
tags: [ai, hallucination, rag, reliability, llm, methods]
---

<style>
.hm-fig{margin:2.5rem 0;}
.hm-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* root vs mask spectrum */
.hm-spectrum{max-width:680px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:.9rem;}
@media(max-width:560px){.hm-spectrum{grid-template-columns:1fr;}}
.hm-spectrum .col{border:1px solid var(--border);border-radius:12px;padding:1.1rem;background:var(--surface);opacity:0;transform:translateY(10px);transition:opacity .5s ease,transform .5s ease;}
.hm-spectrum.go .col{opacity:1;transform:none;} .hm-spectrum.go .col:nth-child(2){transition-delay:.18s;}
.hm-spectrum .col h4{margin:0 0 .5rem;font-size:.82rem;font-family:var(--font-mono);}
.hm-spectrum .col.root{border-color:var(--accent);} .hm-spectrum .col.root h4{color:var(--accent);}
.hm-spectrum .col.mask h4{color:var(--text-2);}
.hm-spectrum .col p{font-size:.85rem;color:var(--text-2);line-height:1.5;margin:0 0 .5rem;}
.hm-spectrum .col .ex{font-family:var(--font-mono);font-size:.74rem;color:var(--text-3);}

/* the layered defense stack */
.hm-stack{max-width:560px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.hm-layer{display:flex;align-items:center;gap:.85rem;border:1px solid var(--border);border-radius:10px;padding:.7rem .9rem;background:var(--surface);opacity:0;transform:translateX(-10px);transition:opacity .5s ease,transform .5s ease;}
.hm-stack.go .hm-layer{opacity:1;transform:none;}
.hm-stack.go .hm-layer:nth-child(1){transition-delay:.1s} .hm-stack.go .hm-layer:nth-child(2){transition-delay:.22s}
.hm-stack.go .hm-layer:nth-child(3){transition-delay:.34s} .hm-stack.go .hm-layer:nth-child(4){transition-delay:.46s}
.hm-stack.go .hm-layer:nth-child(5){transition-delay:.58s}
.hm-layer .n{flex:none;width:26px;height:26px;border-radius:50%;border:1px solid var(--accent);color:var(--accent);font-family:var(--font-mono);font-size:.75rem;display:flex;align-items:center;justify-content:center;}
.hm-layer .t{font-size:.86rem;color:var(--text-2);} .hm-layer .t b{color:var(--text);}

/* cost vs power scatter-ish bars */
.hm-cost{max-width:680px;margin:0 auto;display:flex;flex-direction:column;gap:.7rem;}
.hm-crow{display:grid;grid-template-columns:150px 1fr;gap:.9rem;align-items:center;}
@media(max-width:560px){.hm-crow{grid-template-columns:110px 1fr;}}
.hm-crow .lab{font-family:var(--font-mono);font-size:.76rem;color:var(--text-2);}
.hm-ctrack{height:22px;background:var(--surface-2);border-radius:6px;overflow:hidden;position:relative;}
.hm-cfill{height:100%;width:0;border-radius:6px;transition:width .9s ease;}
.hm-cost.go .hm-cfill{width:var(--w);}
.hm-cfill.free{background:color-mix(in srgb,var(--green) 60%,var(--surface));}
.hm-cfill.mid{background:linear-gradient(90deg,var(--accent),color-mix(in srgb,var(--accent) 50%,var(--surface)));}
.hm-cfill.high{background:linear-gradient(90deg,var(--accent-2),var(--accent));}
.hm-crow .cap{position:absolute;right:8px;top:50%;transform:translateY(-50%);font-family:var(--font-mono);font-size:.68rem;color:var(--text);}

/* effectiveness numbers */
.hm-nums{max-width:680px;margin:0 auto;display:flex;flex-direction:column;gap:.6rem;}
.hm-nrow{display:grid;grid-template-columns:1fr auto;gap:1rem;align-items:center;border:1px solid var(--border);border-radius:10px;padding:.7rem .95rem;background:var(--surface);}
.hm-nrow .d{font-size:.85rem;color:var(--text-2);line-height:1.4;} .hm-nrow .d b{color:var(--text);}
.hm-nrow .d span{font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);display:block;margin-top:.15rem;}
.hm-nrow .big{font-family:var(--font-mono);font-size:1.1rem;color:var(--accent);font-weight:600;white-space:nowrap;}

/* master table */
.hm-tab-wrap{max-width:900px;margin:0 auto;overflow-x:auto;border:1px solid var(--border);border-radius:12px;background:var(--surface);}
.hm-tab{width:100%;border-collapse:collapse;font-size:.83rem;min-width:720px;}
.hm-tab th,.hm-tab td{text-align:left;padding:.6rem .75rem;border-bottom:1px solid var(--border);vertical-align:top;}
.hm-tab th{font-family:var(--font-mono);font-size:.68rem;text-transform:uppercase;letter-spacing:.04em;color:var(--text-3);font-weight:500;white-space:nowrap;}
.hm-tab td:first-child,.hm-tab th:first-child{color:var(--text);font-weight:500;}
.hm-tab tr:last-child td{border-bottom:none;}
.hm-tab .c-free{color:var(--green);font-family:var(--font-mono);font-size:.78rem;}
.hm-tab .c-mid{color:var(--accent);font-family:var(--font-mono);font-size:.78rem;}
.hm-tab .c-high{color:var(--accent-2);font-family:var(--font-mono);font-size:.78rem;}

/* method cards pros/cons */
.hm-method{max-width:720px;margin:1.4rem auto;border:1px solid var(--border);border-radius:14px;background:var(--surface);overflow:hidden;}
.hm-method .top{padding:.9rem 1.1rem;border-bottom:1px solid var(--border);display:flex;align-items:baseline;gap:.6rem;}
.hm-method .top .num{font-family:var(--font-mono);font-size:.75rem;color:var(--accent-ink);background:var(--accent);border-radius:5px;padding:.1rem .45rem;}
.hm-method .top b{color:var(--text);font-size:1rem;}
.hm-method .pc{display:grid;grid-template-columns:1fr 1fr;gap:0;}
@media(max-width:520px){.hm-method .pc{grid-template-columns:1fr;}}
.hm-method .pc .side{padding:.9rem 1.1rem;}
.hm-method .pc .side.pros{border-right:1px solid var(--border);}
@media(max-width:520px){.hm-method .pc .side.pros{border-right:none;border-bottom:1px solid var(--border);}}
.hm-method .grp{font-family:var(--font-mono);font-size:.68rem;letter-spacing:.06em;text-transform:uppercase;margin-bottom:.5rem;}
.hm-method .grp.p{color:var(--green);} .hm-method .grp.c{color:var(--text-3);}
.hm-method .side .li{font-size:.83rem;color:var(--text-2);line-height:1.45;padding-left:1.1rem;position:relative;margin:.35rem 0;}
.hm-method .side .li::before{position:absolute;left:0;font-family:var(--font-mono);}
.hm-method .side.pros .li::before{content:"+";color:var(--green);}
.hm-method .side.cons .li::before{content:"\2212";color:var(--text-3);}
.hm-method .foot{padding:.75rem 1.1rem;background:var(--surface-2);border-top:1px solid var(--border);font-size:.82rem;color:var(--text-2);}
.hm-method .foot b{color:var(--text);}

@media (prefers-reduced-motion: reduce){
  .hm-spectrum .col,.hm-layer{opacity:1!important;transform:none!important;}
  .hm-cost .hm-cfill{transition:none!important;width:var(--w)!important;}
}
</style>

A while back I wrote about [why AI hallucinates](/blog/2026-07-08-why-ai-hallucinates-and-how-we-stop-it/): it predicts plausible tokens, it was trained on incentives that reward a confident guess over an honest "I don't know," and so it will invent a citation, a court case, an API that doesn't exist, and hand it to you with a straight face.

That post answered *why*. This one is the other half, and the more practical half: **what do you actually do about it?**

Because "add some guardrails" is not a plan. There are roughly ten distinct methods to reduce hallucination, and they are wildly different tools. One is nearly free and you should always use it. One multiplies your inference bill by ten. Some fix the problem at the root; others just paper over it so it's harder to notice. If you reach for the wrong one, you spend a lot and fix little.

So this is the ranked, opinionated tour. For each method: how it works, what it genuinely costs, what it *can't* do, and when it's the right call.

## First cut: fix the root, or hide the symptom?

Before the list, one distinction that sorts everything else. Some methods reduce hallucination by giving the model better information to work with. Others let it hallucinate freely and then try to catch the bad output on the way out. Both are useful. They are not the same thing.

<figure class="hm-fig">
<div class="hm-spectrum">
  <div class="col root">
    <h4>Root fixes (upstream)</h4>
    <p>Change what the model has to work with before it generates. Give it the real document, the calculator, the current fact. It hallucinates less because it's guessing less.</p>
    <p class="ex">RAG, tool use, fine-tuning, better prompts</p>
  </div>
  <div class="col mask">
    <h4>Symptom catches (downstream)</h4>
    <p>Let it generate, then inspect the output and block, flag, or fix what looks wrong. Necessary, but it's cleanup: the model still made the thing up, you just caught it.</p>
    <p class="ex">Verifier chains, guardrails, abstention, human review</p>
  </div>
</div>
<figcaption>The biggest wins are upstream. You get far more from feeding the model good context than from policing bad output, though in production you do both.</figcaption>
</figure>

Keep that split in mind. When people say "we added hallucination protection" and mean a single output filter, they've done the cheaper, weaker half and skipped the part that actually moves the numbers.

## The whole toolkit, at a glance

Here's every method side by side before we go deep. Read the cost column carefully: it's the part most guides gloss over.

<figure class="hm-fig">
<div class="hm-tab-wrap">
<table class="hm-tab">
<thead>
<tr><th>Method</th><th>What it does</th><th>Cost / latency</th><th>Fixes root?</th></tr>
</thead>
<tbody>
<tr><td>RAG / grounding</td><td>Retrieve real docs, answer from them</td><td class="c-mid">1 retrieval + bigger prompt</td><td>Yes</td></tr>
<tr><td>Cite + verify</td><td>Check each claim against the source</td><td class="c-mid">+1 model pass</td><td>Catch</td></tr>
<tr><td>Prompt technique</td><td>CoT, "say I don't know", decompose</td><td class="c-free">near free</td><td>Partial</td></tr>
<tr><td>Self-consistency</td><td>Sample N times, majority vote</td><td class="c-high">N times inference</td><td>Catch</td></tr>
<tr><td>LLM-as-judge</td><td>A second model scores the first</td><td class="c-mid">+1 (bigger) call</td><td>Catch</td></tr>
<tr><td>Guardrails / schema</td><td>Force valid structure, enforce rails</td><td class="c-free">cheap to moderate</td><td>Format only</td></tr>
<tr><td>Factuality fine-tuning</td><td>Train the weights toward truth</td><td class="c-high">high upfront, free at run</td><td>Yes</td></tr>
<tr><td>Abstention / uncertainty</td><td>Measure doubt, decline when high</td><td class="c-high">often N times sampling</td><td>Catch</td></tr>
<tr><td>Human-in-the-loop</td><td>A person signs off risky output</td><td class="c-high">slow, labor cost</td><td>Catch</td></tr>
<tr><td>Tool use / KG</td><td>Offload facts to calculator, search, graph</td><td class="c-mid">extra tool calls</td><td>Yes</td></tr>
</tbody>
</table>
</div>
<figcaption>Green is cheap, teal is moderate, purple is expensive. Notice the pattern: the cheapest methods (prompting, schema) are partial fixes, and the strongest root fixes (RAG, tool use, fine-tuning) cost real money or effort. There's no free lunch, only sensible trades.</figcaption>
</figure>

Now the deep version. I've grouped them root-fixes first, catches second, roughly in the order you'd actually adopt them.

## 1. RAG: give it the facts instead of asking it to remember

The single biggest lever, and the first thing to reach for. Instead of hoping the answer is buried somewhere in the model's weights, you retrieve the relevant documents at question time and put them right in the prompt. The model reads facts instead of recalling a fuzzy impression of them.

<div class="hm-method">
  <div class="top"><span class="num">01</span><b>RAG / grounding</b></div>
  <div class="pc">
    <div class="side pros">
      <div class="grp p">Pros</div>
      <div class="li">Attacks hallucination at the root: less guessing</div>
      <div class="li">Updatable without retraining, just change the docs</div>
      <div class="li">Gives you citations for free</div>
    </div>
    <div class="side cons">
      <div class="grp c">Cons</div>
      <div class="li">Only as good as retrieval: wrong chunk in, wrong answer out</div>
      <div class="li">The model can still ignore the context and invent anyway</div>
      <div class="li">Struggles on multi-hop questions</div>
    </div>
  </div>
  <div class="foot"><b>Reach for it when:</b> any domain-specific or knowledge-heavy task. Support bots, internal search, anything over your own data. If you do one thing, do this.</div>
</div>

The honest caveat, which too many RAG guides skip: retrieved does not mean correct. Feed it the right document and a model can still revert to its own invented version, or the retriever fetches the wrong passage entirely. RAG drives hallucination way down; it does not abolish it. Which is exactly why you layer the next methods on top. (I went deep on doing RAG well in the [RAG design post](/blog/2026-07-24-designing-a-rag-system-that-actually-retrieves/).)

## 2. Cite, then actually verify the citation

RAG gets the docs in front of the model. This layer makes it accountable to them. You require a source for every claim, then you check that the source actually says what the model claims it says. That check is the important part, a citation nobody verifies is just decoration.

<div class="hm-method">
  <div class="top"><span class="num">02</span><b>Citation + groundedness verification</b></div>
  <div class="pc">
    <div class="side pros">
      <div class="grp p">Pros</div>
      <div class="li">Catches ungrounded claims plain RAG lets through</div>
      <div class="li">Produces auditable, traceable answers</div>
      <div class="li">Managed options exist, no training needed</div>
    </div>
    <div class="side cons">
      <div class="grp c">Cons</div>
      <div class="li">Adds a second model pass per answer</div>
      <div class="li">The checker has its own error rate</div>
      <div class="li">Auto-correction can quietly distort meaning</div>
    </div>
  </div>
  <div class="foot"><b>Reach for it when:</b> regulated or high-stakes RAG (health, legal, finance) where every claim must trace to a source. Managed versions: AWS Bedrock contextual grounding check, Azure groundedness detection, Vertex grounding.</div>
</div>

The cost here is concrete: one extra classifier or LLM call per response, sometimes per claim. That's latency and money on every single answer. Worth it when a wrong claim is expensive; overkill for a casual chatbot.

## 3. Prompt techniques: the nearly-free baseline

Before you spend anything, spend words. Three prompt moves genuinely reduce hallucination at almost zero cost. Chain-of-thought ("think step by step") makes the model reason before answering. Explicitly permitting "I don't know" gives it an exit that isn't fabrication. And decomposition, breaking a hard question into checkable pieces, stops it from bluffing through a leap it can't make.

<div class="hm-method">
  <div class="top"><span class="num">03</span><b>Prompt techniques (CoT, abstain, decompose)</b></div>
  <div class="pc">
    <div class="side pros">
      <div class="grp p">Pros</div>
      <div class="li">Nearly free, no infra change</div>
      <div class="li">CoT lifts reasoning-heavy accuracy a lot</div>
      <div class="li">"You may say you don't know" cuts confident bluffing</div>
    </div>
    <div class="side cons">
      <div class="grp c">Cons</div>
      <div class="li">CoT can produce plausible-but-wrong reasoning</div>
      <div class="li">Helps big models more than small ones</div>
      <div class="li">Prompt-only gains are fragile, they shift with the model</div>
    </div>
  </div>
  <div class="foot"><b>Reach for it when:</b> always. This is your baseline layer, on top of everything else. Especially good for math and multi-step reasoning, and any task where "no answer" is an acceptable answer.</div>
</div>

The only real cost is tokens: chain-of-thought makes the model write more, which is a bit more latency and spend. Cheap insurance.

## 4 and 5. Sample-and-vote, and the second-opinion model

Two catches that both work by not trusting a single generation.

**Self-consistency** samples the same question several times and takes the majority answer. If the model lands on "42" four times out of five, that's a stronger signal than one lucky (or unlucky) roll. **LLM-as-judge** uses a separate, often stronger model to score the first one's output for factuality before it ships.

<figure class="hm-fig">
<div class="hm-nums">
  <div class="hm-nrow"><div class="d"><b>Self-consistency on GSM8K math</b><span>Wang et al., 2022, arXiv:2203.11171</span></div><div class="big">+17.9%</div></div>
  <div class="hm-nrow"><div class="d"><b>Self-Refine, average gain across tasks</b><span>Madaan et al., 2023, arXiv:2303.17651</span></div><div class="big">~20%</div></div>
  <div class="hm-nrow"><div class="d"><b>Strong LLM judge vs human agreement</b><span>Zheng et al., 2023, arXiv:2306.05685</span></div><div class="big">&gt;80%</div></div>
</div>
<figcaption>These are real reported gains, but each is tied to a specific model and benchmark. Read them as "X reported Y% on task Z," not universal constants. A GPT-4-class judge agreeing with humans over 80% of the time is notable: that's about the rate two humans agree with each other.</figcaption>
</figure>

The catch, literally, is cost. Self-consistency multiplies your inference by however many samples you draw, five, ten, more. A judge adds a full (and usually larger, pricier) model call per answer. These buy real accuracy, and they buy it by the token.

<div class="hm-method">
  <div class="top"><span class="num">04·05</span><b>Self-consistency and LLM-as-judge</b></div>
  <div class="pc">
    <div class="side pros">
      <div class="grp p">Pros</div>
      <div class="li">Large, reliable accuracy gains, no training</div>
      <div class="li">Judge scales up automated quality gates</div>
      <div class="li">Both work purely at inference time</div>
    </div>
    <div class="side cons">
      <div class="grp c">Cons</div>
      <div class="li">Self-consistency costs N times inference</div>
      <div class="li">A judge can share the generator's blind spots</div>
      <div class="li">Judges have biases (verbosity, position, self-preference)</div>
    </div>
  </div>
  <div class="foot"><b>Reach for it when:</b> high-value answers where accuracy beats cost. Self-consistency for math and reasoning with a votable answer; a judge for gating candidate answers before users see them.</div>
</div>

## 6. Guardrails and schema-constrained decoding: structure, not truth

This one is widely misunderstood, so let me be blunt about what it does and doesn't do. Constrained decoding forces the output to match a grammar or JSON schema, so you can't get an invalid enum or a missing field. Guardrail frameworks add programmable input and output rails: topic boundaries, policy checks, validators.

Here's the trap: **constrained decoding guarantees the output is well-formed, not that it's true.** The model can hand you perfectly valid JSON with a completely invented value inside it. Structure is not fact.

<div class="hm-method">
  <div class="top"><span class="num">06</span><b>Guardrails / structured output</b></div>
  <div class="pc">
    <div class="side pros">
      <div class="grp p">Pros</div>
      <div class="li">Eliminates format and enum hallucination outright</div>
      <div class="li">Rails enforce policy and topic boundaries</div>
      <div class="li">Constrained decoding is cheap</div>
    </div>
    <div class="side cons">
      <div class="grp c">Cons</div>
      <div class="li">Guarantees structure, NOT semantic truth</div>
      <div class="li">Rails can over-block valid output</div>
      <div class="li">Adds engineering and per-turn checks</div>
    </div>
  </div>
  <div class="foot"><b>Reach for it when:</b> any structured extraction or tool-calling flow (use schema constraints), and wherever policy plus fact-checks must be enforced in code (use a guardrail framework like NeMo Guardrails or Guardrails AI).</div>
</div>

## 7. Factuality fine-tuning: bake truth into the weights

Everything so far happens at inference. This one changes the model itself. You build preference pairs that reward factual answers over confident-but-wrong ones and train on them (the FactTune work does this with DPO and no human labels). The payoff is unusual: the cost is entirely upfront, and inference stays cheap.

<figure class="hm-fig">
<div class="hm-nums">
  <div class="hm-nrow"><div class="d"><b>Factual error reduction, factuality fine-tuning</b><span>FactTune, Tian et al., 2023, arXiv:2311.08401, on a 7B model</span></div><div class="big">58%</div></div>
</div>
<figcaption>A big drop, but note the fine print: it's a specific model on a specific factuality benchmark, and naive fine-tuning can backfire, teaching a model to confidently state things it doesn't actually know. Done wrong, it makes hallucination worse.</figcaption>
</figure>

<div class="hm-method">
  <div class="top"><span class="num">07</span><b>Factuality fine-tuning / RLHF</b></div>
  <div class="pc">
    <div class="side pros">
      <div class="grp p">Pros</div>
      <div class="li">Bakes factuality into the model, zero per-query overhead</div>
      <div class="li">Durable gains beyond what prompting can reach</div>
      <div class="li">Modern methods need no human labels</div>
    </div>
    <div class="side cons">
      <div class="grp c">Cons</div>
      <div class="li">Expensive and slow to train, needs a pipeline</div>
      <div class="li">Naive tuning can increase hallucination</div>
      <div class="li">Risk of over-refusal or lost capability</div>
    </div>
  </div>
  <div class="foot"><b>Reach for it when:</b> you own the model, have real volume, and want lasting factuality gains that prompting and RAG can't give you. Not a first move.</div>
</div>

## 8. Know when it doesn't know: uncertainty and abstention

A different idea entirely: instead of trying to make every answer right, measure how sure the model is and let it decline when it isn't. The elegant version is **semantic entropy** (a 2024 Nature result): sample several answers, cluster the ones that mean the same thing, and measure the spread of *meanings*. High spread means the model is confabulating, low spread means it's confident. SelfCheckGPT does a black-box cousin of this by checking whether repeated samples agree.

<div class="hm-method">
  <div class="top"><span class="num">08</span><b>Uncertainty estimation / abstention</b></div>
  <div class="pc">
    <div class="side pros">
      <div class="grp p">Pros</div>
      <div class="li">Turns a confident wrong answer into a safe "not sure"</div>
      <div class="li">Semantic entropy needs no ground-truth labels</div>
      <div class="li">Generalizes across tasks and datasets</div>
    </div>
    <div class="side cons">
      <div class="grp c">Cons</div>
      <div class="li">Sampling variants cost N times inference</div>
      <div class="li">Thresholds need tuning per use case</div>
      <div class="li">Abstain too often and the thing becomes useless</div>
    </div>
  </div>
  <div class="foot"><b>Reach for it when:</b> high-stakes settings where a confident wrong answer is worse than no answer. Pairs beautifully with the next method: abstain, then route the uncertain ones to a human.</div>
</div>

## 9 and 10. The human, and the tool

Two last methods that bookend the list.

**Human-in-the-loop** is the highest-reliability option and the least scalable: route the risky, low-confidence, or flagged outputs to a person before they're final. It's slow and it costs labor, and humans have their own failure mode (they wave through fluent, wrong text because it reads well). The trick is not to review everything, but to review only what your uncertainty method (method 8) flagged.

**Tool use and knowledge graphs** attack the root from a different angle: stop asking the stochastic model to be a database. Let it call a calculator for arithmetic, a search API for current facts, a knowledge graph for structured relations. The exact facts live in deterministic systems; the model just orchestrates. This is why a model that's hopeless at mental math becomes reliable the moment you give it a code tool.

<div class="hm-method">
  <div class="top"><span class="num">09·10</span><b>Human review and tool use / knowledge graphs</b></div>
  <div class="pc">
    <div class="side pros">
      <div class="grp p">Pros</div>
      <div class="li">Human review is the highest reliability there is</div>
      <div class="li">Tools move exact facts out of the stochastic model</div>
      <div class="li">Deterministic tools give verifiable, current answers</div>
    </div>
    <div class="side cons">
      <div class="grp c">Cons</div>
      <div class="li">Humans are slow, costly, and miss fluent errors</div>
      <div class="li">Tools and graphs must exist, be correct, and stay current</div>
      <div class="li">The model can still misuse a tool's output</div>
    </div>
  </div>
  <div class="foot"><b>Reach for it when:</b> human review for regulated sign-off and irreversible actions; tools and KGs for anything numeric, computational, or needing live facts.</div>
</div>

## So what do you actually build?

You don't pick one. Every serious production system layers several, cheapest and most-fundamental first, because each catches a different failure.

<figure class="hm-fig">
<div class="hm-stack">
  <div class="hm-layer"><span class="n">1</span><span class="t"><b>Ground it.</b> RAG plus tool use, so it's reading facts, not guessing them. The root fix.</span></div>
  <div class="hm-layer"><span class="n">2</span><span class="t"><b>Prompt it well.</b> Chain-of-thought and explicit permission to say "I don't know." Nearly free.</span></div>
  <div class="hm-layer"><span class="n">3</span><span class="t"><b>Constrain it.</b> Schema and guardrails so the shape is always valid and policy holds.</span></div>
  <div class="hm-layer"><span class="n">4</span><span class="t"><b>Verify it.</b> Check claims against sources, and abstain when uncertainty is high.</span></div>
  <div class="hm-layer"><span class="n">5</span><span class="t"><b>Escalate it.</b> Route the flagged, high-risk answers to a human. Only those.</span></div>
</div>
<figcaption>The layered defense. No single method eliminates hallucination, so you stack them: each layer is cheap relative to the failure it prevents, and together they turn "confidently wrong" into "grounded, cited, and honest about its limits."</figcaption>
</figure>

## The takeaway

Preventing hallucination isn't one trick, it's a budget decision. The cheap methods (good prompts, schema constraints) are worth doing always but only get you part way. The strong methods (RAG, tool use, verification, fine-tuning) cost retrieval calls, extra model passes, training runs, or human time, and you spend those where being wrong is expensive.

The mistake I see most is teams bolting a single output filter onto an ungrounded model and calling it solved. That's the weakest, most downstream layer doing all the work. Start upstream. Give the model the facts, let it admit doubt, and only then spend on catching what slips through. Do that and you don't eliminate hallucination, nobody does, but you push it from "the reason we can't ship" to "a managed, measured risk."

*This is the methods companion to [Why AI Hallucinates, and How We Actually Stop It](/blog/2026-07-08-why-ai-hallucinates-and-how-we-stop-it/). For the grounding layer in depth, see [Designing a RAG System That Actually Retrieves](/blog/2026-07-24-designing-a-rag-system-that-actually-retrieves/).*

## References

Written from scratch; these are the primary sources behind the methods and numbers above. Nothing here is copied from them.

- Chain-of-Thought Prompting, Wei et al., 2022: <https://arxiv.org/abs/2201.11903>
- Self-Consistency Improves Chain of Thought, Wang et al., 2022: <https://arxiv.org/abs/2203.11171>
- Self-Refine: Iterative Refinement with Self-Feedback, Madaan et al., 2023: <https://arxiv.org/abs/2303.17651>
- Retrieval-Augmented Generation, Lewis et al., 2020: <https://arxiv.org/abs/2005.11401>
- SelfCheckGPT, Manakul et al., 2023: <https://arxiv.org/abs/2303.08896>
- Detecting hallucinations with semantic entropy, Farquhar et al., Nature 630:625-630, 2024 (probes follow-up: <https://arxiv.org/abs/2406.15927>)
- Fine-tuning Language Models for Factuality (FactTune), Tian et al., 2023: <https://arxiv.org/abs/2311.08401>
- Judging LLM-as-a-Judge, Zheng et al., 2023: <https://arxiv.org/abs/2306.05685>
- TruthfulQA, Lin et al., 2021: <https://arxiv.org/abs/2109.07958>
- AWS Bedrock, contextual grounding check: <https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-contextual-grounding-check.html>
- AWS Bedrock, Automated Reasoning checks: <https://aws.amazon.com/blogs/aws/minimize-ai-hallucinations-and-deliver-up-to-99-verification-accuracy-with-automated-reasoning-checks-now-available/>
- Azure AI Content Safety, groundedness detection: <https://learn.microsoft.com/en-us/azure/ai-services/content-safety/concepts/groundedness>
- Google Vertex AI, grounding overview: <https://docs.cloud.google.com/vertex-ai/generative-ai/docs/grounding/overview>
- OpenAI, Structured Outputs: <https://developers.openai.com/api/docs/guides/structured-outputs>
- Vectara HHEM hallucination leaderboard: <https://github.com/vectara/hallucination-leaderboard>

<script>
(function(){
  var els=document.querySelectorAll('.hm-spectrum,.hm-stack,.hm-cost,.hm-nums');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.2});
  els.forEach(function(e){io.observe(e)});
})();
</script>
