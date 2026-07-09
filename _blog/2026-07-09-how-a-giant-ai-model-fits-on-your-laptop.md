---
title: "How a Giant AI Model Fits on Your Laptop"
date: 2026-07-09
excerpt: "A serious AI model is billions of numbers, tens of gigabytes at full size. Your laptop has nowhere near the room. Yet it runs, smoothly. The trick is quantization: storing each number in fewer bits, like saving a photo as a JPEG instead of a giant RAW file. Here is exactly how it works, why it barely hurts quality, and where it finally breaks."
tags: [ai, quantization, local-llm, gguf, llm, explainer]
---

<style>
.qz-fig{margin:2.5rem 0;}
.qz-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* the problem: size vs laptop */
.qz-prob{max-width:520px;margin:0 auto;display:flex;flex-direction:column;gap:.6rem;}
.qz-prob .row{display:flex;align-items:center;gap:.8rem;}
.qz-prob .row .lb{flex:none;width:150px;font-family:var(--font-mono);font-size:.76rem;color:var(--text-2);text-align:right;}
.qz-prob .row .track{flex:1;height:26px;border-radius:6px;background:var(--surface-2);border:1px solid var(--border);overflow:hidden;position:relative;}
.qz-prob .row .fill{height:100%;display:flex;align-items:center;padding-left:.5rem;font-family:var(--font-mono);font-size:.7rem;color:var(--accent-ink);white-space:nowrap;}
.qz-prob .row .fill.big{background:var(--surface-2);color:var(--text-2);border-right:2px solid var(--text-3);}
.qz-prob .row .fill.fit{background:var(--grad);}

/* what a weight is - bits */
.qz-bits{max-width:560px;margin:0 auto;}
.qz-bits .num{font-family:var(--font-mono);text-align:center;color:var(--text);font-size:1.1rem;margin-bottom:.6rem;}
.qz-bits .row{display:flex;gap:.5rem;justify-content:center;flex-wrap:wrap;margin-bottom:.5rem;}
.qz-bits .cell{border:1px solid var(--border-2);border-radius:6px;padding:.4rem;text-align:center;min-width:56px;background:var(--surface);}
.qz-bits .cell b{display:block;font-family:var(--font-mono);font-size:.72rem;color:var(--accent);}
.qz-bits .cell span{font-size:.66rem;color:var(--text-3);}

