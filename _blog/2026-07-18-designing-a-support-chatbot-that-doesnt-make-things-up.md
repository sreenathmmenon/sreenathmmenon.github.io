---
title: "Designing a Support Chatbot That Doesn't Make Things Up"
date: 2026-07-18
excerpt: "A company hands you a pile of help docs and asks for a chatbot that answers customer questions from them, accurately, with sources, and without inventing policies that don't exist. It sounds simple. It is not. This is a full walk through how I'd design it: every approach with its honest pros and cons, and, at the end, how DoorDash and LinkedIn actually built theirs, with links to their own write-ups."
tags: [ai, rag, chatbot, architecture, use-case, explainer]
---

<style>
.cb-fig{margin:2.5rem 0;}
.cb-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* naive vs grounded */
.cb-vs{display:grid;grid-template-columns:1fr 1fr;gap:.9rem;max-width:640px;margin:0 auto;}
@media(max-width:560px){.cb-vs{grid-template-columns:1fr;}}
.cb-vs .col{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.cb-vs .col h4{margin:0 0 .5rem;font-size:.85rem;font-family:var(--font-mono);}
.cb-vs .col.bad h4{color:var(--text-2);} .cb-vs .col.good h4{color:var(--accent);}
.cb-vs .col.good{border-color:var(--accent);}
.cb-vs .col p{font-size:.83rem;color:var(--text-2);margin:0;line-height:1.5;}
.cb-vs .col .tag{font-family:var(--font-mono);font-size:.72rem;margin-top:.5rem;}
.cb-vs .col.bad .tag{color:var(--text-3);} .cb-vs .col.good .tag{color:var(--accent);}

/* approach with pros/cons */
.cb-appr{max-width:660px;margin:0 auto;display:flex;flex-direction:column;gap:.8rem;}
.cb-a{border:1px solid var(--border);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.cb-appr.go .cb-a{opacity:1;transform:none;}
.cb-appr.go .cb-a:nth-child(1){transition-delay:.1s} .cb-appr.go .cb-a:nth-child(2){transition-delay:.3s} .cb-appr.go .cb-a:nth-child(3){transition-delay:.5s} .cb-appr.go .cb-a:nth-child(4){transition-delay:.7s}
.cb-a .hd{display:flex;align-items:center;gap:.6rem;margin-bottom:.5rem;}
.cb-a .num{flex:none;width:26px;height:26px;border-radius:50%;background:var(--surface-2);border:1px solid var(--border-2);color:var(--text-2);font-family:var(--font-mono);font-weight:600;font-size:.8rem;display:flex;align-items:center;justify-content:center;}
.cb-a.win .num{background:var(--grad);color:var(--accent-ink);border-color:transparent;}
.cb-a .nm{font-family:var(--font-mono);font-size:.88rem;color:var(--text);font-weight:600;} .cb-a.win .nm{color:var(--accent);}
.cb-a .vd{font-family:var(--font-mono);font-size:.7rem;margin-left:auto;padding:.15rem .5rem;border-radius:999px;border:1px solid var(--border-2);color:var(--text-3);}
.cb-a.win .vd{border-color:var(--accent);color:var(--accent);}
.cb-a .desc{font-size:.84rem;color:var(--text-2);line-height:1.5;margin-bottom:.5rem;}
.cb-a .pc{display:grid;grid-template-columns:1fr 1fr;gap:.5rem;}
@media(max-width:480px){.cb-a .pc{grid-template-columns:1fr;}}
.cb-a .pc .p,.cb-a .pc .c{font-size:.78rem;line-height:1.4;}
.cb-a .pc .p{color:var(--text-2);} .cb-a .pc .p b{color:var(--accent);}
.cb-a .pc .c{color:var(--text-2);} .cb-a .pc .c b{color:var(--text-3);}

/* stages: build vs answer time */
.cb-time{display:grid;grid-template-columns:1fr 1fr;gap:.8rem;max-width:640px;margin:0 auto;}
@media(max-width:560px){.cb-time{grid-template-columns:1fr;}}
.cb-time .c{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.cb-time .c h4{margin:0 0 .5rem;font-size:.84rem;font-family:var(--font-mono);}
.cb-time .c.prep h4{color:var(--text-2);} .cb-time .c.live h4{color:var(--accent);}
.cb-time .c.live{border-color:var(--accent);}
.cb-time .c .l{font-size:.82rem;color:var(--text-2);padding:.2rem 0;}
.cb-time .c .l .a{color:var(--accent);}

/* the pipeline */
.cb-pipe{max-width:660px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.cb-pstep{display:flex;align-items:center;gap:.9rem;border:1px solid var(--border);border-radius:10px;padding:.65rem .9rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.cb-pipe.go .cb-pstep{opacity:1;transform:none;}
.cb-pipe.go .cb-pstep:nth-child(1){transition-delay:.1s} .cb-pipe.go .cb-pstep:nth-child(2){transition-delay:.24s}
.cb-pipe.go .cb-pstep:nth-child(3){transition-delay:.38s} .cb-pipe.go .cb-pstep:nth-child(4){transition-delay:.52s}
.cb-pipe.go .cb-pstep:nth-child(5){transition-delay:.66s} .cb-pipe.go .cb-pstep:nth-child(6){transition-delay:.8s}
.cb-pstep .n{flex:none;width:28px;height:28px;border-radius:50%;background:var(--grad);color:var(--accent-ink);font-family:var(--font-mono);font-weight:600;font-size:.8rem;display:flex;align-items:center;justify-content:center;}
.cb-pstep .t{font-size:.85rem;color:var(--text-2);} .cb-pstep .t b{color:var(--text);}
.cb-pstep.gate{border-color:var(--accent);border-style:dashed;} .cb-pstep.gate .t b{color:var(--accent);}

/* hybrid retrieval */
.cb-hyb{max-width:600px;margin:0 auto;display:flex;gap:.6rem;align-items:stretch;justify-content:center;flex-wrap:wrap;}
.cb-hyb .lane{flex:1;min-width:150px;border:1px solid var(--border-2);border-radius:10px;padding:.8rem;background:var(--surface);text-align:center;}
.cb-hyb .lane b{display:block;font-size:.82rem;color:var(--text);margin-bottom:.15rem;}
.cb-hyb .lane span{font-size:.75rem;color:var(--text-2);line-height:1.4;}
.cb-hyb .merge{align-self:center;font-family:var(--font-mono);font-size:.72rem;color:var(--accent);text-align:center;}

/* confidence gate */
.cb-gate{max-width:560px;margin:0 auto;}
.cb-gate .q{text-align:center;font-family:var(--font-mono);font-size:.82rem;color:var(--accent);border:1px solid var(--accent);border-radius:10px;padding:.6rem;margin-bottom:.8rem;}
.cb-gate .branches{display:grid;grid-template-columns:1fr 1fr;gap:.7rem;}
@media(max-width:480px){.cb-gate .branches{grid-template-columns:1fr;}}
.cb-gate .b{border:1px solid var(--border-2);border-radius:10px;padding:.8rem;text-align:center;background:var(--surface);}
.cb-gate .b .lab{font-family:var(--font-mono);font-size:.74rem;margin-bottom:.3rem;}
.cb-gate .b.weak .lab{color:var(--text-3);} .cb-gate .b.strong .lab{color:var(--accent);}
.cb-gate .b span{font-size:.8rem;color:var(--text-2);line-height:1.4;}
.cb-gate .b.strong{border-color:var(--accent);}

/* guardrails stack */
.cb-guard{max-width:620px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:.7rem;}
@media(max-width:520px){.cb-guard{grid-template-columns:1fr;}}
.cb-guard .c{border:1px solid var(--border-2);border-radius:12px;padding:.85rem 1rem;background:var(--surface);}
.cb-guard .c .k{font-family:var(--font-mono);font-size:.68rem;color:var(--accent);margin-bottom:.25rem;}
.cb-guard .c b{display:block;color:var(--text);font-size:.85rem;margin-bottom:.2rem;}
.cb-guard .c span{font-size:.79rem;color:var(--text-2);line-height:1.45;}

/* company case cards */
.cb-case{max-width:660px;margin:0 auto;display:flex;flex-direction:column;gap:.8rem;}
.cb-co{border:1px solid var(--accent);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);}
.cb-co .hd{display:flex;align-items:baseline;gap:.6rem;margin-bottom:.4rem;flex-wrap:wrap;}
.cb-co .name{font-family:var(--font-mono);font-size:.95rem;color:var(--accent);font-weight:600;}
.cb-co .stat{font-family:var(--font-mono);font-size:.72rem;color:var(--text);background:var(--surface-2);border:1px solid var(--border-2);border-radius:999px;padding:.15rem .55rem;}
.cb-co p{font-size:.85rem;color:var(--text-2);line-height:1.55;margin:.3rem 0;}
.cb-co .src{font-family:var(--font-mono);font-size:.74rem;margin-top:.3rem;}

/* table */
.cb-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.cb-tab th,.cb-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.cb-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.cb-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}

/* extra reveals */
.cb-vs .col{opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.cb-vs.go .col{opacity:1;transform:none;}
.cb-vs.go .col:nth-child(1){transition-delay:.1s} .cb-vs.go .col:nth-child(2){transition-delay:.35s}

.cb-guard .c{opacity:0;transform:translateY(8px);transition:opacity .45s ease,transform .45s ease;}
.cb-guard.go .c{opacity:1;transform:none;}
.cb-guard.go .c:nth-child(1){transition-delay:.1s} .cb-guard.go .c:nth-child(2){transition-delay:.25s}
.cb-guard.go .c:nth-child(3){transition-delay:.4s} .cb-guard.go .c:nth-child(4){transition-delay:.55s}

.cb-case .cb-co{opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.cb-case.go .cb-co{opacity:1;transform:none;}
.cb-case.go .cb-co:nth-child(1){transition-delay:.1s} .cb-case.go .cb-co:nth-child(2){transition-delay:.4s}

@media (prefers-reduced-motion: reduce){
  .cb-pipe .cb-pstep,.cb-appr .cb-a,.cb-vs .col,.cb-guard .c,.cb-case .cb-co{transition:none !important;opacity:1 !important;transform:none !important;}
}
</style>

Here's a request that lands on a lot of engineers' desks, in one form or another. A company has a big pile of help documentation, product guides, policies, FAQs, troubleshooting steps, and they want a chatbot that answers customer questions *from those docs.* Accurately. With links to the source. And, above all, without confidently inventing a refund policy that doesn't exist.

That last part is the whole challenge. If you just hand a raw AI model your question, it'll answer from its general training, which knows nothing about *your* specific return window or *your* shipping rules, and it will happily make something up that sounds right. For a support bot, a made-up answer isn't a cute mistake. It's a promise your company now has to honor, or an angry customer, or worse.

So let me walk through how I'd actually design this system, thoroughly. I'll lay out every approach with its honest pros and cons, build up the real pipeline piece by piece, and, at the very end, show how two big companies, DoorDash and LinkedIn, actually built theirs, with links to their own write-ups so you can read the primary sources. Because the interesting engineering is entirely in the details that stop it from making things up.

## First, what does "good" mean here?

Before any architecture, pin down what success looks like, because it drives every decision:

- **Grounded.** Every answer comes from the company's actual docs, not the model's imagination.
- **Sourced.** The bot can show *which* document it used, so a human can verify it.
- **Honest.** If the docs don't cover the question, it says "I don't know, let me get a human," instead of guessing.
- **Fast and affordable.** Customers won't wait ten seconds, and the company won't pay a fortune per chat.
- **Safe.** No leaking private data, no answering things it shouldn't, no showing one customer another customer's info.

Hold those five. We'll judge every choice against them.

## The four ways you could approach this

Before diving into one design, it's worth seeing the whole menu, because people often grab the wrong option. Here are the four real approaches, each with its honest pros and cons.

<figure class="cb-fig">
  <div class="cb-appr" id="appr">
    <div class="cb-a">
      <div class="hd"><div class="num">1</div><div class="nm">Just the raw model</div><div class="vd">the trap</div></div>
      <div class="desc">Send the question straight to an AI model and show whatever it says. Zero setup.</div>
      <div class="pc"><div class="p"><b>+</b> trivial to build, instant</div><div class="c"><b>&minus;</b> knows nothing about your docs, invents policies, no sources</div></div>
    </div>
    <div class="cb-a">
      <div class="hd"><div class="num">2</div><div class="nm">Fine-tune a model on your docs</div><div class="vd">wrong tool</div></div>
      <div class="desc">Retrain a model on all your help content so it "knows" your policies.</div>
      <div class="pc"><div class="p"><b>+</b> answers feel native, no lookup step</div><div class="c"><b>&minus;</b> expensive, goes stale the moment a doc changes, can't cite sources, still hallucinates</div></div>
    </div>
    <div class="cb-a">
      <div class="hd"><div class="num">3</div><div class="nm">Basic RAG (retrieve then answer)</div><div class="vd">good start</div></div>
      <div class="desc">Find the relevant doc chunks at question-time, hand them to the model, answer from them.</div>
      <div class="pc"><div class="p"><b>+</b> grounded, sourced, cheap, updates when docs do</div><div class="c"><b>&minus;</b> one-shot; no quality check, so weak retrieval still produces confident wrong answers</div></div>
    </div>
    <div class="cb-a win">
      <div class="hd"><div class="num">4</div><div class="nm">Production RAG (retrieve, rerank, gate, guard)</div><div class="vd">what I'd ship</div></div>
      <div class="desc">Basic RAG plus reranking, a confidence gate that refuses when unsure, guardrails, and constant evaluation.</div>
      <div class="pc"><div class="p"><b>+</b> reliable, honest, safe, measurable, the version that survives real customers</div><div class="c"><b>&minus;</b> more moving parts to build and maintain</div></div>
    </div>
  </div>
  <figcaption>Four approaches, and the naming ("basic to advanced") is misleading, the right lens is which one meets the five criteria. Approach 1 fails all of them. Approach 2, fine-tuning, is the one people most often reach for wrongly: it bakes facts into the model that go stale instantly and can't be cited (I dug into why in my fine-tuning-vs-RAG post). Approach 3 is genuinely good and where you start. Approach 4 is what production actually needs. Let me build up to it.</figcaption>
</figure>

Notice why **fine-tuning is the classic wrong turn** here. It feels like "teach the model our business," but policies change weekly, and a fine-tuned model can't tell you *which document* an answer came from, so a customer can't verify it and you can't audit it. Facts belong in *retrieval*, where you can swap a document and the answer updates instantly. Fine-tuning is for changing *behavior* (tone, format), not for teaching *facts*. That single distinction rules out approach 2 for this job.

## The naive version, and why grounding is the fix

The first instinct is approach 1: paste the question in, show the answer. Here's the fix that separates a toy from something real.

<figure class="cb-fig">
  <div class="cb-vs" id="vs">
    <div class="col bad">
      <h4>Naive: model answers from memory</h4>
      <p>Customer asks, the raw model answers from its general training. It doesn't know your policies, so it invents plausible-sounding ones. No source, no way to check.</p>
      <div class="tag">confident, unsourced, often wrong</div>
    </div>
    <div class="col good">
      <h4>Grounded: answer from your docs</h4>
      <p>Before answering, the system finds the relevant passages from your actual docs and makes the model answer <em>only</em> from those, with a citation.</p>
      <div class="tag">accurate, sourced, checkable</div>
    </div>
  </div>
  <figcaption>The core move behind every reliable AI answer: don't let the model recall from fuzzy memory, make it <em>read</em> the real document first. That's RAG (retrieval-augmented generation), the backbone of this design. But basic RAG alone (approach 3) still isn't enough for production, and the gap is exactly what we fill next.</figcaption>
</figure>

## The shape of the system: prepare once, answer live

There are two distinct phases, and keeping them separate is the first design insight. One happens *once, ahead of time* (getting the docs ready to search). The other happens *every time a customer asks*.

<figure class="cb-fig">
  <div class="cb-time">
    <div class="c prep">
      <h4>Prep phase (once, offline)</h4>
      <div class="l">take all the help docs</div>
      <div class="l">split them into small chunks</div>
      <div class="l">turn each chunk into a searchable form</div>
      <div class="l">store them in a searchable index</div>
    </div>
    <div class="c live">
      <h4>Answer phase (every question)</h4>
      <div class="l"><span class="a">find</span> the chunks relevant to the question</div>
      <div class="l"><span class="a">rerank</span> them, keep the best few</div>
      <div class="l"><span class="a">check</span> they're actually good enough</div>
      <div class="l"><span class="a">answer</span> from them, with a citation</div>
    </div>
  </div>
  <figcaption>The expensive "read and index all the docs" work happens once, offline. The live path only does a quick search plus one model call, which is why it can be fast and cheap. Get this split wrong, re-indexing everything on every question, and your bot is slow and expensive for no reason.</figcaption>
</figure>

## The live path, step by step

Here's what happens the moment a customer types a question. Six steps, and each one earns its place.

<figure class="cb-fig">
  <div class="cb-pipe" id="pipe">
    <div class="cb-pstep"><div class="n">1</div><div class="t"><b>Understand the question.</b> Turn "how long do I have to return something?" into a search the system can run over the docs. In a multi-message chat, first condense the whole conversation into the real underlying issue.</div></div>
    <div class="cb-pstep"><div class="n">2</div><div class="t"><b>Retrieve candidates.</b> Pull the most relevant chunks from the index, say the top 20. Cast a wide net first; precision comes next.</div></div>
    <div class="cb-pstep"><div class="n">3</div><div class="t"><b>Rerank.</b> A sharper (slower) model re-scores those 20 and keeps the best 3 to 5. Fast-but-rough retrieval finds candidates; careful reranking picks the winners.</div></div>
    <div class="cb-pstep gate"><div class="n">4</div><div class="t"><b>Confidence gate.</b> Are the top chunks actually relevant? If they're weak, stop here and say "I don't know" or hand off to a human, instead of forcing a bad answer.</div></div>
    <div class="cb-pstep"><div class="n">5</div><div class="t"><b>Generate, grounded.</b> Hand the few good chunks to the model with strict instructions: answer only from this text, and cite it. Nothing invented.</div></div>
    <div class="cb-pstep"><div class="n">6</div><div class="t"><b>Answer with a source.</b> Return the reply plus a link to the doc it came from, so the customer (and the company) can trust and verify it.</div></div>
  </div>
  <figcaption>Six steps, and the ones people skip are 3 and 4. Basic RAG jumps from "retrieve" straight to "answer." The reranking and the confidence gate are exactly what turn a flaky demo into something you'd let touch real customers. Let me unpack the two choices that matter most, and weigh their trade-offs.</figcaption>
</figure>

## Choice #1: how you search, and the trade-off

You'd think "search by meaning" (using embeddings, which match on *concepts* not exact words) is all you need. It's powerful, it treats "return something" and "send an item back" as the same idea. But it has a blind spot: exact terms. A customer typing a specific error code, a product SKU, or an order number needs an *exact keyword* match, which pure meaning-based search can fumble.

<figure class="cb-fig">
  <div class="cb-hyb">
    <div class="lane"><b>Meaning search</b><span>matches concepts and synonyms. Great for "how do I get a refund?"</span></div>
    <div class="merge">+<br>combine</div>
    <div class="lane"><b>Keyword search</b><span>matches exact terms. Great for "error E-4021" or an order ID.</span></div>
  </div>
  <figcaption>The robust answer is <em>hybrid</em> search: run both and combine. <b>Pro:</b> meaning-search catches paraphrases, keyword-search catches exact codes the other misses, together they cover far more questions. <b>Con:</b> it's a bit more to build and tune than a single method. The trade is almost always worth it, because "the bot couldn't find it" is one of the most common real failures, and hybrid quietly fixes a whole class of it.</figcaption>
</figure>

## Choice #2: the confidence gate, the anti-hallucination valve

This is the single most important part of the design, and the one that separates a trustworthy bot from a dangerous one. After retrieval, before generating, ask one question: *are the documents we found actually relevant enough to answer from?*

<figure class="cb-fig">
  <div class="cb-gate">
    <div class="q">Are the retrieved docs strongly relevant to the question?</div>
    <div class="branches">
      <div class="b weak">
        <div class="lab">weak / nothing good</div>
        <span>Don't force an answer. Say "I couldn't find that, let me connect you to a person." Honest beats wrong.</span>
      </div>
      <div class="b strong">
        <div class="lab">strong match</div>
        <span>Go ahead and answer, grounded in those docs, with a citation. This is the safe case.</span>
      </div>
    </div>
  </div>
  <figcaption>The confidence gate is the valve that stops the bot from making things up. <b>Pro:</b> it converts "confidently wrong" into "honestly escalated," which is exactly the behavior that builds trust. <b>Con:</b> tuned too strict, it refuses questions it could have answered and annoys customers; tuned too loose, hallucinations slip through. Finding that threshold is real work, but a support bot that occasionally says "let me get a human" is trustworthy; one that never admits it doesn't know is a liability.</figcaption>
</figure>

## The safety layer you cannot skip

For a real support bot touching real customers, a few guardrails aren't optional extras, they're the line between shippable and reckless.

<figure class="cb-fig">
  <div class="cb-guard" id="guard">
    <div class="c"><div class="k">who-can-see</div><b>Access controls</b><span>Tag each doc with who's allowed to see it, and filter retrieval by the user. A customer must never get an answer built from another customer's data, or an internal-only doc.</span></div>
    <div class="c"><div class="k">privacy</div><b>PII handling</b><span>Strip or protect personal data (emails, card numbers) so it doesn't leak into prompts or logs. Support chats are full of it.</span></div>
    <div class="c"><div class="k">stay-on-topic</div><b>Out-of-scope refusal</b><span>Politely decline questions outside its job (medical advice, competitor comparisons, anything off-policy) instead of improvising.</span></div>
    <div class="c"><div class="k">no-tricks</div><b>Injection defense</b><span>Guard against users trying to trick the bot into ignoring its rules or revealing its hidden instructions. Treat user text as untrusted.</span></div>
  </div>
  <figcaption>The unglamorous parts that never show up in a demo but decide whether the thing can go live. Access control especially: getting retrieval to respect "who is allowed to see what" is a real engineering task, and skipping it is how companies leak data.</figcaption>
</figure>

## How you know it's working: evaluation

Here's what separates people who've actually shipped these from people who've only read about them: **you cannot improve what you don't measure.** A support bot needs a way to tell, continuously, whether it's getting better or quietly regressing.

<figure class="cb-fig">
<table class="cb-tab">
<thead><tr><th>Question</th><th>How you measure it</th></tr></thead>
<tbody>
<tr><td>Did retrieval find the right doc?</td><td>Test on known question-answer pairs, check if the right source shows up</td></tr>
<tr><td>Is the answer faithful to the source?</td><td>Score whether the answer actually matches the cited doc (no invention)</td></tr>
<tr><td>Are customers happy?</td><td>Thumbs up/down, escalation rate, resolved-without-a-human rate</td></tr>
<tr><td>Is it fast and affordable?</td><td>Track response time and cost per conversation over time</td></tr>
<tr><td>Did a change make it worse?</td><td>Re-run the test set before every change; catch regressions early</td></tr>
</tbody>
</table>
  <figcaption>Evaluation turns "it seemed fine in the demo" into "we know it's good and we'll know the instant it breaks." A quiet truth: human review still beats automated scoring for judging real answer quality, so the best setups keep a human reviewing a sample of real conversations. Without measurement, you're flying blind, and as you'll see next, the companies that got this right invested heavily here.</figcaption>
</figure>

## Start simple, add only what you need

One more piece of judgment, because it's easy to over-build. You don't start with all of this on day one. You start with the simplest thing that could work, ship it, watch where it fails, and add complexity *only where the failures are.*

<figure class="cb-fig">
<table class="cb-tab">
<thead><tr><th>Add this...</th><th>...only when</th></tr></thead>
<tbody>
<tr><td>Basic retrieve-then-answer</td><td>Always. This is the starting point.</td></tr>
<tr><td>Reranking</td><td>Retrieval keeps surfacing near-misses; answers feel slightly off.</td></tr>
<tr><td>Hybrid (keyword) search</td><td>Customers ask about exact codes, IDs, product names the bot misses.</td></tr>
<tr><td>Confidence gate</td><td>Basically always for support, it's your anti-hallucination valve.</td></tr>
<tr><td>Access controls</td><td>The moment any doc is not meant for every user. Usually day one.</td></tr>
</tbody>
</table>
  <figcaption>The discipline: don't bolt on reranking, agents, and elaborate pipelines before you've seen the actual failure modes. Ship the simple version, measure it, and let the real problems tell you what to build next. Over-engineering on day one is how simple projects die slowly.</figcaption>
</figure>

## How real companies actually built this

Everything above isn't theory, it's close to what large companies have published about their own production support bots. Here are two, in their own words, with links to their official write-ups so you can read the primary sources.

<figure class="cb-fig">
  <div class="cb-case" id="case">
    <div class="cb-co">
      <div class="hd"><span class="name">DoorDash</span><span class="stat">hallucinations down ~90%</span><span class="stat">compliance issues down ~99%</span></div>
      <p>DoorDash built a RAG support system for their delivery drivers ("Dashers"). When a Dasher reports a problem, the system first <em>condenses the whole conversation</em> into the core issue (exactly step 1 above), then retrieves similar past resolved cases plus the relevant help articles, and generates a grounded answer.</p>
      <p>The part worth stealing: they wrapped it in <em>two extra layers</em>. An <b>LLM Guardrail</b> that checks every answer is actually grounded in the retrieved docs and doesn't violate policy (the confidence-gate idea, taken further), and an <b>LLM Judge</b> that scores quality across five aspects (retrieval correctness, response accuracy, coherence, and more) to catch regressions. Those two layers are what drove the 90% and 99% drops.</p>
      <p class="src">Official write-up: <a href="https://careersatdoordash.com/blog/large-language-modules-based-dasher-support-automation/">DoorDash Engineering, "Path to high-quality LLM-based Dasher support automation"</a></p>
    </div>
    <div class="cb-co">
      <div class="hd"><span class="name">LinkedIn</span><span class="stat">resolution time down ~28.6%</span><span class="stat">deployed ~6 months</span></div>
      <p>LinkedIn's customer-service team took a clever twist on the retrieval step. Instead of treating past support tickets as a flat pile of text, they built a <em>knowledge graph</em> from them, capturing the structure <em>inside</em> each ticket and the relationships <em>between</em> tickets. When a question comes in, they retrieve the relevant piece of that graph, not just loose text chunks.</p>
      <p>The payoff: because the retrieval understands how issues relate, answers are better grounded. They reported a 77.6% jump in a retrieval-quality metric and, deployed in their real support team for about six months, a 28.6% cut in median time to resolve an issue. It's a great example of the idea that <em>better retrieval, not a fancier model, is usually the biggest lever.</em></p>
      <p class="src">Official paper: <a href="https://arxiv.org/abs/2404.17723">Xu et al., "Retrieval-Augmented Generation with Knowledge Graphs for Customer Service Question Answering" (LinkedIn, SIGIR 2024)</a></p>
    </div>
  </div>
  <figcaption>Two real production systems, two lessons. DoorDash's is "wrap generation in verification layers", a guardrail and a judge, and hallucinations collapse. LinkedIn's is "the retrieval step is where the real wins hide", structure your knowledge better and everything downstream improves. Both validate the same spine this whole post is built on: ground it, check it, measure it. Read their write-ups; they're clear and detailed.</figcaption>
</figure>

## The takeaway

A support chatbot that "answers from our docs" sounds like a weekend project, and the naive version is. But the version that won't embarrass the company is a careful little pipeline: prepare the docs once, then for each question, search (by meaning *and* by keyword), rerank to the best few, *check that they're actually good enough*, and only then answer, grounded, with a source, wrapped in guardrails that respect who can see what, and measured constantly so you know it's working.

And notice what the real companies confirm: DoorDash's biggest wins came from *verification layers*, not a bigger model; LinkedIn's came from *better retrieval*, not a bigger model. The lesson underneath the whole thing is the one that runs through all of this, the model is the easy part. The engineering is everything *around* it, the retrieval, the reranking, the honesty gate, the safety, the measurement, that makes it trustworthy enough to put in front of a real customer. Get the model to *read the docs and admit when it can't find the answer*, and you've solved 90% of what makes these systems either wonderful or dangerous.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['appr','pipe','vs','guard','case'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
