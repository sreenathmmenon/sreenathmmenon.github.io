---
title: "Chunking a Form for RAG: Why the Usual Way Breaks"
date: 2026-07-19
excerpt: "RAG works beautifully on paragraphs. Then you point it at a form, an insurance claim, a bank statement, a form full of labels, values, and tables, and it starts getting numbers wrong. The reason is chunking. The usual way of cutting documents into pieces quietly destroys a form's meaning. Here's why, the chunking approaches that fix it, and the honest side effect and accuracy cost of each."
tags: [ai, rag, chunking, forms, retrieval, explainer]
---

<style>
.fm-fig{margin:2.5rem 0;}
.fm-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* a form */
.fm-form{max-width:460px;margin:0 auto;border:1px solid var(--border-2);border-radius:10px;padding:1rem;background:var(--surface);}
.fm-form .row{display:flex;gap:.6rem;align-items:center;margin-bottom:.5rem;}
.fm-form .lab{flex:none;width:130px;font-family:var(--font-mono);font-size:.78rem;color:var(--text-2);text-align:right;}
.fm-form .box{flex:1;border:1px solid var(--border);border-radius:5px;padding:.3rem .5rem;font-family:var(--font-mono);font-size:.78rem;color:var(--accent);background:var(--surface-2);}

/* naive chunk breaks it */
.fm-break{max-width:560px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:.8rem;}
@media(max-width:520px){.fm-break{grid-template-columns:1fr;}}
.fm-break .c{border:1px solid var(--border);border-radius:10px;padding:.85rem;background:var(--surface);}
.fm-break .c h5{margin:0 0 .5rem;font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.05em;}
.fm-break .c.bad h5{color:var(--text-3);} .fm-break .c.good h5{color:var(--accent);}
.fm-break .c.good{border-color:var(--accent);}
.fm-break .chunk{font-family:var(--font-mono);font-size:.72rem;border:1px dashed var(--border-2);border-radius:6px;padding:.4rem .55rem;margin-bottom:.4rem;color:var(--text-2);}
.fm-break .chunk.split{border-color:var(--text-3);color:var(--text-3);}
.fm-break .chunk.whole{border-style:solid;border-color:var(--accent);color:var(--text-2);}
.fm-break .chunk .a{color:var(--accent);}

