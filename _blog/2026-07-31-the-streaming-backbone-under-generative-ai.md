---
title: "The Streaming Backbone Under Generative AI"
date: 2026-07-31
excerpt: "Everyone draws the GenAI diagram with the model in the middle and arrows pointing at it. Nobody draws the pipe those arrows actually are. That pipe is a stream: a durable, ordered, replayable log that sits between the things producing events and the LLMs, embedders, and agents consuming them. This is the plumbing under real-time RAG, live grounding, and agent-to-agent messaging, and it decides whether your system stays up when traffic spikes."
tags: [ai, streaming, kafka, generative-ai, architecture, agents]
---

<style>
.bb-fig{margin:2.5rem 0;}
.bb-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* architecture: producers -> stream(log) -> consumers */
.bb-arch{max-width:840px;margin:0 auto;overflow-x:auto;}
.bb-arch-row{display:flex;flex-wrap:nowrap;gap:.5rem;align-items:stretch;min-width:680px;}
.bb-col{flex:1 1 0;min-width:150px;display:flex;flex-direction:column;gap:.5rem;}
.bb-col>.bb-hd{font-family:var(--font-mono);font-size:.68rem;text-transform:uppercase;letter-spacing:.05em;color:var(--text-3);text-align:center;margin-bottom:.2rem;}
.bb-box{border:1px solid var(--border-2);border-radius:10px;background:var(--surface);padding:.6rem .7rem;opacity:0;transform:translateY(10px);transition:opacity .45s ease,transform .45s ease;}
.bb-arch.go .bb-box{opacity:1;transform:none;}
.bb-arch.go .bb-col:nth-child(3) .bb-box{transition-delay:.15s}
.bb-arch.go .bb-col:nth-child(5) .bb-box{transition-delay:.35s}
.bb-box b{display:block;color:var(--text);font-size:.8rem;line-height:1.25;}
.bb-box span{font-size:.72rem;color:var(--text-3);line-height:1.4;}
.bb-box.acc{border-color:var(--accent);}
.bb-mid{align-self:center;flex:none;color:var(--text-3);font-family:var(--font-mono);font-size:1rem;}
/* the log: partitioned offsets */
.bb-log{border:1px solid var(--accent);border-radius:10px;background:var(--surface-2);padding:.65rem;display:flex;flex-direction:column;gap:.4rem;}
.bb-log .plab{font-family:var(--font-mono);font-size:.66rem;color:var(--accent);}
.bb-part{display:flex;gap:.28rem;}
.bb-cell{flex:1;height:20px;border-radius:4px;background:var(--surface);border:1px solid var(--border);opacity:0;transform:scale(.6);transition:opacity .3s ease,transform .3s ease;}
.bb-arch.go .bb-cell{opacity:1;transform:none;}
.bb-arch.go .bb-part:nth-child(2) .bb-cell:nth-child(1){transition-delay:.4s}
.bb-arch.go .bb-part:nth-child(2) .bb-cell:nth-child(2){transition-delay:.5s}
.bb-arch.go .bb-part:nth-child(2) .bb-cell:nth-child(3){transition-delay:.6s}
.bb-arch.go .bb-part:nth-child(2) .bb-cell:nth-child(4){transition-delay:.7s}
.bb-arch.go .bb-part:nth-child(3) .bb-cell:nth-child(1){transition-delay:.55s}
.bb-arch.go .bb-part:nth-child(3) .bb-cell:nth-child(2){transition-delay:.65s}
.bb-arch.go .bb-part:nth-child(3) .bb-cell:nth-child(3){transition-delay:.75s}
.bb-arch.go .bb-part:nth-child(3) .bb-cell:nth-child(4){transition-delay:.85s}
.bb-cell.fill{background:color-mix(in srgb,var(--accent) 30%,transparent);border-color:var(--accent);}

