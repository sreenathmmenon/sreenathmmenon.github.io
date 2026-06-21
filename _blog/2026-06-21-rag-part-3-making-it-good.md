---
title: "RAG, Part 3: Making It Actually Good (and Why It Still Fails)"
date: 2026-06-21
excerpt: "A basic RAG system retrieves by meaning and calls it a day. The good ones do more: they search by keyword and meaning together, re-judge the results, and respect the strange ways a model loses focus. This is the part that separates a demo from something you would trust."
tags: [rag, retrieval, reranking, hybrid-search, ai, llm, explainer]
---

<style>
.post-fig{margin:2.5rem 0;}
.post-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}
.series-nav{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);border:1px solid var(--border);background:var(--surface);border-radius:10px;padding:.7rem 1rem;margin-bottom:2rem;}
.series-nav b{color:var(--accent);}
.series-nav a{color:var(--accent);}

/* keyword vs meaning demo */
.km-play{max-width:620px;margin:0 auto;background:var(--surface);border:1px solid var(--border-2);border-radius:16px;padding:1.4rem;box-shadow:var(--glow);}
.km-q{display:flex;gap:.5rem;flex-wrap:wrap;margin-bottom:1.2rem;}
.km-qbtn{font-family:var(--font-mono);font-size:.78rem;padding:.4rem .8rem;border-radius:999px;background:var(--surface-2);border:1px solid var(--border-2);color:var(--text-2);cursor:pointer;transition:all .2s var(--ease);}
.km-qbtn:hover{border-color:var(--accent);color:var(--accent);}
.km-qbtn.on{background:var(--grad);color:var(--accent-ink);border-color:transparent;font-weight:600;}
.km-cols{display:grid;grid-template-columns:1fr 1fr;gap:1rem;}
@media(max-width:560px){.km-cols{grid-template-columns:1fr;}}
.km-col h4{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--accent);margin:0 0 .6rem;}
.km-hit{font-size:.86rem;padding:.5rem .6rem;border-radius:8px;background:var(--surface-2);border:1px solid var(--border);color:var(--text-2);margin-bottom:.4rem;}
.km-hit.win{border-color:var(--accent);color:var(--text);}
.km-verdict{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:1.1rem;border-top:1px solid var(--border);padding-top:.9rem;}

/* reranking funnel */
.funnel{max-width:560px;margin:0 auto;display:flex;flex-direction:column;gap:1.3rem;}
.funnel-stage{}
.funnel-label{font-family:var(--font-mono);font-size:.76rem;color:var(--accent);text-transform:uppercase;letter-spacing:.06em;margin-bottom:.5rem;}
.funnel-docs{display:flex;flex-wrap:wrap;gap:.35rem;}
.fdoc{width:26px;height:32px;border-radius:4px;background:var(--surface-2);border:1px solid var(--border-2);}
.fdoc.keep{background:var(--grad);border-color:transparent;}
.fdoc.drop{opacity:.35;}
.funnel-note{font-size:.82rem;color:var(--text-3);margin-top:.5rem;}

/* lost in the middle chart */
.litm{max-width:560px;margin:0 auto;}
.litm-bars{display:flex;align-items:flex-end;gap:.4rem;height:160px;padding:0 .5rem;}
.litm-bars .b{flex:1;background:var(--grad);border-radius:5px 5px 0 0;position:relative;transition:height .4s var(--ease);}
.litm-bars .b span{position:absolute;top:-1.3rem;left:0;right:0;text-align:center;font-family:var(--font-mono);font-size:.68rem;color:var(--text-3);}
.litm-axis{display:flex;justify-content:space-between;font-family:var(--font-mono);font-size:.7rem;color:var(--text-3);margin-top:.5rem;padding:0 .5rem;}

/* generic table */
.ctab{width:100%;border-collapse:collapse;font-size:.9rem;}
.ctab th,.ctab td{text-align:left;padding:.55rem .7rem;border-bottom:1px solid var(--border);}
.ctab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.ctab td:first-child{color:var(--text);}
.ctab .num{font-family:var(--font-mono);color:var(--accent);}
</style>

