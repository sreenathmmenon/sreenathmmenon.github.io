---
title: "RAG, Part 2: Chunking, the Decision That Quietly Decides Everything"
date: 2026-06-21
excerpt: "Before a single answer is retrieved, you have to cut your documents into pieces. Cut them badly and nothing else can save you. Here is every chunking strategy, shown on real text, with honest numbers on what actually works."
tags: [rag, chunking, retrieval, ai, llm, explainer]
---

<style>
.post-fig{margin:2.5rem 0;}
.post-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}
.series-nav{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);border:1px solid var(--border);background:var(--surface);border-radius:10px;padding:.7rem 1rem;margin-bottom:2rem;}
.series-nav b{color:var(--accent);}
.series-nav a{color:var(--accent);}

/* chunk playground */
.chunk-play{max-width:640px;margin:0 auto;background:var(--surface);border:1px solid var(--border-2);border-radius:16px;padding:1.4rem;box-shadow:var(--glow);}
.chunk-strat{display:flex;gap:.5rem;flex-wrap:wrap;margin-bottom:1.1rem;}
.chunk-sbtn{font-family:var(--font-mono);font-size:.78rem;padding:.4rem .8rem;border-radius:999px;background:var(--surface-2);border:1px solid var(--border-2);color:var(--text-2);cursor:pointer;transition:all .2s var(--ease);}
.chunk-sbtn:hover{border-color:var(--accent);color:var(--accent);}
.chunk-sbtn.on{background:var(--grad);color:var(--accent-ink);border-color:transparent;font-weight:600;}
.chunk-out{line-height:1.9;}
.chunk-seg{padding:.1rem .15rem;border-radius:4px;}
.chunk-seg.c0{background:var(--surface-2);}
.chunk-seg.c1{background:transparent;box-shadow:inset 0 0 0 1px var(--accent);}
.chunk-desc{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:1rem;border-top:1px solid var(--border);padding-top:.9rem;}

/* overlap animation */
.ov{max-width:520px;margin:0 auto;display:flex;flex-direction:column;gap:.4rem;font-family:var(--font-mono);font-size:.82rem;}
.ov-row{display:flex;}
.ov-chunk{padding:.4rem .6rem;border-radius:7px;border:1px solid var(--border-2);background:var(--surface);color:var(--text-2);}
.ov-lap{background:var(--grad);color:var(--accent-ink);border-radius:5px;padding:0 .2rem;font-weight:600;}

