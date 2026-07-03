---
title: "Embeddings: How AI Knows Two Things Mean the Same"
date: 2026-07-03
excerpt: "Type 'my laptop keeps freezing' and get back a doc titled 'system hangs on boot.' Not one word matches, yet it's the right answer. That quiet trick is embeddings: turning meaning into numbers you can measure the distance between. Here is how it actually works, from the first intuition to the parts senior engineers still find beautiful."
tags: [ai, embeddings, vectors, semantic-search, rag, explainer]
---

<style>
.em-fig{margin:2.5rem 0;}
.em-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* keyword-fail vs meaning-match */
.em-match{max-width:560px;margin:0 auto;display:flex;flex-direction:column;gap:.7rem;}
.em-match .q{font-family:var(--font-mono);font-size:.85rem;text-align:center;color:var(--accent);border:1px dashed var(--accent);border-radius:10px;padding:.55rem;}
.em-match .cand{display:flex;align-items:center;gap:.7rem;background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:.55rem .8rem;font-size:.86rem;color:var(--text-2);}
.em-match .cand .badge{flex:none;font-family:var(--font-mono);font-size:.72rem;padding:.2rem .5rem;border-radius:999px;border:1px solid var(--border-2);color:var(--text-3);}
.em-match .cand.hit{border-color:var(--accent);}
.em-match .cand.hit .badge{border-color:var(--accent);color:var(--accent);font-weight:600;}

/* vector-as-coordinates */
.em-vec{max-width:520px;margin:0 auto;font-family:var(--font-mono);font-size:.82rem;}
.em-vec .word{color:var(--text);font-weight:600;margin-bottom:.4rem;}
.em-vec .nums{display:flex;flex-wrap:wrap;gap:.35rem;}
.em-vec .n{padding:.25rem .5rem;border:1px solid var(--border-2);border-radius:6px;background:var(--surface);color:var(--text-2);}
.em-vec .n.dim{color:var(--accent);border-color:var(--accent);}
.em-vec .dots{color:var(--text-3);align-self:center;}

/* 2D meaning-space plot */
.em-space{max-width:520px;margin:0 auto;position:relative;height:300px;border:1px solid var(--border);border-radius:14px;background:var(--surface);overflow:hidden;}
.em-space .axis{position:absolute;background:var(--border-2);}
.em-space .ax-x{left:8%;right:8%;top:50%;height:1px;}
.em-space .ax-y{top:8%;bottom:8%;left:50%;width:1px;}
.em-space .pt{position:absolute;transform:translate(-50%,-50%);text-align:center;opacity:0;transition:opacity .5s ease;}
.em-space.go .pt{opacity:1;}
.em-space.go .pt:nth-child(3){transition-delay:.15s}
.em-space.go .pt:nth-child(4){transition-delay:.3s}
.em-space.go .pt:nth-child(5){transition-delay:.45s}
.em-space.go .pt:nth-child(6){transition-delay:.6s}
.em-space.go .pt:nth-child(7){transition-delay:.75s}
.em-space.go .pt:nth-child(8){transition-delay:.9s}
.em-space .pt .dot{width:11px;height:11px;border-radius:50%;background:var(--grad);margin:0 auto .2rem;box-shadow:var(--glow);}
.em-space .pt .lbl{font-family:var(--font-mono);font-size:.72rem;color:var(--text-2);white-space:nowrap;}
.em-space .pt.far .dot{background:var(--surface-2);border:1px solid var(--text-3);}

/* cosine angle dial */
.em-cos{max-width:600px;margin:0 auto;display:flex;flex-wrap:wrap;gap:1rem;justify-content:center;}
.em-cos .dial{width:150px;text-align:center;}
.em-cos svg{width:150px;height:110px;}
.em-cos .cap{font-family:var(--font-mono);font-size:.74rem;color:var(--text-2);margin-top:.3rem;}
.em-cos .cap b{color:var(--accent);display:block;}
.em-cos .ray{stroke:var(--accent);stroke-width:2.5;stroke-linecap:round;transform-origin:20px 90px;opacity:0;transition:transform 1s cubic-bezier(.2,.7,.2,1),opacity .5s ease;}
.em-cos.go .ray{opacity:1;}
.em-cos.go .d1 .ray2{transform:rotate(-8deg);}
.em-cos.go .d2 .ray2{transform:rotate(-52deg);}
.em-cos.go .d3 .ray2{transform:rotate(-92deg);}
.em-cos .base{stroke:var(--text-3);stroke-width:2;stroke-linecap:round;}

