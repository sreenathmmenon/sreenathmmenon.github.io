---
title: "Designing an AI Tool to Debug a 5GB Tarball of Logs"
date: 2026-08-15
excerpt: "A support engineer gets handed a 5GB tar. Inside: nested tars from log rotation, 20+ microservices each in its own folder, each rotated on its own rules, and somewhere in there, the reason a customer is angry. The obvious move is to throw an LLM at it. That's also the fastest way to build something that lies to you during an incident. Here's how I'd actually architect an on-prem, AI-assisted log tool where the whole design rests on one uncomfortable idea: the LLM is the least trustworthy part, so give it the least to do."
tags: [ai, system-design, observability, on-prem, agents, rag]
---

<style>
.lg-fig{margin:2.5rem 0;}
.lg-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.8rem;text-align:center;line-height:1.5;}

/* the inverted-trust thesis banner */
.lg-thesis{max-width:640px;margin:0 auto;border:1px solid var(--accent);border-radius:14px;background:var(--surface);padding:1.3rem 1.4rem;text-align:center;}
.lg-thesis .big{font-family:var(--font-display);font-size:1.35rem;color:var(--accent);line-height:1.3;margin-bottom:.5rem;}
.lg-thesis p{font-size:.88rem;color:var(--text-2);line-height:1.55;margin:0;}

