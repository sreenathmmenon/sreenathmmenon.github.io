---
title: "Designing a Recommendation System: The Funnel"
date: 2026-07-24
excerpt: "You open an app and the 'up next' somehow nails it, out of millions of things, in the time it took the page to paint. That's not one clever model. It's a funnel: narrow millions to a handful in stages, spending more compute the fewer items you have left. Here's the whole thing end to end, the two-tower trick at its core, and the cloud services that fill each box on AWS, GCP, and Azure."
tags: [ai, recommendations, ranking, system-design, embeddings, architecture]
---

<style>
.rec-fig{margin:2.5rem 0;}
.rec-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* 1. serving funnel */
.rec-funnel{max-width:640px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.rec-stage{position:relative;border:1px solid var(--border);border-radius:10px;padding:.7rem .95rem;background:var(--surface);display:flex;align-items:center;gap:.9rem;opacity:0;transform:translateY(10px);transition:opacity .5s ease,transform .5s ease;}
.rec-funnel.go .rec-stage{opacity:1;transform:none;}
.rec-funnel.go .rec-stage:nth-child(1){transition-delay:.05s}
.rec-funnel.go .rec-stage:nth-child(2){transition-delay:.18s}
.rec-funnel.go .rec-stage:nth-child(3){transition-delay:.31s}
.rec-funnel.go .rec-stage:nth-child(4){transition-delay:.44s}
.rec-funnel.go .rec-stage:nth-child(5){transition-delay:.57s}
.rec-funnel.go .rec-stage:nth-child(6){transition-delay:.7s}
.rec-stage::before{content:"";position:absolute;left:0;top:0;bottom:0;width:3px;border-radius:3px 0 0 3px;background:var(--accent);opacity:.6;}
.rec-stage.heavy::before{width:5px;opacity:1;background:var(--grad);}
.rec-stage .rs-k{flex:none;font-family:var(--font-mono);font-size:.66rem;letter-spacing:.06em;text-transform:uppercase;color:var(--text-3);width:66px;}
.rec-stage .rs-b{flex:1;}
.rec-stage .rs-b b{display:block;font-size:.9rem;color:var(--text);font-family:var(--font-mono);}
.rec-stage .rs-b span{font-size:.8rem;color:var(--text-2);line-height:1.45;}
.rec-stage .rs-n{flex:none;font-family:var(--font-mono);font-size:.82rem;font-weight:600;color:var(--accent);text-align:right;width:78px;}
.rec-stage.heavy .rs-b b{color:var(--accent);}

/* 2. two-tower */
.rec-tt{max-width:660px;margin:0 auto;}
.rec-tt svg{width:100%;height:auto;overflow:visible;font-family:var(--font-mono);}
.rec-tt .tt-box{fill:var(--surface);stroke:var(--border-2);stroke-width:1;}
.rec-tt .tt-box.u{stroke:var(--accent);} .rec-tt .tt-box.i{stroke:var(--accent-2);}
.rec-tt .tt-lab{fill:var(--text);font-size:11px;font-weight:600;text-anchor:middle;}
.rec-tt .tt-sub{fill:var(--text-3);font-size:8px;text-anchor:middle;}
.rec-tt .tt-u{fill:var(--accent);} .rec-tt .tt-i{fill:var(--accent-2);}
.rec-tt .tt-space{fill:var(--surface-2);stroke:var(--border);stroke-width:1;}
.rec-tt .tt-dot{opacity:0;transition:opacity .5s ease;}
.rec-tt.go .tt-dot{opacity:1;}
.rec-tt.go .tt-dot.d1{transition-delay:.5s} .rec-tt.go .tt-dot.d2{transition-delay:.6s}
.rec-tt.go .tt-dot.d3{transition-delay:.7s} .rec-tt.go .tt-dot.d4{transition-delay:.8s}
.rec-tt.go .tt-dot.uq{transition-delay:1s}
.rec-tt .tt-flow{stroke-dasharray:4 4;stroke:var(--border-2);stroke-width:1;fill:none;stroke-dashoffset:40;transition:stroke-dashoffset 1s ease;}
.rec-tt.go .tt-flow{stroke-dashoffset:0;}
.rec-tt .tt-ann{fill:none;stroke:var(--accent);stroke-width:1.5;opacity:0;transition:opacity .5s ease .9s;}
.rec-tt.go .tt-ann{opacity:1;}

/* 3. batch vs real-time */
.rec-split{display:grid;grid-template-columns:1fr 1fr;gap:.8rem;max-width:660px;margin:0 auto;}
@media(max-width:560px){.rec-split{grid-template-columns:1fr;}}
.rec-split .col{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);opacity:0;transform:translateY(10px);transition:opacity .5s ease,transform .5s ease;}
.rec-split.go .col{opacity:1;transform:none;}
.rec-split.go .col.rt{transition-delay:.2s;}
.rec-split .col.b{border-color:var(--border-2);} .rec-split .col.rt{border-color:var(--accent);}
.rec-split .col h4{margin:0 0 .3rem;font-family:var(--font-mono);font-size:.82rem;}
.rec-split .col.b h4{color:var(--text-2);} .rec-split .col.rt h4{color:var(--accent);}
.rec-split .col .cad{font-family:var(--font-mono);font-size:.68rem;color:var(--text-3);margin-bottom:.6rem;}
.rec-split .col .row{font-size:.8rem;color:var(--text-2);padding:.22rem 0;line-height:1.4;}
.rec-split .col .row b{color:var(--text);font-weight:500;}
.rec-merge{max-width:660px;margin:.8rem auto 0;text-align:center;font-family:var(--font-mono);font-size:.78rem;color:var(--accent);border:1px dashed var(--accent);border-radius:10px;padding:.6rem;opacity:0;transition:opacity .5s ease .5s;}
.rec-split.go + .rec-merge{opacity:1;}

