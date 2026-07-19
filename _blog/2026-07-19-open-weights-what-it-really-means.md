---
title: "Open Weights: What It Really Means (and Why It's Not Open Source)"
date: 2026-07-19
excerpt: "You keep hearing that Llama, DeepSeek, and Mistral are 'open source' AI. Most of them aren't, not really. They're open weight, which is a different, more limited thing that people constantly mix up. Here's what open weights actually means, how it differs from open source and closed models, in plain language, with a simple analogy anyone can follow."
tags: [ai, open-weights, open-source, models, explainer]
---

<style>
.ow-fig{margin:2.5rem 0;}
.ow-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* the cake analogy */
.ow-cake{display:grid;grid-template-columns:repeat(3,1fr);gap:.7rem;max-width:700px;margin:0 auto;}
@media(max-width:600px){.ow-cake{grid-template-columns:1fr;}}
.ow-cake .c{border:1px solid var(--border-2);border-radius:12px;padding:1rem;background:var(--surface);text-align:center;opacity:0;transform:translateY(8px);transition:opacity .45s ease,transform .45s ease;}
.ow-cake.go .c{opacity:1;transform:none;}
.ow-cake.go .c:nth-child(1){transition-delay:.1s} .ow-cake.go .c:nth-child(2){transition-delay:.3s} .ow-cake.go .c:nth-child(3){transition-delay:.5s}
.ow-cake .c.mid{border-color:var(--accent);}
.ow-cake .c .top{font-family:var(--font-mono);font-size:.7rem;color:var(--text-3);margin-bottom:.3rem;}
.ow-cake .c.mid .top{color:var(--accent);}
.ow-cake .c b{display:block;color:var(--text);font-size:.9rem;margin-bottom:.35rem;}
.ow-cake .c.mid b{color:var(--accent);}
.ow-cake .c span{font-size:.82rem;color:var(--text-2);line-height:1.45;}

