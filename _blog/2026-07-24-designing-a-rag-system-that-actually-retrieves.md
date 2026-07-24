---
title: "Designing a RAG System That Actually Retrieves"
date: 2026-07-24
excerpt: "Everyone can bolt a vector database onto an LLM and call it RAG. The hard part is the retrieve step actually finding the right paragraph, out of thousands of internal docs that changed this morning, with the user's permissions respected and a citation on the answer. This is the full design, box by box, plus the cloud services that fill each box on AWS, GCP, and Azure."
tags: [ai, rag, system-design, retrieval, embeddings, architecture]
---

<style>
.rag-fig{margin:2.5rem 0;}
.rag-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* pipeline flow */
.rag-flow{max-width:680px;margin:0 auto;display:flex;flex-direction:column;gap:1.3rem;}
.rag-flow .path{display:flex;flex-direction:column;gap:.5rem;}
.rag-flow .plabel{font-family:var(--font-mono);font-size:.72rem;letter-spacing:.06em;text-transform:uppercase;color:var(--accent);}
.rag-flow .nodes{display:flex;align-items:stretch;gap:.4rem;flex-wrap:wrap;}
.rag-flow .node{flex:1;min-width:96px;border:1px solid var(--border-2);border-radius:10px;padding:.6rem .55rem;background:var(--surface-2);text-align:center;opacity:0;transform:translateY(8px);transition:opacity .45s ease,transform .45s ease;}
.rag-flow.go .node{opacity:1;transform:none;}
.rag-flow.go .path:nth-child(2) .node:nth-child(1){transition-delay:.05s}
.rag-flow.go .path:nth-child(2) .node:nth-child(2){transition-delay:.15s}
.rag-flow.go .path:nth-child(2) .node:nth-child(3){transition-delay:.25s}
.rag-flow.go .path:nth-child(2) .node:nth-child(4){transition-delay:.35s}
.rag-flow.go .path:nth-child(2) .node:nth-child(5){transition-delay:.45s}
.rag-flow.go .path:nth-child(4) .node:nth-child(1){transition-delay:.55s}
.rag-flow.go .path:nth-child(4) .node:nth-child(2){transition-delay:.65s}
.rag-flow.go .path:nth-child(4) .node:nth-child(3){transition-delay:.75s}
.rag-flow.go .path:nth-child(4) .node:nth-child(4){transition-delay:.85s}
.rag-flow.go .path:nth-child(4) .node:nth-child(5){transition-delay:.95s}
.rag-flow .node .n{display:block;font-family:var(--font-mono);font-size:.66rem;color:var(--accent);margin-bottom:.2rem;}
.rag-flow .node b{display:block;font-size:.8rem;color:var(--text);line-height:1.25;}
.rag-flow .node span{display:block;font-size:.68rem;color:var(--text-3);margin-top:.15rem;line-height:1.3;}
.rag-flow .node.live{background:var(--surface);border-color:var(--accent);}
.rag-flow .bridge{text-align:center;font-family:var(--font-mono);font-size:.7rem;color:var(--text-3);}