/* 4. feature store / skew */
.rec-fs{max-width:660px;margin:0 auto;}
.rec-fs svg{width:100%;height:auto;overflow:visible;font-family:var(--font-mono);}
.rec-fs .fs-def{fill:var(--surface-2);stroke:var(--accent);stroke-width:1.5;}
.rec-fs .fs-box{fill:var(--surface);stroke:var(--border-2);stroke-width:1;}
.rec-fs .fs-lab{fill:var(--text);font-size:10px;text-anchor:middle;font-weight:600;}
.rec-fs .fs-sub{fill:var(--text-3);font-size:8px;text-anchor:middle;}
.rec-fs .fs-line{stroke:var(--accent);stroke-width:1.5;fill:none;stroke-dasharray:3 3;stroke-dashoffset:60;transition:stroke-dashoffset 1s ease;}
.rec-fs.go .fs-line{stroke-dashoffset:0;}
.rec-fs .fs-line.d{transition-delay:.3s;}
.rec-fs .fs-skew{fill:var(--text-3);font-size:8.5px;text-anchor:middle;opacity:0;transition:opacity .5s ease .9s;}
.rec-fs.go .fs-skew{opacity:1;}

/* 5. cloud table */
.rec-tab-wrap{max-width:820px;margin:0 auto;overflow-x:auto;}
.rec-tab{width:100%;border-collapse:collapse;font-size:.83rem;min-width:640px;}
.rec-tab th,.rec-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.rec-tab th{font-family:var(--font-mono);font-size:.68rem;text-transform:uppercase;letter-spacing:.05em;color:var(--text-3);}
.rec-tab td:first-child,.rec-tab th:first-child{color:var(--text);font-weight:500;font-family:var(--font-mono);font-size:.76rem;}
.rec-tab td{color:var(--text-2);}
.rec-tab tbody tr:hover td{background:var(--surface-2);}
.rec-tab code{font-family:var(--font-mono);font-size:.82em;color:var(--accent);}

