---
title: "Streaming Data Into RAG: Keeping the Index Live"
date: 2026-07-31
excerpt: "A normal RAG index is a photograph. You embed your documents once, and from that second on the index is frozen while the world keeps moving. That's fine for a policy handbook and useless for anything that changes: tickets, prices, inventory, chat. This post is about closing that gap, wiring a stream of changes straight into your vector index so retrieval reflects now, not last night. AWS in the loop, with the GCP and Azure equivalents at the end."
tags: [ai, rag, streaming, aws, real-time, architecture]
---

<style>
.st-fig{margin:2.5rem 0;}
.st-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* snapshot vs live */
.st-vs{display:grid;grid-template-columns:1fr 1fr;gap:.9rem;max-width:660px;margin:0 auto;}
@media(max-width:560px){.st-vs{grid-template-columns:1fr;}}
.st-vs .col{border:1px solid var(--border);border-radius:12px;padding:1.1rem;background:var(--surface);opacity:0;transform:translateY(10px);transition:opacity .5s ease,transform .5s ease;}
.st-vs.go .col{opacity:1;transform:none;} .st-vs.go .col:nth-child(2){transition-delay:.18s;}
.st-vs .col h4{margin:0 0 .6rem;font-size:.85rem;font-family:var(--font-mono);}
.st-vs .col.snap{border-color:var(--border-2);} .st-vs .col.snap h4{color:var(--text-2);}
.st-vs .col.live{border-color:var(--accent);} .st-vs .col.live h4{color:var(--accent);}
.st-vs .col p{font-size:.85rem;color:var(--text-2);line-height:1.55;margin:0;}
.st-vs .col .frozen{font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);margin-top:.7rem;}
.st-vs .col.live .frozen{color:var(--accent);}

/* the pipeline */
.st-pipe{max-width:820px;margin:0 auto;overflow-x:auto;}
.st-flow{display:flex;flex-wrap:nowrap;gap:.4rem;align-items:stretch;min-width:640px;}
.st-node{flex:1 1 0;min-width:110px;border:1px solid var(--border-2);border-radius:11px;background:var(--surface);padding:.75rem .7rem;opacity:0;transform:translateY(10px);transition:opacity .45s ease,transform .45s ease;}
.st-flow.go .st-node{opacity:1;transform:none;}
.st-flow.go .st-node:nth-child(1){transition-delay:.05s} .st-flow.go .st-node:nth-child(3){transition-delay:.2s}
.st-flow.go .st-node:nth-child(5){transition-delay:.35s} .st-flow.go .st-node:nth-child(7){transition-delay:.5s}
.st-flow.go .st-node:nth-child(9){transition-delay:.65s}
.st-node .svc{font-family:var(--font-mono);font-size:.66rem;color:var(--accent);}
.st-node b{display:block;color:var(--text);font-size:.82rem;margin:.2rem 0 .12rem;line-height:1.25;}
.st-node span{font-size:.73rem;color:var(--text-3);line-height:1.4;}
.st-arrow{align-self:center;color:var(--text-3);font-family:var(--font-mono);flex:none;font-size:.9rem;}

/* batch vs stream freshness timeline */
.st-time{max-width:640px;margin:0 auto;display:flex;flex-direction:column;gap:1rem;}
.st-track{border:1px solid var(--border);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);}
.st-track .lb{font-family:var(--font-mono);font-size:.74rem;margin-bottom:.7rem;}
.st-track.batch .lb{color:var(--text-2);} .st-track.stream .lb{color:var(--accent);}
.st-bar{height:14px;border-radius:7px;background:var(--surface-2);position:relative;overflow:hidden;}
.st-bar .stale{position:absolute;top:0;left:0;bottom:0;width:0;transition:width 1.1s ease;}
.st-track.batch .stale{background:linear-gradient(90deg,color-mix(in srgb,var(--text-3) 55%,transparent),color-mix(in srgb,var(--text-3) 20%,transparent));}
.st-track.stream .stale{background:linear-gradient(90deg,var(--accent),color-mix(in srgb,var(--accent) 30%,transparent));}
.st-time.go .st-track.batch .stale{width:100%;} .st-time.go .st-track.stream .stale{width:6%;}
.st-track .cap{font-family:var(--font-mono);font-size:.7rem;color:var(--text-3);margin-top:.5rem;}