/* naive vs structure-aware chunking */
.rag-chunk{max-width:660px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:.9rem;}
@media(max-width:560px){.rag-chunk{grid-template-columns:1fr;}}
.rag-chunk .col{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.rag-chunk .col h4{margin:0 0 .6rem;font-size:.82rem;font-family:var(--font-mono);}
.rag-chunk .col.bad h4{color:var(--text-2);} .rag-chunk .col.good h4{color:var(--accent);}
.rag-chunk .col.good{border-color:var(--accent);}
.rag-chunk .doc{display:flex;flex-direction:column;gap:.3rem;}
.rag-chunk .line{height:9px;border-radius:3px;background:var(--surface-2);position:relative;overflow:hidden;}
.rag-chunk .line i{position:absolute;inset:0;width:0;background:var(--grad);opacity:.5;transition:width .6s ease;}
.rag-chunk.go .line i{width:100%;}
.rag-chunk .cut{position:relative;margin:.15rem 0;}
.rag-chunk .cut::before{content:"";display:block;height:2px;background:var(--text-3);opacity:.6;}
.rag-chunk .cut.severed::before{background:repeating-linear-gradient(90deg,var(--text-3) 0 6px,transparent 6px 12px);}
.rag-chunk .cut .cl{position:absolute;right:0;top:-.9rem;font-family:var(--font-mono);font-size:.6rem;}
.rag-chunk .bad .cut .cl{color:var(--text-3);} .rag-chunk .good .cut .cl{color:var(--accent);}
.rag-chunk .col p{font-size:.78rem;color:var(--text-2);margin:.7rem 0 0;line-height:1.5;}

/* hybrid search */
.rag-hyb{max-width:640px;margin:0 auto;}
.rag-hyb .lanes{display:grid;grid-template-columns:1fr 1fr;gap:.8rem;}
@media(max-width:520px){.rag-hyb .lanes{grid-template-columns:1fr;}}
.rag-hyb .lane{border:1px solid var(--border-2);border-radius:12px;padding:.9rem;background:var(--surface);opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.rag-hyb.go .lane{opacity:1;transform:none;}
.rag-hyb.go .lane:nth-child(1){transition-delay:.1s} .rag-hyb.go .lane:nth-child(2){transition-delay:.3s}
.rag-hyb .lane .k{font-family:var(--font-mono);font-size:.72rem;color:var(--accent);margin-bottom:.15rem;}
.rag-hyb .lane b{display:block;font-size:.85rem;color:var(--text);margin-bottom:.4rem;}
.rag-hyb .lane .catch{font-size:.76rem;color:var(--text-2);line-height:1.45;}
.rag-hyb .lane .miss{font-size:.72rem;color:var(--text-3);margin-top:.4rem;line-height:1.4;}
.rag-hyb .chips{display:flex;flex-wrap:wrap;gap:.3rem;margin-top:.5rem;}
.rag-hyb .chip{font-family:var(--font-mono);font-size:.68rem;padding:.15rem .45rem;border-radius:6px;background:var(--surface-2);border:1px solid var(--border-2);color:var(--text-2);}
.rag-hyb .fuse{text-align:center;margin-top:.9rem;font-family:var(--font-mono);font-size:.76rem;color:var(--accent);border:1px solid var(--accent);border-radius:10px;padding:.55rem;opacity:0;transition:opacity .5s ease;transition-delay:.55s;}
.rag-hyb.go .fuse{opacity:1;}

/* retrieve then rerank funnel */
.rag-rr{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.rag-rr .stage{border:1px solid var(--border-2);border-radius:10px;padding:.7rem .9rem;background:var(--surface);display:flex;align-items:center;gap:.9rem;opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.rag-rr.go .stage{opacity:1;transform:none;}
.rag-rr.go .stage:nth-child(1){transition-delay:.1s} .rag-rr.go .stage:nth-child(2){transition-delay:.35s} .rag-rr.go .stage:nth-child(3){transition-delay:.6s}
.rag-rr .stage.narrow{border-color:var(--accent);}
.rag-rr .count{flex:none;width:64px;text-align:center;font-family:var(--font-mono);}
.rag-rr .count .big{display:block;font-size:1.25rem;color:var(--accent);font-weight:600;line-height:1;}
.rag-rr .count .sm{display:block;font-size:.62rem;color:var(--text-3);margin-top:.15rem;}
.rag-rr .body b{display:block;font-size:.84rem;color:var(--text);}
.rag-rr .body span{font-size:.76rem;color:var(--text-2);line-height:1.45;}
.rag-rr .bar{height:8px;border-radius:4px;background:var(--surface-2);margin-top:.4rem;overflow:hidden;}
.rag-rr .bar i{display:block;height:100%;width:0;background:var(--grad);transition:width .7s ease;}
.rag-rr.go .stage:nth-child(1) .bar i{width:100%;transition-delay:.2s}
.rag-rr.go .stage:nth-child(2) .bar i{width:32%;transition-delay:.45s}
.rag-rr.go .stage:nth-child(3) .bar i{width:12%;transition-delay:.7s}

/* cloud table */
.rag-tab-wrap{max-width:820px;margin:0 auto;overflow-x:auto;opacity:0;transition:opacity .6s ease;}
.rag-tab-wrap.go{opacity:1;}
.rag-tab{width:100%;border-collapse:collapse;font-size:.84rem;min-width:640px;}
.rag-tab th,.rag-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.rag-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.rag-tab td:first-child,.rag-tab th:first-child{color:var(--text);font-weight:500;font-family:var(--font-mono);font-size:.76rem;white-space:nowrap;}
.rag-tab tbody tr:hover{background:var(--surface-2);}

/* gotchas */
.rag-got{max-width:660px;margin:0 auto;display:flex;flex-direction:column;gap:.8rem;}
.rag-g{border:1px solid var(--border);border-left:3px solid var(--accent);border-radius:10px;padding:.9rem 1.05rem;background:var(--surface);opacity:0;transform:translateY(10px);transition:opacity .5s ease,transform .5s ease;}
.rag-got.go .rag-g{opacity:1;transform:none;}
.rag-got.go .rag-g:nth-child(1){transition-delay:.1s} .rag-got.go .rag-g:nth-child(2){transition-delay:.32s} .rag-got.go .rag-g:nth-child(3){transition-delay:.54s}
.rag-g .hd{display:flex;align-items:baseline;gap:.6rem;margin-bottom:.35rem;flex-wrap:wrap;}
.rag-g .tag{font-family:var(--font-mono);font-size:.7rem;color:var(--accent);text-transform:uppercase;letter-spacing:.05em;}
.rag-g b{font-size:.9rem;color:var(--text);}
.rag-g p{font-size:.82rem;color:var(--text-2);line-height:1.55;margin:0;}
.rag-g p .fix{color:var(--accent);font-family:var(--font-mono);font-size:.76rem;}

.rag-note{max-width:600px;margin:0 auto;text-align:center;border:1px solid var(--accent);border-radius:12px;padding:1rem 1.2rem;background:var(--surface);}
.rag-note p{font-size:.9rem;color:var(--text-2);line-height:1.6;margin:0;} .rag-note b{color:var(--accent);}

@media (prefers-reduced-motion: reduce){
  .rag-flow .node,.rag-chunk .line i,.rag-hyb .lane,.rag-hyb .fuse,.rag-rr .stage,.rag-rr .bar i,.rag-tab-wrap,.rag-got .rag-g{
    transition:none !important;opacity:1 !important;transform:none !important;width:auto;
  }
  .rag-chunk .line i{width:100%!important;}
  .rag-rr .stage:nth-child(1) .bar i{width:100%!important;} .rag-rr .stage:nth-child(2) .bar i{width:32%!important;} .rag-rr .stage:nth-child(3) .bar i{width:12%!important;}
}
</style>

Someone in your company types "what's our refund window on annual plans?" into a chat box and hits enter. They want one sentence back, correct, with a link to the actual policy doc, and they want it before they've finished reaching for their coffee. The policy changed last Tuesday. There are four thousand documents in the system and this person can only legally see about six hundred of them.

That's the job. Not "put an LLM on top of a vector store." Anyone can do that in an afternoon and demo it and feel great. The demo answers the three questions you tested it on and then someone asks the fourth question and it confidently cites a policy that was deleted in March.

This is post two in the [AI System Design on the Cloud](/blog/2026-07-24-three-ai-systems-same-design/) series. In [the first post](/blog/2026-07-24-three-ai-systems-same-design/) I made the case that RAG, agents, and recommendations are the same funnel: cheap wide recall, then expensive narrow precision, then policy. This post works the RAG funnel out in full, box by box, and names the cloud service that fills each box. The whole time, keep one thing in mind: **the model is rarely the problem. Retrieval is.**

## Why RAG exists at all

Quick sanity check, because people reach for RAG reflexively and sometimes they shouldn't.

You use retrieval-augmented generation when three things are true at once. Your corpus is too big to stuff into a context window, thousands of docs, no contest. Your content changes too often to bake into the model, you can't retrain because someone edited a pricing table this morning. And you need citations, an answer with no source is useless when the question is "what's our policy," because nobody will trust it and legal will have opinions.

RAG solves all three by keeping the knowledge outside the model. The docs live in an index. At question time you go find the relevant pieces and hand them to the model as context. Edit a doc, re-index that doc, done. The model never memorized anything, so it can't be out of date, as long as your index isn't.

That "as long as your index isn't" is doing a lot of work. Hold that thought.

## The two pipelines

The mistake beginners make is thinking of RAG as one flow. It's two, and they run at completely different times.

<figure class="rag-fig">
<div class="rag-flow rag-anim">
  <div class="path">
    <div class="plabel">Offline · ingestion (runs when docs change)</div>
  </div>
  <div class="path nodes">
    <div class="node"><span class="n">01</span><b>Connectors</b><span>pull from wikis, drives, tickets</span></div>
    <div class="node"><span class="n">02</span><b>Parse / OCR</b><span>PDFs, scans, tables to text</span></div>
    <div class="node"><span class="n">03</span><b>Chunk</b><span>split by structure</span></div>
    <div class="node"><span class="n">04</span><b>Embed</b><span>text to vectors</span></div>
    <div class="node"><span class="n">05</span><b>Upsert</b><span>into the index + metadata</span></div>
  </div>
  <div class="bridge">the two pipelines meet at the index</div>
  <div class="path">
    <div class="plabel">Online · query (runs on every question, in milliseconds)</div>
  </div>
  <div class="path nodes">
    <div class="node live"><span class="n">01</span><b>Understand</b><span>rewrite, expand query</span></div>
    <div class="node live"><span class="n">02</span><b>Hybrid retrieve</b><span>dense + sparse, filtered</span></div>
    <div class="node live"><span class="n">03</span><b>Rerank</b><span>cross-encoder, top ~50</span></div>
    <div class="node live"><span class="n">04</span><b>Generate</b><span>grounded answer + cites</span></div>
    <div class="node live"><span class="n">05</span><b>Guard / cache</b><span>safety, store result</span></div>
  </div>
</div>
<figcaption>Offline you prepare the index. Online you query it. Most of the interesting engineering is offline, where nobody's watching.</figcaption>
</figure>

The offline path is where quality is quietly won or lost. It runs on a schedule or on a change event, and it's allowed to be slow. The online path runs on every keystroke-ish and has a latency budget measured in a second or two. If you only optimize the online path, you're polishing the wrong pipeline. Let's walk the offline one first, because that's where the retrieval you're so worried about actually gets built.

## Chunking: where good RAG is won or lost

You can't embed a fifty-page document as one vector. It'd be mush, an average of everything, specific about nothing. So you split it into chunks. The naive way is to count characters: every 500 tokens, cut. Simple, and it quietly wrecks your retrieval.

<figure class="rag-fig">
<div class="rag-chunk rag-anim">
  <div class="col bad">
    <h4>Fixed-size split (every 500 tokens)</h4>
    <div class="doc">
      <div class="line"><i></i></div>
      <div class="line"><i></i></div>
      <div class="cut severed"><span class="cl">cut</span></div>
      <div class="line"><i></i></div>
      <div class="line"><i></i></div>
      <div class="cut severed"><span class="cl">cut mid-table</span></div>
      <div class="line"><i></i></div>
    </div>
    <p>The cut lands wherever the counter hits 500. A sentence gets sliced in half. A table's header ends up in one chunk and its rows in the next, so neither chunk means anything on its own.</p>
  </div>
  <div class="col good">
    <h4>Structure-aware split</h4>
    <div class="doc">
      <div class="line"><i></i></div>
      <div class="line"><i></i></div>
      <div class="cut"><span class="cl">section end</span></div>
      <div class="line"><i></i></div>
      <div class="line"><i></i></div>
      <div class="cut"><span class="cl">table kept whole</span></div>
      <div class="line"><i></i></div>
    </div>
    <p>Cuts follow the document's own seams: headings, paragraphs, table boundaries. Each chunk is a complete thought. A little overlap between chunks means a sentence spanning a boundary still survives in one of them.</p>
  </div>
</div>
<figcaption>Fixed-size chunking severs meaning at arbitrary points. Structure-aware chunking respects the seams the author already put there.</figcaption>
</figure>

Two more moves make chunking genuinely good. **Overlap:** let consecutive chunks share a little text, so an idea that straddles a boundary lives intact in at least one of them. And **contextual retrieval:** before you embed a chunk, prepend a one-line summary of the section and document it came from. "From the Refunds section of the 2026 Annual Plan Terms:" glued to the front of a chunk about the 30-day window turns an ambiguous fragment into something that retrieves cleanly, because now the chunk carries its own context instead of assuming the reader remembers what section they're in. It costs a bit at index time and pays you back on every query.

If you get one thing right in the whole system, get chunking right. A great model reranking bad chunks is a great chef working with spoiled ingredients.

## Retrieval: dense and sparse, together

Now the part everyone thinks *is* RAG. You've got the question, you need the right chunks. There are two ways to search, and the trap is picking one.

**Dense (semantic) search** embeds the query and finds chunks whose vectors are nearby. It understands meaning. "How long do I have to get my money back" finds the refund-window chunk even though it shares almost no words with it. But dense search is fuzzy about exact tokens. Ask about error code `TLS_0x8F` or SKU `AX-2200` and the embedding blurs it into "some error code, some product," and you get near-misses.

**Sparse (keyword / BM25) search** matches literal terms. It nails the error code, the SKU, the acronym, the exact product name. But ask it a paraphrase and it shrugs, because none of the words line up.

<figure class="rag-fig">
<div class="rag-hyb rag-anim">
  <div class="lanes">
    <div class="lane">
      <div class="k">Dense · semantic</div>
      <b>Catches meaning</b>
      <div class="catch">Paraphrases, synonyms, "get my money back" = "refund."</div>
      <div class="chips"><span class="chip">refund window</span><span class="chip">cancel &amp; get money back</span><span class="chip">return period</span></div>
      <div class="miss">Misses: exact tokens like TLS_0x8F, AX-2200, SOC 2.</div>
    </div>
    <div class="lane">
      <div class="k">Sparse · keyword</div>
      <b>Catches exact tokens</b>
      <div class="catch">Error codes, SKUs, acronyms, precise strings.</div>
      <div class="chips"><span class="chip">TLS_0x8F</span><span class="chip">AX-2200</span><span class="chip">SOC 2</span></div>
      <div class="miss">Misses: paraphrases with no shared words.</div>
    </div>
  </div>
  <div class="fuse">Rank fusion &rarr; one ranked list that catches both</div>
</div>
<figcaption>Dense and sparse fail in opposite directions. Run both, fuse the results, and cover each other's blind spots.</figcaption>
</figure>

So you run both and fuse the rankings, usually with reciprocal rank fusion, which is a fancy name for a simple, robust way of blending two ranked lists without having to calibrate their scores against each other. Hybrid retrieval isn't a nice-to-have. In a real corpus full of product names and codes and jargon, it's the difference between "usually finds it" and "finds it."

And this is exactly the stage where permissions belong. More on that in a second.

## Rerank: cheap and wide, then expensive and narrow

Hybrid search gives you a decent ranked list, but "decent" isn't good enough to feed the model. The top result by vector similarity is often the third-best answer to the actual question. So you add a second, smarter pass.

<figure class="rag-fig">
<div class="rag-rr rag-anim">
  <div class="stage">
    <div class="count"><span class="big">~50</span><span class="sm">candidates</span></div>
    <div class="body"><b>Retrieve wide (bi-encoder)</b><span>Cheap. Compares pre-computed vectors. Its only job: make sure the right chunk is somewhere in this pool.</span><div class="bar"><i></i></div></div>
  </div>
  <div class="stage narrow">
    <div class="count"><span class="big">rank</span><span class="sm">cross-encoder</span></div>
    <div class="body"><b>Rerank (reads query + chunk together)</b><span>Expensive per item, so you only run it on the ~50. It actually reads each chunk against the question and scores real relevance.</span><div class="bar"><i></i></div></div>
  </div>
  <div class="stage narrow">
    <div class="count"><span class="big">8</span><span class="sm">to the LLM</span></div>
    <div class="body"><b>Keep top-K</b><span>A tight, high-precision set. Fewer, better chunks beat a big pile of mediocre ones every time.</span><div class="bar"><i></i></div></div>
  </div>
</div>
<figcaption>Wide cheap recall, then narrow expensive precision. The cross-encoder is too slow to run on the whole corpus, which is the entire reason retrieval runs first.</figcaption>
</figure>

The bi-encoder in the retrieval step encodes the query and every chunk *separately* and compares vectors, which is fast because the chunk vectors were computed offline. A **cross-encoder** reranker is different: it takes the query and one chunk together, as a pair, and reads them jointly. That joint reading is dramatically more accurate at judging "does this chunk actually answer this question," and it's also far too slow to run against ten thousand chunks per query. So you don't. You let cheap retrieval narrow ten thousand down to fifty, then spend the expensive cross-encoder only on those fifty, and hand the model the top eight.

That's the funnel from post one, sitting right in the middle of RAG. Cheap wide net, expensive narrow judgment, and it's forced on you by the same constraint every time: your best step is too slow to run on everything.

## Permissions belong in retrieval, not generation

Here's the mistake that turns a clever system into a data-leak incident. It's tempting to retrieve everything, generate an answer, and *then* check whether the user was allowed to see the sources. That's backwards. By the time you're filtering the output, the model has already read documents this person can't access, and even a well-behaved model can leak the gist of a restricted doc in a paraphrased answer.

Permissions have to be a **metadata filter at retrieval time.** Every chunk carries the access tags of its source document. When the query runs, you filter the candidate set to only what this user can see *before* anything gets ranked or read. The model never touches a document the person isn't cleared for, so there's nothing to leak. Filter at the door, not on the way out.

## Grounding, citations, and the courage to say "I don't know"

Now the model finally speaks. Its instructions are narrow on purpose: answer *only* from the chunks provided, cite which chunk each claim came from, and if the chunks don't contain the answer, say so instead of guessing.

That last part is the hard cultural sell and the most important. A RAG system that says "I don't have that in the docs" when it genuinely doesn't is worth ten times more than one that produces a fluent, confident, wrong answer. Confident and wrong is the failure mode that erodes trust and, in a policy context, gets people into real trouble. Prefer the honest miss. Make "I don't know" a first-class, encouraged output, not an embarrassment the prompt tries to avoid.

Citations do double duty here: they let the user verify, and they let *you* audit. When an answer is wrong, the citation tells you instantly whether retrieval handed the model bad chunks or the model ignored good ones. Which brings us to the thing most teams skip.

## Caching, because you'll answer the same thing a thousand times

People ask the same questions. "What's the refund policy," "how do I reset my password," phrased forty different ways. Three layers of cache save you real money and latency:

**Exact-match cache** for identical queries, trivial and instant. **Semantic cache** that recognizes "how do I get a refund" and "what's your refund process" as the same question via embedding similarity, and returns the stored answer without re-running the whole pipeline. And **prompt / context caching** at the model layer, where the provider caches the fixed part of your prompt (system instructions, retrieved context that repeats) so you're not paying to re-process the same tokens every call. Stack all three and a big chunk of your traffic never hits the expensive path.

## The same design, filled in three ways

None of this requires building from scratch. Every cloud has a managed service for each box. Here's the map as of mid-2026.

<figure class="rag-fig">
<div class="rag-tab-wrap rag-anim">
<table class="rag-tab">
<thead>
<tr><th>Pipeline box</th><th>AWS</th><th>GCP</th><th>Azure</th></tr>
</thead>
<tbody>
<tr><td>Parse / OCR</td><td>Textract</td><td>Document AI</td><td>Document Intelligence</td></tr>
<tr><td>Managed RAG</td><td>Bedrock Knowledge Bases</td><td>Vertex AI Search</td><td>Azure AI Search (integrated vectorization)</td></tr>
<tr><td>Embeddings</td><td>Titan or Cohere on Bedrock</td><td>gemini-embedding</td><td>Azure OpenAI embeddings</td></tr>
<tr><td>Vector store</td><td>OpenSearch Serverless, Aurora pgvector, or S3 Vectors</td><td>Vertex Vector Search or AlloyDB pgvector</td><td>Azure AI Search or Cosmos DB</td></tr>
<tr><td>Reranker</td><td>Bedrock Rerank (Cohere)</td><td>Vertex Ranking API</td><td>AI Search semantic ranker</td></tr>
<tr><td>LLM</td><td>Bedrock (Claude)</td><td>Vertex (Gemini)</td><td>Microsoft Foundry (GPT / Claude)</td></tr>
<tr><td>Guardrails</td><td>Bedrock Guardrails</td><td>Model Armor</td><td>Azure AI Content Safety</td></tr>
</tbody>
</table>
</div>
<figcaption>Same seven boxes, three vendors. The managed-RAG row can collapse several of the others into one service, which is great for getting started and less great when you need to tune a specific box.</figcaption>
</figure>

A word of caution on the "Managed RAG" row. Bedrock Knowledge Bases, Vertex AI Search, and Azure AI Search will each do chunking, embedding, retrieval, and reranking behind one API, and that's a genuinely good way to ship a first version fast. The catch is that the moment your retrieval quality plateaus, you'll want to reach in and change the chunking strategy or swap the reranker, and how much the managed service lets you do that varies a lot. Start managed, but know which box you'll want to pry open first.

## The gotchas nobody demos

The demo works. Then it meets reality. Three failures show up in basically every RAG deployment, and none of them are the model's fault.

<figure class="rag-fig">
<div class="rag-got rag-anim">
  <div class="rag-g">
    <div class="hd"><span class="tag">01</span><b>The stale index</b></div>
    <p>A doc gets edited or deleted, but its old chunks linger in the index. The bot confidently cites a policy that no longer exists. Retrieval did its job perfectly, it found the chunk, the chunk was just a ghost. <span class="fix">Fix: delete and upsert on every change, and stamp chunks with version metadata so superseded content can be filtered out.</span></p>
  </div>
  <div class="rag-g">
    <div class="hd"><span class="tag">02</span><b>Retrieval miss, confident answer</b></div>
    <p>The right chunk never made it into the top-K. So the model, handed nothing useful, falls back on what it half-remembers from training and produces a fluent, authoritative, wrong answer. It looks like a hallucination. It's a retrieval failure wearing a hallucination costume. <span class="fix">Fix: measure retrieval recall separately, so you catch the miss before the user does.</span></p>
  </div>
  <div class="rag-g">
    <div class="hd"><span class="tag">03</span><b>Embedding-model drift</b></div>
    <p>You upgrade the embedding model for better quality. But your index was built with the old one, and vectors from two different models don't live in the same space, so queries and chunks stop lining up and retrieval quietly degrades. <span class="fix">Fix: changing the embedding model means re-embedding the entire corpus. Pin the model, and treat a change as a full reindex, never a hot swap.</span></p>
  </div>
</div>
<figcaption>Every one of these is an infrastructure failure that the user experiences as "the AI is dumb." It usually isn't the AI.</figcaption>
</figure>

## Evaluate retrieval separately from generation

That second gotcha points at the single most useful discipline in RAG, so it gets its own section: **measure retrieval and generation as two different things.**

Teams love to evaluate the final answer, thumbs up, thumbs down, "was this helpful." That tells you *something* went wrong but not *where*. Split it. Measure retrieval on its own with **recall@k** (did the right chunk make it into the top k?) and **MRR** (how high up was it?). Measure generation on its own with **groundedness / faithfulness** (did the answer stick to the retrieved chunks, or did it invent?).

<div class="rag-note">
<p><b>The number that matters most:</b> most RAG failures are retrieval failures. If recall@k is low, no model on earth can save the answer, because the answer was never in the room. Fix retrieval first, always.</p>
</div>

When you split the metrics, debugging stops being guesswork. Low recall means work on chunking, hybrid search, or reranking. High recall but low groundedness means the model is ignoring good context, so tighten the prompt or the model. Without the split, you'll spend a week tuning prompts to fix a problem that lived in your chunker.

## The takeaway

RAG is not "an LLM with a vector database." It's a retrieval system that happens to end in a language model, and almost all of the engineering, and almost all of the failure, lives in the retrieval half. Get the chunks right, search dense and sparse together, funnel wide-then-narrow through a reranker, filter permissions at the door, ground every claim in a citation, and measure the retrieval step on its own so you know where it breaks.

Do that and the model's job becomes almost boring: read eight good chunks, write one honest sentence, cite the source. Boring is exactly what you want. The magic was never in the model. It was in making sure the right paragraph showed up before the model ever opened its mouth.

*Next in the series: [designing the agent](/blog/2026-07-24-designing-an-agent-that-doesnt-go-off-the-rails/), where the same funnel drives a loop instead of a single pass, and [the recommendation system](/blog/2026-07-24-designing-a-recommendation-system-the-funnel/), the funnel in its purest, highest-scale form. And if you skipped it, [the first post](/blog/2026-07-24-three-ai-systems-same-design/) is the map for why all three share this shape.*

<script>
(function(){
  var els=document.querySelectorAll('.rag-anim');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.2});
  els.forEach(function(e){io.observe(e)});
})();
</script>