/* 6. gotchas */
.rec-gotchas{max-width:660px;margin:0 auto;display:flex;flex-direction:column;gap:.8rem;}
.rec-g{border:1px solid var(--border);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);border-left:3px solid var(--accent-2);opacity:0;transform:translateX(-10px);transition:opacity .5s ease,transform .5s ease;}
.rec-gotchas.go .rec-g{opacity:1;transform:none;}
.rec-gotchas.go .rec-g:nth-child(1){transition-delay:.1s}
.rec-gotchas.go .rec-g:nth-child(2){transition-delay:.3s}
.rec-gotchas.go .rec-g:nth-child(3){transition-delay:.5s}
.rec-g h4{margin:0 0 .35rem;font-family:var(--font-mono);font-size:.85rem;color:var(--text);}
.rec-g h4 .num{color:var(--accent-2);}
.rec-g p{margin:0;font-size:.83rem;color:var(--text-2);line-height:1.55;}
.rec-g p b{color:var(--accent);}

@media (prefers-reduced-motion: reduce){
  .rec-stage,.rec-split .col,.rec-g{opacity:1!important;transform:none!important;}
  .rec-tt .tt-dot,.rec-tt .tt-ann,.rec-fs .fs-skew,.rec-merge{opacity:1!important;}
  .rec-tt .tt-flow,.rec-fs .fs-line{stroke-dashoffset:0!important;}
}
</style>

You know the moment. You finish a video, and the "up next" is somehow exactly the thing you didn't know you wanted. Or you open a shopping app and the home feed leads with the one product you'd been half-thinking about. It feels like the app read your mind.

It didn't. What it did was pick a handful of items out of millions, and it did it in the time the page took to paint. Tens of milliseconds. That constraint, millions of candidates, a few slots, almost no time, is the whole reason recommendation systems look the way they do.

This is the last post in the series, and it's the clearest example of the funnel the [three-systems post](/blog/2026-07-24-three-ai-systems-same-design/) was about. RAG had it. Agents had it. But recommendations are the textbook version: cheap wide net, expensive narrow scoring, then the rules a model won't learn. Let me build the whole thing box by box.

## The problem, stated honestly

You can't score millions of items with a good model on every request. A rich ranking model that looks at this user crossed with this item, with hundreds of features, might take a millisecond per item. Times ten million items, times every user hitting the page, and you're nowhere near your latency budget or your compute budget.

So the field gave up on scoring everything. Instead it narrows in stages, spending almost nothing per item when there are millions, and spending real compute per item only once there are a few hundred left.

<figure class="rec-fig rec-funnel rec-anim">
  <div class="rec-stage">
    <span class="rs-k">Context</span>
    <span class="rs-b"><b>Request context</b><span>user, session, device, page, time of day</span></span>
    <span class="rs-n">1 user</span>
  </div>
  <div class="rec-stage">
    <span class="rs-k">Fetch</span>
    <span class="rs-b"><b>Feature fetch</b><span>low-latency read from the online feature store</span></span>
    <span class="rs-n">~1 ms</span>
  </div>
  <div class="rec-stage">
    <span class="rs-k">Retrieve</span>
    <span class="rs-b"><b>Candidate generation</b><span>two-tower ANN + collaborative + trending, fused</span></span>
    <span class="rs-n">millions to hundreds</span>
  </div>
  <div class="rec-stage">
    <span class="rs-k">Filter</span>
    <span class="rs-b"><b>Filtering</b><span>drop seen, out-of-stock, region-locked, blocked</span></span>
    <span class="rs-n">hundreds</span>
  </div>
  <div class="rec-stage heavy">
    <span class="rs-k">Rank</span>
    <span class="rs-b"><b>Ranking (heavy model)</b><span>score each candidate for p(click / watch / convert)</span></span>
    <span class="rs-n">hundreds</span>
  </div>
  <div class="rec-stage heavy">
    <span class="rs-k">Re-rank</span>
    <span class="rs-b"><b>Re-ranking (policy)</b><span>diversity, freshness, exploration, ads, dedupe</span></span>
    <span class="rs-n">a final few</span>
  </div>
  <figcaption>The serving path. Compute per item goes up as the count comes down. Then you log every impression and the features you served, because that's your training data.</figcaption>
