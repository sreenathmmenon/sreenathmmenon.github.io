---
title: "Streaming LLM Output: Tokens and Tool Calls"
date: 2026-07-31
excerpt: "An LLM writes its answer one token at a time either way. Streaming just shows you those tokens as they land instead of hiding them behind a blank screen. Same total time, wildly different feeling. Here's how it works over the wire, and why streaming through an agent loop is the part that quietly breaks."
tags: [ai, streaming, llm, sse, generative-ai, architecture]
---

<style>
.so-fig{margin:2.5rem 0;}
.so-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* 1. typing vs wall of text */
.so-hook{display:grid;grid-template-columns:1fr 1fr;gap:.9rem;max-width:720px;margin:0 auto;}
@media(max-width:620px){.so-hook{grid-template-columns:1fr;}}
.so-hook .pane{border:1px solid var(--border);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);min-height:190px;display:flex;flex-direction:column;}
.so-hook .pane h4{margin:0 0 .7rem;font-size:.78rem;font-family:var(--font-mono);text-transform:uppercase;letter-spacing:.06em;}
.so-hook .buffered h4{color:var(--text-3);}
.so-hook .streamed{border-color:var(--accent);}
.so-hook .streamed h4{color:var(--accent);}
.so-hook .screen{flex:1;font-family:var(--font-mono);font-size:.8rem;line-height:1.55;color:var(--text-2);}
.so-hook .wait{color:var(--text-3);font-style:italic;display:flex;align-items:center;gap:.5rem;}
.so-hook .spin{width:11px;height:11px;border:2px solid var(--border-2);border-top-color:var(--text-3);border-radius:50%;display:inline-block;}
.so-hook.go .buffered .spin{animation:so-spin 1s linear infinite;}
@keyframes so-spin{to{transform:rotate(360deg);}}
.so-hook .buffered .dump{opacity:0;}
.so-hook.go .buffered .dump{animation:so-dump 0s linear 4.2s forwards;}
@keyframes so-dump{to{opacity:1;}}
.so-hook.go .buffered .wait{animation:so-hide 0s linear 4.2s forwards;}
@keyframes so-hide{to{display:none;opacity:0;}}
.so-hook .streamed .tok{opacity:0;}
.so-hook.go .streamed .tok{animation:so-fade .18s ease forwards;}
.so-hook.go .streamed .tok:nth-child(1){animation-delay:.3s}
.so-hook.go .streamed .tok:nth-child(2){animation-delay:.5s}
.so-hook.go .streamed .tok:nth-child(3){animation-delay:.7s}
.so-hook.go .streamed .tok:nth-child(4){animation-delay:.95s}
.so-hook.go .streamed .tok:nth-child(5){animation-delay:1.2s}
.so-hook.go .streamed .tok:nth-child(6){animation-delay:1.5s}
.so-hook.go .streamed .tok:nth-child(7){animation-delay:1.85s}
.so-hook.go .streamed .tok:nth-child(8){animation-delay:2.2s}
.so-hook.go .streamed .tok:nth-child(9){animation-delay:2.6s}
.so-hook.go .streamed .tok:nth-child(10){animation-delay:3s}
@keyframes so-fade{to{opacity:1;}}
.so-cur{display:inline-block;width:7px;height:1.05em;background:var(--accent);vertical-align:text-bottom;margin-left:1px;}
.so-hook.go .streamed .so-cur{animation:so-blink 1s step-end infinite;}
@keyframes so-blink{50%{opacity:0;}}

/* 2. SSE event flow */
.so-sse{max-width:560px;margin:0 auto;border:1px solid var(--border);border-radius:12px;background:var(--surface);overflow:hidden;}
.so-sse .conn{font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);padding:.55rem .9rem;border-bottom:1px solid var(--border);background:var(--surface-2);display:flex;justify-content:space-between;}
.so-sse .conn b{color:var(--green);}
.so-sse .ev{font-family:var(--font-mono);font-size:.78rem;padding:.45rem .9rem;border-bottom:1px solid var(--border);display:flex;gap:.7rem;align-items:baseline;opacity:0;transform:translateX(-8px);}
.so-sse.go .ev{animation:so-ev .35s ease forwards;}
.so-sse .ev .nm{color:var(--accent);flex:none;width:170px;}
.so-sse .ev .py{color:var(--text-3);}
.so-sse .ev.rep .nm{color:var(--green);}
.so-sse .ev:last-child{border-bottom:none;}
.so-sse.go .ev:nth-child(2){animation-delay:.15s}
.so-sse.go .ev:nth-child(3){animation-delay:.5s}
.so-sse.go .ev:nth-child(4){animation-delay:.75s}
.so-sse.go .ev:nth-child(5){animation-delay:1s}
.so-sse.go .ev:nth-child(6){animation-delay:1.25s}
.so-sse.go .ev:nth-child(7){animation-delay:1.6s}
.so-sse.go .ev:nth-child(8){animation-delay:1.85s}
@keyframes so-ev{to{opacity:1;transform:none;}}