/* what you get: three ingredients */
.ow-parts{max-width:640px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.ow-prow{display:flex;align-items:center;gap:.9rem;border:1px solid var(--border);border-radius:10px;padding:.7rem .9rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.ow-parts.go .ow-prow{opacity:1;transform:none;}
.ow-parts.go .ow-prow:nth-child(1){transition-delay:.1s} .ow-parts.go .ow-prow:nth-child(2){transition-delay:.3s} .ow-parts.go .ow-prow:nth-child(3){transition-delay:.5s}
.ow-prow .nm{flex:none;width:130px;font-family:var(--font-mono);font-size:.8rem;font-weight:600;color:var(--text);}
.ow-prow .d{font-size:.84rem;color:var(--text-2);} .ow-prow .d b{color:var(--accent);}

/* who gives what (matrix) */
.ow-matrix{max-width:640px;margin:0 auto;}
.ow-mtab{width:100%;border-collapse:collapse;font-size:.86rem;}
.ow-mtab th,.ow-mtab td{padding:.55rem .6rem;border-bottom:1px solid var(--border);text-align:center;}
.ow-mtab th{font-family:var(--font-mono);font-size:.7rem;text-transform:uppercase;letter-spacing:.05em;color:var(--text-3);}
.ow-mtab td:first-child,.ow-mtab th:first-child{text-align:left;color:var(--text);font-weight:500;}
.ow-mtab .y{color:var(--accent);font-weight:600;} .ow-mtab .n{color:var(--text-3);}

/* what open weights lets you do */
.ow-can{display:grid;grid-template-columns:1fr 1fr;gap:.8rem;max-width:640px;margin:0 auto;}
@media(max-width:520px){.ow-can{grid-template-columns:1fr;}}
.ow-can .side{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.ow-can .side h4{margin:0 0 .6rem;font-size:.82rem;font-family:var(--font-mono);}
.ow-can .side.yes{border-color:var(--accent);} .ow-can .side.yes h4{color:var(--accent);}
.ow-can .side.no h4{color:var(--text-2);}
.ow-can .side .item{font-size:.83rem;color:var(--text-2);margin:.4rem 0;line-height:1.45;padding-left:1.1rem;position:relative;}
.ow-can .side .item::before{position:absolute;left:0;font-family:var(--font-mono);}
.ow-can .side.yes .item::before{content:"+";color:var(--accent);}
.ow-can .side.no .item::before{content:"\2212";color:var(--text-3);}

/* openwashing */
.ow-wash{max-width:560px;margin:0 auto;text-align:center;border:1px solid var(--accent);border-radius:12px;padding:1.1rem 1.3rem;background:var(--surface);}
.ow-wash b{color:var(--accent);} .ow-wash p{font-size:.9rem;color:var(--text-2);line-height:1.6;margin:0;}

/* examples chips */
.ow-ex{max-width:660px;margin:0 auto;display:flex;flex-direction:column;gap:.6rem;}
.ow-erow{border:1px solid var(--border);border-radius:10px;padding:.7rem .9rem;background:var(--surface);}
.ow-erow .lb{font-family:var(--font-mono);font-size:.72rem;color:var(--accent);margin-bottom:.3rem;}
.ow-erow .items{font-family:var(--font-mono);font-size:.82rem;color:var(--text-2);}
.ow-erow .items b{color:var(--text);}

/* table */
.ow-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.ow-tab th,.ow-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.ow-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.ow-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}

/* extra reveals */
.ow-can .side{opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.ow-can.go .side{opacity:1;transform:none;}
.ow-can.go .side:nth-child(1){transition-delay:.1s} .ow-can.go .side:nth-child(2){transition-delay:.35s}

.ow-wash{opacity:0;transform:scale(.98);transition:opacity .6s ease,transform .6s ease;}
.ow-wash.go{opacity:1;transform:none;}

.ow-ex .ow-erow{opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.ow-ex.go .ow-erow{opacity:1;transform:none;}
.ow-ex.go .ow-erow:nth-child(1){transition-delay:.1s} .ow-ex.go .ow-erow:nth-child(2){transition-delay:.3s} .ow-ex.go .ow-erow:nth-child(3){transition-delay:.5s}

@media (prefers-reduced-motion: reduce){
  .ow-cake .c,.ow-parts .ow-prow,.ow-can .side,.ow-wash,.ow-ex .ow-erow{transition:none !important;opacity:1 !important;transform:none !important;}
}
</style>

You've probably read that Llama, DeepSeek, Mistral, and Qwen are "open source" AI models, the free, community versions you can download and run yourself, unlike the locked-away GPT and Claude. It's a nice story. It's also, for most of them, not quite true.

Almost all of those models are **open weight**, not open source. And the difference isn't nitpicking, it's a real, meaningful gap that changes what you can actually do with the model, and how much you can trust it. People mix these two terms up constantly, including press releases and marketing that really should know better. So let me clear it up properly, in plain language, with an analogy anyone can follow. By the end you'll know exactly what "open weights" means, why it isn't the same as open source, and why it matters.

## First, what even are the "weights"?

Quick foundation. An AI model is, underneath, a giant pile of numbers, billions of them, called its **weights**. Those numbers are what the model *learned* during training; they're the finished brain. When you run a model, you're running its weights. (I've written about how these are stored and shrunk if you want the deep version.)

So "open weights" is about one specific thing: **can you download that finished pile of numbers and run it yourself?** That's it. It says nothing about whether you can see *how* those numbers were made.

## The cake analogy (the whole idea in one picture)

Here's the mental model that makes everything click. Think of an AI model like a cake.

<figure class="ow-fig">
  <div class="ow-cake" id="cake">
    <div class="c">
      <div class="top">closed</div>
      <b>Buy a slice at the shop</b>
      <span>You can taste it, but only at their counter. You can't take the cake home, and you certainly don't get the recipe. This is GPT, Claude, Gemini, use it through their service only.</span>
    </div>
    <div class="c mid">
      <div class="top">open weights</div>
      <b>Take the whole cake home</b>
      <span>They hand you the finished cake. You can eat it, re-decorate it, serve it anywhere. But you don't get the recipe or the ingredient list, so you can't know exactly how it was made. This is Llama, DeepSeek, Mistral.</span>
    </div>
    <div class="c">
      <div class="top">open source</div>
      <b>Cake plus the full recipe</b>
      <span>You get the cake and the complete recipe and ingredients, so you could bake your own from scratch, inspect exactly what went in, and change anything. This is rarer than people think.</span>
    </div>
  </div>
  <figcaption>Three levels of openness. Closed: you never leave the shop. Open weights: you get the finished cake, yours to use and modify, but no recipe. Open source: cake <em>and</em> recipe, fully reproducible. Most "open" AI models today are the middle one, the cake without the recipe.</figcaption>
</figure>

## The three ingredients of a truly open model

To see why open weights isn't open source, you need to know that "making an AI model" has three parts. A truly open-source model gives you all three. An open-weight model gives you only the first.

<figure class="ow-fig">
  <div class="ow-parts" id="parts">
    <div class="ow-prow"><div class="nm">The weights</div><div class="d">The finished model, the pile of learned numbers. <b>Open weights gives you this.</b> Download, run, fine-tune.</div></div>
    <div class="ow-prow"><div class="nm">The training code</div><div class="d">The recipe: the exact code and process used to train it. Open weights usually keeps this secret.</div></div>
    <div class="ow-prow"><div class="nm">The training data</div><div class="d">The ingredients: what it was actually trained on. Open weights almost never reveals this.</div></div>
  </div>
  <figcaption>Weights, code, data. Open weights hands you the finished product (the weights) but keeps the recipe (code) and ingredients (data) private. Open source gives you all three, so an independent team could rebuild the exact model, check what data shaped it, and audit it honestly. The real test of "open source" is one word: <em>reproducibility.</em> Can someone else recreate it? With just weights, no.</figcaption>
</figure>

## So who actually gives what?

Here's the honest scorecard across the three kinds of models.

<figure class="ow-fig">
  <div class="ow-matrix">
    <table class="ow-mtab">
      <thead><tr><th>You get...</th><th>Closed</th><th>Open weights</th><th>Open source</th></tr></thead>
      <tbody>
        <tr><td>Run it yourself</td><td class="n">no</td><td class="y">yes</td><td class="y">yes</td></tr>
        <tr><td>Fine-tune it on your data</td><td class="n">no</td><td class="y">yes</td><td class="y">yes</td></tr>
        <tr><td>The training code</td><td class="n">no</td><td class="n">no</td><td class="y">yes</td></tr>
        <tr><td>The training data</td><td class="n">no</td><td class="n">no</td><td class="y">yes</td></tr>
        <tr><td>Rebuild / fully audit it</td><td class="n">no</td><td class="n">no</td><td class="y">yes</td></tr>
      </tbody>
    </table>
  </div>
  <figcaption>The two middle columns look similar but they're not. Open weights and open source both let you run and fine-tune the model, that's the practical freedom most people care about. But only open source lets you see the recipe and ingredients, and therefore verify what the model really is. That gap in the bottom three rows is the whole story.</figcaption>
</figure>

## What open weights genuinely lets you do (and what it doesn't)

Let's be fair to open weights, because it gives you a lot, just not everything.

<figure class="ow-fig">
  <div class="ow-can" id="can">
    <div class="side yes">
      <h4>What you CAN do</h4>
      <div class="item">Run it on your own machines, your data never leaves</div>
      <div class="item">Fine-tune it on your own data for your own tasks</div>
      <div class="item">Deploy it in your product with no per-use fee to a vendor</div>
      <div class="item">Escape a vendor's pricing, rules, and "we're retiring this model" decisions</div>
    </div>
    <div class="side no">
      <h4>What you CAN'T do</h4>
      <div class="item">See what data it was trained on (bias? copyrighted material?)</div>
      <div class="item">Reproduce it from scratch or fully audit how it behaves</div>
      <div class="item">Truly verify claims about how it was built</div>
      <div class="item">Always use it freely, some "open weight" licenses have real restrictions</div>
    </div>
  </div>
  <figcaption>The honest balance. Open weights buys you the three things enterprises care about most, run it yourself, tune it yourself, no vendor lock-in. What it doesn't buy is <em>transparency</em>: you're trusting a finished brain without knowing what went into making it. For most practical uses that's fine; for auditing bias or verifying safety claims, it's a real limitation.</figcaption>
</figure>

## The word for calling open weights "open source": openwashing

Here's the honest part the marketing skips. Because "open source" sounds more generous and trustworthy than "open weight," a lot of companies label their open-weight models "open source" when they aren't. There's even a name for it.

<figure class="ow-fig">
  <div class="ow-wash" id="wash">
    <p><b>Openwashing</b> is marketing an open-weight model as "open source" when it isn't, taking credit for openness you didn't actually provide. The Open Source Initiative, the body that literally defines "open source," has been blunt that weights alone are "a downloadable artifact, not a reproducible one." A cake is not a recipe.</p>
  </div>
  <figcaption>This is why the distinction matters beyond pedantry. "Open source" carries a promise, anyone can inspect, verify, and rebuild it. Open weights doesn't carry that promise. Calling one the other quietly overstates how transparent and trustworthy the model really is.</figcaption>
</figure>

## Real examples, labelled honestly

So where do the models you've heard of actually land? Here's the honest map.

<figure class="ow-fig">
  <div class="ow-ex" id="ex">
    <div class="ow-erow"><div class="lb">closed (API only)</div><div class="items"><b>GPT, Claude, Gemini</b>: you use them through a service; you can't download them.</div></div>
    <div class="ow-erow"><div class="lb">open weights (most "open" models)</div><div class="items"><b>Llama, DeepSeek, Mistral, Qwen, Gemma</b>: download and run them, but the training data stays secret. Widely (and wrongly) called "open source."</div></div>
    <div class="ow-erow"><div class="lb">genuinely open source (rarer)</div><div class="items"><b>OLMo (Allen Institute), Pythia (EleutherAI)</b>: weights AND training code AND the dataset, fully reproducible and auditable.</div></div>
  </div>
  <figcaption>Notice the middle row is where nearly all the famous "open" models sit, they're open weight, not open source, even when marketed otherwise. The genuinely-open-source models exist (OLMo and Pythia are the go-to examples) but they're the exception, usually from research institutes rather than companies protecting a competitive edge or worried about what's in their training data.</figcaption>
</figure>

## Why companies stop at open weights

A fair question: if open source is more open, why don't they just do that? Two honest reasons:

- **Competitive advantage.** The training data and recipe are the secret sauce that took millions of dollars and huge effort to build. Giving those away hands rivals a free head start.
- **Legal caution.** Training data often contains copyrighted material scraped from the web. Publishing exactly what's in it invites lawsuits. Keeping it secret avoids that fight.

So open weights is a genuine middle ground: generous enough to let the world *use* and *build on* the model, guarded enough to protect the company's edge and dodge the legal minefield. That's why it's become the dominant flavor of "open" AI, and why it's here to stay.

## The quick reference

<figure class="ow-fig">
<table class="ow-tab">
<thead><tr><th></th><th>Closed</th><th>Open weights</th><th>Open source</th></tr></thead>
<tbody>
<tr><td>In one line</td><td>Rent it via API</td><td>Own the finished model</td><td>Own the model + the recipe</td></tr>
<tr><td>Run it yourself?</td><td>No</td><td>Yes</td><td>Yes</td></tr>
<tr><td>See how it was made?</td><td>No</td><td>No</td><td>Yes</td></tr>
<tr><td>Reproducible / auditable?</td><td>No</td><td>No</td><td>Yes</td></tr>
<tr><td>Examples</td><td>GPT, Claude, Gemini</td><td>Llama, DeepSeek, Mistral</td><td>OLMo, Pythia</td></tr>
</tbody>
</table>
  <figcaption>The whole thing on one card. The line most people get wrong is the middle one: "open weights" is real openness, but it is <em>not</em> "open source," because you can't see how it was made. Keep the cake-versus-recipe picture and you'll never confuse them again.</figcaption>
</figure>

## The takeaway

"Open weights" means one clear thing: you can download the finished AI model, run it on your own hardware, and fine-tune it, without asking anyone's permission or paying per use. That's genuinely valuable, and it's why models like Llama and DeepSeek changed the landscape. But it is *not* the same as open source. Open weights gives you the cake; open source gives you the cake *and* the recipe *and* the ingredient list, so you could bake it yourself and check exactly what went in.

The next time you see a model called "open source," it's worth a small, healthy dose of skepticism: do they actually share the training data and code, or just the finished weights? Most of the time, it's just the weights. That's not a scandal, open weights is a real and useful kind of openness. It's just a smaller promise than "open source," and now you know exactly where the line sits, and why anyone who cares about trust, bias, or reproducibility should care where a model falls.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['cake','parts','can','wash','ex'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