</figure>

Read that top to bottom and the shape is the point. The first stages are cheap and wide. They're allowed to be sloppy, as long as they don't miss the good stuff. The bottom stages are expensive and narrow. They can afford to be careful because there's almost nothing left to be careful about.

Walk the boxes:

- **Request context.** Who's asking, from what device, on what page, at what time. A product page wants "similar and complementary items." A home feed wants "things you'll open." Same machine, different context.
- **Feature fetch.** A single low-latency read pulls this user's features (recent activity, long-term profile, current session) from an online store. Milliseconds, or the whole budget is blown before you've done anything.
- **Candidate generation.** The big narrowing. Millions to a few hundred, usually by *fusing several sources*: a two-tower vector search (more on that below), collaborative filtering (people like you liked this), and plain rules (trending, recent, editorial picks). No single source is enough, so you union them.
- **Filtering.** Remove what you can't or shouldn't show: things they've already seen, out-of-stock items, region-locked content, anything blocked. Cheap set operations, big impact.
- **Ranking.** Now the heavy model earns its keep. On a few hundred candidates it can afford rich *user times item cross features*, the interactions the retrieval stage literally couldn't look at, and score each one for the probability you'll click, watch, or buy.
- **Re-ranking.** The last pass isn't about accuracy. It's diversity so you're not shown ten near-identical items, freshness, a little exploration, deduping, blending ads, fairness. Business objectives an accuracy model will never optimize on its own.

Then you log it all. Which impressions you served, in what order, with what features. That log, joined later with what the user actually did, becomes the labels you train on. The serving path and the training path are two loops around the same data.

## The two-tower trick, which is the whole backbone

The retrieval stage is where the magic lives, and it runs on one idea that's worth slowing down for: **two towers.**

You train two separate encoders. One is a user (or query) tower: it takes everything you know about the user and context and produces a vector. The other is an item tower: it takes everything about an item and produces a vector in the *same* space. You train them jointly so that a user's vector lands close to the items they'll actually engage with. Closeness is just a dot product or cosine similarity.

<figure class="rec-fig rec-tt rec-anim">
<svg viewBox="0 0 620 260" role="img" aria-label="Two-tower model: a user tower and an item tower produce embeddings into a shared space; item embeddings are precomputed and indexed, the user embedding is computed at request time, and an approximate nearest neighbour lookup retrieves the closest items.">
  <!-- user tower -->
  <rect class="tt-box u" x="20" y="30" width="120" height="52" rx="8"></rect>
  <text class="tt-lab" x="80" y="52">User / query tower</text>
  <text class="tt-sub" x="80" y="68">at request time · once</text>
  <!-- item tower -->
  <rect class="tt-box i" x="480" y="30" width="120" height="52" rx="8"></rect>
  <text class="tt-lab" x="540" y="52">Item tower</text>
  <text class="tt-sub" x="540" y="68">offline · precomputed</text>
  <!-- flows to space -->
  <path class="tt-flow" d="M80 82 L80 130 L250 150"></path>
  <path class="tt-flow" d="M540 82 L540 130 L370 150"></path>
  <!-- shared space -->
  <rect class="tt-space" x="200" y="110" width="220" height="120" rx="10"></rect>
  <text class="tt-sub" x="310" y="128" style="font-size:9px;fill:var(--text-2)">shared embedding space</text>
  <!-- item dots (precomputed, indexed) -->
  <circle class="tt-dot d1 tt-i" cx="250" cy="160" r="4"></circle>
  <circle class="tt-dot d2 tt-i" cx="290" cy="200" r="4"></circle>
  <circle class="tt-dot d3 tt-i" cx="355" cy="175" r="4"></circle>
  <circle class="tt-dot d4 tt-i" cx="380" cy="205" r="4"></circle>
  <circle class="tt-dot d1 tt-i" cx="330" cy="145" r="4"></circle>
  <circle class="tt-dot d2 tt-i" cx="230" cy="200" r="4"></circle>
  <!-- user dot -->
  <circle class="tt-dot uq tt-u" cx="300" cy="172" r="6"></circle>
  <text class="tt-dot uq tt-sub" x="300" y="192" style="fill:var(--accent);font-size:8px">you</text>
  <!-- ANN circle -->
  <circle class="tt-ann" cx="300" cy="172" r="42"></circle>
  <text class="tt-dot uq tt-sub" x="420" y="250" style="fill:var(--accent);font-size:8.5px;text-anchor:end">ANN lookup = nearest items in ms</text>