/* 3. TTFT timeline */
.so-ttft{max-width:640px;margin:0 auto;display:flex;flex-direction:column;gap:1.1rem;}
.so-ttft .lane{}
.so-ttft .lane .cap{font-family:var(--font-mono);font-size:.75rem;margin-bottom:.4rem;display:flex;justify-content:space-between;}
.so-ttft .lane.buf .cap{color:var(--text-3);}
.so-ttft .lane.str .cap{color:var(--accent);}
.so-ttft .track{height:30px;border-radius:7px;background:var(--surface-2);border:1px solid var(--border-2);position:relative;overflow:hidden;}
.so-ttft .fill{position:absolute;top:0;bottom:0;left:0;width:0;}
.so-ttft .lane.buf .fill{background:repeating-linear-gradient(45deg,var(--border-2),var(--border-2) 6px,transparent 6px,transparent 12px);}
.so-ttft .lane.str .fill{background:var(--accent);opacity:.25;}
.so-ttft.go .lane.buf .fill{animation:so-grow 4s linear forwards;}
.so-ttft.go .lane.str .fill{animation:so-grow 4s linear forwards;}
@keyframes so-grow{to{width:100%;}}
.so-ttft .mark{position:absolute;top:0;bottom:0;width:2px;background:var(--accent);opacity:0;}
.so-ttft.go .lane.str .mark{animation:so-show 0s linear .35s forwards;}
@keyframes so-show{to{opacity:1;}}
.so-ttft .mark .lbl{position:absolute;top:-1.4rem;left:4px;font-family:var(--font-mono);font-size:.65rem;color:var(--accent);white-space:nowrap;}
.so-ttft .txt{position:absolute;top:0;bottom:0;left:8px;right:8px;display:flex;align-items:center;font-family:var(--font-mono);font-size:.72rem;color:var(--text-2);opacity:0;}
.so-ttft.go .lane.str .txt{animation:so-show 0s linear .5s forwards;}
.so-ttft.go .lane.buf .txt{animation:so-show 0s linear 4s forwards;}

