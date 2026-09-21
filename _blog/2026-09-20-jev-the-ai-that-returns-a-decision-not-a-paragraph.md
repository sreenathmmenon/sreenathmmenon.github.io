---
title: "Jev: The AI That Returns a Decision, Not a Paragraph"
date: 2026-09-20
excerpt: "Typesafe just shipped Jev, a different shape of AI model. You don't ask it to write text and then parse the answer back out. You hand it your program's state and a typed question, and it returns a typed decision with a calibrated confidence score, in one pass, in milliseconds. It can't hallucinate a field that doesn't exist, because the set of possible answers is fixed before it runs. Here's what a 'System One' model actually is, how it works, where it wins, and where it doesn't."
tags: [ai, models, system-design]
---

<style>
.jv-fig{margin:2.4rem 0;}
.jv-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.8rem;text-align:center;line-height:1.5;}

/* hero: text-out vs decision-out */
.jv-hero{max-width:720px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:1rem;}
@media(max-width:600px){.jv-hero{grid-template-columns:1fr;}}
.jv-card{border:1px solid var(--border);border-radius:14px;background:var(--surface);overflow:hidden;opacity:0;transform:translateY(10px);transition:opacity .5s var(--ease),transform .5s var(--ease);}
.jv-hero.go .jv-card{opacity:1;transform:none;} .jv-hero.go .jv-card:nth-child(2){transition-delay:.16s;}
.jv-card .h{padding:.75rem 1rem;border-bottom:1px solid var(--border);font-family:var(--font-mono);font-size:.8rem;font-weight:600;}
.jv-card.llm .h{color:var(--accent-2);} .jv-card.jev .h{color:var(--accent);}
.jv-card .b{padding:.95rem 1rem;font-size:.86rem;color:var(--text-2);line-height:1.55;}
.jv-card .io{font-family:var(--font-mono);font-size:.76rem;background:var(--surface-2);border-radius:8px;padding:.5rem .7rem;margin:.4rem 0;color:var(--text);}
.jv-card .io .k{color:var(--text-3);}
.jv-card.jev .io .v{color:var(--accent);} .jv-card.llm .io .v{color:var(--accent-2);}

/* autoregressive vs parallel */
.jv-gen{max-width:720px;margin:0 auto;display:flex;flex-direction:column;gap:1.1rem;}
.jv-lane{border:1px solid var(--border);border-radius:12px;background:var(--surface);padding:.9rem 1rem;}
.jv-lane .lh{font-family:var(--font-mono);font-size:.74rem;font-weight:600;margin-bottom:.6rem;}
.jv-lane.seq .lh{color:var(--accent-2);} .jv-lane.par .lh{color:var(--accent);}
.jv-toks{display:flex;flex-wrap:wrap;gap:.35rem;}
.jv-tok{font-family:var(--font-mono);font-size:.72rem;border-radius:6px;padding:.2rem .5rem;background:var(--surface-2);color:var(--text-2);opacity:0;transform:scale(.8);}
.jv-lane.seq.go .jv-tok{opacity:1;transform:none;transition:opacity .25s var(--ease),transform .25s var(--ease);}
.jv-lane.seq.go .jv-tok:nth-child(1){transition-delay:.05s}.jv-lane.seq.go .jv-tok:nth-child(2){transition-delay:.3s}.jv-lane.seq.go .jv-tok:nth-child(3){transition-delay:.55s}.jv-lane.seq.go .jv-tok:nth-child(4){transition-delay:.8s}.jv-lane.seq.go .jv-tok:nth-child(5){transition-delay:1.05s}.jv-lane.seq.go .jv-tok:nth-child(6){transition-delay:1.3s}.jv-lane.seq.go .jv-tok:nth-child(7){transition-delay:1.55s}.jv-lane.seq.go .jv-tok:nth-child(8){transition-delay:1.8s}
.jv-lane.par.go .jv-tok{opacity:1;transform:none;transition:opacity .4s var(--ease);transition-delay:.15s;}
.jv-lane .ln{font-family:var(--font-mono);font-size:.7rem;color:var(--text-3);margin-top:.55rem;}