/* ordering / partitioning */
.bb-ord{max-width:680px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:.9rem;}
@media(max-width:540px){.bb-ord{grid-template-columns:1fr;}}
.bb-ord .side{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.bb-ord .side h4{margin:0 0 .7rem;font-size:.8rem;font-family:var(--font-mono);}
.bb-ord .side.same{border-color:var(--accent);} .bb-ord .side.same h4{color:var(--accent);}
.bb-ord .side.diff h4{color:var(--text-2);}
.bb-lane{display:flex;align-items:center;gap:.4rem;margin:.5rem 0;font-family:var(--font-mono);font-size:.72rem;}
.bb-lane .key{flex:none;color:var(--text-3);width:2.6rem;}
.bb-ev{flex:none;width:22px;height:22px;border-radius:5px;border:1px solid var(--border-2);background:var(--surface-2);display:flex;align-items:center;justify-content:center;font-size:.66rem;color:var(--text-2);opacity:0;transform:translateX(-8px);transition:opacity .4s ease,transform .4s ease;}
.bb-ord.go .bb-ev{opacity:1;transform:none;}
.bb-ord.go .bb-lane .bb-ev:nth-child(2){transition-delay:.05s}
.bb-ord.go .bb-lane .bb-ev:nth-child(3){transition-delay:.2s}
.bb-ord.go .bb-lane .bb-ev:nth-child(4){transition-delay:.35s}
.bb-ord.go .bb-lane .bb-ev:nth-child(5){transition-delay:.5s}
.bb-ev.acc{border-color:var(--accent);background:color-mix(in srgb,var(--accent) 22%,transparent);color:var(--accent);}
.bb-ord .note{font-family:var(--font-mono);font-size:.7rem;color:var(--text-3);margin-top:.6rem;line-height:1.4;}
.bb-ord .side.same .note{color:var(--accent);}

/* delivery semantics */
.bb-del{max-width:720px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr 1fr;gap:.7rem;}
@media(max-width:620px){.bb-del{grid-template-columns:1fr;}}
.bb-d{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);opacity:0;transform:translateY(10px);transition:opacity .5s ease,transform .5s ease;}
.bb-del.go .bb-d{opacity:1;transform:none;}
.bb-del.go .bb-d:nth-child(2){transition-delay:.15s} .bb-del.go .bb-d:nth-child(3){transition-delay:.3s}
.bb-d h4{margin:0 0 .5rem;font-size:.78rem;font-family:var(--font-mono);color:var(--text);}
.bb-d.good{border-color:var(--accent);} .bb-d.good h4{color:var(--accent);}
.bb-d .glyph{font-family:var(--font-mono);font-size:.9rem;letter-spacing:.15em;margin:.3rem 0 .6rem;}
.bb-d .glyph .drop{color:var(--text-3);} .bb-d .glyph .dup{color:var(--text-2);} .bb-d .glyph .ok{color:var(--accent);}
.bb-d p{font-size:.8rem;color:var(--text-2);line-height:1.5;margin:0;}
.bb-d .cost{font-family:var(--font-mono);font-size:.68rem;color:var(--text-3);margin-top:.6rem;display:block;}

/* back-pressure */
.bb-bp{max-width:700px;margin:0 auto;display:flex;flex-direction:column;gap:1rem;}
.bb-bp .track{border:1px solid var(--border);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);}
.bb-bp .track .lb{font-family:var(--font-mono);font-size:.74rem;margin-bottom:.7rem;}
.bb-bp .track.melt .lb{color:var(--text-2);} .bb-bp .track.buf .lb{color:var(--accent);}
.bb-pipe-row{display:flex;align-items:center;gap:.5rem;}
.bb-src,.bb-sink{flex:none;font-family:var(--font-mono);font-size:.68rem;color:var(--text-3);width:4.5rem;line-height:1.3;}
.bb-flow{flex:1;height:26px;border-radius:6px;background:var(--surface-2);position:relative;overflow:hidden;border:1px solid var(--border);}
.bb-surge{position:absolute;top:0;bottom:0;left:0;width:0;transition:width 1.2s ease;}
.bb-bp.go .bb-flow .bb-surge{width:100%;}
.bb-bp .track.melt .bb-surge{background:linear-gradient(90deg,color-mix(in srgb,var(--text-3) 55%,transparent),color-mix(in srgb,var(--text-2) 45%,transparent));}
.bb-bp .track.buf .bb-buffer{position:absolute;top:3px;bottom:3px;left:3px;right:3px;border-radius:4px;border:1px dashed var(--accent);}
.bb-bp .track.buf .bb-surge{background:linear-gradient(90deg,var(--accent),color-mix(in srgb,var(--accent) 25%,transparent));width:0;}
.bb-bp.go .track.buf .bb-surge{width:35%;}
.bb-bp .cap{font-family:var(--font-mono);font-size:.7rem;color:var(--text-3);margin-top:.55rem;}
.bb-bp .track.buf .cap{color:var(--accent);}