<div class="series-nav">The final part of a 3-part series on RAG. <a href="/blog/2026-06-21-rag-part-1-what-it-really-is/">Part 1</a> was the foundation, <a href="/blog/2026-06-21-rag-part-2-chunking/">Part 2</a> was chunking. <b>This part is about making retrieval actually good, and the ways it still breaks.</b></div>

By now you can build a basic RAG system. Chunk your documents, turn them into points on a map of meaning, grab the nearest ones to the question, hand them to the model. For a weekend demo, that is genuinely enough.

It is also where most people stop, and it is why most RAG demos feel slightly broken in ways nobody can quite explain. The answers are mostly right and occasionally, confidently, wrong. The gap between that demo and something you would actually trust is the subject of this part. There are three moves that close it.

## Move one: meaning search has a blind spot

In Part 1 I sold you on searching by meaning, and I meant it. But meaning search, on its own, has one real weakness, and it is worth seeing clearly.

It is fuzzy by design. It is brilliant at "money back" matching "refund," which is the whole point. But that same fuzziness makes it surprisingly bad at exact, literal things. Product codes. Error numbers. A specific person's name. The model that knows "refund" and "money back" are cousins might also think "error 404" and "error 403" are basically the same, because they look and feel almost identical in meaning-space. They are not. One of them is the answer and one of them is noise.

There is an older, dumber kind of search that is perfect at exactly this: keyword search. The classic version is called BM25, and it does not care about meaning at all. It cares about exact words and how rare and important they are. Ask it for "error 404" and it finds the documents that literally contain "404," ranked by how much those words stand out.

So you have two tools with opposite strengths. Watch them on different questions.

<figure class="post-fig">
  <div class="km-play">
    <div class="km-q" id="kmQ"></div>
    <div class="km-cols">
      <div class="km-col"><h4>Keyword (BM25)</h4><div id="kmKw"></div></div>
      <div class="km-col"><h4>Meaning (vectors)</h4><div id="kmVec"></div></div>
    </div>
    <div class="km-verdict" id="kmVerdict"></div>
  </div>
  <figcaption>Two searchers, opposite strengths. Keyword nails the exact-term query and fumbles the paraphrase. Meaning does the reverse. The accented result is the one that actually answers the question.</figcaption>
</figure>

<script>
(function(){
  var data = {
    'How do I get my money back?': {
      kw: [['Picnic refund form (2019 archive)', false], ['Server money market report', false], ['No strong keyword match found', false]],
      vec: [['Our refund procedure: contact support within 30 days', true], ['You can cancel a subscription in settings', false], ['Billing FAQ overview', false]],
      verdict: 'Paraphrase question. Meaning search wins easily: it connects "money back" to "refund" with no shared words. Keyword search flails, because none of the right words appear literally.'
    },
    'What causes error 404?': {
      kw: [['Error 404: the page or resource was not found', true], ['Error 403: access forbidden', false], ['Error 500: internal server error', false]],
      vec: [['Error 403: access forbidden', false], ['Error 404: the page or resource was not found', true], ['General troubleshooting guide', false]],
      verdict: 'Exact-term question. Keyword search wins cleanly: it locks onto the literal "404". Meaning search nearly trips, because to a vector model 403 and 404 feel almost identical. This is its blind spot.'
    }
  };
  var qbar = document.getElementById('kmQ');
  if(!qbar) return;
  var kw = document.getElementById('kmKw'), vec = document.getElementById('kmVec'), verdict = document.getElementById('kmVerdict');
  function rows(arr){ return arr.map(function(r){ return '<div class="km-hit'+(r[1]?' win':'')+'">'+r[0]+'</div>'; }).join(''); }
  function render(q){
    Array.prototype.forEach.call(qbar.children, function(b){ b.classList.toggle('on', b.dataset.q===q); });
    kw.innerHTML = rows(data[q].kw); vec.innerHTML = rows(data[q].vec); verdict.textContent = data[q].verdict;
  }
  Object.keys(data).forEach(function(q){
    var b=document.createElement('button'); b.className='km-qbtn'; b.textContent=q; b.dataset.q=q;
    b.addEventListener('click', function(){ render(q); }); qbar.appendChild(b);
  });
  render(Object.keys(data)[0]);
})();
</script>