/* the architecture pipeline */
.lg-pipe{max-width:760px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.lg-stage{display:flex;align-items:center;gap:.85rem;border:1px solid var(--border);border-radius:11px;background:var(--surface);padding:.75rem .95rem;opacity:0;transform:translateX(-10px);transition:opacity .5s var(--ease),transform .5s var(--ease);}
.lg-pipe.go .lg-stage{opacity:1;transform:none;}
.lg-pipe.go .lg-stage:nth-child(1){transition-delay:.05s} .lg-pipe.go .lg-stage:nth-child(2){transition-delay:.14s} .lg-pipe.go .lg-stage:nth-child(3){transition-delay:.23s} .lg-pipe.go .lg-stage:nth-child(4){transition-delay:.32s} .lg-pipe.go .lg-stage:nth-child(5){transition-delay:.41s} .lg-pipe.go .lg-stage:nth-child(6){transition-delay:.5s}
.lg-stage .tag{flex:none;font-family:var(--font-mono);font-size:.64rem;letter-spacing:.04em;text-transform:uppercase;border-radius:5px;padding:.18rem .5rem;min-width:76px;text-align:center;}
.lg-stage .tag.code{color:var(--accent);border:1px solid var(--accent);}
.lg-stage .tag.model{color:var(--accent-2);border:1px solid var(--accent-2);}
.lg-stage b{color:var(--text);font-size:.9rem;} .lg-stage .d{font-size:.82rem;color:var(--text-2);line-height:1.4;}
.lg-stage .body{flex:1;}

/* rotation reconciliation visual */
.lg-rot{max-width:640px;margin:0 auto;border:1px solid var(--border);border-radius:12px;background:var(--surface);padding:1.1rem;}
.lg-rot .row{display:flex;align-items:center;gap:.5rem;font-family:var(--font-mono);font-size:.74rem;margin:.35rem 0;flex-wrap:wrap;}
.lg-rot .file{border:1px solid var(--border-2);border-radius:6px;padding:.15rem .45rem;color:var(--text-3);}
.lg-rot .file.live{color:var(--accent);border-color:var(--accent);}
.lg-rot .arrow{color:var(--text-3);}
.lg-rot .note{font-size:.8rem;color:var(--text-2);margin-top:.7rem;line-height:1.5;}
.lg-rot .note b{color:var(--text);}

/* guardrails */
.lg-guards{max-width:720px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.lg-guard{display:flex;gap:.85rem;border:1px solid var(--border);border-radius:10px;background:var(--surface-2);padding:.75rem .95rem;align-items:flex-start;opacity:0;transform:translateY(8px);transition:opacity .45s var(--ease),transform .45s var(--ease);}
.lg-guards.go .lg-guard{opacity:1;transform:none;}
.lg-guards.go .lg-guard:nth-child(1){transition-delay:.06s} .lg-guards.go .lg-guard:nth-child(2){transition-delay:.14s} .lg-guards.go .lg-guard:nth-child(3){transition-delay:.22s} .lg-guards.go .lg-guard:nth-child(4){transition-delay:.30s} .lg-guards.go .lg-guard:nth-child(5){transition-delay:.38s} .lg-guards.go .lg-guard:nth-child(6){transition-delay:.46s}
.lg-guard .n{flex:none;font-family:var(--font-mono);font-size:.72rem;color:var(--accent);border:1px solid var(--accent);border-radius:6px;padding:.12rem .45rem;margin-top:.05rem;}
.lg-guard p{font-size:.85rem;color:var(--text-2);line-height:1.5;margin:0;} .lg-guard p b{color:var(--text);}

/* eval table */
.lg-tab-wrap{max-width:780px;margin:0 auto;overflow-x:auto;border:1px solid var(--border);border-radius:12px;background:var(--surface);}
.lg-tab{width:100%;border-collapse:collapse;font-size:.84rem;min-width:600px;}
.lg-tab th,.lg-tab td{text-align:left;padding:.6rem .8rem;border-bottom:1px solid var(--border);vertical-align:top;}
.lg-tab th{font-family:var(--font-mono);font-size:.68rem;text-transform:uppercase;letter-spacing:.04em;color:var(--text-3);font-weight:500;}
.lg-tab td:first-child{color:var(--text);font-weight:500;font-family:var(--font-mono);font-size:.78rem;}
.lg-tab tr:last-child td{border-bottom:none;}
.lg-tab .det{color:var(--green);font-family:var(--font-mono);font-size:.74rem;} .lg-tab .judge{color:var(--accent-2);font-family:var(--font-mono);font-size:.74rem;}

/* model split */
.lg-split{max-width:700px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:1rem;}
@media(max-width:600px){.lg-split{grid-template-columns:1fr;}}
.lg-col{border:1px solid var(--border);border-radius:13px;background:var(--surface);overflow:hidden;opacity:0;transform:translateY(10px);transition:opacity .5s var(--ease),transform .5s var(--ease);}
.lg-split.go .lg-col{opacity:1;transform:none;} .lg-split.go .lg-col:nth-child(2){transition-delay:.16s;}
.lg-col .h{padding:.75rem 1rem;border-bottom:1px solid var(--border);font-family:var(--font-mono);font-size:.8rem;font-weight:600;}
.lg-col.small .h{color:var(--accent);} .lg-col.big .h{color:var(--accent-2);}
.lg-col .b{padding:.9rem 1rem;font-size:.84rem;color:var(--text-2);line-height:1.5;}
.lg-col .b .li{padding-left:1.1rem;position:relative;margin:.35rem 0;}
.lg-col .b .li::before{content:"\2713";position:absolute;left:0;color:var(--accent);font-family:var(--font-mono);}

/* surfaces */
.lg-surf{max-width:640px;margin:0 auto;display:grid;grid-template-columns:repeat(3,1fr);gap:.7rem;}
@media(max-width:560px){.lg-surf{grid-template-columns:1fr;}}
.lg-s{border:1px solid var(--border-2);border-radius:12px;background:var(--surface);padding:1rem;text-align:center;opacity:0;transform:scale(.95);transition:opacity .45s var(--ease),transform .45s var(--ease);}
.lg-surf.go .lg-s{opacity:1;transform:none;}
.lg-surf.go .lg-s:nth-child(1){transition-delay:.08s} .lg-surf.go .lg-s:nth-child(2){transition-delay:.2s} .lg-surf.go .lg-s:nth-child(3){transition-delay:.32s}
.lg-s .t{font-family:var(--font-mono);font-size:.82rem;color:var(--accent);font-weight:600;margin-bottom:.3rem;}
.lg-s .d{font-size:.8rem;color:var(--text-2);line-height:1.45;}

@media (prefers-reduced-motion: reduce){
  .lg-stage,.lg-guard,.lg-col,.lg-s{opacity:1!important;transform:none!important;}
}
</style>

A support engineer gets an email: *"customer's checkout is failing intermittently, logs attached."* Attached is a 5GB tar. Inside it: more tars, because log rotation packaged the older logs as nested archives. Inside *those*: 20-plus microservices, each in its own folder, each rotated on its own schedule with its own naming and its own timestamp format. Somewhere in that pile is the reason checkout is failing. The engineer has maybe an hour before the customer escalates.

This is a real problem, and it's a great one to design an AI tool for, on the condition that you're honest about what the AI can and can't be trusted to do. Because the obvious build, "pipe the logs into an LLM and ask what's wrong," is also the fastest way to ship something that confidently lies to an engineer in the middle of an incident. And a confident wrong answer during an incident isn't a neutral miss. It sends someone down a three-hour dead end while the customer keeps escalating.

So this post is how I'd actually architect it, for the constraint that matters most in this world: **on-prem**. The logs can't leave the customer's box. The available model varies wildly per customer, some have a real GPU and a capable local model, some have a small model on CPU, some are fully air-gapped. And the whole thing rests on one idea I want to put up front, because everything else follows from it.

<figure class="lg-fig">
<div class="lg-thesis">
  <div class="big">The LLM is the least trustworthy part of this system. So give it the least to do.</div>
  <p>Deterministic code establishes the facts, parsing, indexing, correlation, ordering. The model only narrates over evidence it is forced to cite. Invert that, thin parsing and a fat prompt, and you get a demo that dazzles and a tool that hallucinates in the field.</p>
</div>
</figure>

That isn't a hot take. It's what the serious observability teams actually do. Two of the best-known "AI root cause" features, Datadog's Watchdog and Grafana's Sift, are classical statistics and ML, not LLMs at all. The LLM, where there is one, just puts their findings into English. Keep that in mind every time someone sells you "AI for your logs."

## Step zero: the boring part is the hard part

Before any intelligence, you have to turn that 5GB mess into something answerable. This is where most of the real engineering lives, and it's genuinely nasty.

**A tar has no index.** Unlike a zip (which has a directory you can seek to), a tar is just header, data, header, data, with no map. You cannot cheaply pull one service's logs out of a 5GB bundle; you either stream it member by member in one pass, or build your own offset index first. And a `.tar.gz` is one continuous compressed stream, so you can't even seek by byte without special handling, and gzip can't be decompressed in parallel, so it's often the serial bottleneck of the whole pipeline. The rule is: **stream, never "extract all."** Feed a nested tar's inner stream straight into another reader, and guard against the classics, decompression bombs (a nested gzip can expand from megabytes to hundreds of gigabytes) and tar-slip path traversal (`../../etc/passwd`).

**Log rotation is a trap.** You'd think reassembling `app.log`, `app.log.1`, `app.log.2.gz` into one timeline is trivial. It isn't. Default logrotate numbering runs *backwards* in time (higher number = older), so a naive alphabetical sort is both reversed and wrong (it orders `.1, .10, .2`). Some services compress on a delay, so compressed and uncompressed rotations coexist. Some use dates instead of numbers. And `copytruncate` can drop or duplicate lines right at the seam. Across 20 independently-configured services, you'll meet all of these in one bundle.

<figure class="lg-fig">
<div class="lg-rot">
  <div class="row"><span class="arrow">oldest</span><span class="file">app.log.4.gz</span><span class="arrow">&rarr;</span><span class="file">app.log.3.gz</span><span class="arrow">&rarr;</span><span class="file">app.log.2.gz</span><span class="arrow">&rarr;</span><span class="file">app.log.1</span><span class="arrow">&rarr;</span><span class="file live">app.log</span><span class="arrow">newest</span></div>
  <div class="note"><b>Filename order is a hint. Content timestamps are the (weak) truth.</b> Merge the rotated files by parsed timestamp, de-duplicate at overlap seams, and, this is the part that matters, <b>flag the gaps</b> instead of presenting a clean-looking timeline that's silently missing an hour.</div>
</div>
<figcaption>Reconciling one service's rotated logs into a coherent timeline. Now do it for 20 services, each configured differently, then merge those into one cross-service view.</figcaption>
</figure>

Then normalize every timestamp to UTC, and be humble about it: normalization fixes *how* a time is written, not *whether the clock was right*. NTP drift of ~150 milliseconds is enough to make effect look like it came before cause. And remember a stack trace is one event across forty lines with a timestamp only on the first, so multi-line assembly has to happen *before* rotation stitching, or you tear a trace across a seam.

None of this is glamorous. All of it is load-bearing. Get it wrong and every clever thing downstream is reasoning over corrupted facts.

## The architecture: code finds the truth, the model explains it

Here's the whole pipeline. Read the tags on the left: almost every stage is deterministic code. The model appears exactly once, at the end, working only on evidence the code already found and structured.

<figure class="lg-fig">
<div class="lg-pipe wm-anim">
  <div class="lg-stage"><span class="tag code">code</span><div class="body"><b>Ingest</b><div class="d">Stream the nested tars, assemble multi-line events, stitch rotations per service, normalize timestamps to UTC, flag gaps.</div></div></div>
  <div class="lg-stage"><span class="tag code">code</span><div class="body"><b>Extract structure</b><div class="d">Pull errors, exceptions, log levels, stack traces, and correlation / trace IDs. Mine templates (Drain-style) so millions of lines become a few hundred patterns.</div></div></div>
  <div class="lg-stage"><span class="tag code">code</span><div class="body"><b>Redact</b><div class="d">Strip tokens, keys, emails, IPs before anything is indexed or shown to a model. This is a hard gate, not an output filter.</div></div></div>
  <div class="lg-stage"><span class="tag code">code</span><div class="body"><b>Index</b><div class="d">Every line gets a stable ID and its exact <span class="mono">service : file : line @ timestamp</span>. Build a correlation-ID index to stitch a request across services. Hybrid search: keyword + vector.</div></div></div>
  <div class="lg-stage"><span class="tag code">code</span><div class="body"><b>Correlate + rank</b><div class="d">For a time window or a trace, pull the real lines. Find the first error vs the downstream cascade. Rank the root, not the loudest.</div></div></div>
  <div class="lg-stage"><span class="tag model">model</span><div class="body"><b>Explain</b><div class="d">The LLM narrates the incident over the already-found evidence, and must cite every claim to a real line ID. That's its entire job.</div></div></div>
</div>
<figcaption>The deterministic core does the work; the model does the writing. The payoff: swap the last box for a frontier local model or a quantized 7B and nothing upstream changes. Better hardware buys a richer explanation, not different facts.</figcaption>
</figure>

Two design choices in there deserve a spotlight, because they're where the accuracy actually comes from.

**Hybrid search, not just vectors.** For logs, pure semantic search is weak. If an engineer searches for `ERR_CONN_RST` or a specific correlation ID, vector similarity returns passages that are *semantically nearby* but may never contain the literal string. Keyword search nails exact error codes and IDs; vector search catches paraphrase. You run both and fuse the rankings. In log analysis, keyword search never stopped being essential, and anyone who replaced it wholesale with embeddings regretted it.

**The loudest error is usually the victim, not the cause.** When one service fails, the failure climbs *up* the call tree through timeouts and retries. The service screaming the most errors is typically the user-facing one at the top, timing out because something deep and quiet broke first. So root-cause localization means finding the *earliest* anomalous event on the *most upstream* service in the dependency graph, not the most frequent or most recent error. This is exactly the judgment an exhausted engineer gets wrong at 2am, and exactly where deterministic correlation earns its place, before the model ever speaks.

## Guardrails: accuracy is the entire product

For most AI features, a guardrail is a safety net. Here it *is* the product. An answer you can't verify is worse than no answer. These are the guardrails I'd consider non-negotiable.

<figure class="lg-fig">
<div class="lg-guards wm-anim">
  <div class="lg-guard"><span class="n">1</span><p><b>Cite from the index, never emit a raw line.</b> The model references evidence by line ID; the code then checks each quoted line byte-for-byte against the index and rejects any non-match before it reaches the user. "Did it invent a timestamp?" becomes a deterministic string-equality check, not a hope.</p></div>
  <div class="lg-guard"><span class="n">2</span><p><b>Abstention is a first-class, rewarded answer.</b> "I don't have logs showing that" must be an acceptable, even encouraged, output. Models hallucinate partly because evals score a confident wrong answer the same as an honest "I don't know", so guessing is rational. Reward the refusal and you get fewer confident fabrications.</p></div>
  <div class="lg-guard"><span class="n">3</span><p><b>Scope-locked to this tar.</b> The tool answers only from the provided bundle. No training-data memory, no "in general, checkout failures are usually…". If it isn't in these logs, it doesn't exist for this answer.</p></div>
  <div class="lg-guard"><span class="n">4</span><p><b>Redaction before the model, not after.</b> Logs leak bearer tokens, API keys, IPs, emails. Scrub at ingestion (tools like Presidio do the detection). The honest catch: redaction is a precision/recall trap, over-redact and you destroy the correlating key you needed; under-redact and you leak a secret. Layer regex + entity detection + allowlists, and never claim "fully scrubbed".</p></div>
  <div class="lg-guard"><span class="n">5</span><p><b>Flag, don't paper over.</b> Missing hour in a service's timeline? Duplicate lines at a rotation seam? Clock skew between two services? Surface it. A tool that shows a clean, confident, silently-incomplete timeline is more dangerous than one that says "there's a 12-minute gap here I can't see into."</p></div>
  <div class="lg-guard"><span class="n">6</span><p><b>Grounding is not correctness.</b> The uncomfortable one. A faithfully-quoted wrong log line is still wrong, and a citation proves a pointer exists, not that it supports the claim. Even frontier models top out around 85% grounded factuality on "answer only from this document" tasks. Design for a human who verifies, not one who trusts.</p></div>
</div>
<figcaption>Notice how many of these are deterministic checks wrapped around the model, not pleas to the model. That's the pattern: constrain with code, don't hope with prompts.</figcaption>
</figure>

## Can you even evaluate this? Yes, and mostly without an LLM

If you can't measure it, you can't trust it, and "the demo looked good" is not measurement. The good news: most of what matters here is checkable deterministically.

<figure class="lg-fig">
<div class="lg-tab-wrap">
<table class="lg-tab">
<thead><tr><th>What you measure</th><th>Why it matters</th><th>How</th></tr></thead>
<tbody>
<tr><td>Retrieval recall@k</td><td>Did the right evidence lines even get pulled? Most failures are retrieval failures.</td><td class="det">deterministic</td></tr>
<tr><td>Citation validity</td><td>Do the cited line IDs exist and match the source byte-for-byte?</td><td class="det">deterministic</td></tr>
<tr><td>Abstention correctness</td><td>Does it say "I don't know" when there's no evidence, and only then?</td><td class="det">deterministic</td></tr>
<tr><td>Root-cause accuracy</td><td>Did it name the actual originating fault, against a labeled incident?</td><td class="judge">judge + labels</td></tr>
<tr><td>Groundedness</td><td>Does each cited line actually support the claim it's attached to?</td><td class="judge">LLM-as-judge</td></tr>
<tr><td>False-positive rate</td><td>How often does it assert a confident wrong cause? Measure separately from over-abstention.</td><td class="judge">judge + labels</td></tr>
</tbody>
</table>
</div>
<figcaption>Lean on the green rows. Citation-exists, quote-matches-source, recall against labeled evidence, abstention-when-empty, all cheap, reproducible, and impossible to game. Use an LLM judge only for the genuinely semantic rows, and validate that judge against human labels first, or it's an opinion, not a metric.</figcaption>
</figure>

## The golden dataset: possible, with an honest asterisk

You asked whether a golden dataset is even feasible here. It is, and the cleanest way is to **cause the faults yourself.** Inject a known failure (chaos-engineering tools do this), capture the resulting tar, and label the true root cause plus the exact evidence lines. That gives you near-free, precise ground truth at volume. Supplement it with a smaller set of replayed real incidents to keep the eval honest, because injected faults are suspiciously clean and a model can overfit tidy signatures. There are public log datasets too (Loghub and friends), though they're mostly labeled for parsing and anomaly detection, not full root-cause chains.

Now the asterisk, because pretending this is easy would be the dishonest part:

- **Real incidents rarely have one clean cause.** Good postmortems say "contributing causes," plural. A single-label golden set encodes a simplification and will unfairly punish a model that names a real *contributing* cause. Let labels accept a *set* of acceptable causes and evidence lines.
- **Which line is "the evidence"?** For a chain like DB timeout ← pool exhaustion ← slow dependency, the true evidence is several lines across several services. One gold line isn't enough.
- **Sometimes the decisive event was never logged.** Then ground truth requires inferring a cause with no evidence line, which directly conflicts with a tool that correctly refuses to answer without evidence. Your abstention metric and your golden labels can legitimately disagree, and you have to decide, in advance, which one wins.
- **Labelers disagree, and postmortems are written under hindsight.** Measure inter-annotator agreement; the "official" cause is sometimes just the convenient one.

That's not a reason to skip the golden set. It's a reason to build it with humility and to report accuracy with its caveats, not as a single triumphant number. For context, the best *deployed* root-cause accuracy in the research I trust sits around 0.77, and that was with hand-built, per-category diagnostic handlers, not a clever prompt. Anyone claiming near-perfect automated RCA is selling something.

## On-prem: one engine, any model, any surface

The on-prem constraint is what makes the deterministic-first design non-optional, and it turns out to be a feature. Because the model only does thin work at the end, the tool degrades gracefully across whatever hardware a customer has.

<figure class="lg-fig">
<div class="lg-split wm-anim">
  <div class="lg-col small">
    <div class="h">A small local model is enough for</div>
    <div class="b">
      <div class="li">Structured extraction and error categorization</div>
      <div class="li">Template and known-error matching</div>
      <div class="li">Short summaries of an already-selected cluster</div>
      <div class="li">Anything single-shot with schema-constrained output</div>
    </div>
  </div>
  <div class="lg-col big">
    <div class="h">A bigger model earns its keep for</div>
    <div class="b">
      <div class="li">Multi-step synthesis across many services</div>
      <div class="li">The root-cause narrative and the causal chain</div>
      <div class="li">Weighing competing hypotheses</div>
      <div class="li">Long, multi-hop reasoning where errors compound</div>
    </div>
  </div>
</div>
<figcaption>The dividing line: single-shot structural transforms go to the small model; long reasoning where mistakes snowball wants a bigger one. And on the tiniest air-gapped box, the model step is optional, the deterministic core still emits correlated evidence and categorized errors. That's a downgrade, not a failure.</figcaption>
</figure>

The last piece is delivery, and here the customers genuinely differ: some want a command line, some want it wired into their AI IDE, some want a UI. The answer is to build the deterministic core as a library with a stable API, then put thin adapters on top. Same engine, three faces.

<figure class="lg-fig">
<div class="lg-surf wm-anim">
  <div class="lg-s"><div class="t">CLI</div><div class="d">For ops, cron, CI, and air-gapped boxes. The deterministic outputs need no model at all.</div></div>
  <div class="lg-s"><div class="t">MCP server</div><div class="d">Expose <span class="mono">search_logs</span>, <span class="mono">correlate_by_trace</span>, <span class="mono">explain_incident</span> as tools, and the customer's own AI IDE drives your engine.</div></div>
  <div class="lg-s"><div class="t">UI</div><div class="d">A web frontend over the same API: a timeline, the evidence, the citations, for people who want to see it.</div></div>
</div>
<figcaption>One core, three adapters. MCP is the interesting one: it turns your engine into tools any capable AI IDE can call, so the customer brings their own model and your tool becomes its hands. (If you're new to MCP, I wrote about it <a href="/blog/2026-06-30-mcp-the-port-that-let-ai-touch-the-world/">here</a> and <a href="/blog/2026-08-04-webmcp-teaching-websites-to-talk-to-ai-agents/">here</a>.)</figcaption>
</figure>

## The honest bottom line

If you take one thing from this: **the hard, valuable engineering is the deterministic core, not the prompt.** Streaming nested tars, reconciling 20 services' worth of rotated logs into a trustworthy timeline, correlating a request across services, finding the earliest upstream fault instead of the loudest downstream symptom, redacting secrets without destroying evidence. Do that well, and even a modest local model can write a genuinely useful, cited explanation on top. Skip it and reach for a bigger model, and you've built something that sounds like an expert and misleads like a stranger.

There are real ceilings, and a credible tool names them: grounding isn't correctness, the best deployed root-cause accuracy is far from perfect, redaction can't promise to catch everything, and if the decisive event was never logged, no tool can conjure it, instrumentation is the ceiling, not model cleverness. The right product doesn't hide those. It shows its evidence, flags its gaps, abstains when it's blind, and keeps the human holding the judgment.

That's the tool I'd actually want handed to me at 2am with a 5GB tar and an angry customer. Not one that tells me what's wrong. One that shows me, line by cited line, what the logs actually say, and admits what they don't.

## References

Written from scratch after reading the primary sources; the engineering claims and honest limitations trace to these. Nothing here is copied from them.

- Drain log parser (He et al., ICWS 2017) and Drain3: <https://github.com/logpai/Drain3>
- Honeycomb, the hard stuff nobody talks about (LLMs): <https://www.honeycomb.io/blog/hard-stuff-nobody-talks-about-llm>
- logrotate configuration reference: <https://man7.org/linux/man-pages/man5/logrotate.conf.5.html>
- W3C Trace Context (correlating requests across services): <https://www.w3.org/TR/trace-context/>
- Reciprocal Rank Fusion (Cormack et al., SIGIR 2009): <https://cormack.uwaterloo.ca/cormacksigir09-rrf.pdf>
- RCACopilot, grounded LLM root-cause (Microsoft, EuroSys 2024): <https://arxiv.org/abs/2305.15778>
- Why Language Models Hallucinate (2025): <https://arxiv.org/abs/2509.04664>
- Microsoft Presidio (PII/secret redaction): <https://github.com/microsoft/presidio>
- Loghub, real-world log datasets: <https://github.com/logpai/loghub>
- Model Context Protocol, tools spec: <https://modelcontextprotocol.io/specification/2025-06-18/server/tools>

*Related system-design pieces: [designing a RAG system that actually retrieves](/blog/2026-07-24-designing-a-rag-system-that-actually-retrieves/), [designing an agent that doesn't go off the rails](/blog/2026-07-24-designing-an-agent-that-doesnt-go-off-the-rails/), and [LLM security](/blog/2026-08-02-llm-security-prompt-injection-and-the-cost-of-defending/) for the redaction and prompt-injection angles.*

<script>
(function(){
  var els=document.querySelectorAll('.lg-pipe,.lg-guards,.lg-split,.lg-surf');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.18});
  els.forEach(function(e){io.observe(e)});
})();
</script>