/* three primitives */
.jv-prims{max-width:720px;margin:0 auto;display:grid;grid-template-columns:repeat(3,1fr);gap:.8rem;}
@media(max-width:600px){.jv-prims{grid-template-columns:1fr;}}
.jv-prim{border:1px solid var(--border-2);border-radius:12px;background:var(--surface);padding:1rem;opacity:0;transform:translateY(8px);transition:opacity .45s var(--ease),transform .45s var(--ease);}
.jv-prims.go .jv-prim{opacity:1;transform:none;}
.jv-prims.go .jv-prim:nth-child(1){transition-delay:.08s}.jv-prims.go .jv-prim:nth-child(2){transition-delay:.2s}.jv-prims.go .jv-prim:nth-child(3){transition-delay:.32s}
.jv-prim .pt{font-family:var(--font-mono);font-size:.9rem;color:var(--accent);font-weight:600;margin-bottom:.35rem;}
.jv-prim .pd{font-size:.83rem;color:var(--text-2);line-height:1.5;}
.jv-prim .pe{font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);margin-top:.5rem;border-top:1px solid var(--border);padding-top:.5rem;}

/* speed + cost chart */
.jv-chart{max-width:680px;margin:0 auto;border:1px solid var(--border);border-radius:14px;background:var(--surface);padding:1.2rem 1.1rem 1rem;}
.jv-chart .clab{font-family:var(--font-mono);font-size:.74rem;color:var(--text-3);margin-bottom:1rem;}
.jv-chart .clab b{color:var(--text);}
.jv-crow{display:flex;align-items:center;gap:.7rem;margin:.6rem 0;}
.jv-crow .cn{flex:none;width:120px;font-family:var(--font-mono);font-size:.74rem;color:var(--text-2);}
.jv-crow .ct{flex:1;background:var(--surface-2);border-radius:6px;height:24px;overflow:hidden;}
.jv-crow .cb{height:100%;width:0;border-radius:6px;transition:width .9s var(--ease);}
.jv-crow.llm .cb{background:var(--accent-2);} .jv-crow.jev .cb{background:var(--accent);}
.jv-chart.go .jv-crow .cb{width:var(--w);}
.jv-crow .cv{flex:none;width:96px;font-family:var(--font-mono);font-size:.74rem;color:var(--text);text-align:right;}
.jv-chart .cnote{margin-top:.9rem;font-family:var(--font-mono);font-size:.7rem;color:var(--text-3);text-align:center;line-height:1.5;}

/* calibration */
.jv-cal{max-width:600px;margin:0 auto;border:1px solid var(--border);border-radius:14px;background:var(--surface);padding:1.2rem;}
.jv-cal .ct{font-family:var(--font-mono);font-size:.74rem;color:var(--text-3);margin-bottom:.9rem;text-align:center;}
.jv-calrow{display:flex;align-items:center;gap:.7rem;margin:.5rem 0;opacity:0;transform:translateX(-8px);transition:opacity .4s var(--ease),transform .4s var(--ease);}
.jv-cal.go .jv-calrow{opacity:1;transform:none;}
.jv-cal.go .jv-calrow:nth-child(2){transition-delay:.1s}.jv-cal.go .jv-calrow:nth-child(3){transition-delay:.22s}.jv-cal.go .jv-calrow:nth-child(4){transition-delay:.34s}
.jv-calrow .said{flex:none;width:120px;font-family:var(--font-mono);font-size:.74rem;color:var(--accent);}
.jv-calrow .track{flex:1;background:var(--surface-2);border-radius:6px;height:16px;overflow:hidden;}
.jv-calrow .fill{height:100%;width:0;background:var(--green);border-radius:6px;transition:width .8s var(--ease);}
.jv-cal.go .jv-calrow .fill{width:var(--w);}
.jv-calrow .act{flex:none;width:64px;font-family:var(--font-mono);font-size:.72rem;color:var(--text-2);text-align:right;}
.jv-cal .cfoot{margin-top:.9rem;font-size:.82rem;color:var(--text-2);line-height:1.5;text-align:center;}