The obvious move, once you see this, is to stop choosing. Run both searches and combine them. That is called hybrid search, and it is the quiet baseline of every serious system. You get the literal precision of keywords and the conceptual reach of meaning, and the questions that would have broken either one alone now get answered.

## A small, elegant trick for combining them

There is a wrinkle. Keyword search and meaning search return scores on totally different scales. BM25 might say a document scores 14.2. The vector search says 0.87. You cannot just add those; they do not mean the same thing.

The clean fix is almost suspiciously simple, and it is worth knowing because it shows up everywhere. Instead of combining the scores, you combine the *ranks*. Forget that one document scored 14.2 and another 0.87. Just ask: where did each document place on each list? First? Third? Then a document that ranked high on *both* lists floats to the top, and a document that only one searcher loved gets a more modest boost.

This is called Reciprocal Rank Fusion. The actual rule, from a 2009 paper by Cormack and colleagues, is a single line: a document's combined score is the sum, across both lists, of one divided by (a small constant plus its rank). The constant is usually 60. You do not need to memorise that. The idea is what matters: **reward agreement between the two searchers, not the loudness of any single score.** It is parameter-free, it needs no tuning, and it is why it became the default fusion method in basically every search engine that does hybrid retrieval.

## Move two: retrieve fast, then judge slowly

Here is the second move, and it is the one that gives the biggest quality jump for the least conceptual effort.

Everything so far, keyword and vector search, has a hidden compromise baked in. To search a million documents quickly, the system compared your question to each document *separately*. Your question became one vector, each document became its own vector, and it measured distances. That is fast, because all the document vectors were computed ahead of time. But it is shallow. The question and the document never actually met. It is like deciding two strangers are compatible by reading their dating profiles side by side, without ever letting them have a conversation.

So you do it in two stages. First, the fast shallow search casts a wide net and pulls back, say, the top fifty candidates. It is allowed to be a bit sloppy, because its only job is to not miss the good stuff. Then a second, slower, much smarter model called a reranker looks at each of those fifty candidates *together with the question, at the same time*, and scores how well they actually fit. The question and the document finally have their conversation. The reranker is too slow to run on a million documents, but on fifty it is fast enough, and it is dramatically more accurate.

<figure class="post-fig">
  <div class="funnel">
    <div class="funnel-stage">
      <div class="funnel-label">stage 1: fast retrieval (cast a wide net)</div>
      <div class="funnel-docs" id="funnelTop"></div>
      <div class="funnel-note">Cheap and shallow. Grabs ~50 maybe-relevant candidates from millions. Job: do not miss the good ones.</div>
    </div>
    <div class="funnel-stage">
      <div class="funnel-label">stage 2: reranker (judge each one properly)</div>
      <div class="funnel-docs" id="funnelKeep"></div>
      <div class="funnel-note">Slow and precise. Reads the question and each candidate together, keeps the best handful for the model.</div>
    </div>
  </div>
  <figcaption>The retrieve-then-rerank pattern. A wide cheap net, then a careful judge. This two-stage shape is how nearly every serious search and RAG system works today.</figcaption>
</figure>

<script>
(function(){
  var top = document.getElementById('funnelTop'); if(!top) return;
  var keep = document.getElementById('funnelKeep');
  for(var i=0;i<18;i++){ var d=document.createElement('div'); d.className='fdoc'; top.appendChild(d); }
  for(var j=0;j<18;j++){ var d2=document.createElement('div'); d2.className='fdoc '+(j<4?'keep':'drop'); keep.appendChild(d2); }
})();
</script>

How much does this actually help? From the production write-ups I have seen benchmarked, hybrid search on its own lands somewhere around the high seventies in accuracy, and adding a reranker on top pushes it into the low nineties, for a small cost in latency. That is a large jump from one extra step. The reason it is not free is that the reranker is a heavier model, so you only ever run it on the shortlist, never the whole library.

## Move three: respect how the model loses focus

The last move is not about retrieval at all. It is about what happens *after* you have found good chunks and pasted them into the prompt. And it is the most counter-intuitive thing in this whole series.