/* windowing */
.bb-win{max-width:700px;margin:0 auto;display:flex;flex-direction:column;gap:1.3rem;}
.bb-win .wrow{border:1px solid var(--border);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);}
.bb-win .wrow .lb{font-family:var(--font-mono);font-size:.74rem;color:var(--text-2);margin-bottom:.8rem;}
.bb-win .wrow.slide .lb{color:var(--accent);}
.bb-stream{position:relative;height:44px;}
.bb-dots{display:flex;gap:.55rem;position:relative;z-index:2;padding-top:14px;}
.bb-dot{width:12px;height:12px;border-radius:50%;background:var(--accent);opacity:.75;flex:none;}
.bb-wins{position:absolute;top:0;left:0;right:0;bottom:0;z-index:1;}
.bb-w{position:absolute;top:0;bottom:0;border:1px solid var(--accent);border-radius:6px;background:color-mix(in srgb,var(--accent) 8%,transparent);opacity:0;transition:opacity .5s ease;}
.bb-win.go .bb-w{opacity:1;}
.bb-win.go .wrow.slide .bb-w:nth-child(2){transition-delay:.2s}
.bb-win.go .wrow.slide .bb-w:nth-child(3){transition-delay:.4s}
.bb-win .cap{font-family:var(--font-mono);font-size:.7rem;color:var(--text-3);margin-top:.6rem;}

/* cloud table */
.bb-tab-wrap{max-width:820px;margin:0 auto;overflow-x:auto;border:1px solid var(--border);border-radius:12px;background:var(--surface);}
.bb-tab{width:100%;border-collapse:collapse;font-size:.85rem;min-width:640px;}
.bb-tab th,.bb-tab td{text-align:left;padding:.65rem .8rem;border-bottom:1px solid var(--border);vertical-align:top;}
.bb-tab th{font-family:var(--font-mono);font-size:.7rem;text-transform:uppercase;letter-spacing:.05em;color:var(--text-3);font-weight:500;}
.bb-tab td:first-child,.bb-tab th:first-child{color:var(--text);font-weight:500;font-family:var(--font-mono);font-size:.74rem;}
.bb-tab tr:last-child td{border-bottom:none;}
.bb-tab .k-c{color:var(--accent);}

/* references */
.bb-refs{max-width:680px;margin:0 auto;border:1px solid var(--border);border-radius:12px;background:var(--surface-2);padding:.4rem 1.1rem;}
.bb-refs a{display:block;padding:.6rem 0;border-top:1px solid var(--border);font-size:.85rem;color:var(--text-2);line-height:1.4;}
.bb-refs a:first-child{border-top:none;}
.bb-refs a:hover{color:var(--accent);}
.bb-refs a .src{font-family:var(--font-mono);font-size:.68rem;color:var(--text-3);display:block;margin-bottom:.15rem;letter-spacing:.03em;}

@media (prefers-reduced-motion: reduce){
  .bb-box,.bb-cell,.bb-ev,.bb-d,.bb-w{opacity:1!important;transform:none!important;}
  .bb-surge{transition:none!important;}
  .bb-bp.go .track.melt .bb-surge{width:100%;} .bb-bp.go .track.buf .bb-surge{width:35%;}
}
</style>