</svg>
<figcaption>Because the towers are independent, every item vector is computed offline and indexed. At request time you compute one user vector, then an approximate-nearest-neighbour lookup finds the closest items out of millions in milliseconds.</figcaption>
</figure>

Here's why that structure is so powerful. Because the two towers are *independent*, the item tower never needs the user to run. So you compute every item's embedding **offline, in a batch job**, and load them all into an approximate-nearest-neighbour (ANN) index. That index can search millions of vectors in single-digit milliseconds.

Then at request time you do almost nothing. You run the user tower once to get a single vector, and you ask the ANN index for its nearest item vectors. That's how you retrieve a few hundred relevant items out of ten million without breaking a sweat. All the heavy lifting happened last night in a batch job; serving is just one encode plus one lookup.

There's a catch, and it's the reason the funnel has more stages after this. The towers *can't see user-times-item interactions.* The user vector is computed before it's ever compared to an item, so the model can't learn "this user loves this item specifically because of some interaction between them." That's the price of precomputability. And it's exactly why you need a separate ranking stage downstream, one that *can* look at those cross features, on the small candidate set where you can afford to.

Retrieval is cheap and cares about **recall**: get the good items into the pool, don't miss them. Ranking is expensive and cares about **precision**: order the small pool well. A brilliant ranker can't rescue an item that retrieval never surfaced, which is why retrieval quality quietly decides your ceiling.

## What to precompute, and what to do live

The two-tower split hints at a bigger design principle that runs through the whole system: **decide what's stable enough to precompute, and what's too volatile to.**

<div class="rec-fig rec-split rec-anim">
  <div class="col b">
    <h4>Batch / offline</h4>
    <div class="cad">hourly to daily · cheap at scale</div>
    <div class="row"><b>Item embeddings</b>, indexed in ANN</div>
    <div class="row"><b>Long-horizon user profile</b>: tastes over months</div>
    <div class="row"><b>Popularity + trending</b> aggregates</div>
    <div class="row"><b>Model training</b> itself</div>
  </div>
  <div class="col rt">
    <h4>Real-time / online</h4>
    <div class="cad">at request · relevance over cost</div>
    <div class="row"><b>In-session behavior</b>: "you just watched X"</div>
    <div class="row"><b>Current context</b>: page, device, time now</div>
    <div class="row"><b>The user embedding</b>, one encode</div>
    <div class="row"><b>Ranking + re-ranking</b> on the candidates</div>
  </div>
</div>
<div class="rec-merge rec-anim">the classic win: batch profile + live session signals, combined at request time</div>

Stable things go in a batch job because batch compute is cheap and you can afford to redo the world overnight. Volatile things have to be computed live because they didn't exist a minute ago. The move that makes recommendations feel alive is combining the two: a rich, slowly-built profile of who you are, adjusted at request time by what you're doing *right now*. The "you just watched X, so here's more like X" reaction is that combination in action. Neither half alone would feel as sharp.

## The feature store, and the bug it exists to prevent

There's an unglamorous box in every serious recsys that turns out to matter more than the models: the feature store. And its real job isn't storage. It's **consistency.**