You would assume that if the right answer is somewhere in the context you gave the model, it will find it. It will not, reliably. Researchers at Stanford and elsewhere showed something now famous, in a 2024 paper titled "Lost in the Middle." When they buried the crucial piece of information at different positions in a long context and measured how often the model used it, they found a clear, almost embarrassing pattern. The model paid close attention to whatever came at the very start and at the very end, and went a little blind in the middle.

<figure class="post-fig">
  <div class="litm">
    <div class="litm-bars">
      <div class="b" style="height:92%"><span>strong</span></div>
      <div class="b" style="height:62%"><span></span></div>
      <div class="b" style="height:46%"><span></span></div>
      <div class="b" style="height:42%"><span>weak</span></div>
      <div class="b" style="height:48%"><span></span></div>
      <div class="b" style="height:66%"><span></span></div>
      <div class="b" style="height:90%"><span>strong</span></div>
    </div>
    <div class="litm-axis"><span>start of context</span><span>middle</span><span>end of context</span></div>
  </div>
  <figcaption>The shape researchers found: a U-curve. How well a model uses a fact depends heavily on where that fact sits in the context. Start and end get attention. The middle gets neglected. The bars here illustrate the pattern, not exact figures from any one run.</figcaption>
</figure>

This is wild when you sit with it. It means you can retrieve the perfect chunk, hand it to the model, and still get a wrong answer purely because the chunk landed in the forgetful middle of a long prompt. The information was right there. The model just did not look hard enough at that spot.

The practical lessons are simple once you know the curve exists. Do not drown the model in context; more is not better past a point. Put your most important retrieved chunks near the beginning or the end, not buried in the middle. And this is a strong argument for everything earlier in this series: the better your retrieval and reranking, the fewer chunks you need to include, and the less you expose yourself to this problem at all.

## The uncomfortable truth: it still fails, and quietly

I want to end honestly, because most writing about RAG stops at the happy path.

Even with hybrid search, reranking, and careful context ordering, RAG fails. What makes it hard is that it fails in three different ways that look identical from the outside, a wrong or useless answer, but each needs a completely different fix:

<figure class="post-fig">
  <table class="ctab">
    <thead><tr><th>What you see</th><th>What actually went wrong</th><th>Where to look</th></tr></thead>
    <tbody>
      <tr><td>Wrong answer</td><td>The right chunk was never retrieved</td><td class="num">retrieval / chunking</td></tr>
      <tr><td>Wrong answer</td><td>Right chunk retrieved, model ignored or misread it</td><td class="num">context order, prompt</td></tr>
      <tr><td>Wrong answer</td><td>Right chunk retrieved and read, but the question was ambiguous</td><td class="num">the query itself</td></tr>
    </tbody>
  </table>
  <figcaption>Three failures, one symptom. This is why RAG is harder than it looks: the same broken answer can come from three different places, and fixing the wrong one wastes days.</figcaption>
</figure>

This is also why evaluation is its own quiet trap. It is tempting to measure only whether the final answer was good. But a good answer can come from bad retrieval by luck, and a bad answer can come from perfect retrieval that the model fumbled. If you only watch the final answer, you cannot tell which machine broke. The teams who get RAG right measure the *retrieval* and the *generation* separately, so when something breaks they know which half to fix.

## Where this leaves you

So here is the whole journey, in one breath. RAG hands a model the right documents at question time instead of hoping it memorised them. To find the right documents you cut them into honest chunks, turn meaning into geometry, and search by closeness. To do it well you search by keyword and meaning together, fuse the two by rank, and rerank the shortlist with a slower, sharper judge. And you stay humble about the model's wandering attention, and about the three quiet ways the whole thing can still fail.

None of it is magic. All of it is understandable. And every piece, from chunking to reranking, exists for one plain reason: to put true, relevant, checkable information in front of a model at the moment you ask, so its brilliance has something solid to stand on.

That is RAG. Thanks for reading all three parts. If you build something with this, I would genuinely love to hear what you made.

*Key references for this part: the "Lost in the Middle" paper (Liu et al., 2024) and the Reciprocal Rank Fusion method (Cormack et al., 2009). The explanations, examples, and diagrams here are my own.*