/* comparison table */
.jv-tab-wrap{max-width:760px;margin:0 auto;overflow-x:auto;border:1px solid var(--border);border-radius:12px;background:var(--surface);}
.jv-tab{width:100%;border-collapse:collapse;font-size:.86rem;min-width:560px;}
.jv-tab th,.jv-tab td{text-align:left;padding:.65rem .85rem;border-bottom:1px solid var(--border);vertical-align:top;line-height:1.45;}
.jv-tab thead th{font-family:var(--font-mono);font-size:.7rem;text-transform:uppercase;letter-spacing:.04em;font-weight:600;}
.jv-tab thead th:nth-child(2){color:var(--accent-2);} .jv-tab thead th:nth-child(3){color:var(--accent);}
.jv-tab tbody td:first-child{color:var(--text);font-weight:500;}
.jv-tab td{color:var(--text-2);} .jv-tab code{font-family:var(--font-mono);font-size:.82em;}
.jv-tab tr:last-child td{border-bottom:none;}

@media (prefers-reduced-motion: reduce){
  .jv-card,.jv-tok,.jv-prim,.jv-calrow,.jv-drow{opacity:1!important;transform:none!important;transition:none!important;}
  .jv-chart .jv-crow .cb,.jv-cal .jv-calrow .fill,.jv-demo .jv-drow .dfill{transition:none!important;}
}

/* real demo: confidence split */
.jv-demo{max-width:620px;margin:0 auto;border:1px solid var(--border);border-radius:14px;background:var(--surface);padding:1.2rem;}
.jv-demo .dh{font-family:var(--font-mono);font-size:.74rem;color:var(--text-3);margin-bottom:1rem;text-align:center;}
.jv-drow{display:flex;align-items:center;gap:.7rem;margin:.7rem 0;opacity:0;transform:translateX(-8px);transition:opacity .45s var(--ease),transform .45s var(--ease);}
.jv-demo.go .jv-drow{opacity:1;transform:none;}
.jv-demo.go .jv-drow:nth-child(3){transition-delay:.12s}
.jv-drow .dlab{flex:none;width:110px;font-family:var(--font-mono);font-size:.72rem;line-height:1.3;}
.jv-drow.clear .dlab{color:var(--accent);} .jv-drow.amb .dlab{color:var(--accent-2);}
.jv-drow .dtrack{flex:1;background:var(--surface-2);border-radius:6px;height:26px;overflow:hidden;position:relative;}
.jv-drow .dfill{height:100%;width:0;border-radius:6px;transition:width 1s var(--ease);}
.jv-drow.clear .dfill{background:var(--accent);} .jv-drow.amb .dfill{background:var(--accent-2);}
.jv-demo.go .jv-drow .dfill{width:var(--w);}
.jv-drow .dval{flex:none;width:54px;font-family:var(--font-mono);font-size:.8rem;color:var(--text);text-align:right;}
.jv-demo .dex{margin-top:1rem;border-top:1px solid var(--border);padding-top:.8rem;font-size:.83rem;color:var(--text-2);line-height:1.6;}
.jv-demo .dex .q{font-family:var(--font-mono);font-size:.76rem;color:var(--text);}
.jv-demo .dex .c{font-family:var(--font-mono);font-size:.76rem;}
.jv-demo .dex .c.hi{color:var(--accent);} .jv-demo .dex .c.lo{color:var(--accent-2);}
</style>

Most AI you use writes. You ask a question, it produces a paragraph, and if your program needs a real answer out of that paragraph, you parse it, hope the JSON is valid, and add a retry for when it isn't. The intelligence is great. The *shape* of the output is a hassle: it's text, and text has to be turned back into something your code can use.

[Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev), released by Typesafe in September 2026, is a different shape. You hand it your program's state and a typed question, and it hands back a typed decision with a confidence score. No paragraph. No parsing. No "please respond only in JSON." The answer is already the thing your code needed.