Think about it. Your model trains on features computed in a big offline job over historical data. But at serving time, those same features have to be computed live, in a different codebase, under a latency budget. If the offline "average watch time last 7 days" is computed even slightly differently from the online one, your model sees one thing in training and a different thing in production. That gap has a name: **train/serve skew.** And it's the most insidious bug in the whole field, because the model looks fantastic in offline evaluation and quietly falls apart live.

<figure class="rec-fig rec-fs rec-anim">
<svg viewBox="0 0 620 220" role="img" aria-label="One set of feature definitions feeds both an offline store for training and an online store for serving; if the two diverge, you get train/serve skew.">
  <!-- shared definitions -->
  <rect class="fs-def" x="230" y="20" width="160" height="46" rx="9"></rect>
  <text class="fs-lab" x="310" y="40">Feature definitions</text>
  <text class="fs-sub" x="310" y="55">written once</text>
  <!-- offline store -->
  <rect class="fs-box" x="40" y="140" width="200" height="54" rx="9"></rect>
  <text class="fs-lab" x="140" y="162">Offline store</text>
  <text class="fs-sub" x="140" y="178">training · point-in-time joins</text>
  <!-- online store -->
  <rect class="fs-box" x="380" y="140" width="200" height="54" rx="9"></rect>
  <text class="fs-lab" x="480" y="162">Online store</text>
  <text class="fs-sub" x="480" y="178">serving · low-latency read</text>
  <!-- lines -->
  <path class="fs-line" d="M290 66 L140 140"></path>
  <path class="fs-line d" d="M330 66 L480 140"></path>
  <text class="fs-skew" x="310" y="120">same definition both sides = no skew</text>
  <text class="fs-skew" x="310" y="210" style="fill:var(--accent-2)">diverge here and the model looks great offline, tanks live</text>
</svg>
<figcaption>One definition, two stores. Training reads the offline store, serving reads the online store, and the feature store's whole reason to exist is that both compute the feature the same way.</figcaption>
</figure>

So the feature store defines each feature once and serves it to both worlds: an offline store for training (with point-in-time-correct joins, so you never accidentally leak the future into a training row) and an online store for serving (fast reads). Same definition, both sides. That's the guarantee you're paying for. It's boring, and it's the difference between a model that works and one that only looked like it did.

## Where LLMs and embeddings fold in

Given everything else in this series has been about language models, the honest answer is: they help, but they don't change the shape.

Content and semantic embeddings power the item tower, and they're the cleanest fix for **cold start.** A brand-new item has zero interaction history, so collaborative signals have nothing to work with and it never gets retrieved. But you still have its text, its metadata, its thumbnail. A semantic embedding of that content places the new item near similar existing ones in the shared space, so it can be retrieved on day one. LLMs also generate richer features (summaries, categories, extracted attributes) that feed both towers.

There's a newer thread too: generative and sequence recommenders that treat a user's history as a sequence and "generate" the likely next item, the same next-token idea from [how LLMs actually work](/blog/2026-06-20-how-llms-actually-work/), pointed at item IDs instead of words. Interesting, and real. But the retrieve-then-rank funnel is still the production backbone. LLMs mostly improve the *representations* inside the boxes, not the arrangement of the boxes. The funnel earns its keep whether the embeddings come from a classic model or a language one.

## The cloud services that fill each box

None of this is something you build from raw metal anymore. Every box on the diagram maps to a managed service on each major cloud. Here's the honest mapping as of mid-2026:

<div class="rec-fig rec-tab-wrap rec-anim">
<table class="rec-tab">
<thead>
<tr><th>Box</th><th>AWS</th><th>GCP</th><th>Azure</th></tr>
</thead>
<tbody>
<tr><td>Feature store</td><td>SageMaker Feature Store</td><td>Vertex AI Feature Store</td><td>Azure ML feature store</td></tr>
<tr><td>Embeddings / training</td><td>SageMaker</td><td>Vertex AI Training</td><td>Azure ML</td></tr>
<tr><td>ANN retrieval</td><td>OpenSearch kNN or Aurora <code>pgvector</code></td><td>Vertex Vector Search (ScaNN)</td><td>Azure AI Search vectors</td></tr>
<tr><td>Ranking serving</td><td>SageMaker endpoints</td><td>Vertex AI Prediction</td><td>Azure ML online endpoints</td></tr>
<tr><td>Low-latency feature cache</td><td>ElastiCache / DynamoDB</td><td>Memorystore / Bigtable</td><td>Azure Cache for Redis / Cosmos DB</td></tr>
<tr><td>Streaming events</td><td>Kinesis / MSK</td><td>Pub/Sub + Dataflow</td><td>Event Hubs</td></tr>
<tr><td>Offline data / labels</td><td>S3 + EMR / Glue</td><td>BigQuery + Dataflow</td><td>Synapse / Databricks</td></tr>
</tbody>
</table>
</div>

The rows are the architecture. Pick a cloud and you're mostly choosing which brand fills each box, not changing what the boxes do. If you understand the funnel, you can port the design to any of the three by reading down a column.

## The gotchas that actually sink these systems

The architecture is the easy part. Here's what quietly kills recommendation systems in production, in rough order of how often it happens.

<div class="rec-fig rec-gotchas rec-anim">
  <div class="rec-g">
    <h4><span class="num">1.</span> Train/serve skew and feature leakage</h4>
    <p>Features computed differently offline vs online, or a feature that secretly encodes the label and won't exist at serving time, gives you a model that's brilliant in evaluation and useless live. <b>Point-in-time-correct joins and a shared feature store</b> are the fix. This is the number one production killer, full stop.</p>
  </div>
  <div class="rec-g">
    <h4><span class="num">2.</span> Feedback loops and popularity bias</h4>
    <p>The model recommends what it already surfaces, those items get the clicks, training reinforces them, and the catalog collapses toward a popular few. Rich get richer. <b>Mitigate with exploration</b> (bandits, a dash of randomization), diversity in re-ranking, and logging what <i>could</i> have been shown, not just what was.</p>
  </div>
  <div class="rec-g">
    <h4><span class="num">3.</span> Cold start and offline/online divergence</h4>
    <p>New items have no signal and never get retrieved; new users have no profile. Bridge both with content and semantic embeddings plus sensible context defaults. And when a lift in NDCG doesn't move the real business metric, <b>your offline eval isn't measuring reality</b>, so the final arbiter is always an online A/B test.</p>
  </div>
</div>

That last point deserves the final word. You can chase offline metrics, AUC, NDCG, recall@k, forever, and they're useful for catching regressions cheaply. But they're a proxy. A model that scores higher offline can lose in production, because offline you're grading against a world your own past recommendations shaped. The system is a loop, and the only honest measurement of a loop is to run it live against a control. The A/B test is the truth. Everything before it is a hypothesis.

## The upshot

A recommendation system feels like mind-reading, and it's really just discipline about compute. You can't be smart about millions of items, so you're cheap and wide first (two-tower retrieval, high recall), then expensive and narrow (a heavy ranker on a few hundred, high precision), then you apply the rules a scoring model would never learn on its own (diversity, freshness, fairness, ads). Cheap recall, expensive precision, policy. The same funnel this whole series has been circling.

That's the series. Four posts, [all built on the same machine](/blog/2026-07-24-three-ai-systems-same-design/): a [RAG chatbot](/blog/2026-07-24-designing-a-rag-system-that-actually-retrieves/), an [agent that acts](/blog/2026-07-24-designing-an-agent-that-doesnt-go-off-the-rails/), and this one that ranks. Learn the funnel once and the next system you're handed is just a matter of asking what's cheap, what's expensive, and which rules the model will never figure out for you.

<script>
(function(){
  var els=document.querySelectorAll('.rec-anim');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.2});
  els.forEach(function(e){io.observe(e)});
})();
</script>