/* similarity bars */
.em-bars{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:.9rem;}
.em-bars .row .lab{display:flex;justify-content:space-between;font-family:var(--font-mono);font-size:.8rem;margin-bottom:.3rem;}
.em-bars .row .lab b{color:var(--text);}
.em-bars .row .lab span{color:var(--accent);}
.em-bars .track{height:16px;border-radius:999px;background:var(--surface-2);border:1px solid var(--border);overflow:hidden;}
.em-bars .fill{height:100%;border-radius:999px;background:var(--grad);width:0;transition:width 1.1s cubic-bezier(.2,.7,.2,1);}
.em-bars.go .fill{width:var(--w);}
.em-bars .fill.low{background:var(--surface-2);border-right:2px solid var(--text-3);}

/* pipeline */
.em-pipe{max-width:640px;margin:0 auto;display:flex;flex-direction:column;gap:.6rem;}
.em-pstep{display:flex;align-items:center;gap:.9rem;background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:.7rem .9rem;}
.em-pstep .n{flex:none;width:28px;height:28px;border-radius:50%;background:var(--grad);color:var(--accent-ink);font-family:var(--font-mono);font-weight:600;font-size:.82rem;display:flex;align-items:center;justify-content:center;}
.em-pstep .t{font-size:.88rem;color:var(--text-2);}
.em-pstep .t b{color:var(--text);}