It's worth knowing who's behind it, because it sets the ambition: founder Diogo Almeida says he helped invent ChatGPT, then spent two years in stealth on a new training method and this new model. His launch framing was blunt, "20 to 200x faster, 40 to 400x cheaper, output tokens free," and it landed: tens of millions of views in a day. Big claims deserve a close read, so let's actually look at what it is.

<figure class="jv-fig">
<div class="jv-hero wm-anim">
  <div class="jv-card llm">
    <div class="h">A normal LLM</div>
    <div class="b">
      <div class="io"><span class="k">in:</span> "I was charged twice, help ASAP"</div>
      <div class="io"><span class="k">out:</span> <span class="v">"It sounds like you're dealing with a billing issue. I'd be happy to help..."</span></div>
      Now your code parses that back into fields, and handles the times it comes out malformed.
    </div>
  </div>
  <div class="jv-card jev">
    <div class="h">Jev</div>
    <div class="b">
      <div class="io"><span class="k">in:</span> "I was charged twice, help ASAP"</div>
      <div class="io"><span class="k">out:</span> <span class="v">billing: 0.98 · tone: angry · urgency: high</span></div>
      Typed values with confidence, ready to branch on. Nothing to parse, nothing to retry.
    </div>
  </div>
</div>
<figcaption>Same input. One returns prose you have to decode; the other returns the decision itself. Typesafe calls this a "System One" model: fast, structured decisions software can use directly.</figcaption>
</figure>

## System One, not System Two

The name is a nod to Kahneman: System 1 is fast, intuitive, automatic; System 2 is slow, deliberate reasoning. Today's frontier LLMs are wonderful System 2 machines, they think out loud, at length, and that's exactly what you want for writing code or an essay. But a huge amount of what we actually wire AI into isn't essay-writing. It's small, fast judgments: *is this spam? which queue does this ticket go to? how risky is this transaction? does this text mention billing?*

Those are System One tasks, and using a text-generating model for them is like hiring a novelist to answer yes/no questions. It works, but it's slow, expensive, and it might write a paragraph when you needed a boolean. Jev is built for that fast lane instead.

## How it actually works: one pass, not a token loop

This is the core idea, and it's worth slowing down on. An LLM generates text one token at a time. To produce a 500-token answer it runs itself about 500 times in sequence, each run feeding the next. That sequential loop is why responses take seconds and why you pay per output token.

Jev doesn't generate text at all. Because the set of possible answers is fixed *before* it runs (pick one of these options, a score on this scale, a yes/no probability), it can score every possible answer in a single forward pass.

<figure class="jv-fig">
<div class="jv-gen">
  <div class="jv-lane seq wm-anim">
    <div class="lh">LLM: one token at a time</div>
    <div class="jv-toks">
      <span class="jv-tok">It</span><span class="jv-tok">sounds</span><span class="jv-tok">like</span><span class="jv-tok">a</span><span class="jv-tok">billing</span><span class="jv-tok">issue</span><span class="jv-tok">, so</span><span class="jv-tok">...</span>
    </div>
    <div class="ln">hundreds of sequential passes → seconds, billed per output token</div>
  </div>
  <div class="jv-lane par wm-anim">
    <div class="lh">Jev: all answers at once</div>
    <div class="jv-toks">
      <span class="jv-tok">billing 0.98</span><span class="jv-tok">tone: angry</span><span class="jv-tok">urgency: high</span>
    </div>
    <div class="ln">one pass over a fixed answer set → milliseconds, output not metered</div>
  </div>
</div>
<figcaption>The top row appears one token at a time, that's the LLM's decode loop, the real bottleneck. The bottom row lands all at once. When there's no token stream to generate, there's no per-token output cost, which is why Typesafe meters input only.</figcaption>
</figure>