/* 4. agent-loop partial JSON */
.so-buf{max-width:660px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:.9rem;}
@media(max-width:600px){.so-buf{grid-template-columns:1fr;}}
.so-buf .frags,.so-buf .acc{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.so-buf .acc{border-color:var(--accent);}
.so-buf h4{margin:0 0 .7rem;font-size:.74rem;font-family:var(--font-mono);text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.so-buf .acc h4{color:var(--accent);}
.so-buf .frag{font-family:var(--font-mono);font-size:.75rem;color:var(--text-2);background:var(--surface-2);border:1px solid var(--border-2);border-radius:6px;padding:.35rem .55rem;margin-bottom:.4rem;opacity:0;transform:translateY(6px);}
.so-buf.go .frag{animation:so-fr .3s ease forwards;}
.so-buf.go .frag:nth-child(2){animation-delay:.3s}
.so-buf.go .frag:nth-child(3){animation-delay:.7s}
.so-buf.go .frag:nth-child(4){animation-delay:1.1s}
.so-buf.go .frag:nth-child(5){animation-delay:1.5s}
@keyframes so-fr{to{opacity:1;transform:none;}}
.so-buf .glass{font-family:var(--font-mono);font-size:.78rem;color:var(--text);word-break:break-all;line-height:1.5;min-height:2.6em;}
.so-buf .glass .g{opacity:0;}
.so-buf.go .glass .g{animation:so-fade .25s ease forwards;}
.so-buf.go .glass .g:nth-child(1){animation-delay:.45s}
.so-buf.go .glass .g:nth-child(2){animation-delay:.85s}
.so-buf.go .glass .g:nth-child(3){animation-delay:1.25s}
.so-buf.go .glass .g:nth-child(4){animation-delay:1.65s}
.so-buf .status{margin-top:.7rem;font-family:var(--font-mono);font-size:.72rem;padding:.35rem .55rem;border-radius:6px;border:1px solid var(--border-2);color:var(--text-3);opacity:0;}
.so-buf.go .status{animation:so-flip 0s linear 2s forwards;}
@keyframes so-flip{to{opacity:1;border-color:var(--green);color:var(--green);}}

/* 5. provider table */
.so-tab-wrap{max-width:840px;margin:0 auto;overflow-x:auto;}
.so-tab{width:100%;border-collapse:collapse;font-size:.83rem;min-width:660px;}
.so-tab th,.so-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.so-tab th{font-family:var(--font-mono);font-size:.7rem;text-transform:uppercase;letter-spacing:.05em;color:var(--text-3);}
.so-tab td:first-child,.so-tab th:first-child{color:var(--text);font-weight:500;font-family:var(--font-mono);font-size:.76rem;white-space:nowrap;}
.so-tab td{color:var(--text-2);font-family:var(--font-mono);font-size:.76rem;}

/* 6. gotchas */
.so-got{max-width:680px;margin:0 auto;display:flex;flex-direction:column;gap:.6rem;}
.so-got .g{border:1px solid var(--border);border-left:3px solid var(--accent);border-radius:10px;padding:.8rem 1rem;background:var(--surface);opacity:0;transform:translateX(-10px);}
.so-got.go .g{animation:so-slide .45s ease forwards;}
.so-got.go .g:nth-child(1){animation-delay:.1s}
.so-got.go .g:nth-child(2){animation-delay:.3s}
.so-got.go .g:nth-child(3){animation-delay:.5s}
.so-got.go .g:nth-child(4){animation-delay:.7s}
@keyframes so-slide{to{opacity:1;transform:none;}}
.so-got .g b{display:block;font-family:var(--font-mono);font-size:.82rem;color:var(--text);margin-bottom:.3rem;}
.so-got .g p{margin:0;font-size:.83rem;color:var(--text-2);line-height:1.5;}

@media (prefers-reduced-motion: reduce){
  .so-hook .pane *,.so-sse .ev,.so-ttft .fill,.so-ttft .mark,.so-ttft .txt,
  .so-buf .frag,.so-buf .glass .g,.so-buf .status,.so-got .g,.so-cur,.so-hook .spin{
    animation:none!important;opacity:1!important;transform:none!important;width:auto;}
  .so-ttft .fill{width:100%!important;}
  .so-hook .buffered .wait{display:none!important;}
}
</style>

You send a question and watch a blank rectangle. Nothing. A little spinner, maybe. Four seconds, six, eight. Then the whole answer slaps onto the screen at once, three paragraphs you now have to start reading from the top. That's the buffered experience, and it's the default if you don't do anything special.

Now the same model, same question, same eight seconds of total work. Except this time a word appears almost immediately, then another, then a sentence, and you're already reading while the model is still writing the end. By the time it finishes you've half-consumed the answer. Nothing about the model got faster. You just stopped hiding the tokens.

<figure class="so-fig">
  <div class="so-hook so-anim" id="hook">
    <div class="pane buffered">
      <h4>Buffered</h4>
      <div class="screen">
        <div class="wait"><span class="spin"></span> waiting for full response...</div>
        <div class="dump">Streaming shows tokens as the model produces them instead of holding the whole answer back.</div>
      </div>
    </div>
    <div class="pane streamed">
      <h4>Streamed</h4>
      <div class="screen">
        <span class="tok">Streaming </span><span class="tok">shows </span><span class="tok">tokens </span><span class="tok">as </span><span class="tok">the </span><span class="tok">model </span><span class="tok">produces </span><span class="tok">them, </span><span class="tok">so </span><span class="tok">you </span><span class="tok">read </span><span class="tok">as </span><span class="tok">it </span><span class="tok">writes</span><span class="so-cur"></span>
      </div>
    </div>
  </div>
  <figcaption>Left: dead air, then a wall of text. Right: same total time, text you can start reading at once. The model generates token by token either way. Streaming is just the decision to show its work.</figcaption>
</figure>

The whole input side of generative AI, retrieval, context assembly, the RAG plumbing, gets most of the attention. But the output side is where the user actually lives. And the mechanics are simpler than they look, right up until you try to stream a tool call, which is where it gets genuinely nasty. That last part is the point of this post.

## Why bother: perceived latency is the real latency

An LLM is autoregressive. It produces one token, feeds it back in, produces the next, and repeats until it decides to stop. That loop runs whether or not you stream. The only question is when the tokens reach the user: after the last one is generated (buffered), or as each one comes out (streamed).

The metric that matters here is **time to first token**, TTFT. It's the LLM's version of time-to-first-byte: how long from "you hit send" to "the first readable thing appears." Buffered responses have no meaningful TTFT, the first thing you see is also the last. Streaming splits one long wait into a short wait plus visible, readable progress. TTFT has become a standard enough concern that OpenTelemetry's GenAI semantic conventions define `gen_ai.server.time_to_first_token` as a first-class metric, which is a good sign it's a real SLO and not a vibe.

<figure class="so-fig">
  <div class="so-ttft so-anim" id="ttft">
    <div class="lane buf">
      <div class="cap"><span>Buffered</span><span>total: 4.0s</span></div>
      <div class="track"><div class="fill"></div><div class="txt">nothing... nothing... (answer appears at the very end)</div></div>
    </div>
    <div class="lane str">
      <div class="cap"><span>Streamed</span><span>total: 4.0s</span></div>
      <div class="track"><div class="fill"></div><div class="mark" style="left:9%"><span class="lbl">TTFT ~0.35s</span></div><div class="txt">first token, then a steady flow you're already reading</div></div>
    </div>
  </div>
  <figcaption>Same 4.0s of total generation. The buffered bar delivers everything at t=4.0. The streamed bar delivers something at t=0.35 and keeps going. Perceived latency is what people mean when they say an app "feels fast," and streaming wins it for free.</figcaption>
</figure>

If you've read [What Tokens Actually Are](/blog/2026-06-21-what-tokens-actually-are/), the unit flowing across the wire here is exactly that token, one chunk of the model's output at a time.

## How it works over the wire: Server-Sent Events

The transport under almost every streaming LLM API is **Server-Sent Events**, SSE. It's an old, boring, wonderfully reliable web standard, and boring is what you want under something that has to run for thirty seconds without hiccupping.

The shape: the client opens one long-lived HTTP connection, the server responds with `Content-Type: text/event-stream`, and instead of closing after one payload, it holds the connection open and pushes a sequence of events down it. Each event is plain text, optionally a named `event:` line and one or more `data:` lines, separated by a blank line. The client reads them as they arrive. That's the entire protocol, and it's specified in the WHATWG HTML standard, not some vendor's blog post.

The Anthropic Messages API is a clean example of the event sequence. You get a `message_start`, then for each piece of the reply a `content_block_start`, a run of `content_block_delta` events carrying the actual tokens, and a `content_block_stop`. Then a `message_delta` with top-level metadata like stop reason, and finally `message_stop`. Text arrives as `text_delta` inside those content-block deltas. The repeated delta events in the middle are the stream, everything else is bookkeeping around them.

<figure class="so-fig">
  <div class="so-sse so-anim" id="sse">
    <div class="conn"><span>one open connection · text/event-stream</span><span><b>● open</b></span></div>
    <div class="ev"><span class="nm">message_start</span><span class="py">role, model, usage</span></div>
    <div class="ev"><span class="nm">content_block_start</span><span class="py">index 0, type: text</span></div>
    <div class="ev rep"><span class="nm">content_block_delta</span><span class="py">text_delta: "Stream"</span></div>
    <div class="ev rep"><span class="nm">content_block_delta</span><span class="py">text_delta: "ing is"</span></div>
    <div class="ev rep"><span class="nm">content_block_delta</span><span class="py">text_delta: " simple"</span></div>
    <div class="ev"><span class="nm">content_block_stop</span><span class="py">index 0</span></div>
    <div class="ev"><span class="nm">message_stop</span><span class="py">done</span></div>
  </div>
  <figcaption>Events arriving one at a time over a single connection. The green content_block_delta rows are the tokens, everything around them is framing. Same connection stays open start to stop, no polling, no reconnect per token.</figcaption>
</figure>

Every major cloud speaks a variant of this. AWS Bedrock's `ConverseStream` gives you a unified, cross-model event set (`messageStart`, `contentBlockDelta`, `messageStop`), so the same parsing code works whether the underlying model is Claude, Llama, or something else. Google Vertex uses `streamGenerateContent`. Azure OpenAI is the familiar `stream: true` flag returning SSE chunks. Different names, same idea: one connection, a header of metadata, a middle of deltas, a footer that closes it out.

<figure class="so-fig">
  <div class="so-tab-wrap">
    <table class="so-tab">
      <thead>
        <tr><th>Aspect</th><th>Anthropic Claude</th><th>AWS Bedrock</th><th>Google Vertex</th><th>Azure OpenAI</th></tr>
      </thead>
      <tbody>
        <tr><td>Streaming call</td><td>Messages, stream:true</td><td>ConverseStream</td><td>streamGenerateContent</td><td>chat completions, stream:true</td></tr>
        <tr><td>Transport</td><td>SSE (text/event-stream)</td><td>event stream (AWS)</td><td>SSE / chunked</td><td>SSE (text/event-stream)</td></tr>
        <tr><td>Delta event</td><td>content_block_delta</td><td>contentBlockDelta</td><td>candidates[].content parts</td><td>choices[].delta</td></tr>
      </tbody>
    </table>
  </div>
  <figcaption>Four providers, one pattern. If you build your client around "open a stream, read framed deltas until stop," porting between them is mostly a rename job.</figcaption>
</figure>

## The hard part: streaming through an agent loop

Here's where naive streaming falls apart. Streaming text is easy, each delta is a printable fragment, you append it and move on. But a model doesn't only emit text. In an agent it emits **tool calls**, and a tool call has a structured JSON argument. The model streams that JSON the same way it streams prose: token by token, as fragments.

So the input to your `get_weather` tool doesn't arrive as `{"city": "Tokyo", "units": "c"}`. It arrives as `{"ci`, then `ty": "To`, then `kyo", "un`, then `its": "c"}`. Those are `input_json_delta` events (partial JSON) under one content block. You cannot parse fragment three. It isn't valid JSON. It isn't even valid until the last fragment lands and the block stops.

The rule, and Anthropic's fine-grained tool streaming docs are explicit about this: **accumulate the partial-JSON fragments per content-block index, and only parse when that block's stop event arrives.** The index matters because a single turn can stream several tool calls interleaved or in sequence, and you must not mix fragment 2 of block 0 into block 1. Buffer by index, concatenate the pieces, and `JSON.parse` exactly once, at `content_block_stop`. Try to act on a half-formed call and you'll either crash or fire a tool with garbage arguments.

<figure class="so-fig">
  <div class="so-buf so-anim" id="buf">
    <div class="frags">
      <h4>input_json_delta (block index 0)</h4>
      <div class="frag">{"ci</div>
      <div class="frag">ty": "To</div>
      <div class="frag">kyo", "un</div>
      <div class="frag">its": "c"}</div>
    </div>
    <div class="acc">
      <h4>Accumulated buffer</h4>
      <div class="glass"><span class="g">{"ci</span><span class="g">ty": "To</span><span class="g">kyo", "un</span><span class="g">its": "c"}</span></div>
      <div class="status">content_block_stop → JSON.parse → call get_weather</div>
    </div>
  </div>
  <figcaption>Tool arguments stream as partial JSON. No single fragment is parseable. You concatenate by content-block index and parse exactly once, when the block stops. This is why streaming and tool use don't naively compose: text you can print immediately, a tool call you must wait to complete.</figcaption>
</figure>

There's a subtle trap even after you get the buffering right: you can accidentally add latency by buffering *too much*. LiveKit's agent framework hit exactly this. Their LLM stream buffered tool-call deltas and, on every turn that involved a tool, introduced roughly 0.6 to 1 second of dead air before the agent spoke ([issue #5826](https://github.com/livekit/agents/issues/5826)). The fix is to keep streaming the text portion to the user while you quietly accumulate the tool-call JSON in the background, rather than stalling the whole output until the JSON is complete. The lesson: buffering is necessary for the tool arguments, and poison for everything else. Stream what you can, buffer only what you must.

Structured output, the JSON-mode and schema-constrained responses you'd use in a [RAG or extraction pipeline](/blog/2026-07-24-designing-a-rag-system-that-actually-retrieves/), streams the same way and needs the same buffer-then-parse discipline. If you're wiring streaming into a real agent, the loop design in [Designing an Agent That Doesn't Go Off the Rails](/blog/2026-07-24-designing-an-agent-that-doesnt-go-off-the-rails/) is where the tool-call handling actually lives.

## The gotchas that show up in production

None of these are exotic. All of them will find you eventually.

<figure class="so-fig">
  <div class="so-got so-anim" id="got">
    <div class="g">
      <b>Moderation on partial deltas</b>
      <p>Content filters run mid-stream, so a chunk can get flagged after you've already shown the user the tokens before it. You need a plan for retracting or halting a stream that's turned unsafe partway through, not just filtering the final blob. Azure's content-streaming docs cover the asynchronous-filter tradeoff directly.</p>
    </div>
    <div class="g">
      <b>Proxies that buffer your stream to death</b>
      <p>Load balancers, reverse proxies, and CDNs love to buffer responses for efficiency, which quietly re-assembles your SSE stream into one big delayed blob. All your TTFT work, gone. You have to explicitly disable response buffering on the path (think nginx proxy_buffering off), or the stream never reaches the client as a stream.</p>
    </div>
    <div class="g">
      <b>Reconnection and Last-Event-ID</b>
      <p>SSE has built-in reconnect: the browser remembers the last event id and sends Last-Event-ID on reconnect so the server can resume. That only works if your server actually emits ids and knows how to pick up mid-generation. Most LLM streams don't resume gracefully, so decide up front whether a dropped connection means retry-from-scratch or resume.</p>
    </div>
    <div class="g">
      <b>The ~6-connection-per-domain limit</b>
      <p>Over HTTP/1.x, browsers cap concurrent connections per domain at around six, and a held-open SSE stream eats one for its whole lifetime. A few open streams plus normal traffic and you're blocked. HTTP/2 multiplexes and mostly dissolves this, which is one more reason to serve streams over HTTP/2.</p>
    </div>
  </div>
  <figcaption>Streaming is boring to get working and finicky to get right. Every one of these is a "worked in dev, mysteriously broke in prod behind the load balancer" story waiting to happen.</figcaption>
</figure>

## The takeaway

Streaming doesn't make your model faster. It makes your model honest about the fact that it's already writing one token at a time, and it hands those tokens to the user the instant they exist instead of hoarding them for a dramatic reveal nobody asked for. The transport is a thirty-year-old web standard doing exactly what it was built to do. The perceived-latency win is basically free.

The one place it stops being free is the agent loop, where tool-call arguments arrive as partial JSON that you have to buffer by index and parse only when the block completes, streaming the prose while quietly accumulating the structure. Get that split right and you get an agent that talks while it thinks. Get it wrong and you either crash on half-formed JSON or add a second of dead silence to every tool call, which, as LiveKit found out, users notice immediately. Show the tokens. Buffer only the JSON. Ship it over HTTP/2 and turn off the proxy buffering before you wonder why none of it works.

This is one of a set of streaming posts, the [streaming backbone under generative AI](/blog/2026-07-31-the-streaming-backbone-under-generative-ai/) and [streaming context into LLMs and agents](/blog/2026-07-31-streaming-context-into-llms-and-agents/), and it sits alongside the broader [three AI systems, same design](/blog/2026-07-24-three-ai-systems-same-design/) series.

## References

- [Anthropic: streaming Messages (SSE event flow)](https://platform.claude.com/docs/en/build-with-claude/streaming)
- [Anthropic: fine-grained tool streaming (partial JSON, accumulate per index)](https://platform.claude.com/docs/en/agents-and-tools/tool-use/fine-grained-tool-streaming)
- [Claude Agent SDK: streaming output](https://code.claude.com/docs/en/agent-sdk/streaming-output)
- [AWS Bedrock ConverseStream API reference](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_ConverseStream.html)
- [Google Vertex: streamGenerateContent](https://cloud.google.com/vertex-ai/generative-ai/docs/reference/rest/v1/projects.locations.publishers.models/streamGenerateContent)
- [Azure OpenAI: content streaming](https://learn.microsoft.com/en-us/azure/foundry/openai/concepts/content-streaming)
- [WHATWG: Server-Sent Events spec](https://html.spec.whatwg.org/multipage/server-sent-events.html)
- [MDN: using server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)
- [OpenTelemetry: GenAI metrics (TTFT)](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-metrics.md)
- [Vercel AI SDK 5 (streaming architecture, typed stream parts)](https://vercel.com/blog/ai-sdk-5)
- [LiveKit Agents #5826: buffered tool-call deltas adding dead air](https://github.com/livekit/agents/issues/5826)
- [Parsing Anthropic SSE tool calls without loading the whole message](https://dev.to/gabrielanhaia/streaming-tool-calls-parse-anthropic-sse-without-loading-the-whole-message-2on)

<script>
(function(){
  var els=document.querySelectorAll('.so-anim');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.2});
  els.forEach(function(e){io.observe(e)});
})();
</script>