/* approaches with side effect + accuracy */
.fm-appr{max-width:680px;margin:0 auto;display:flex;flex-direction:column;gap:.8rem;}
.fm-a{border:1px solid var(--border);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.fm-appr.go .fm-a{opacity:1;transform:none;}
.fm-appr.go .fm-a:nth-child(1){transition-delay:.1s} .fm-appr.go .fm-a:nth-child(2){transition-delay:.28s} .fm-appr.go .fm-a:nth-child(3){transition-delay:.46s} .fm-appr.go .fm-a:nth-child(4){transition-delay:.64s} .fm-appr.go .fm-a:nth-child(5){transition-delay:.82s}
.fm-a .hd{display:flex;align-items:center;gap:.6rem;margin-bottom:.5rem;}
.fm-a .num{flex:none;width:26px;height:26px;border-radius:50%;background:var(--surface-2);border:1px solid var(--border-2);color:var(--text-2);font-family:var(--font-mono);font-weight:600;font-size:.8rem;display:flex;align-items:center;justify-content:center;}
.fm-a.win .num{background:var(--grad);color:var(--accent-ink);border-color:transparent;}
.fm-a .nm{font-family:var(--font-mono);font-size:.87rem;color:var(--text);font-weight:600;} .fm-a.win .nm{color:var(--accent);}
.fm-a .acc{font-family:var(--font-mono);font-size:.7rem;margin-left:auto;padding:.15rem .5rem;border-radius:999px;border:1px solid var(--border-2);color:var(--text-3);}
.fm-a.win .acc{border-color:var(--accent);color:var(--accent);}
.fm-a .desc{font-size:.84rem;color:var(--text-2);line-height:1.5;margin-bottom:.5rem;}
.fm-a .se{font-size:.8rem;color:var(--text-2);line-height:1.45;border-left:2px solid var(--border-2);padding-left:.7rem;}
.fm-a.win .se{border-color:var(--accent);}
.fm-a .se b{color:var(--text-3);} .fm-a.win .se b{color:var(--accent);}

/* the golden rule */
.fm-rule{max-width:540px;margin:0 auto;text-align:center;border:1px solid var(--accent);border-radius:12px;padding:1.1rem 1.3rem;background:var(--surface);}
.fm-rule b{color:var(--accent);} .fm-rule p{font-size:.9rem;color:var(--text-2);line-height:1.6;margin:0;}

/* parent-child viz */
.fm-pc{max-width:560px;margin:0 auto;}
.fm-pc svg{width:100%;height:180px;overflow:visible;}
.fm-pc .box{fill:var(--surface);stroke:var(--border-2);stroke-width:1.5;} .fm-pc .box.acc{stroke:var(--accent);stroke-width:2;}
.fm-pc .tx{fill:var(--text);font-family:var(--font-mono);font-size:10px;text-anchor:middle;}
.fm-pc .sub{fill:var(--text-3);font-family:var(--font-mono);font-size:8px;text-anchor:middle;}
.fm-pc .ln{stroke:var(--accent);stroke-width:1.5;fill:none;}

/* table row chunking */
.fm-trow{max-width:560px;margin:0 auto;}
.fm-trow .tbl{border:1px solid var(--border);border-radius:8px;overflow:hidden;font-family:var(--font-mono);font-size:.74rem;}
.fm-trow .tr{display:flex;border-bottom:1px solid var(--border);}
.fm-trow .tr:last-child{border-bottom:none;}
.fm-trow .tr.head{background:var(--surface-2);color:var(--accent);}
.fm-trow .td{flex:1;padding:.4rem .55rem;color:var(--text-2);border-right:1px solid var(--border);}
.fm-trow .td:last-child{border-right:none;}
.fm-trow .tr.chunked{outline:2px solid var(--accent);outline-offset:-2px;}

/* metadata tag flow */
.fm-meta{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.fm-mrow{display:flex;align-items:center;gap:.9rem;border:1px solid var(--border);border-radius:10px;padding:.6rem .9rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.fm-meta.go .fm-mrow{opacity:1;transform:none;}
.fm-meta.go .fm-mrow:nth-child(1){transition-delay:.1s} .fm-meta.go .fm-mrow:nth-child(2){transition-delay:.3s} .fm-meta.go .fm-mrow:nth-child(3){transition-delay:.5s}
.fm-mrow .tag{flex:none;font-family:var(--font-mono);font-size:.72rem;color:var(--accent-ink);background:var(--grad);border-radius:6px;padding:.2rem .5rem;}
.fm-mrow .t{font-size:.84rem;color:var(--text-2);} .fm-mrow .t b{color:var(--text);}

/* table */
.fm-tab{width:100%;border-collapse:collapse;font-size:.88rem;}
.fm-tab th,.fm-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.fm-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.fm-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}

/* extra reveals */
.fm-break .c{opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.fm-break.go .c{opacity:1;transform:none;}
.fm-break.go .c:nth-child(1){transition-delay:.1s} .fm-break.go .c:nth-child(2){transition-delay:.35s}

.fm-pc svg{opacity:0;transform:translateY(8px);transition:opacity .6s ease,transform .6s ease;}
.fm-pc.go svg{opacity:1;transform:none;}

.fm-rule{opacity:0;transform:scale(.98);transition:opacity .6s ease,transform .6s ease;}
.fm-rule.go{opacity:1;transform:none;}

@media (prefers-reduced-motion: reduce){
  .fm-appr .fm-a,.fm-meta .fm-mrow,.fm-break .c,.fm-pc svg,.fm-rule{transition:none !important;opacity:1 !important;transform:none !important;}
}
</style>

RAG is a beautiful trick when your documents are paragraphs. You chop the text into chunks, find the chunk that matches the question, hand it to the model, and get a grounded answer. I've written about it a few times. It just works.

Then someone points it at a *form*, an insurance claim, a bank statement, a loan application, and it starts quietly getting numbers wrong. It confuses one amount for another. It answers "what's the policy number?" with the account number. It reads a checkbox wrong. Nothing crashes; it just returns confidently incorrect values, which in insurance or banking is the worst kind of failure.

The culprit is almost never the model. It's **chunking**, the step where you cut the document into pieces before storing it. The way you'd naturally chunk a paragraph *destroys* a form, because a form's meaning lives in its structure, and naive chunking shreds that structure. This post is about exactly that: why the usual chunking breaks forms, the approaches that fix it, and, honestly, the side effect and accuracy cost of each. (I'll assume the text is already pulled off the page; getting text off a scan is its own topic. Here we care about what happens *after* you have the text: how you cut it up.)

## The one thing that makes a form different

Quick reminder of why forms aren't prose. In a paragraph, meaning flows in a line, you can cut it almost anywhere and each piece still makes sense. In a form, meaning is a *pairing*: a label and its value belong together, and a table cell only means something next to its row and column headers.

<figure class="fm-fig">
  <div class="fm-form">
    <div class="row"><div class="lab">Policy Number:</div><div class="box">POL-88231</div></div>
    <div class="row"><div class="lab">Claim Amount:</div><div class="box">$4,500</div></div>
    <div class="row"><div class="lab">Deductible:</div><div class="box">$500</div></div>
  </div>
  <figcaption>"$4,500" only means "claim amount" because it sits next to that label. "$500" is the deductible for the same reason. Separate a value from its label, or a number from its header, and the meaning is gone. Hold this: on a form, <em>a value and the thing that names it must never be split apart.</em> That single rule explains every chunking decision below.</figcaption>
</figure>

## Why the usual chunking breaks it

The default chunking strategy is dead simple: cut the text every ~500 words, maybe with a little overlap. On prose, fine. On a form, watch what it does.

<figure class="fm-fig">
  <div class="fm-break" id="break">
    <div class="c bad">
      <h5>Naive chunking (cut by size)</h5>
      <div class="chunk split">...Deductible: <span>[cut here]</span></div>
      <div class="chunk split">$500 Claim Amount: $4,500 Policy...</div>
      <div style="font-size:.72rem;color:var(--text-3);margin-top:.4rem">"$500" got separated from "Deductible". Now which amount is which?</div>
    </div>
    <div class="c good">
      <h5>Structure-aware chunking</h5>
      <div class="chunk whole">Deductible <span class="a">= $500</span></div>
      <div class="chunk whole">Claim Amount <span class="a">= $4,500</span></div>
      <div style="font-size:.72rem;color:var(--text-3);margin-top:.4rem">Each label stays glued to its value. Meaning preserved.</div>
    </div>
  </div>
  <figcaption>Left: cutting by size slices right through a label-value pair, orphaning "$500" from "Deductible". Later, when someone asks "what's the deductible?", retrieval finds a chunk with a lonely "$500" or a jumble of amounts, and the model guesses. Right: cut along the form's <em>structure</em> instead, keep each pair whole, and the value never loses its label. Same document, wildly different reliability.</figcaption>
</figure>

That's the whole problem in a nutshell: **naive, size-based chunking treats a form like prose, and forms aren't prose.** So how do you chunk them properly? There are a few approaches, and each buys accuracy at a different price.

## The chunking approaches, with their real trade-offs

Here's the menu, from naive to what I'd actually reach for, each with its honest side effect and its effect on accuracy.

<figure class="fm-fig">
  <div class="fm-appr" id="appr">
    <div class="fm-a">
      <div class="hd"><div class="num">1</div><div class="nm">Fixed-size chunking</div><div class="acc">low accuracy on forms</div></div>
      <div class="desc">Cut every N words, ignore structure. The default that works on prose.</div>
      <div class="se"><b>Side effect:</b> splits labels from values and rows from headers, so retrieval returns orphaned numbers and the model mismatches fields. Fine for paragraphs, actively harmful for forms.</div>
    </div>
    <div class="fm-a">
      <div class="hd"><div class="num">2</div><div class="nm">Field-aware chunking (one pair per chunk)</div><div class="acc">high on simple forms</div></div>
      <div class="desc">Chunk along the form's structure: each label-value pair (or small group of related fields) becomes its own chunk, kept whole.</div>
      <div class="se"><b>Side effect:</b> you need to <em>detect</em> the structure first (which requires knowing the form's layout), and very tiny chunks can lose surrounding context. But for key-value forms, accuracy jumps because nothing gets orphaned.</div>
    </div>
    <div class="fm-a">
      <div class="hd"><div class="num">3</div><div class="nm">Table-aware chunking (one row per chunk)</div><div class="acc">high for row lookups</div></div>
      <div class="desc">For tables, index each <em>row</em> as a chunk, carrying its column headers with it, so a row's cells stay tied to what they mean.</div>
      <div class="se"><b>Side effect:</b> great for "what's in row X?" questions, but it can lose cross-row context, "which row has the highest amount?" needs the whole table, not one row. Research on structure-aware table chunking reports strong faithfulness gains, but you trade away some table-wide reasoning.</div>
    </div>
    <div class="fm-a win">
      <div class="hd"><div class="num">4</div><div class="nm">Parent-child chunking</div><div class="acc">best all-rounder</div></div>
      <div class="desc">Keep <em>two</em> sizes: small child chunks (a single field or row) for precise <em>finding</em>, linked to a larger parent chunk (the whole section or table) for <em>understanding</em>. You search on the small ones, but hand the model the big one.</div>
      <div class="se"><b>Side effect:</b> you maintain two index layers and it's more work to update when a doc changes. But it resolves the core tension, precise retrieval <em>and</em> enough context, which is why it's the single biggest quality win for most setups.</div>
    </div>
    <div class="fm-a">
      <div class="hd"><div class="num">5</div><div class="nm">Turn it into structured data (skip RAG)</div><div class="acc">highest, when it fits</div></div>
      <div class="desc">If the form always has the same fields, don't chunk-and-search at all, extract the fields into a proper table (policy_number, amount, ...) and query them directly.</div>
      <div class="se"><b>Side effect:</b> only works when fields are known and consistent, and you lose RAG's flexibility for free-form questions. But when it fits, it's the most accurate and verifiable option, exact values, no fuzzy retrieval, no hallucinated numbers.</div>
    </div>
  </div>
  <figcaption>Five approaches, one theme: the more you respect the form's structure, the higher your accuracy, at the cost of more setup. Fixed-size (1) is easy and wrong for forms. Field- and table-aware (2, 3) fix the orphaning but need structure detection. Parent-child (4) is the workhorse. And sometimes (5) the right move is to stop treating it as a search problem at all.</figcaption>
</figure>

## The golden rule behind all of them

Every good approach above is really following one principle. If you remember nothing else, remember this:

<figure class="fm-fig">
  <div class="fm-rule" id="rule">
    <p><b>Never split a value from the thing that names it.</b> A number without its label is noise. Chunk along the form's structure, label-with-value, row-with-headers, so every piece that gets stored can still be understood on its own.</p>
  </div>
  <figcaption>This is the whole art of chunking a form. Prose chunking asks "where's a natural sentence break?" Form chunking asks "what's the smallest piece that still carries its own meaning?" For a form, that piece is a labelled field or a headed row, never an arbitrary slice of text.</figcaption>
</figure>

## A closer look at the workhorse: parent-child

Because parent-child chunking is the one I'd reach for most, here's how it actually works, and it's a neat idea. You store two linked versions of the content: tiny pieces to *search* with, and big pieces to *answer* from.

<figure class="fm-fig">
  <div class="fm-pc" id="pc">
    <svg viewBox="0 0 400 180">
      <rect class="box acc" x="120" y="10" width="160" height="42" rx="10"/>
      <text class="tx" x="200" y="30">PARENT: whole section</text>
      <text class="sub" x="200" y="44">handed to the model to answer</text>
      <rect class="box" x="20" y="120" width="100" height="42" rx="10"/>
      <text class="tx" x="70" y="140">child</text><text class="sub" x="70" y="154">Policy No.</text>
      <rect class="box" x="150" y="120" width="100" height="42" rx="10"/>
      <text class="tx" x="200" y="140">child</text><text class="sub" x="200" y="154">Claim Amt</text>
      <rect class="box" x="280" y="120" width="100" height="42" rx="10"/>
      <text class="tx" x="330" y="140">child</text><text class="sub" x="330" y="154">Deductible</text>
      <path class="ln" d="M180,52 L70,118"/><path class="ln" d="M200,52 L200,118"/><path class="ln" d="M220,52 L330,118"/>
    </svg>
  </div>
  <figcaption>You <em>search</em> on the small child chunks (each a single precise field), so retrieval is sharp, "claim amount" matches exactly the right little piece. But you <em>answer</em> from the parent (the whole section), so the model has the surrounding context and doesn't misread. Small chunks for finding, big chunks for understanding. That combination is why it beats using one chunk size for both.</figcaption>
</figure>

## The trick that quietly does a lot: metadata

One more technique that pairs with all of these, and it's easy to overlook. When you store each chunk, tag it with *metadata*, extra labels about what it is: which form, which field, which section. Then retrieval can *filter* before it even searches.

<figure class="fm-fig">
  <div class="fm-meta" id="meta">
    <div class="fm-mrow"><span class="tag">form: ACORD-25</span><div class="t">tag every chunk with the form type</div></div>
    <div class="fm-mrow"><span class="tag">field: claim_amount</span><div class="t">and which field it holds</div></div>
    <div class="fm-mrow"><span class="tag">filter first</span><div class="t"><b>Query "claim amount on the auto claim" filters to just those chunks, then searches. Faster and far more precise.</b></div></div>
  </div>
  <figcaption>Metadata turns a fuzzy search into a targeted one. Instead of searching all chunks and hoping the right field floats up, you first narrow to "claim-amount fields on this form type," then retrieve. It's cheap to add and it sharply cuts the wrong-field mistakes that plague form RAG. On structured documents, good metadata is often worth more than a fancier model.</figcaption>
</figure>

## Which one should you actually pick?

The honest decision guide, matched to your forms and your risk.

<figure class="fm-fig">
<table class="fm-tab">
<thead><tr><th>Your situation</th><th>Reach for</th><th>Because</th></tr></thead>
<tbody>
<tr><td>Simple, consistent key-value forms</td><td>Field-aware chunking</td><td>Keeps each pair whole; big accuracy win, low effort</td></tr>
<tr><td>Lots of tables, row-level questions</td><td>Table-aware (row) chunking</td><td>Ties each row's cells to their headers</td></tr>
<tr><td>Mixed forms, varied questions</td><td>Parent-child chunking</td><td>Precise finding plus enough context; the best all-rounder</td></tr>
<tr><td>Same fields every time, exact values matter</td><td>Extract to structured data, skip RAG</td><td>Most accurate and verifiable when the shape is fixed</td></tr>
<tr><td>Any of the above, high stakes</td><td>+ metadata + a confidence check</td><td>Filter to the right fields, and flag uncertain ones for a human</td></tr>
</tbody>
</table>
  <figcaption>No single winner, it depends on your forms. But the through-line is constant: the accuracy problem with form RAG is almost always a <em>chunking</em> problem, not a model problem. Fix how you cut the document, and the same model that was getting numbers wrong starts getting them right.</figcaption>
</figure>

## Want the real research?

If you want to go deeper than this overview, these are solid primary sources on chunking structured and tabular data for RAG:

- **Structure-aware chunking for tabular data**, on why row-and-header-aware chunking beats naive splitting, with faithfulness numbers: [Structure-Aware Chunking for Tabular Data in Retrieval-Augmented Generation](https://arxiv.org/html/2507.12425v1).
- **Semi-structured table question answering**, a look at answering questions over messy real tables: [ST-Raptor (arXiv 2508.18190)](https://arxiv.org/abs/2508.18190).
- **Adaptive retrieval over tabular formats**, on structured querying instead of blind chunk search: [SQuARE (arXiv 2512.04292)](https://arxiv.org/abs/2512.04292).

## The takeaway

When RAG gets a number wrong on a form, the instinct is to blame the model or reach for a bigger one. Nine times out of ten, the real fault is upstream, in how you chopped the document. Naive size-based chunking treats a form like a paragraph and slices right through the label-value pairs that carry its meaning, leaving retrieval with orphaned numbers to guess from.

The fix isn't fancier AI, it's chunking that respects the form's shape: keep each label glued to its value, each row glued to its headers, use parent-child so you can find precisely and answer with context, tag everything with metadata so you can filter to the right field, and when the form is fixed and the stakes are high, skip fuzzy search and pull the fields into a real table. Every one of these has a cost, more structure detection, more index layers, less flexibility, but they all buy the same thing: numbers that come out right. On a form, accuracy was never really about the model. It was about not cutting the meaning in half before the model ever sees it.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['appr','meta','break','pc','rule'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