/* upsert not append */
.st-up{max-width:640px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:.9rem;}
@media(max-width:520px){.st-up{grid-template-columns:1fr;}}
.st-up .side{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.st-up .side h4{margin:0 0 .55rem;font-size:.8rem;font-family:var(--font-mono);}
.st-up .side.bad h4{color:var(--text-2);} .st-up .side.good{border-color:var(--accent);} .st-up .side.good h4{color:var(--accent);}
.st-up .side .item{font-size:.82rem;color:var(--text-2);line-height:1.45;padding-left:1.1rem;position:relative;margin:.4rem 0;}
.st-up .side .item::before{position:absolute;left:0;font-family:var(--font-mono);}
.st-up .side.bad .item::before{content:"\2212";color:var(--text-3);}
.st-up .side.good .item::before{content:"+";color:var(--accent);}

/* gotchas */
.st-got{max-width:660px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.st-g{display:flex;gap:.85rem;border:1px solid var(--border);border-radius:10px;background:var(--surface-2);padding:.75rem .95rem;align-items:flex-start;opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.st-got.go .st-g{opacity:1;transform:none;}
.st-got.go .st-g:nth-child(1){transition-delay:.1s} .st-got.go .st-g:nth-child(2){transition-delay:.25s}
.st-got.go .st-g:nth-child(3){transition-delay:.4s} .st-got.go .st-g:nth-child(4){transition-delay:.55s}
.st-g .no{flex:none;font-family:var(--font-mono);font-size:.72rem;color:var(--accent);border:1px solid var(--accent);border-radius:6px;padding:.12rem .45rem;margin-top:.05rem;}
.st-g p{font-size:.85rem;color:var(--text-2);line-height:1.5;margin:0;}
.st-g p b{color:var(--text);}

/* cloud table */
.st-tab-wrap{max-width:760px;margin:0 auto;overflow-x:auto;border:1px solid var(--border);border-radius:12px;background:var(--surface);}
.st-tab{width:100%;border-collapse:collapse;font-size:.86rem;min-width:560px;}
.st-tab th,.st-tab td{text-align:left;padding:.65rem .8rem;border-bottom:1px solid var(--border);vertical-align:top;}
.st-tab th{font-family:var(--font-mono);font-size:.7rem;text-transform:uppercase;letter-spacing:.05em;color:var(--text-3);font-weight:500;}
.st-tab td:first-child,.st-tab th:first-child{color:var(--text);font-weight:500;font-family:var(--font-mono);font-size:.76rem;}
.st-tab tr:last-child td{border-bottom:none;}
.st-tab .aws-c{color:var(--accent);}

/* references */
.st-refs{max-width:660px;margin:0 auto;border:1px solid var(--border);border-radius:12px;background:var(--surface-2);padding:.4rem 1.1rem;}
.st-refs a{display:block;padding:.6rem 0;border-top:1px solid var(--border);font-size:.85rem;color:var(--text-2);line-height:1.4;}
.st-refs a:first-child{border-top:none;}
.st-refs a:hover{color:var(--accent);}
.st-refs a .src{font-family:var(--font-mono);font-size:.68rem;color:var(--text-3);display:block;margin-bottom:.15rem;letter-spacing:.03em;}

@media (prefers-reduced-motion: reduce){
  .st-vs .col,.st-node,.st-g{opacity:1!important;transform:none!important;}
  .st-time .stale{transition:none!important;}
  .st-time.go .st-track.batch .stale{width:100%;} .st-time.go .st-track.stream .stale{width:6%;}
}
</style>

If you've read the [RAG series](/blog/2026-06-21-rag-part-1-what-it-really-is/) here, you know the machinery: chunk your documents, embed them, drop the vectors in a store, and at query time retrieve the closest ones and hand them to the model. It works. But there's a quiet assumption baked into every diagram of it, and almost nobody says it out loud.

The index is a **photograph**.

You embed your documents once. From that second, the vectors are frozen. The world keeps moving, tickets get updated, prices change, an order ships, someone edits the policy, and your index sits there holding a picture of how things were at the last reindex. Ask the bot "what's the status of order 4471?" and it cheerfully answers from last night's snapshot. Confidently. Wrongly.

For a policy handbook that changes twice a year, a nightly batch reindex is completely fine. For anything that actually changes, it's a slow-motion correctness bug. This post is about closing that gap: wiring a **stream** of changes straight into the vector index, so retrieval reflects *now*.

## Snapshot RAG vs live RAG

<figure class="st-fig">
<div class="st-vs">
  <div class="col snap">
    <h4>Snapshot RAG</h4>
    <p>Embed the whole corpus in a batch job. Re-run it on a schedule, nightly, hourly, whenever. Between runs, the index is a fixed picture. Simple, cheap, and correct only until the moment the first document changes.</p>
    <p class="frozen">freshness = time since last batch</p>
  </div>
  <div class="col live">
    <h4>Live RAG</h4>
    <p>Every change to a source becomes an event. The event flows through a stream, gets re-chunked and re-embedded, and updates just that item in the index within seconds. The picture is never more than a moment old.</p>
    <p class="frozen">freshness = seconds</p>
  </div>
</div>
<figcaption>Same retrieval at query time. The difference is entirely in how the index gets updated.</figcaption>
</figure>

Notice what does *not* change: the query path. Retrieval, reranking, the model, all of it stays exactly as the [design post](/blog/2026-07-24-designing-a-rag-system-that-actually-retrieves/) laid it out. Streaming only rewrites the **ingestion** side, the offline path that keeps the index current. That's the whole trick. You're not building a different RAG system. You're replacing the batch loader with a live one.

## The use cases that actually need this

Before the pipeline, the honest question: do you need it? Streaming ingestion is more moving parts and more cost, so reach for it only when staleness is a real bug. It usually is when the corpus is one of these:

- **Support and operations.** A bot answering over live tickets, order status, account state. "It shipped an hour ago" needs to be retrievable an hour ago, not tomorrow.
- **Commerce.** Prices, inventory, availability. Retrieving "in stock" for something that sold out twenty minutes back is a customer-trust problem.
- **Conversations and collaboration.** Chat logs, docs, comment threads that people expect the assistant to have already read.
- **Signals and telemetry.** Logs, metrics, fraud events, IoT readings, anything where the useful window is minutes wide.

If your corpus is contracts, manuals, or published articles, close the tab and keep your nightly batch. You don't need any of what follows.

## The pipeline, AWS in the loop

Here's the shape end to end. A change happens at the source, and instead of waiting for a batch job to notice, you turn it into an event and push it through a stream that ends at the vector store.

<figure class="st-fig">
<div class="st-pipe">
<div class="st-flow">
  <div class="st-node"><span class="svc">source + CDC</span><b>Change happens</b><span>DynamoDB Streams, DB CDC, an S3 upload, an app event</span></div>
  <div class="st-arrow">&rarr;</div>
  <div class="st-node"><span class="svc">Kinesis / MSK</span><b>Stream</b><span>Durable, ordered, buffered event log</span></div>
  <div class="st-arrow">&rarr;</div>
  <div class="st-node"><span class="svc">Lambda</span><b>Chunk + embed</b><span>Re-chunk the item, call Bedrock for the vector</span></div>
  <div class="st-arrow">&rarr;</div>
  <div class="st-node"><span class="svc">Bedrock</span><b>Embedding</b><span>Titan or Cohere turns text into a vector</span></div>
  <div class="st-arrow">&rarr;</div>
  <div class="st-node"><span class="svc">OpenSearch / S3 Vectors</span><b>Upsert</b><span>Replace that item's vector by stable ID</span></div>
</div>
</div>
<figcaption>The change never waits for a schedule. It rides the stream and updates one item in seconds.</figcaption>
</figure>

Walking the boxes, and why each one is what it is:

**The change, captured.** The cleanest source of "something changed" is change data capture. If your data lives in DynamoDB, **DynamoDB Streams** hands you an event for every write, no polling. A relational database gives you CDC through Database Migration Service or Debezium onto a stream. A document dropped in **S3** fires an **EventBridge** or S3 notification. The point is the same: don't scan the whole corpus looking for diffs, let the source tell you exactly what moved.

**The stream itself.** This is the load-bearing part, and it's why streaming beats "just call the embedder in your write path." **Kinesis Data Streams** (or **MSK**, managed Kafka, if you want Kafka semantics) sits between the firehose of changes and your embedding step. It buffers, so a burst of updates doesn't melt your embedder. It's durable, so if the consumer dies mid-batch, the events are still there. And it preserves order within a shard, which matters more than you'd think, we'll get to that. Without a stream, a spike in source changes goes straight at Bedrock and either throttles or bankrupts you.

**Chunk and embed.** A **Lambda** function consumes from the stream in small batches. For each changed item it does the same work your batch job did, but for one thing: re-chunk it (structure-aware, with overlap, exactly as [Part 2 of the RAG series](/blog/2026-06-21-rag-part-2-chunking/) argued), then call **Bedrock** to embed each chunk. Lambda is the right tool here because the work is bursty and embarrassingly parallel: scale to zero when nothing's changing, fan out when the stream floods. Watch cold starts on the latency-sensitive edge, but for ingestion a few hundred milliseconds is nothing.

**Upsert, not append.** The last box is where most people quietly introduce a bug, so it gets its own section.

## The upsert trap

The instinct, when a new event arrives, is to embed it and add it to the index. Add. That's the bug.

<figure class="st-fig">
<div class="st-up">
  <div class="side bad">
    <h4>Append (wrong)</h4>
    <div class="item">Every edit adds a new vector for the same underlying item</div>
    <div class="item">The old, stale chunk is still in there, still retrievable</div>
    <div class="item">Retrieval returns both, the model sees contradictions</div>
    <div class="item">Deletes never happen, so wrong data lives forever</div>
  </div>
  <div class="side good">
    <h4>Upsert by stable ID (right)</h4>
    <div class="item">Each chunk carries a deterministic ID from source key + chunk index</div>
    <div class="item">An edit replaces that ID's vector in place</div>
    <div class="item">A delete event removes it, tombstones and all</div>
    <div class="item">The index holds exactly one current version of each thing</div>
  </div>
</div>
</figure>

If you append, your index becomes a graveyard of every version a document ever had, and the stale ones outnumber the fresh. The bot retrieves the old price *and* the new one and picks whichever the reranker liked. The fix is boring and essential: give every chunk a **stable, deterministic ID** derived from the source's primary key plus the chunk index, and **upsert** on it, replace if it exists, insert if it doesn't. And handle **deletes** as first-class events: when the source row is deleted, the stream carries a delete, and your consumer removes those chunk IDs. An index that can't delete is an index that lies eventually.

## Why the freshness gap is the whole point

<figure class="st-fig">
<div class="st-time">
  <div class="st-track batch">
    <div class="lb">Batch reindex, nightly</div>
    <div class="st-bar"><div class="stale"></div></div>
    <div class="cap">staleness grows all day until the next run</div>
  </div>
  <div class="st-track stream">
    <div class="lb">Streaming ingestion</div>
    <div class="st-bar"><div class="stale"></div></div>
    <div class="cap">staleness stays near zero, always</div>
  </div>
</div>
<figcaption>With batch, your worst-case staleness is the interval between runs. With streaming, it's the time to process one event.</figcaption>
</figure>

With a nightly batch, the honest description of your index is "correct as of 2am, degrading until tomorrow." At 5pm it's fifteen hours stale in the worst case. You can shrink that by reindexing hourly, but hourly reindex of a big corpus is expensive and wasteful, you re-embed millions of unchanged chunks to catch the handful that moved. Streaming flips the economics: you only ever embed what actually changed, and the freshness is measured in seconds, not hours. You do more small work and far less total work.

## The gotchas nobody warns you about

Streaming ingestion has a specific set of ways to hurt you. None are dealbreakers, all are worth designing for up front.

<figure class="st-fig">
<div class="st-got">
  <div class="st-g"><span class="no">1</span><p><b>Out-of-order events.</b> Two edits to the same item can arrive out of order, and now your "current" vector is actually the older one. Order within a Kinesis shard is guaranteed, so partition by the item's key so all its edits land on the same shard. Or carry a version or timestamp on each event and refuse to overwrite a newer vector with an older one.</p></div>
  <div class="st-g"><span class="no">2</span><p><b>Embedding cost per event.</b> Every change triggers an embed call, and a chatty source (a row that updates on every click) can turn into a Bedrock bill you didn't plan for. Debounce: collapse rapid repeated edits to the same item, and only re-embed when the text that actually gets embedded changed, not on every metadata tweak.</p></div>
  <div class="st-g"><span class="no">3</span><p><b>Back-pressure and poison events.</b> A traffic spike or a single malformed record can stall the consumer. Let the stream buffer the spike (that's its job), set sane batch sizes and concurrency on the Lambda, and send records that fail repeatedly to a dead-letter queue instead of blocking the shard behind them forever.</p></div>
  <div class="st-g"><span class="no">4</span><p><b>Embedding-model drift, now live.</b> The old rule still holds: change the embedding model and old vectors are incompatible with new queries. In a streaming world you can't just "re-run the batch." You need a reindex strategy, usually a parallel index you backfill and cut over to, because the live one never stops taking writes.</p></div>
</div>
</figure>

That last one is the sharp edge of live systems generally: there's no quiet window where nothing is happening. Every migration is a migration under load. Plan for a shadow index and a cutover, not a maintenance window.

## The same pipeline on the other clouds

The pattern is cloud-agnostic, stream the changes, embed on the way through, upsert by ID. Here's how the boxes map if you're not on AWS:

<figure class="st-fig">
<div class="st-tab-wrap">
<table class="st-tab">
<thead>
<tr><th>Box</th><th class="aws-c">AWS</th><th>GCP</th><th>Azure</th></tr>
</thead>
<tbody>
<tr><td>Change capture</td><td>DynamoDB Streams / DMS CDC</td><td>Datastream / Firestore triggers</td><td>Cosmos DB change feed</td></tr>
<tr><td>The stream</td><td>Kinesis / MSK</td><td>Pub/Sub</td><td>Event Hubs</td></tr>
<tr><td>Chunk + embed</td><td>Lambda</td><td>Dataflow / Cloud Functions</td><td>Azure Functions</td></tr>
<tr><td>Embedding model</td><td>Bedrock (Titan / Cohere)</td><td>Vertex (gemini-embedding)</td><td>Azure OpenAI embeddings</td></tr>
<tr><td>Vector store</td><td>OpenSearch / S3 Vectors</td><td>Vertex Vector Search / AlloyDB</td><td>Azure AI Search / Cosmos DB</td></tr>
</tbody>
</table>
</div>
<figcaption>Different names, identical shape. Stream in, embed through, upsert out.</figcaption>
</figure>

## The takeaway

A batch-loaded RAG index answers questions about the past and pretends it's the present. For a lot of corpora that's fine. For the ones that move, the fix isn't a smarter model or a bigger context window, it's admitting the index has to be *fed*, not photographed.

Streaming ingestion is that feeding tube. A change becomes an event, the event rides a durable stream, gets embedded once on the way through, and upserts a single item in the index within seconds. The query side never even knows. And the two things that turn it from a demo into a system you can trust are the least glamorous parts: **upsert by stable ID, and handle deletes.** Get those right and your bot stops answering yesterday's questions with yesterday's data.

## References

I built the argument and the diagrams from scratch, but the pattern isn't mine to claim: AWS publishes an official reference architecture for exactly this, and the surrounding docs are worth reading if you're implementing it. Everything below is a link, nothing here is copied from them.

<div class="st-refs">
<a href="https://docs.aws.amazon.com/architecture-diagrams/latest/exploring-real-time-streaming-for-retrieval-augmented-generation/exploring-real-time-streaming-for-retrieval-augmented-generation.html"><span class="src">AWS Architecture Diagrams</span>Exploring real-time streaming for retrieval-augmented generation, the official reference architecture for this whole pipeline</a>
<a href="https://aws.amazon.com/blogs/database/vector-search-for-amazon-dynamodb-with-zero-etl-for-amazon-opensearch-service/"><span class="src">AWS Database Blog</span>Vector search for Amazon DynamoDB with zero-ETL for Amazon OpenSearch Service</a>
<a href="https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/OpenSearchIngestionForDynamoDB.html"><span class="src">Amazon DynamoDB docs</span>DynamoDB zero-ETL integration with Amazon OpenSearch Service, DynamoDB Streams replicating changes in near real time</a>
<a href="https://aws.amazon.com/blogs/big-data/generate-vector-embeddings-for-your-data-using-aws-lambda-as-a-processor-for-amazon-opensearch-ingestion/"><span class="src">AWS Big Data Blog</span>Generate vector embeddings using AWS Lambda as a processor for Amazon OpenSearch Ingestion</a>
<a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/configure-client-ddb.html"><span class="src">Amazon OpenSearch Service docs</span>Using an OpenSearch Ingestion pipeline with Amazon DynamoDB</a>
</div>

*This extends the RAG thread: start with [what RAG really is](/blog/2026-06-21-rag-part-1-what-it-really-is/), then [chunking](/blog/2026-06-21-rag-part-2-chunking/), then [why it still fails](/blog/2026-06-21-rag-part-3-making-it-good/). For the full serving-side design and the cloud service map, see [Designing a RAG System That Actually Retrieves](/blog/2026-07-24-designing-a-rag-system-that-actually-retrieves/), part of the [AI System Design on the Cloud](/blog/2026-07-24-three-ai-systems-same-design/) series.*

<script>
(function(){
  var els=document.querySelectorAll('.st-vs,.st-flow,.st-time,.st-got');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.2});
  els.forEach(function(e){io.observe(e)});
})();
</script>