Two consequences fall straight out of this. First, it's fast: Typesafe quotes 70 to 500 milliseconds end to end. Second, it structurally **can't hallucinate a field that doesn't exist** or emit malformed output, because strings were never the output format. If the only allowed answers are `calm` or `angry`, it cannot return `slightly miffed` or a broken bracket. That's a 0% *type-error* rate, guaranteed by construction, not by hoping.

## The three things you can ask it

You compose real logic out of three primitives. Choice and Score return their value plus a calibrated confidence; Noul returns a 0 to 1 probability that is itself the calibrated answer. You can bundle many into one call, and they all run in parallel and in isolation against the same state.

<figure class="jv-fig">
<div class="jv-prims wm-anim">
  <div class="jv-prim">
    <div class="pt">Choice</div>
    <div class="pd">Pick one from a fixed list of options (up to 255).</div>
    <div class="pe">tone → calm | angry</div>
  </div>
  <div class="jv-prim">
    <div class="pt">Score</div>
    <div class="pd">Rate against a scale or rubric you define.</div>
    <div class="pe">urgency → low | medium | high</div>
  </div>
  <div class="jv-prim">
    <div class="pt">Noul</div>
    <div class="pd">A calibrated yes/no, returned as a probability from 0 to 1.</div>
    <div class="pe">billing? → 0.98</div>
  </div>
</div>
<figcaption>Choice, Score, Noul. Typesafe's guidance: keep each question narrow and specific, then combine them in your own code, rather than asking one giant multi-part question.</figcaption>
</figure>

In practice a call reads like a set of typed questions asked of one piece of state:

```python
from typesafe_sdk import TypeSafeClient, Choice, Score, Noul

client = TypeSafeClient()
result = client.system_one(
    "I was charged twice. Please help ASAP.",
    {
        "billing": Noul(instructions="Is this about billing?"),
        "tone":    Choice(instructions="What is the tone?",
                          criteria={"calm": None, "angry": None}),
        "urgency": Score(instructions="How urgent is this?",
                         criteria=["low", "medium", "high"]),
    },
)

result.nouls["billing"].noul   # 0.98
result.choices["tone"].choice  # "angry"
result.scores["urgency"].score # "high"
```

## Confidence you can actually branch on

Here's the part that matters more than the speed. Jev is trained with a method Typesafe calls RLCD, Reinforcement Learning for Calibrated Decisions. Where ChatGPT-style RLHF rewards answers humans *prefer* (which quietly trains confident-sounding over honest), RLCD rewards the confidence numbers being *right on average*, using proper scoring rules like the Brier score.

Calibration means exactly what a good weather forecast means: on the days a forecaster says "70% chance of rain," it should rain about 70% of the time. Applied to your code, that's the useful bit, a confidence score you can set a threshold on.

<figure class="jv-fig">
<div class="jv-cal wm-anim">
  <div class="ct">when Jev says X% confident, it's right about X% of the time</div>
  <div class="jv-calrow"><span class="said">says 95%</span><span class="track"><span class="fill" style="--w:95%"></span></span><span class="act">act</span></div>
  <div class="jv-calrow"><span class="said">says 80%</span><span class="track"><span class="fill" style="--w:80%"></span></span><span class="act">act</span></div>
  <div class="jv-calrow"><span class="said">says 55%</span><span class="track"><span class="fill" style="--w:55%"></span></span><span class="act">escalate</span></div>
  <div class="cfoot">High confidence, let the code act. Low confidence, route it to a human or a slower model. The score is a real dial, not decoration.</div>
</div>
<figcaption>An overconfident model gives you a number you can't trust, so you can't threshold on it. A calibrated one turns "how sure are you?" into an if-statement.</figcaption>
</figure>

## I ran it, so here's real data, not a claim

I got an API key and pointed Jev at 320 product reviews where I already knew the true sentiment, including 19 deliberately mixed ones. The whole run: **320 decisions in 7.8 seconds**, **97.8% accurate**, and it cost me **under half a cent** (Jev bills input only, and this run was about 92k input tokens). The same 320 decisions on a mid-tier LLM, made to emit a structured answer each time, works out to roughly **72x more expensive** on token pricing alone.

