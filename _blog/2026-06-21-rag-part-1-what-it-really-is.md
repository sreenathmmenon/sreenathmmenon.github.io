---
title: "RAG, Part 1: What It Really Is (and the Problem It Solves)"
date: 2026-06-21
excerpt: "A language model that has read the whole internet still cannot tell you what is in your own company handbook. RAG is how we fix that. This is the foundation: the problem, the idea, and how a machine decides what counts as relevant."
tags: [rag, retrieval, embeddings, ai, llm, explainer]
---

<style>
.post-fig{margin:2.5rem 0;}
.post-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}
.series-nav{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);border:1px solid var(--border);background:var(--surface);border-radius:10px;padding:.7rem 1rem;margin-bottom:2rem;}
.series-nav b{color:var(--accent);}

/* pipeline diagram */
.pipe{display:flex;align-items:stretch;gap:.5rem;max-width:640px;margin:0 auto;flex-wrap:wrap;justify-content:center;}
.pipe-box{flex:1;min-width:120px;background:var(--surface);border:1px solid var(--border-2);border-radius:12px;padding:.9rem .7rem;text-align:center;}
.pipe-box .pipe-ic{font-family:var(--font-mono);font-size:.72rem;color:var(--accent);text-transform:uppercase;letter-spacing:.08em;display:block;margin-bottom:.4rem;}
.pipe-box .pipe-t{font-size:.88rem;color:var(--text);}
.pipe-arrow{display:flex;align-items:center;color:var(--text-3);font-size:1.2rem;}
@media(max-width:560px){.pipe-arrow{transform:rotate(90deg);justify-content:center;}}

/* embedding similarity playground */
.sim-play{max-width:600px;margin:0 auto;background:var(--surface);border:1px solid var(--border-2);border-radius:16px;padding:1.4rem;box-shadow:var(--glow);}
.sim-q{font-family:var(--font-mono);font-size:.85rem;color:var(--text-2);margin-bottom:1rem;}
.sim-q b{color:var(--accent);}
.sim-list{display:flex;flex-direction:column;gap:.6rem;}
.sim-item{display:flex;align-items:center;gap:.8rem;}
.sim-text{flex:1;font-size:.92rem;color:var(--text);}
.sim-track{width:120px;flex:none;background:var(--surface-2);border-radius:999px;height:14px;overflow:hidden;border:1px solid var(--border);}
.sim-fill{height:100%;background:var(--grad);border-radius:999px;transition:width .5s var(--ease);}
.sim-score{width:3rem;flex:none;text-align:right;font-family:var(--font-mono);font-size:.8rem;color:var(--accent);}
.sim-buttons{display:flex;gap:.5rem;flex-wrap:wrap;margin-bottom:1.1rem;}
.sim-btn{font-family:var(--font-mono);font-size:.8rem;padding:.4rem .8rem;border-radius:999px;background:var(--surface-2);border:1px solid var(--border-2);color:var(--text-2);cursor:pointer;transition:all .2s var(--ease);}
.sim-btn:hover{border-color:var(--accent);color:var(--accent);}
.sim-btn.on{background:var(--grad);color:var(--accent-ink);border-color:transparent;font-weight:600;}
</style>

<div class="series-nav">A 3-part series on RAG. <b>You are reading Part 1: the foundation.</b> Part 2 is about chunking. Part 3 is about making it actually good.</div>

Let me start with a small, slightly embarrassing story about how smart these models are not.

You can ask a modern language model almost anything. The causes of the French Revolution. How to reverse a linked list. A decent risotto recipe. It has read a staggering amount and will answer with total confidence.

Now ask it this: "What is our company's refund policy?"

It has no idea. It cannot know. Your company handbook was never in its training data, and it never will be. So one of two things happens. The honest model says "I do not have access to that." The overconfident one invents a refund policy that sounds perfectly reasonable and is completely made up. Neither is useful, and the second is dangerous.

This is the gap RAG was built to close. And once you see the shape of the problem, the solution is almost obvious.

## The model is brilliant and blind

Here is the core tension. A language model knows a lot, but only about the world it was trained on, frozen at a moment in the past. It knows nothing about:

- Your private documents
- Anything that happened after its training cut-off
- The specific, niche, true details of your particular situation

You could try to fix this by retraining the model on your data, but that is slow, expensive, and you would have to do it again every time a document changes. That is a sledgehammer for a problem that needs a scalpel.

RAG is the scalpel. The trick is to stop expecting the model to *know* your information, and instead *hand it the right information at the moment you ask the question.*

That is the whole idea, in one sentence: before the model answers, go find the relevant documents and put them in front of it. Retrieval, then generation. RAG.

## A librarian, not a know-it-all

The analogy that finally made it click for me is a library.

Imagine you walk up to a reference librarian with a question. The librarian does not have every fact memorised. What they have is something better: they know how to *find* the right book, fast, and they will read you the relevant passage before answering.

A plain language model is the know-it-all at the party who answers everything from memory and is sometimes confidently wrong. A RAG system is the librarian. Same intelligence, but it looks things up first, and it can point you to the exact page it used.

<figure class="post-fig">
  <div class="pipe">
    <div class="pipe-box"><span class="pipe-ic">you ask</span><span class="pipe-t">"What is our refund policy?"</span></div>
    <div class="pipe-arrow">&rarr;</div>
    <div class="pipe-box"><span class="pipe-ic">retrieve</span><span class="pipe-t">find the matching pages from your documents</span></div>
    <div class="pipe-arrow">&rarr;</div>
    <div class="pipe-box"><span class="pipe-ic">augment</span><span class="pipe-t">paste those pages next to your question</span></div>
    <div class="pipe-arrow">&rarr;</div>
    <div class="pipe-box"><span class="pipe-ic">generate</span><span class="pipe-t">model answers using that context</span></div>
  </div>
  <figcaption>The RAG loop. Retrieve, augment, generate. The model never had to "know" your policy. It just had to read the right page at the right time.</figcaption>