A load test hits a demo I once watched go sideways. Not the model, the model was fine. The moment traffic tripled, the app started calling the embedder in its own request path, one embed per incoming event, synchronously. The embedder throttled, the requests backed up, the whole thing fell over in about forty seconds. The fix wasn't a bigger model or a faster GPU. It was a pipe. A stream in the middle that soaked up the burst and let the embedder work at its own pace.

That pipe is the part nobody draws. Every generative-AI architecture diagram puts the LLM in the center with arrows pointing at it, and treats those arrows as if they're free. They aren't. They're the streaming backbone, and it's doing more work than the model on a bad day.

## What the stream actually does for GenAI

Strip away the vendor names and a stream is one thing: a **durable, ordered, replayable log** sitting between the things that produce events and the things that consume them. Producers append. Consumers read at their own pace and remember where they were. Nobody blocks anybody. In a generative-AI system it quietly does three jobs.

- **Feeds live context into LLMs.** Real-time grounding. A prompt built from what's happening *now*, the last few minutes of alerts, the current order state, this session's clicks, instead of whatever was true at deploy time.
- **Keeps RAG indexes fresh.** Every change to a source becomes an event, gets re-embedded on the way through, and upserts a single vector so retrieval reflects reality. (That's its own post, the [companion piece on streaming into RAG](/blog/2026-07-31-streaming-data-into-rag-keeping-the-index-live/).)
- **Moves events between agents.** Agent-to-agent messaging. One agent emits a result, others react, without every agent holding a direct HTTP connection to every other agent.

Same pipe, three jobs. Here's its shape.

## The backbone, end to end

<figure class="bb-fig bb-arch bb-anim">
<div class="bb-arch">
<div class="bb-arch-row">
  <div class="bb-col">
    <div class="bb-hd">Producers</div>
    <div class="bb-box"><b>App events</b><span>clicks, orders, messages</span></div>
    <div class="bb-box"><b>Database CDC</b><span>row changed, row deleted</span></div>
    <div class="bb-box"><b>Signals</b><span>logs, metrics, IoT</span></div>
  </div>
  <div class="bb-mid">&rarr;</div>
  <div class="bb-col">
    <div class="bb-hd">The stream (a log)</div>
    <div class="bb-log">
      <div class="plab">topic, split into partitions</div>
      <div class="bb-part"><span style="font-family:var(--font-mono);font-size:.6rem;color:var(--text-3);width:2.2rem;flex:none;align-self:center;">p0</span><div class="bb-cell fill"></div><div class="bb-cell fill"></div><div class="bb-cell"></div><div class="bb-cell"></div></div>
      <div class="bb-part"><span style="font-family:var(--font-mono);font-size:.6rem;color:var(--text-3);width:2.2rem;flex:none;align-self:center;">p1</span><div class="bb-cell fill"></div><div class="bb-cell fill"></div><div class="bb-cell fill"></div><div class="bb-cell"></div></div>
      <div class="bb-part"><span style="font-family:var(--font-mono);font-size:.6rem;color:var(--text-3);width:2.2rem;flex:none;align-self:center;">p2</span><div class="bb-cell fill"></div><div class="bb-cell fill"></div><div class="bb-cell"></div><div class="bb-cell"></div></div>
      <div class="plab" style="color:var(--text-3);">offsets grow left to right, never rewritten</div>
    </div>
  </div>
  <div class="bb-mid">&rarr;</div>
  <div class="bb-col">
    <div class="bb-hd">Consumers</div>
    <div class="bb-box acc"><b>Embedder</b><span>text to vector, upsert index</span></div>
    <div class="bb-box acc"><b>LLM feeder</b><span>build prompt, call the model</span></div>
    <div class="bb-box acc"><b>Agents</b><span>react to events, emit new ones</span></div>
  </div>
</div>
</div>
<figcaption>Producers append, the log holds ordered records per partition, consumers read at their own pace and track their own offset.</figcaption>
</figure>

The log is the interesting box. It's not a queue that deletes on read, it's an append-only log split into **partitions**, and each record has a position (an **offset** in Kafka, a **sequence number** in Kinesis). Records aren't removed when a consumer reads them, they age out on a retention policy. So a slow consumer can catch up later, a new consumer can replay from the beginning, and a crashed one resumes from its last committed offset instead of losing everything. That single property, replayability, is what makes the backbone safe to build a GenAI pipeline on.

Four clouds, one idea, slightly different vocabulary. Kinesis gives you **shards, partition keys, and sequence numbers**. Kafka gives you **topics, partitions, offsets, and consumer groups**. Pub/Sub gives you **topics with ordering keys**. Event Hubs gives you **partitions, consumer groups, and checkpoints**, and it speaks the Kafka protocol so a lot of Kafka tooling just works against it. Learn one, and the others are mostly renaming.

## Ordering is only a per-partition promise

Here's the thing that bites people. A stream does not guarantee global order. It guarantees order **within a single partition**, and that's it. Which means where a record lands is a decision you make, through the **partition key**.

<figure class="bb-fig bb-ord bb-anim">
<div class="bb-ord">
  <div class="side same">
    <h4>Same key, same partition</h4>
    <div class="bb-lane"><span class="key">order-42</span><span class="bb-ev acc">1</span><span class="bb-ev acc">2</span><span class="bb-ev acc">3</span><span class="bb-ev acc">4</span></div>
    <p class="note">All of order-42's events route to one partition. They stay in order. The consumer sees created, then paid, then shipped, never shuffled.</p>
  </div>
  <div class="side diff">
    <h4>Different keys, parallel</h4>
    <div class="bb-lane"><span class="key">order-42</span><span class="bb-ev">1</span><span class="bb-ev">2</span></div>
    <div class="bb-lane"><span class="key">order-77</span><span class="bb-ev">1</span><span class="bb-ev">2</span></div>
    <div class="bb-lane"><span class="key">order-88</span><span class="bb-ev">1</span></div>
    <p class="note">Different orders spread across partitions and process in parallel. Fast, and fine, because you never needed order-42 and order-77 ordered against each other.</p>
  </div>
</div>
<figcaption>Route by entity key. Everything for one entity stays ordered, everything across entities runs parallel.</figcaption>
</figure>

Why a GenAI system cares: out-of-order events quietly corrupt state. If two edits to the same document arrive out of order, the *older* embedding can overwrite the *newer* one, and now your fresh vector is silently stale. If an agent watching an order sees "shipped" before "paid," it reasons off a state that never existed. The rule is boring and load-bearing: **route all events for one entity to the same partition** by using the entity's ID as the partition key. Everything for that entity stays ordered, everything across entities still runs in parallel. You get correctness where you need it and throughput everywhere else.

## Delivery semantics: pick your poison

When you wire a producer to a stream to a consumer, exactly-once sounds like the obvious thing to want. It's rarely what you want to pay for. There are three modes, and they trade the same two failures against cost.

<figure class="bb-fig bb-del bb-anim">
<div class="bb-del">
  <div class="bb-d">
    <h4>At-most-once</h4>
    <div class="glyph"><span class="drop">● ○ ● ●</span></div>
    <p>Fire and forget. On failure, the record is just gone. Fastest, cheapest, and it loses data. Fine for lossy telemetry, wrong for anything that matters.</p>
    <span class="cost">risk: silent loss</span>
  </div>
  <div class="bb-d good">
    <h4>At-least-once</h4>
    <div class="glyph"><span class="ok">● ● ●● ●</span></div>
    <p>Retry until acknowledged. Nothing is lost, but a retry after a lost ack means the same record shows up twice. The common default, and duplicates are manageable.</p>
    <span class="cost">risk: duplicates</span>
  </div>
  <div class="bb-d">
    <h4>Exactly-once</h4>
    <div class="glyph"><span class="ok">● ● ● ●</span></div>
    <p>Idempotent producer plus transactions dedupe end to end. No loss, no dupes, at a throughput cost (Confluent measures roughly 3% on Kafka). Real, not free.</p>
    <span class="cost">cost: ~3% throughput</span>
  </div>
</div>
<figcaption>Loss, duplicates, or overhead. Every streaming system is choosing two of the three to tolerate.</figcaption>
</figure>

For generative AI there's a pragmatic combo that beats reaching for exactly-once by reflex: **at-least-once delivery plus an idempotent upsert-by-ID on the consumer.** Think about what a duplicate actually does in a GenAI pipeline. A repeated "document 42 changed" event just re-embeds the same text and upserts it to the same vector ID. The second write lands on top of the first and produces the identical result. Harmless. You made the operation idempotent at the sink, so you don't need the stream to guarantee no duplicates. You get exactly-once *effects* without paying for exactly-once *delivery*. (Worth noting the 3% figure comes from Confluent, who are Kafka-aligned, but the shape of the tradeoff holds regardless of vendor.)

## Back-pressure is the reason the pipe exists

Remember the demo that fell over in forty seconds. This is the visual of why.

<figure class="bb-fig bb-bp bb-anim">
<div class="bb-bp">
  <div class="track melt">
    <div class="lb">No stream: producer calls the embedder directly</div>
    <div class="bb-pipe-row">
      <span class="bb-src">burst of events</span>
      <div class="bb-flow"><div class="bb-surge"></div></div>
      <span class="bb-sink">embedder throttles</span>
    </div>
    <div class="cap">the spike hits the model at full force, it throttles or the bill explodes</div>
  </div>
  <div class="track buf">
    <div class="lb">With a stream: the log absorbs the spike</div>
    <div class="bb-pipe-row">
      <span class="bb-src">burst of events</span>
      <div class="bb-flow"><div class="bb-buffer"></div><div class="bb-surge"></div></div>
      <span class="bb-sink">embedder steady</span>
    </div>
    <div class="cap">the buffer soaks the burst, the consumer pulls at a safe rate, back-pressure travels upstream</div>
  </div>
</div>
<figcaption>Without a stream, a spike goes straight at the model. With one, the log is a shock absorber.</figcaption>
</figure>

Back-pressure is the polite word for "the consumer is slower than the producer, so slow the producer down." Without a stream, there's nowhere to put the overflow, so the pressure lands on whatever is downstream, usually your embedder or your model endpoint, and it throttles you or runs up a bill you didn't budget for. With a stream, the log is a buffer. Producers keep appending, the log holds the backlog, and consumers **pull** at whatever rate they can sustain. A traffic spike becomes a temporary rise in lag instead of an outage. This is the single biggest reason a GenAI system in production has a stream in the middle rather than a direct call. Models and embedders have rate limits and real per-token cost. You do not want a viral moment translating directly into throttled requests or a five-figure invoice.

## Windowing: aggregating before you spend a token

Sometimes you don't want to feed the LLM every event. You want a *summary* of a slice of time. "Summarize the last five minutes of alerts." That's a window, and the stream-processing layer (Apache Flink and friends) gives you two shapes of it.

<figure class="bb-fig bb-win bb-anim">
<div class="bb-win">
  <div class="wrow">
    <div class="lb">Tumbling: fixed, non-overlapping</div>
    <div class="bb-stream">
      <div class="bb-wins">
        <div class="bb-w" style="left:0;width:31%;"></div>
        <div class="bb-w" style="left:34.5%;width:31%;"></div>
        <div class="bb-w" style="left:69%;width:31%;"></div>
      </div>
      <div class="bb-dots"><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span></div>
    </div>
    <div class="cap">each event lands in exactly one window, one summary per bucket</div>
  </div>
  <div class="wrow slide">
    <div class="lb">Sliding: fixed size, overlapping</div>
    <div class="bb-stream">
      <div class="bb-wins">
        <div class="bb-w" style="left:0;width:40%;"></div>
        <div class="bb-w" style="left:22%;width:40%;"></div>
        <div class="bb-w" style="left:44%;width:40%;"></div>
      </div>
      <div class="bb-dots"><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span><span class="bb-dot"></span></div>
    </div>
    <div class="cap">windows overlap, an event can belong to several, smoother rolling view</div>
  </div>
</div>
<figcaption>Tumbling buckets partition time cleanly. Sliding windows overlap for a rolling read. Pick per question.</figcaption>
</figure>

A tumbling window chops time into fixed, non-overlapping buckets: every five minutes, take everything that arrived and produce one summary. Clean, one output per bucket, and each event counts once. A sliding window is a fixed span that advances by a smaller step, so windows overlap and an event can belong to several. You'd use tumbling for "give me a fresh five-minute digest every five minutes" and sliding for "keep me a rolling five-minute view that updates every minute." Either way, the point for GenAI is the same: you aggregate the raw firehose down to something worth a prompt *before* you spend tokens on it, instead of calling the model once per raw event.

## The stream-processing layer

The log holds events. Something has to transform them before the GenAI step, chunk text, compute an embedding, aggregate a window, filter noise. That's the stream-processing layer, and it sits directly on the stream: **Apache Flink**, its managed forms (AWS runs a Managed Service for Apache Flink), or **Kafka Streams** if you're in Kafka's world. AWS publishes the canonical shape of this for GenAI: **Kinesis to Managed Flink to Bedrock (Claude) to OpenSearch.** Events land in Kinesis, Flink reads and windows and enriches them, calls Bedrock to run a model or an embedding, and writes the result to OpenSearch for retrieval. Every box is doing one job, and the stream is what lets them run at independent speeds without one starving or drowning the next.

## Events between agents, not HTTP between agents

The newest job for the backbone is the most interesting. As soon as you have more than a couple of agents talking to each other, the naive design (every agent holds a direct HTTP connection to every other agent) turns into a mess. Ten agents wired point to point is up to ninety directional connections to build, secure, and debug, and it grows with the square of the count. Add one agent and you touch every other.

Put a stream in the middle and it goes linear. Each agent **publishes** its results to a topic and **subscribes** to the topics it cares about. Nobody needs a direct line to anybody. An agent can be added, restarted, or scaled without rewiring the others, and because it's a durable log, a new agent can even replay history to catch up on what it missed. This is the emerging pattern behind event-driven multi-agent systems, and it's why people connecting agent-to-agent protocols like A2A and MCP keep landing on Kafka as the broker underneath: the coordination problem between agents is the same pub/sub problem the streaming world solved a decade ago. (Confluent has written the loudest version of this argument, and yes, they sell Kafka, but the quadratic-to-linear point stands on its own.)

## The same backbone across clouds

Four vendors, one mental model. Here's the concept map so a diagram on one cloud reads on any other.

<figure class="bb-fig">
<div class="bb-tab-wrap">
<table class="bb-tab">
<thead>
<tr><th>Concept</th><th class="k-c">Kafka / Kinesis</th><th>Pub/Sub</th><th>Event Hubs</th></tr>
</thead>
<tbody>
<tr><td>The stream</td><td>Topic / stream</td><td>Topic</td><td>Event hub (topic)</td></tr>
<tr><td>Ordering unit</td><td>Partition / shard (+ key)</td><td>Ordering key</td><td>Partition (+ key)</td></tr>
<tr><td>Delivery guarantee</td><td>At-least-once, exactly-once opt-in</td><td>At-least-once, ordered opt-in</td><td>At-least-once, checkpointed</td></tr>
<tr><td>Managed offering</td><td>MSK / Confluent / Kinesis</td><td>Pub/Sub (fully managed)</td><td>Event Hubs (fully managed)</td></tr>
<tr><td>Stream processing</td><td>Flink / Kafka Streams</td><td>Dataflow (Beam)</td><td>Stream Analytics / Flink</td></tr>
</tbody>
</table>
</div>
<figcaption>Different names, one shape. A log you partition for order, read at your own pace, and process on the way through.</figcaption>
</figure>

## The takeaway

The model gets the credit. The backbone keeps it standing. Every serious generative-AI system, real-time grounding, live RAG, multi-agent coordination, has a durable ordered log in the middle doing the unglamorous work: absorbing spikes so a viral moment doesn't throttle your endpoint, preserving per-entity order so a stale vector never overwrites a fresh one, and letting agents talk through topics instead of a spaghetti of HTTP. Partition by entity key for the ordering you need. Lean on at-least-once plus idempotent upserts and skip paying for exactly-once you don't. Let the log take the back-pressure. Do that and the interesting failure mode stops being "the whole thing fell over in forty seconds" and starts being "lag went up for a minute, then recovered." Which is the entire point.

This is one of three siblings on the streaming backbone. This post is the plumbing; *Streaming Context Into LLMs and Agents* covers feeding live context in on the read side, and *Streaming LLM Output: Tokens and Tool Calls* covers streaming the model's output back out. Read together they're the full round trip.

## References

I wrote the argument and drew every diagram from scratch. The patterns aren't mine to claim, though, and the docs and posts below are where the concepts are specified and where the AWS reference pipeline lives. Nothing here is copied from them.

<div class="bb-refs">
<a href="https://aws.amazon.com/blogs/big-data/build-a-real-time-streaming-generative-ai-application-using-amazon-bedrock-amazon-managed-service-for-apache-flink-and-amazon-kinesis-data-streams/"><span class="src">AWS Big Data Blog</span>Build a real-time streaming generative AI application with Bedrock, Managed Service for Apache Flink, and Kinesis Data Streams (the Kinesis to Flink to Bedrock to OpenSearch pattern)</a>
<a href="https://docs.aws.amazon.com/architecture-diagrams/latest/exploring-real-time-streaming-for-retrieval-augmented-generation/exploring-real-time-streaming-for-retrieval-augmented-generation.html"><span class="src">AWS Architecture Diagrams</span>Exploring real-time streaming for retrieval-augmented generation (CDC to stream to vector store to Bedrock)</a>
<a href="https://www.confluent.io/blog/mastering-real-time-retrieval-augmented-generation-rag-with-flink/"><span class="src">Confluent Blog</span>Mastering real-time retrieval-augmented generation with Flink</a>
<a href="https://www.confluent.io/blog/google-agent2agent-protocol-needs-kafka/"><span class="src">Confluent Blog</span>Why Google's Agent2Agent protocol needs Kafka (events between agents)</a>
<a href="https://docs.aws.amazon.com/streams/latest/dev/key-concepts.html"><span class="src">Amazon Kinesis docs</span>Kinesis Data Streams key concepts: shards, partition keys, sequence numbers</a>
<a href="https://docs.confluent.io/kafka/design/delivery-semantics.html"><span class="src">Confluent / Kafka docs</span>Message delivery semantics: at-most-once, at-least-once, exactly-once</a>
<a href="https://docs.cloud.google.com/pubsub/docs/ordering"><span class="src">Google Cloud docs</span>Pub/Sub message ordering with ordering keys</a>
<a href="https://learn.microsoft.com/en-us/azure/event-hubs/event-hubs-features"><span class="src">Microsoft Learn</span>Azure Event Hubs features: partitions, consumer groups, checkpointing, Kafka compatibility</a>
</div>

*Part of the [AI System Design on the Cloud](/blog/2026-07-24-three-ai-systems-same-design/) series. For the systems this backbone feeds, see [Designing a RAG System That Actually Retrieves](/blog/2026-07-24-designing-a-rag-system-that-actually-retrieves/) and [Designing an Agent That Doesn't Go Off the Rails](/blog/2026-07-24-designing-an-agent-that-doesnt-go-off-the-rails/). The direct companion is [Streaming Data Into RAG: Keeping the Index Live](/blog/2026-07-31-streaming-data-into-rag-keeping-the-index-live/).*

<script>
(function(){
  var els=document.querySelectorAll('.bb-anim');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.2});
  els.forEach(function(e){io.observe(e)});
})();
</script>