/* matryoshka nesting */
.em-mat{max-width:560px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.em-mat .layer{border:1px solid var(--border-2);border-radius:8px;padding:.55rem .8rem;font-family:var(--font-mono);font-size:.78rem;display:flex;justify-content:space-between;align-items:center;color:var(--text-2);background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.em-mat.go .layer{opacity:1;transform:none;}
.em-mat.go .layer:nth-child(1){transition-delay:.1s}
.em-mat.go .layer:nth-child(2){transition-delay:.3s}
.em-mat.go .layer:nth-child(3){transition-delay:.5s}
.em-mat.go .layer:nth-child(4){transition-delay:.7s}
.em-mat .layer .tag{color:var(--accent);}
.em-mat .layer.full{border-color:var(--accent);}

/* table */
.em-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.em-tab th,.em-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.em-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.em-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}
.em-tab code{font-size:.82rem;}

@media (prefers-reduced-motion: reduce){
  .em-space .pt,.em-cos .ray,.em-bars .fill,.em-mat .layer{transition:none !important;}
  .em-space .pt{opacity:1 !important;}
  .em-bars .fill{width:var(--w) !important;}
  .em-mat .layer{opacity:1 !important;transform:none !important;}
  .em-cos.go .d1 .ray2{transform:rotate(-8deg);}
  .em-cos.go .d2 .ray2{transform:rotate(-52deg);}
  .em-cos.go .d3 .ray2{transform:rotate(-92deg);}
}
</style>

Here is a small thing that should not work, and does.

You type "my laptop keeps freezing" into a search box. Back comes a help article titled "system hangs on boot." Read those two phrases again. They share *no words*. Not "laptop," not "freezing," nothing. An old-fashioned search, the kind that matches letters, would have shrugged and returned nothing. Yet the modern one found the exact right answer, because somewhere underneath, the machine understood that a freezing laptop and a hanging system are the *same idea wearing different clothes*.

That quiet understanding is the single most useful trick in modern AI, and it has a name: **embeddings**. Almost everything you find impressive, semantic search, RAG, recommendations, an assistant that remembers what you meant three messages ago, is standing on this one idea. So I want to build it up with you slowly, from the first honest intuition all the way to the parts that still make senior engineers smile. No prior math needed to start. By the end you will understand something real.

## The core move: turn meaning into a place

Computers cannot compare meanings. They can only compare numbers. So the whole game is a single audacious move: **take a piece of text and turn its meaning into a list of numbers, chosen so that similar meanings get similar numbers.**

That list of numbers is called a vector, or an embedding. Think of the numbers as coordinates. Two coordinates, and you have a point on a map. Three, and it is a point in a room. An embedding just uses *many* coordinates, hundreds or thousands, to place a piece of meaning somewhere in a vast space. You cannot picture that space, and that is fine, nobody can. The point is what it *buys* you: once meaning is a location, "these two things are similar" becomes "these two points are close together." A fuzzy human question turns into plain geometry.

<figure class="em-fig">
  <div class="em-vec">
    <div class="word">"king"</div>
    <div class="nums">
      <span class="n dim">0.21</span><span class="n">-0.44</span><span class="n dim">0.68</span><span class="n">0.05</span><span class="n">-0.31</span><span class="n dim">0.52</span><span class="dots">… 762 more</span>
    </div>
  </div>
  <figcaption>The word "king" as an embedding: just a long list of numbers. Each one is a coordinate along some axis of meaning. No single number means anything on its own; together they pin "king" to one exact spot in a space of meaning.</figcaption>
</figure>

## Seeing the space (a flat version of a huge idea)

Real embedding space has hundreds of dimensions, which is unpicturable. But the *behaviour* survives if we squash it down to two, so let me show you a flat cartoon of the real thing. Watch where the words land:

<figure class="em-fig">
  <div class="em-space" id="space">
    <div class="axis ax-x"></div>
    <div class="axis ax-y"></div>
    <div class="pt" style="left:30%;top:32%"><div class="dot"></div><div class="lbl">king</div></div>
    <div class="pt" style="left:38%;top:26%"><div class="dot"></div><div class="lbl">queen</div></div>
    <div class="pt" style="left:26%;top:40%"><div class="dot"></div><div class="lbl">prince</div></div>
    <div class="pt far" style="left:72%;top:70%"><div class="dot"></div><div class="lbl">banana</div></div>
    <div class="pt far" style="left:78%;top:62%"><div class="dot"></div><div class="lbl">apple</div></div>
    <div class="pt far" style="left:66%;top:78%"><div class="dot"></div><div class="lbl">mango</div></div>
  </div>
  <figcaption>A 2D cartoon of meaning-space. Royalty clusters in one corner, fruit in another. Nobody told the model "these are royals." It learned, from oceans of text, that these words show up in similar company, and so it placed them near each other. Distance became a proxy for meaning.</figcaption>
</figure>

The famous demonstration of how *structured* this space is: take the vector for "king," subtract "man," add "woman," and the point you land on sits almost exactly on "queen." The space encodes not just topics but *relationships*, direction itself carries meaning ("the male-to-female direction," "the singular-to-plural direction"). That is not a party trick someone hard-coded. It fell out of learning, and the first time you see it work you feel a small jolt. I still do.

## Measuring "close": the angle, not the distance

So similar meanings land nearby. How do we *measure* nearby? Your instinct says "distance between the points," and that is reasonable, but the tool everyone actually reaches for measures the **angle** between the two vectors instead. It is called **cosine similarity**, and it is worth understanding why angle beats distance.

Picture each embedding as an arrow from the origin out to its point. Two arrows pointing the *same direction* mean the same thing, regardless of how long the arrows are. Cosine similarity reads that angle and hands you one number:

<figure class="em-fig">
  <div class="em-cos" id="cos">
    <div class="dial d1">
      <svg viewBox="0 0 160 100"><line class="base" x1="20" y1="90" x2="140" y2="90"/><line class="ray ray2" x1="20" y1="90" x2="140" y2="90"/></svg>
      <div class="cap"><b>≈ 1.0</b>same direction<br>"king" · "queen"</div>
    </div>
    <div class="dial d2">
      <svg viewBox="0 0 160 100"><line class="base" x1="20" y1="90" x2="140" y2="90"/><line class="ray ray2" x1="20" y1="90" x2="140" y2="90"/></svg>
      <div class="cap"><b>≈ 0.5</b>loosely related<br>"king" · "crown"</div>
    </div>
    <div class="dial d3">
      <svg viewBox="0 0 160 100"><line class="base" x1="20" y1="90" x2="140" y2="90"/><line class="ray ray2" x1="20" y1="90" x2="140" y2="90"/></svg>
      <div class="cap"><b>≈ 0.0</b>unrelated<br>"king" · "banana"</div>
    </div>
  </div>
  <figcaption>Cosine similarity is just the cosine of the angle between two arrows. Zero angle gives 1 (identical meaning). A right angle gives 0 (unrelated). Opposite directions give -1. One clean number, from -1 to 1, for "how alike are these two meanings."</figcaption>
</figure>

The mechanics, in plain words: multiply the two vectors dimension by dimension and add it all up (that is the dot product), then divide by each arrow's length. Dividing by the lengths is the important part, because it *throws the lengths away* and leaves only direction.

Why deliberately ignore length? Two reasons, one practical and one deep. The practical one: a long document about laptops should not rank as "more about laptops" than a short sentence about laptops just because it has more words. Meaning is about *which way you point*, not how far. The deep one is a genuine senior-level fact worth carrying: in very high-dimensional spaces, plain straight-line distances between points all start to look eerily similar, everything drifts toward equally-far-apart. It is called the curse of dimensionality, and it quietly wrecks distance-based comparison. Angle survives it. That is the real reason cosine is the default, not habit.

## Where do the numbers come from? (the honest mechanics)

Fair question: who decides that "king" is `[0.21, -0.44, ...]`? Nobody hand-writes these. A neural network *learns* them. Here is the pipeline without the hand-waving, and it holds from a fresher's mental model to what actually ships.

<figure class="em-fig">
  <div class="em-pipe">
    <div class="em-pstep"><div class="n">1</div><div class="t"><b>Tokenize.</b> The text is chopped into tokens (word-ish pieces). Each token starts as a lookup in a big learned table of vectors.</div></div>
    <div class="em-pstep"><div class="n">2</div><div class="t"><b>Read in context.</b> A transformer passes those token vectors through its layers, letting every token adjust based on the words around it. "Bank" near "river" ends up different from "bank" near "money."</div></div>
    <div class="em-pstep"><div class="n">3</div><div class="t"><b>Pool into one vector.</b> A sentence is many token vectors; we need one. Usually you average them (mean pooling), or use a special summary token. One fixed-size vector now stands for the whole text.</div></div>
    <div class="em-pstep"><div class="n">4</div><div class="t"><b>That vector is the embedding.</b> Typically 768 to 4096 numbers, every one carrying signal. Ready to compare against any other.</div></div>
  </div>
  <figcaption>Raw text goes in, one meaning-vector comes out. The transformer's job in step 2 is the clever bit: it reads each word in the light of its neighbours, so the final vector reflects meaning-in-context, not just a dictionary lookup.</figcaption>
</figure>

Two words there deserve a beat, because they separate "I sort of get it" from "I actually get it."

**Dense, not sparse.** An older approach gave each word in the dictionary its very own slot, so a vector was tens of thousands of numbers, almost all zero, one slot lit up per word present. That is *sparse*, and it is why old search thought "laptop freezing" and "system hanging" were total strangers: different slots, zero overlap. Modern embeddings are *dense*: a few hundred to a few thousand numbers where nearly every value means something, and meaning is spread across all of them. That density is exactly what lets unrelated *words* land on nearby *meanings*.

**Trained by contrast.** How does the model learn to place similar things together? You show it pairs that *should* be close (a question and its correct answer) and let everything else in the batch count as things that should be far. The model nudges the close pairs together and shoves the rest apart, over and over, across billions of examples. This is contrastive learning, and it is why "laptop freezing" and "system hangs" drift into the same neighbourhood despite sharing no letters. They kept appearing in similar roles, so the model learned to point them the same way.

## The thing this unlocks: semantic search

Now the payoff, and it is beautifully simple once the pieces are in place. Searching by *meaning* instead of by *matching letters* is four steps:

<figure class="em-fig">
  <div class="em-pipe">
    <div class="em-pstep"><div class="n">1</div><div class="t"><b>Embed everything you own</b> once, up front. Every doc becomes a vector, stored in a vector database.</div></div>
    <div class="em-pstep"><div class="n">2</div><div class="t"><b>Embed the query</b> with the exact same model when someone searches.</div></div>
    <div class="em-pstep"><div class="n">3</div><div class="t"><b>Find nearest neighbours</b> by cosine similarity, the stored vectors pointing most nearly the same way as the query.</div></div>
    <div class="em-pstep"><div class="n">4</div><div class="t"><b>Return those</b> (and optionally rerank the top few more carefully).</div></div>
  </div>
  <figcaption>This is the engine under semantic search and under the "retrieval" in RAG. Notice step 1 happens once and offline; only the tiny query embedding and the nearest-neighbour lookup happen live. That is why it feels instant.</figcaption>
</figure>

And this is exactly why our opening magic trick worked. "my laptop keeps freezing" and "system hangs on boot" get embedded into arrows pointing almost the same direction. High cosine similarity. The letters never mattered; the *meanings* pointed the same way.

Here is that same idea as scores, so you can feel the gradient between a strong match and a weak one:

<figure class="em-fig">
  <div class="em-bars" id="bars">
    <div class="row"><div class="lab"><b>"system hangs on boot"</b><span>0.83</span></div><div class="track"><div class="fill" style="--w:83%"></div></div></div>
    <div class="row"><div class="lab"><b>"pc won't respond, screen frozen"</b><span>0.79</span></div><div class="track"><div class="fill" style="--w:79%"></div></div></div>
    <div class="row"><div class="lab"><b>"how to speed up a slow computer"</b><span>0.51</span></div><div class="track"><div class="fill" style="--w:51%"></div></div></div>
    <div class="row"><div class="lab"><b>"best banana bread recipe"</b><span>0.06</span></div><div class="track"><div class="fill low" style="--w:8%"></div></div></div>
  </div>
  <figcaption>Cosine similarity of each candidate against the query "my laptop keeps freezing." The two real matches score high with zero shared keywords; the banana bread, sharing the word count but none of the meaning, sinks to the floor. The number, not the words, does the ranking.</figcaption>
</figure>

## Where you'll actually meet embeddings

This is not a lab curiosity. It is quietly running under most useful AI you touch:

<figure class="em-fig">
<table class="em-tab">
<thead><tr><th>Use case</th><th>What gets embedded</th><th>Why it works</th></tr></thead>
<tbody>
<tr><td>Semantic search</td><td>Your documents and the query</td><td>Finds meaning, not keywords, so synonyms and paraphrases still match</td></tr>
<tr><td>RAG</td><td>Chunks of your knowledge base</td><td>Retrieves the passages nearest the question to feed the model real context</td></tr>
<tr><td>Recommendations</td><td>Items, and a user's history</td><td>"More like this" becomes "nearest neighbours in item-space"</td></tr>
<tr><td>Deduplication / clustering</td><td>Records, tickets, reviews</td><td>Near-identical meanings cluster together even when worded differently</td></tr>
<tr><td>Agent memory</td><td>Past notes and decisions</td><td>Recall what's *relevant* to now, by nearness, instead of scrolling everything</td></tr>
<tr><td>Classification</td><td>The input text</td><td>Similar inputs sit near labelled examples, so the nearest ones vote</td></tr>
</tbody>
</table>
  <figcaption>Every row is the same primitive: turn things into vectors, then reason with nearness. Once you see it, you spot embeddings everywhere.</figcaption>
</figure>

I have leaned on this myself. A tool I built keeps a working memory across sessions, and the way it decides what past context is *relevant* to your current task is exactly this: embed the notes, embed the moment, pull the nearest. It is a small idea doing enormous work.

## One more, for the senior in the room: nested embeddings

Here is a recent, genuinely elegant twist that rewards understanding the basics. A 3072-dimension embedding is powerful but *heavy*, more storage, slower to compare, at scale that is real money. You would love to use a shorter vector when you can afford lower precision, and a longer one when you need the best. Normally that means training a whole separate model per size. Annoying.

The fix is a training trick called **Matryoshka representation learning**, named for the Russian nesting dolls, and the name is the whole idea. The model is trained so that the *most important meaning is packed into the earliest dimensions*, and every prefix of the vector is a complete, usable embedding on its own.

<figure class="em-fig">
  <div class="em-mat" id="mat">
    <div class="layer full"><span>full vector · first 3072 dims</span><span class="tag">best quality</span></div>
    <div class="layer"><span>first 768 dims</span><span class="tag">great, 4× smaller</span></div>
    <div class="layer"><span>first 256 dims</span><span class="tag">good, 12× smaller</span></div>
    <div class="layer"><span>first 64 dims</span><span class="tag">coarse but usable</span></div>
  </div>
  <figcaption>One vector, many honest sizes. Because training forced the front of the vector to carry the heaviest meaning, you can just chop the tail off to shrink it, no new model, no re-embedding. Doll inside a doll inside a doll, each one complete.</figcaption>
</figure>

Why it matters in practice: you can do a fast, cheap first pass with the *short* prefix to narrow millions of candidates down to a few hundred, then rerank just those with the *full* vector for precision. Best of both. The remarkable part is that a well-trained short prefix often matches a much longer vector from an ordinary model, real quality at a fraction of the cost. That is the kind of design that makes you sit back a little.

## The one sentence to keep

Strip everything away and embeddings are this: **meaning becomes a place, and similarity becomes distance.** Turn text into a point in a space built so that alike things sit close, and suddenly a computer that only knows how to compare numbers can answer questions about *meaning*, find the right doc with none of the right words, remember what's relevant, group what belongs together.

The next time a search understands you better than the words you typed, or an assistant surfaces exactly the note you needed, you will know what happened underneath. Your meaning was turned into an arrow, and somewhere in a space too big to picture, it pointed at the answer.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['space','cos','bars','mat'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