But the number I actually care about is this one. When I split Jev's confidence by whether a review was clear-cut or genuinely mixed, the confidence *tracked the ambiguity on its own*:

<figure class="jv-fig">
<div class="jv-demo wm-anim">
  <div class="dh">Jev's average confidence, split by how clear the review was (real run, n=320)</div>
  <div class="jv-drow clear"><span class="dlab">clear-cut reviews</span><span class="dtrack"><span class="dfill" style="--w:98.1%"></span></span><span class="dval">98%</span></div>
  <div class="jv-drow amb"><span class="dlab">genuinely mixed reviews</span><span class="dtrack"><span class="dfill" style="--w:64.2%"></span></span><span class="dval">64%</span></div>
  <div class="dex">
    <div><span class="q">"Overpriced and underwhelming in every way."</span> → negative, <span class="c hi">0.99</span></div>
    <div><span class="q">"Loved the location, hated the noise."</span> → <span class="c lo">0.53</span>, it's genuinely torn, and it says so</div>
  </div>
</div>
<figcaption>Nobody told Jev which reviews were ambiguous. It reported high confidence on the obvious ones and low confidence on the truly mixed ones, entirely on its own. That gap is the whole product: you can auto-act on the 98% ones and route the 53% ones to a human. Ask an LLM the same and it'll say "99%" to both.</figcaption>
</figure>

This is the thing an LLM can't hand you. Not the answer, LLMs classify sentiment fine, but a confidence number honest enough to *branch on*, produced fast and cheap enough to run over your whole dataset. That combination is what "System One" is for.