/* precision ladder animation */
.qz-ladder{max-width:620px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.qz-rung{display:flex;align-items:center;gap:.8rem;border:1px solid var(--border);border-radius:10px;padding:.6rem .9rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.qz-ladder.go .qz-rung{opacity:1;transform:none;}
.qz-ladder.go .qz-rung:nth-child(1){transition-delay:.1s} .qz-ladder.go .qz-rung:nth-child(2){transition-delay:.3s}
.qz-ladder.go .qz-rung:nth-child(3){transition-delay:.5s} .qz-ladder.go .qz-rung:nth-child(4){transition-delay:.7s}
.qz-rung .fmt{flex:none;width:70px;font-family:var(--font-mono);font-size:.8rem;font-weight:600;color:var(--accent);}
.qz-rung .bar{flex:1;height:16px;border-radius:4px;background:var(--surface-2);border:1px solid var(--border);overflow:hidden;}
.qz-rung .bfill{height:100%;background:var(--grad);}
.qz-rung .sz{flex:none;width:88px;font-family:var(--font-mono);font-size:.72rem;color:var(--text-2);text-align:right;}

/* the mapping: float to int (the core mechanic) */
.qz-map{max-width:600px;margin:0 auto;}
.qz-map .band{position:relative;height:44px;border:1px solid var(--border);border-radius:8px;background:var(--surface-2);margin:.4rem 0;}
.qz-map .tick{position:absolute;top:0;bottom:0;width:1px;background:var(--border-2);}
.qz-map .lbl{font-family:var(--font-mono);font-size:.72rem;color:var(--text-2);display:flex;justify-content:space-between;}
.qz-map .dot{position:absolute;top:50%;transform:translate(-50%,-50%);width:10px;height:10px;border-radius:50%;background:var(--accent);opacity:0;transition:opacity .4s ease;}
.qz-map.go .dot{opacity:1;}
.qz-map .snap{position:absolute;top:50%;transform:translate(-50%,-50%);font-family:var(--font-mono);font-size:.64rem;color:var(--accent);opacity:0;transition:opacity .4s ease .6s;}
.qz-map.go .snap{opacity:1;}

/* worked example card */
.qz-work{max-width:520px;margin:0 auto;background:var(--surface);border:1px solid var(--border-2);border-radius:12px;padding:1rem 1.2rem;font-family:var(--font-mono);font-size:.82rem;line-height:1.9;color:var(--text-2);}
.qz-work .k{color:var(--accent);}
.qz-work .c{color:var(--text-3);}
.qz-work .r{color:var(--text);}

/* jpeg analogy */
.qz-jpeg{max-width:600px;margin:0 auto;display:flex;gap:.8rem;justify-content:center;flex-wrap:wrap;}
.qz-jpeg .c{flex:1;min-width:150px;border:1px solid var(--border-2);border-radius:10px;padding:.9rem;text-align:center;background:var(--surface);}
.qz-jpeg .c.acc{border-color:var(--accent);}
.qz-jpeg .c b{display:block;color:var(--text);font-size:.85rem;margin-bottom:.2rem;}
.qz-jpeg .c.acc b{color:var(--accent);}
.qz-jpeg .c span{font-size:.76rem;color:var(--text-2);line-height:1.4;}

/* accuracy retention bars */
.qz-acc{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:.8rem;}
.qz-acc .row{display:flex;align-items:center;gap:.8rem;}
.qz-acc .row .d{flex:none;width:96px;font-family:var(--font-mono);font-size:.74rem;color:var(--text-2);}
.qz-acc .row .track{flex:1;height:18px;border-radius:6px;background:var(--surface-2);border:1px solid var(--border);overflow:hidden;}
.qz-acc .row .fill{height:100%;width:0;transition:width 1s cubic-bezier(.2,.7,.2,1);}
.qz-acc.go .row .fill{width:var(--w);}
.qz-acc .row .fill.good{background:var(--grad);}
.qz-acc .row .fill.cliff{background:var(--surface-2);border-right:2px solid var(--text-3);}
.qz-acc .row .n{flex:none;width:74px;font-family:var(--font-mono);font-size:.72rem;color:var(--accent);text-align:right;}
.qz-acc.go .r1 .fill{transition-delay:.1s} .qz-acc.go .r2 .fill{transition-delay:.3s} .qz-acc.go .r3 .fill{transition-delay:.5s}
.qz-acc.go .r4 .fill{transition-delay:.7s} .qz-acc.go .r5 .fill{transition-delay:.9s}

/* outlier illustration */
.qz-out{max-width:560px;margin:0 auto;display:flex;gap:3px;justify-content:center;align-items:flex-end;height:90px;}
.qz-out .w{flex:1;max-width:14px;background:var(--surface-2);border:1px solid var(--border);border-radius:2px 2px 0 0;}
.qz-out .w.outlier{background:var(--grad);border-color:var(--accent);}

/* table */
.qz-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.qz-tab th,.qz-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.qz-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.qz-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}

/* extra reveal animations */
.qz-prob .row .fill{width:0 !important;transition:width 1s cubic-bezier(.2,.7,.2,1);}
.qz-prob.go .row:nth-child(1) .fill{width:100% !important;transition-delay:.1s}
.qz-prob.go .row:nth-child(2) .fill{width:50% !important;transition-delay:.3s}
.qz-prob.go .row:nth-child(3) .fill{width:31% !important;transition-delay:.5s}

.qz-work{opacity:0;transform:translateY(8px);transition:opacity .6s ease,transform .6s ease;}
.qz-work.go{opacity:1;transform:none;}

.qz-jpeg .c{opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.qz-jpeg.go .c{opacity:1;transform:none;}
.qz-jpeg.go .c:nth-child(1){transition-delay:.1s} .qz-jpeg.go .c:nth-child(2){transition-delay:.4s}

.qz-out .w{height:4px !important;transition:height .7s cubic-bezier(.2,.7,.2,1);align-self:flex-end;}
.qz-out.go .w{height:var(--h) !important;}

@media (prefers-reduced-motion: reduce){
  .qz-ladder .qz-rung,.qz-map .dot,.qz-map .snap,.qz-acc .row .fill,
  .qz-prob .row .fill,.qz-work,.qz-jpeg .c,.qz-out .w{transition:none !important;}
  .qz-ladder .qz-rung{opacity:1 !important;transform:none !important;}
  .qz-map .dot,.qz-map .snap{opacity:1 !important;}
  .qz-acc .row .fill{width:var(--w) !important;}
  .qz-work{opacity:1 !important;transform:none !important;} .qz-jpeg .c{opacity:1 !important;transform:none !important;}
  .qz-prob.go .row:nth-child(1) .fill{width:100% !important;} .qz-prob.go .row:nth-child(2) .fill{width:50% !important;} .qz-prob.go .row:nth-child(3) .fill{width:31% !important;}
  .qz-out .w{height:var(--h) !important;}
}
</style>

Something quietly impossible happens every time you run an AI model on your laptop.

A capable modern model is, at its heart, a giant pile of numbers, its "weights." A small one has around 8 billion of them. Stored the normal way, that's roughly 16 gigabytes of pure numbers, and bigger models run to hundreds of gigabytes. Your laptop's memory is nowhere near enough to hold that comfortably. By rights, it shouldn't fit. And yet you type one command and the thing runs, smoothly, answering you in real time.

How? The answer is one of the most elegant tricks in all of AI, and it's called **quantization**. The one-line version: *store each of those billions of numbers using fewer bits.* It's the exact same idea as saving a photo as a compact JPEG instead of a massive RAW file, you lose a tiny sliver of quality you can barely perceive, and in return the thing shrinks to a fraction of its size.

I touched this in my Ollama post, but it deserves the full treatment, because once you actually see *how* it works, it stops feeling like magic and starts feeling like clever engineering you could have thought of yourself. Let me build it up from the very bottom: what a single weight even is, how you shrink it, why that barely hurts, and where the trick finally falls apart.

## The problem, drawn to scale

First, feel the size mismatch that makes this necessary:

<figure class="qz-fig">
  <div class="qz-prob" id="prob">
    <div class="row"><div class="lb">8B model, full size</div><div class="track"><div class="fill big" style="width:100%">~16 GB</div></div></div>
    <div class="row"><div class="lb">typical laptop RAM</div><div class="track"><div class="fill big" style="width:50%">~8 to 16 GB</div></div></div>
    <div class="row"><div class="lb">same model, quantized</div><div class="track"><div class="fill fit" style="width:31%">~5 GB</div></div></div>
  </div>
  <figcaption>The squeeze. At full size, even a "small" 8B model barely fits (or doesn't) alongside everything else your laptop is doing. Quantized down to a quarter of the size, it slots in with room to spare. That shrink is the whole reason local AI is possible at all.</figcaption>
</figure>

## Step 1: what is a single weight, really?

Before you can shrink billions of numbers, you have to know what one of them looks like *to the computer.* A weight is just a number like `0.4732`. But the computer doesn't store "0.4732" as text, it stores it in **bits**, using a format called a floating-point number. The standard format for AI, **FP16**, uses **16 bits** per number.

<figure class="qz-fig">
  <div class="qz-bits">
    <div class="num">0.4732</div>
    <div class="row">
      <div class="cell"><b>1 bit</b><span>sign</span></div>
      <div class="cell"><b>5 bits</b><span>exponent</span></div>
      <div class="cell"><b>10 bits</b><span>fraction</span></div>
    </div>
    <div style="text-align:center;font-family:var(--font-mono);font-size:.74rem;color:var(--text-3)">= 16 bits (FP16) to store one weight, precisely</div>
  </div>
  <figcaption>One weight in FP16: 16 bits, split into a sign, an exponent, and a fraction. This gives lots of precision, it can represent a huge range of values very finely. But precision costs space. 16 bits per weight, times 8 billion weights, is where the 16 gigabytes comes from. The question quantization asks is: do we really need all 16 bits?</figcaption>
</figure>

## Step 2: the ladder of precision

Here's the key realization: you don't have to store every weight in 16 bits. You can use fewer, and the fewer you use, the smaller the model. This is a ladder, and each step down roughly halves the size:

<figure class="qz-fig">
  <div class="qz-ladder" id="ladder">
    <div class="qz-rung"><div class="fmt">FP16</div><div class="bar"><div class="bfill" style="width:100%"></div></div><div class="sz">16 bits · ~16 GB</div></div>
    <div class="qz-rung"><div class="fmt">INT8</div><div class="bar"><div class="bfill" style="width:50%"></div></div><div class="sz">8 bits · ~8 GB</div></div>
    <div class="qz-rung"><div class="fmt">INT4</div><div class="bar"><div class="bfill" style="width:25%"></div></div><div class="sz">4 bits · ~4 GB</div></div>
    <div class="qz-rung"><div class="fmt">INT2</div><div class="bar"><div class="bfill" style="width:12%"></div></div><div class="sz">2 bits · ~2 GB (rough)</div></div>
  </div>
  <figcaption>Each step down the ladder cuts the storage roughly in half. FP16 is full quality. INT8 halves it. INT4, the popular default for laptops, quarters it. Go lower and it keeps shrinking, but as we'll see, quality starts to crumble past a point. The art is finding how low you can go before it hurts.</figcaption>
</figure>

But wait, this should bother you. A 16-bit float can represent a smooth, near-infinite range of values. A 4-bit integer can only represent **16 distinct values** (2 to the power of 4). How on earth do you cram a delicate weight like `0.4732` into a system that only has 16 possible slots? That's the heart of the trick, and it's beautifully simple.

## Step 3: the actual trick, mapping floats onto a grid

Here's how you turn a precise float into a low-bit integer. Imagine the possible integer values as a **grid of evenly-spaced pegs.** Quantization takes each real weight and *snaps it to the nearest peg.* You lose the tiny distance between the true value and the peg, that's the "rounding error", but you gain the ability to store just which peg it landed on, which takes far fewer bits.

<figure class="qz-fig">
  <div class="qz-map" id="map">
    <div class="lbl"><span>-max</span><span>0</span><span>+max</span></div>
    <div class="band">
      <div class="tick" style="left:6%"></div><div class="tick" style="left:19%"></div><div class="tick" style="left:31%"></div><div class="tick" style="left:44%"></div><div class="tick" style="left:56%"></div><div class="tick" style="left:69%"></div><div class="tick" style="left:81%"></div><div class="tick" style="left:94%"></div>
      <div class="dot" style="left:38%"></div>
      <div class="snap" style="left:44%;top:130%">snaps to nearest peg</div>
    </div>
  </div>
  <figcaption>The real weight (the dot) sits somewhere between the pegs. Quantization snaps it to the closest one and stores just that peg's number. The vertical lines are the only values a 4-bit integer can hold, 16 of them. The trick isn't storing the value exactly; it's storing <em>which peg</em> is closest, cheaply, and accepting the tiny miss.</figcaption>
</figure>

But there's a catch you may have spotted: those pegs are evenly spaced from `-max` to `+max`. How does the computer know what `max` is, so it knows how wide to space the pegs? That's where the one crucial extra number comes in: the **scale factor.**

The recipe (this is the whole mechanic, in plain arithmetic):

<figure class="qz-fig">
  <div class="qz-work" id="work">
<span class="c"># Say your biggest weight in a group is 2.5,</span>
<span class="c"># and you're quantizing to INT8 (range up to 127).</span>

<span class="k">scale</span> = 127 / 2.5 = <span class="r">50.8</span>   <span class="c"># pegs per unit</span>

<span class="c"># To store weight 0.4732:</span>
<span class="k">store</span> = round(0.4732 × 50.8) = <span class="r">24</span>   <span class="c"># just an integer!</span>

<span class="c"># To use it later (dequantize):</span>
<span class="k">recover</span> = 24 / 50.8 = <span class="r">0.4724</span>   <span class="c"># ≈ 0.4732 ✓</span>
  </div>
  <figcaption>The full round-trip. You store the integer <b>24</b> (tiny) plus one shared scale factor for the whole group. To use the weight, you multiply back by the scale to recover approximately the original. The recovered <b>0.4724</b> is a hair off the true <b>0.4732</b>, that gap is the only quality you lose. Multiply this tiny, forgivable error across billions of weights and it mostly averages out.</figcaption>
</figure>

The elegant part: you don't store a scale for *every* weight (that would defeat the purpose). You group weights into blocks, say 32 at a time, and store **one** scale factor per block. So the real cost of "4-bit" is a touch more than 4 bits: 32 four-bit weights (128 bits) plus one 16-bit scale is 144 bits over 32 weights, about **4.5 bits each.** Honest math, and still a massive saving.

## Why this barely hurts: the JPEG intuition

Now the reassuring part. You might expect throwing away precision to wreck the model. It mostly doesn't, and the reason is the same reason JPEG works on photos:

<figure class="qz-fig">
  <div class="qz-jpeg" id="jpeg">
    <div class="c"><b>RAW photo</b><span>every pixel in perfect detail. Huge file. More precision than your eye can even see.</span></div>
    <div class="c acc"><b>JPEG photo</b><span>drops detail you'd never notice. A fraction of the size. Looks identical.</span></div>
  </div>
  <figcaption>A neural network, like a photograph, carries far more precision than it strictly needs. Most of a weight's exact decimal places don't change the model's answer, they're below the threshold that matters, just like the subtle color detail a JPEG discards is below what your eye catches. Quantization throws away the precision that wasn't doing much work. That's why the model keeps behaving almost identically.</figcaption>
</figure>

And "almost identically" is measurable, not a hope. Here's roughly how much of the original quality survives at each level (from large-scale evaluations):

<figure class="qz-fig">
  <div class="qz-acc" id="acc">
    <div class="row r1"><div class="d">FP16 (full)</div><div class="track"><div class="fill good" style="--w:100%"></div></div><div class="n">100%</div></div>
    <div class="row r2"><div class="d">INT8 / Q8</div><div class="track"><div class="fill good" style="--w:99.5%"></div></div><div class="n">~99%+</div></div>
    <div class="row r3"><div class="d">Q4_K_M (4-bit)</div><div class="track"><div class="fill good" style="--w:98%"></div></div><div class="n">~97-99%</div></div>
    <div class="row r4"><div class="d">Q3</div><div class="track"><div class="fill good" style="--w:90%"></div></div><div class="n">~90%</div></div>
    <div class="row r5"><div class="d">Q2</div><div class="track"><div class="fill cliff" style="--w:70%"></div></div><div class="n">falls off</div></div>
  </div>
  <figcaption>The size-quality tradeoff, as bars. 8-bit keeps over 99%. The popular 4-bit (Q4_K_M) still holds around 97 to 99%, losing just 1 to 3% on small models, and even less on big ones (a 70B model at 4-bit barely notices). But look at the bottom: around 2-bit, quality falls off a cliff. There's a sweet spot, and 4-bit is usually it.</figcaption>
</figure>

## The wrinkle the pros handle: outliers

Here's where it gets genuinely interesting, and where naive quantization breaks. In a real model, the weights aren't all similar sizes. A tiny fraction of them, often around **0.1%**, are wild **outliers**: values far bigger than the rest. And they matter enormously to the output.

<figure class="qz-fig">
  <div class="qz-out" id="out">
    <div class="w" style="--h:30%"></div><div class="w" style="--h:25%"></div><div class="w" style="--h:35%"></div><div class="w" style="--h:28%"></div><div class="w" style="--h:32%"></div><div class="w" style="--h:22%"></div><div class="w outlier" style="--h:95%"></div><div class="w" style="--h:30%"></div><div class="w" style="--h:26%"></div><div class="w" style="--h:33%"></div><div class="w" style="--h:29%"></div><div class="w" style="--h:24%"></div><div class="w" style="--h:31%"></div><div class="w outlier" style="--h:88%"></div><div class="w" style="--h:27%"></div><div class="w" style="--h:34%"></div><div class="w" style="--h:23%"></div><div class="w" style="--h:30%"></div>
  </div>
  <figcaption>Most weights are small and similar (the short bars); a rare few are huge outliers (the tall accent bars). Here's the problem: if you set your quantization <em>max</em> to fit those giant outliers, all the normal weights get squeezed into just a few pegs near zero, and lose almost all their detail. One bully weight ruins the grid for everyone.</figcaption>
</figure>

The clever fixes are why modern quantization is so good. Techniques like **LLM.int8()** keep just those rare outlier weights in full precision while quantizing the other 99.9% aggressively, a mixed-precision compromise. Others, like **AWQ**, analyze which weights actually matter most to the output (only about 1% are "salient") and protect *those* while squeezing the rest. And the popular **K-quant** scheme (the "K" in `Q4_K_M`) uses smart block structures with shared meta-constants so even the scale factors are stored efficiently. This is why a well-made 4-bit model is so much better than a crude one: it's not quantizing everything blindly, it's protecting what matters.

## Decoding the cryptic names

Now those mysterious model filenames make sense. When you see `Q4_K_M`, you can read it:

<figure class="qz-fig">
<table class="qz-tab">
<thead><tr><th>Piece</th><th>Means</th></tr></thead>
<tbody>
<tr><td><code>Q4</code></td><td>Quantized to about 4 bits per weight</td></tr>
<tr><td><code>K</code></td><td>Uses the smart "K-quant" block structure (better than the old plain method)</td></tr>
<tr><td><code>M</code></td><td>Medium variant, a size within the family (S = small, M = medium, L = large)</td></tr>
</tbody>
</table>
  <figcaption>"Q4_K_M" = 4-bit, K-quant method, medium size. It's the community's most popular default because it hits the sweet spot: a quarter of the memory, nearly all the quality. Now you can pick a quantization level on purpose instead of guessing.</figcaption>
</figure>

## One honest detail: it unpacks to compute

A subtle point worth knowing so you're not misled. The weights are *stored* in 4 bits to save space, but during the actual math, the computer usually **unpacks them back to full precision** for each calculation (this is called dequantization). So quantization mainly saves **memory and loading time**, not necessarily the raw math speed, though smaller data moving around does often make things faster too. The headline win is the one that lets it run at all: it fits.

## The full picture

<figure class="qz-fig">
<table class="qz-tab">
<thead><tr><th>Level</th><th>Size (8B model)</th><th>Quality kept</th><th>Use when</th></tr></thead>
<tbody>
<tr><td>FP16</td><td>~16 GB</td><td>100%</td><td>You have the hardware and want max fidelity</td></tr>
<tr><td>Q8</td><td>~8.5 GB</td><td>~99%+</td><td>Plenty of memory, want near-perfect</td></tr>
<tr><td>Q4_K_M</td><td>~5 GB</td><td>~97-99%</td><td>The default. Best balance for most laptops</td></tr>
<tr><td>Q3</td><td>~4 GB</td><td>~90%</td><td>Tight on memory, can tolerate some slip</td></tr>
<tr><td>Q2</td><td>~3 GB</td><td>drops sharply</td><td>Rarely worth it, quality falls off</td></tr>
</tbody>
</table>
  <figcaption>The whole tradeoff on one card. The rule of thumb: start at Q4_K_M. Go up to Q8 if you have the memory and want the last 1 to 2%. Only drop to Q3 if you're squeezed. Q2 is usually a false economy. Bigger models tolerate aggressive quantization better, so a 70B at 4-bit is safer than an 8B at 4-bit.</figcaption>
</figure>

## The takeaway

The reason a model that "shouldn't fit" runs on your laptop isn't magic, it's a chain of clever, understandable ideas. A weight is just a number in bits. You don't need all those bits, most of the precision was never doing real work, exactly like the detail a JPEG throws away. So you snap each weight to the nearest peg on a grid, store just the peg plus one shared scale factor per block, and reverse it when needed. The rare outlier weights that actually matter get protected. The result: a quarter of the size, nearly all of the intelligence.

The next time you watch a genuinely capable AI answer you with your Wi-Fi off, on a laptop that has no business running something that large, you'll know the trick underneath. Billions of numbers, each quietly rounded to the nearest peg, small enough to fit, sharp enough to still think. That's quantization, and it's one of the quiet pieces of engineering that put real AI in everyone's hands.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['ladder','map','acc','prob','work','jpeg','out'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