</figure>

Notice what this buys you, beyond just correct answers:

- The answer is **grounded** in real documents, so it hallucinates far less.
- You can **show the source**, which means a human can check it.
- When a document changes, you change the document, and the next answer is already up to date. No retraining.

That last point is why RAG quietly became the default way to build serious AI systems on private or fast-moving data. It is practical in a way that retraining never is.

## But what does "find the relevant pages" actually mean?

Here is where most explanations wave their hands, and where the real idea lives. When I say "go find the relevant documents," how does a computer decide what is relevant?

It cannot just search for matching words. If you ask "how do I get my money back" and the document says "refund procedure," there is not a single shared word, yet they obviously mean the same thing. Keyword search would miss it.

So RAG uses something cleverer: it turns meaning into numbers.

Every chunk of text, your documents and your question alike, gets passed through an embedding model. This is a cousin of the language model whose only job is to convert text into a long list of numbers, a vector, arranged so that **things which mean similar things end up close together.** "Refund" and "money back" land near each other. "Refund" and "photosynthesis" land far apart. The text becomes a point in a kind of map of meaning.

I covered this map idea briefly in my post on how LLMs work. In RAG, it becomes the whole engine.

## Closeness is the answer

Once your question and all your document chunks are points on this map, "find the relevant ones" turns into something a computer is great at: find the points nearest to my question.

The usual way to measure "near" is called cosine similarity. You do not need the maths. Just the intuition: it gives a score, where **1 means the two pieces of text point in the same direction (very similar)** and **0 means they are unrelated.** The system scores your question against every chunk and grabs the top few.

Try it. Pick a question below and watch how the same set of document chunks gets scored differently depending on what you asked.

<figure class="post-fig">
  <div class="sim-play">
    <div class="sim-buttons" id="simButtons"></div>
    <div class="sim-q">your question: <b id="simQ"></b></div>
    <div class="sim-list" id="simList"></div>
  </div>
  <figcaption>Illustrative scores, not a live model, but the behaviour is exactly right: the chunk that means the same thing rises to the top, even when it shares no words with the question. That is the magic keyword search cannot do.</figcaption>
</figure>

<script>
(function(){
  var data = {
    "How do I get my money back?": [
      ["Our refund procedure: contact support within 30 days.", 0.91],
      ["Shipping usually takes three to five business days.", 0.34],
      ["The annual company picnic is held in July.", 0.08],
      ["You can cancel a subscription from account settings.", 0.57]
    ],
    "When will my order arrive?": [
      ["Our refund procedure: contact support within 30 days.", 0.29],
      ["Shipping usually takes three to five business days.", 0.93],
      ["The annual company picnic is held in July.", 0.07],
      ["You can cancel a subscription from account settings.", 0.22]
    ],
    "How do I stop being billed?": [
      ["Our refund procedure: contact support within 30 days.", 0.55],
      ["Shipping usually takes three to five business days.", 0.18],
      ["The annual company picnic is held in July.", 0.06],
      ["You can cancel a subscription from account settings.", 0.88]
    ]
  };
  var questions = Object.keys(data);
  var btns = document.getElementById('simButtons');
  var qEl = document.getElementById('simQ');
  var list = document.getElementById('simList');
  if(!btns) return;

  function render(q){
    qEl.textContent = q;
    Array.prototype.forEach.call(btns.children, function(b){ b.classList.toggle('on', b.dataset.q === q); });
    var rows = data[q].slice().sort(function(a,b){ return b[1]-a[1]; });
    list.innerHTML = '';
    rows.forEach(function(r){
      var item = document.createElement('div'); item.className = 'sim-item';
      item.innerHTML = '<div class="sim-text">'+r[0]+'</div>'+
        '<div class="sim-track"><div class="sim-fill" style="width:'+Math.round(r[1]*100)+'%"></div></div>'+
        '<div class="sim-score">'+r[1].toFixed(2)+'</div>';
      list.appendChild(item);
    });
  }
  questions.forEach(function(q){
    var b = document.createElement('button'); b.className='sim-btn'; b.textContent=q; b.dataset.q=q;
    b.addEventListener('click', function(){ render(q); });
    btns.appendChild(b);
  });
  render(questions[0]);
})();
</script>

Play with it for a second. Ask "how do I get my money back" and the refund line wins, even though they share no words. Ask "how do I stop being billed" and suddenly the cancellation line jumps ahead of the refund line, because cancelling is closer in meaning to your intent. The system is not matching letters. It is matching meaning. That is the quiet superpower underneath every RAG system.

## So, the whole picture

Let me put Part 1 together in plain terms.

A language model is brilliant but blind to your private and recent information. RAG fixes that not by teaching the model new facts, but by fetching the right facts at question time and handing them over. To fetch the right facts, it turns every piece of text into a point on a map of meaning, then grabs the points nearest your question. The model reads those, and answers grounded in real, checkable sources.

That is RAG at its core, and honestly, if you only ever understood this much, you would understand more than most people shipping these systems.

But here is the catch, and it is the thing that separates a toy demo from something that actually works: **what you retrieve is only as good as how you cut up your documents in the first place.** If your chunks are bad, your retrieval is bad, and no clever model can save you.

That unglamorous, make-or-break decision is called chunking, and it is the entire subject of Part 2. It is the part most people get wrong, and the part where a little understanding pays off enormously.

See you there.