**Try it yourself:** I put live versions up at **[jev-live-demo-production.up.railway.app](https://jev-live-demo-production.up.railway.app)**, each runs Jev **and a real LLM side by side**:

- **The sentiment demo** (the home page): the same review to both at once. Type a genuinely mixed one ("loved the location, hated the noise") and watch the gaps: Jev answers in ~150ms and honestly drops to around 50%, while the LLM takes over a second and still says 80%. That overconfidence, live and next to Jev, is the whole argument in one screen.
- **[The maze race](https://jev-live-demo-production.up.railway.app/)**: two runners navigate the same maze, each move a decision. Jev decides ~10 a second and reaches the goal; the LLM stalls, because you can't put a one-second, few-cents call inside a real-time loop.
- **[Jev routing WebMCP tools](https://jev-live-demo-production.up.railway.app/webmcp)**: a page declaring 13 real [WebMCP](/blog/2026-08-04-webmcp-teaching-websites-to-talk-to-ai-agents/) tools. Say what you want in plain words; Jev picks the tool with a calibrated confidence and gates the one that acts in your name, while the LLM picks a tool but gives you no honest number to gate on.

Typesafe's own headline numbers put the speed and cost side on one picture. Read them as vendor claims, not gospel (more on that below), but the *shape* matches what I measured:

<figure class="jv-fig">
<div class="jv-chart wm-anim">
  <div class="clab">Typesafe's published claims: <b>a customer-support triage decision</b></div>
  <div class="jv-crow llm"><span class="cn">LLM latency</span><span class="ct"><span class="cb" style="--w:100%"></span></span><span class="cv">3 to 329 s</span></div>
  <div class="jv-crow jev"><span class="cn">Jev latency</span><span class="ct"><span class="cb" style="--w:2%"></span></span><span class="cv">70 to 500 ms</span></div>
  <div class="jv-crow llm"><span class="cn">LLM cost / Mtok</span><span class="ct"><span class="cb" style="--w:100%"></span></span><span class="cv">$0.20 to $10</span></div>
  <div class="jv-crow jev"><span class="cn">Jev cost / Mtok</span><span class="ct"><span class="cb" style="--w:1%"></span></span><span class="cv">$0.042 + free out</span></div>
  <div class="cnote">bars are relative, not to scale across the two pairs · 20 to 200x faster, and cheaper still, on decision-shaped tasks</div>
</div>
<figcaption>The gap is roughly two orders of magnitude on both axes. That is the entire reason to care: not that Jev is smarter, but that for a small decision it is dramatically faster and cheaper than making an LLM write one.</figcaption>
</figure>

## The pitch, in one table

<figure class="jv-fig">
<div class="jv-tab-wrap">
<table class="jv-tab">
<thead><tr><th></th><th>Frontier LLM</th><th>Jev (System One)</th></tr></thead>
<tbody>
<tr><td>Output</td><td>Text you parse</td><td>A typed value, ready to use</td></tr>
<tr><td>How it runs</td><td>Token by token, in sequence</td><td>All answers in one pass</td></tr>
<tr><td>Latency (Typesafe's numbers)</td><td>3 to 329 seconds</td><td>70 to 500 ms</td></tr>
<tr><td>Cost</td><td><code>$0.20</code> to <code>$10</code> / Mtok in, output ~5x more</td><td><code>$0.042</code> / Mtok in, output free</td></tr>
<tr><td>Malformed / invalid output</td><td>Possible, needs retries</td><td>Impossible by construction (0% type errors)</td></tr>
<tr><td>Confidence</td><td>Often overconfident, hard to trust</td><td>Calibrated, safe to threshold on</td></tr>
<tr><td>Best at</td><td>Open-ended writing, reasoning, code</td><td>Fast structured decisions: classify, route, score, extract</td></tr>
</tbody>
</table>
</div>
<figcaption>Numbers are Typesafe's own published claims. They're not competing with LLMs at writing, they're competing at the small decisions you currently overpay an LLM to make.</figcaption>
</figure>

Good places to reach for it: classification, routing, scoring, extraction, real-time loops (their demo plays Doom at ~10 decisions a second), map-reducing a judgment over a big dataset, and guardrailing an LLM's output. Bad places: anything that needs to *write*, reason step by step, or hold a conversation. The thing that makes it fast is the thing that makes it narrow.

## Where I'd be skeptical

I like this a lot, and I want to be honest about what isn't proven yet, because it's early and the marketing is loud.

**The benchmarks aren't independent.** Typesafe evaluated Jev with "workflow evals" that compare its answers to the *average of two frontier LLMs*, not to verified ground truth. That measures agreement with other models, not correctness. If both reference models are wrong, Jev can "win" while being wrong too. On at least one chart they shared, its raw accuracy sat below a strong LLM even as it dominated the cost-and-speed frontier.

**"Can't hallucinate" is a precise, narrow claim.** It means it can't emit an invalid *type*, a value outside the allowed set. It can absolutely still be *wrong*: pick a valid option that's the incorrect one. Typesafe says as much in a follow-up titled "Where Jev Actually Fails." Type-safe is not the same as correct.

**There's no paper.** As of now the architecture and the RLCD loss function are unpublished. The CEO has confirmed RLCD exists by name; the rest ("encoder-only? diffusion?") is informed speculation. Treat the internals as a black box for now.

**Free output "too cheap to meter"** is a launch price, and the founder openly says they can't yet prove it isn't subsidized. Nice while it lasts; don't architect around it being free forever.

## The idea worth keeping

Even if you never use Jev, the framing is the takeaway: a lot of what we bolt LLMs onto isn't a writing task, it's a decision task wearing a writing task's clothes. We've been paying for a paragraph and a JSON parser when we wanted a typed value and a confidence score. A model shaped like the decision, fast, bounded, calibrated, is a genuinely good idea. Whether Jev is the one that nails it is what the next year will tell.

*Sources, all primary: Typesafe's [launch post](https://typesafe.ai/blog/introducing-system-one-models-and-jev), their [docs](https://docs.typesafe.ai/), and the code example from the [Python SDK docs](https://docs.typesafe.ai/sdk/python/usage.md). Written from scratch; nothing copied.*

<script>
(function(){
  var els=document.querySelectorAll('.jv-hero,.jv-lane,.jv-prims,.jv-chart,.jv-cal,.jv-demo');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.18});
  els.forEach(function(e){io.observe(e)});
})();
</script>