/* comparison table */
.ctab{width:100%;border-collapse:collapse;font-size:.9rem;margin:0 auto;}
.ctab th,.ctab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);}
.ctab th{font-family:var(--font-mono);font-size:.74rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.ctab td:first-child{color:var(--text);font-weight:500;}
.ctab .num{font-family:var(--font-mono);color:var(--accent);}
.ctab tr:hover td{background:var(--surface);}
</style>

<div class="series-nav">Part 2 of a 3-part series on RAG. <a href="/blog/2026-06-21-rag-part-1-what-it-really-is/">Part 1</a> covered what RAG is and how retrieval finds meaning. <b>This part is about chunking.</b> Part 3 is about making it good.</div>

In Part 1, I made retrieval sound clean. Turn your documents into points on a map of meaning, find the points nearest the question, hand them to the model. Simple.

I left out the messy step that happens first, and it is the step that most quietly decides whether your whole system works.

Before any of that map-of-meaning magic, you have to take your documents and cut them into pieces. You cannot embed a 300-page manual as one giant blob. So you slice it up, and each slice becomes one searchable chunk. The question is: where do you cut?

It sounds trivial. It is not. Cut in the wrong places and you will retrieve garbage no matter how good your model is. This is the part people skip past in tutorials, and then wonder why their RAG demo gives confident, useless answers. So let us actually look at it.

## Why you cannot just split every 500 characters

The lazy approach is to chop the text every N characters and move on. Fast, simple, and it will hurt you. Here is why.

Imagine your document has this sentence: "Refunds are processed within 30 days. To start one, email support." If your blind cut lands right in the middle, you get one chunk ending at "within 30" and the next starting at "days. To start one." Now someone asks how long refunds take, and the retriever finds a fragment that says "Refunds are processed within 30" and stops. The actual answer got sliced in half and scattered across two chunks. Neither chunk is complete. Both are slightly wrong.

A good chunk is like a good paragraph: it holds one idea, completely, with enough around it to make sense on its own. The art of chunking is cutting on the natural seams of meaning instead of in the middle of them.

In the real world the documents you chunk are rarely tidy. They are sprawling OpenStack manuals, product FAQs, internal wikis, policy PDFs, the messy stuff people actually need answers from. To keep the demo readable I will use a tiny passage of my own below, but picture it as a few sentences pulled from a page of one of those. Click through the strategies and watch where each one decides to cut.

<figure class="post-fig">
  <div class="chunk-play">
    <div class="chunk-strat" id="chunkStrat"></div>
    <div class="chunk-out" id="chunkOut"></div>
    <div class="chunk-desc" id="chunkDesc"></div>
  </div>
  <figcaption>The same short passage, cut by four different strategies. Alternating shading and outlines mark where one chunk ends and the next begins. Watch how the smarter strategies refuse to cut mid-sentence.</figcaption>
</figure>

<script>
(function(){
  // original example text (my own writing, not copied), 4 short sentences
  var text = "Tea was first brewed in ancient China. Traders carried it west along old mountain routes. By the 1600s it had reached Europe and become a luxury. Today it is the most consumed drink on earth after water.";
  var sentences = text.match(/[^.]+\./g).map(function(s){ return s.trim(); });

  var strategies = {
    "Fixed-size": {
      desc: "Cuts every ~60 characters, ignoring meaning. Fast, but notice it slices straight through the middle of sentences.",
      chunks: function(){
        var out=[], size=60;
        for(var i=0;i<text.length;i+=size){ out.push(text.slice(i,i+size)); }
        return out;
      }
    },
    "Recursive": {
      desc: "Tries to keep paragraphs together, then sentences, then words, only splitting further when a chunk is too big. The sensible default. Here it lands cleanly on sentence boundaries.",
      chunks: function(){ return sentences.map(function(s){ return s+' '; }); }
    },
    "Sentence-window": {
      desc: "Indexes one sentence at a time for precise retrieval, but remembers its neighbours so the model still gets surrounding context. Best of both: pinpoint matching, full meaning.",
      chunks: function(){ return sentences.map(function(s){ return s+' '; }); }
    },
    "Semantic": {
      desc: "Embeds each sentence and cuts only where the topic actually shifts. Here it groups the two history sentences together and splits before the 'today' sentence, because the meaning changed.",
      chunks: function(){
        return [ sentences[0]+' '+sentences[1]+' ', sentences[2]+' ', sentences[3]+' ' ];
      }
    }
  };

  var stratBar = document.getElementById('chunkStrat');
  var out = document.getElementById('chunkOut');
  var desc = document.getElementById('chunkDesc');
  if(!stratBar) return;

  function render(name){
    Array.prototype.forEach.call(stratBar.children, function(b){ b.classList.toggle('on', b.dataset.n === name); });
    var chunks = strategies[name].chunks();
    out.innerHTML = chunks.map(function(c,i){
      return '<span class="chunk-seg c'+(i%2)+'">'+c+'</span>';
    }).join('');
    desc.textContent = strategies[name].desc;
  }
  Object.keys(strategies).forEach(function(name){
    var b=document.createElement('button'); b.className='chunk-sbtn'; b.textContent=name; b.dataset.n=name;
    b.addEventListener('click', function(){ render(name); });
    stratBar.appendChild(b);
  });
  render('Fixed-size');
})();
</script>

## The strategies, in plain terms

Let me name what you just watched.

**Fixed-size.** Cut every N characters or tokens, full stop. It is the fastest and the dumbest. It happily guillotines sentences. People use it because it is one line of code, and then they are confused when retrieval is poor. It is not useless, but it is rarely the right default.

**Recursive.** This is the workhorse, and the one I reach for first. Instead of cutting blindly, it tries to split on the biggest natural boundary first: paragraphs. If a piece is still too big, it splits that piece on sentences. Still too big, it splits on words. It keeps the strongest units of meaning whole for as long as it can. The popular implementation tries separators in order, roughly "paragraph break, then line break, then space, then last-resort character." Simple idea, very good results.

**Sentence-window.** A lovely trick. You index single sentences, which makes retrieval pinpoint-accurate, because a one-sentence chunk is unambiguous about what it is. But a lone sentence is thin context for the model. So at answer time, you quietly expand the match to include the sentences around it. You get precise matching and rich context at once.

**Semantic.** The clever one. Instead of counting characters, it embeds each sentence and measures how much the meaning shifts from one to the next. Where the topic genuinely changes, it cuts. Where ideas belong together, it keeps them together. It produces the most coherent chunks. It is also the most expensive, because you are running an embedding model across the whole document just to decide where to cut.

## The unsung hero: overlap

There is one more idea that sits on top of all of these, and it fixes the sliced-sentence problem almost for free.

Overlap. When you cut chunk two, you let it start a little before chunk one ended. The pieces share a small overlapping band. So if an idea straddles a boundary, it survives intact in at least one chunk instead of being orphaned across two.

<figure class="post-fig">
  <div class="ov">
    <div class="ov-row"><span class="ov-chunk">Refunds are processed within <span class="ov-lap">30 days. To start</span></span></div>
    <div class="ov-row" style="padding-left:9rem"><span class="ov-chunk"><span class="ov-lap">30 days. To start</span> one, email support.</span></div>
  </div>
  <figcaption>The accent band is the overlap, shared by both chunks. The phrase that used to get cut in half now lives complete inside both pieces. A small, cheap insurance policy against bad cuts.</figcaption>
</figure>

A common setting is a chunk of a few hundred tokens with an overlap of ten to twenty percent. It costs you a little storage and a little redundancy, and it saves you from a whole category of silent failures. Almost always worth it.

## Now the honest part: which one actually wins?

Here is where I want to be careful, because this is exactly the kind of thing people repeat without checking. So these numbers are from Chroma's published chunking evaluation, not my guesses, and I will link it.

Their team tested many strategies on the same benchmark and measured recall, which is roughly "of all the chunks that should have been retrieved, what fraction did we actually get." Higher is better. A few real results:

<figure class="post-fig">
  <table class="ctab">
    <thead><tr><th>Strategy</th><th>Chunk setup</th><th>Recall</th><th>Cost</th></tr></thead>
    <tbody>
      <tr><td>Recursive</td><td class="num">400 / 200 overlap</td><td class="num">88.3%</td><td>cheap</td></tr>
      <tr><td>Recursive</td><td class="num">200 / no overlap</td><td class="num">88.5%</td><td>cheap</td></tr>
      <tr><td>LLM semantic</td><td class="num">~240 tokens</td><td class="num">91.7%</td><td>very slow</td></tr>
      <tr><td>Cluster semantic</td><td class="num">~400 setting</td><td class="num">92.1%</td><td>expensive</td></tr>
    </tbody>
  </table>
  <figcaption>Selected results from Chroma's chunking evaluation (text-embedding-3-large, top 5 retrieved). The semantic methods do win on recall, by a few points. But read the cost column before you celebrate.</figcaption>
</figure>

So semantic chunking is the best, right? Pack it up, go home?

Not so fast, and this is the lesson I most want you to take from Part 2. The semantic methods won by a few points of recall. But the Chroma team noted the LLM-based chunker took **tens of minutes** to run, which makes it impractical for most real document collections. Meanwhile plain recursive chunking, the cheap and simple one, landed around 88 percent, just a few points behind, in a tiny fraction of the time and cost.

Read that gap honestly. You are paying a large amount of extra compute to move from 88 to 92. Sometimes those four points matter enormously, in medicine, in law, where a missed chunk is a real harm. Often they do not, and the simple thing is the right thing.

## What I actually do

After all the theory, here is the boring, effective advice I would give a friend starting out:

- Start with **recursive chunking**, a few hundred tokens, ten to twenty percent overlap. It is cheap, simple, and gets you most of the way.
- If you find retrieval is missing context that sits right next to a good match, add a **sentence-window** layer so the model sees the neighbourhood.
- Only reach for **semantic chunking** when you have measured a real problem that the simpler methods cannot fix, and you can afford the cost. Do not start there.
- Whatever you pick, **measure it on your own documents.** The benchmark winner on someone else's data is not guaranteed to win on yours. Your handbook is not their dataset.

That last point matters more than any strategy name. The teams that win at RAG are not the ones with the fanciest chunker. They are the ones who actually looked at what their system retrieved, saw it was bad, and fixed the cut.

Chunking is unglamorous. It is also the difference between a demo and a product. Get this right and you have done the hard, quiet work that most people skip.

In Part 3, we leave chunking behind and tackle retrieval itself: why pure meaning-search misses exact terms, how keyword search and vector search work better together, what reranking is, and the strange ways RAG still fails even when everything looks right. That is where it gets genuinely fun.

*Benchmark figures in this post are from [Chroma's "Evaluating Chunking Strategies for Retrieval"](https://research.trychroma.com/evaluating-chunking) research. The explanations and example are my own.*
