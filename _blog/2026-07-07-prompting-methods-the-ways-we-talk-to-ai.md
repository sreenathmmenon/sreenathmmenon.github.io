---
title: "Prompting Methods: The Different Ways We Talk to AI"
date: 2026-07-07
excerpt: "The exact same model can give you a mediocre answer or a brilliant one, depending only on how you asked. Over the last few years people worked out a real toolkit of prompting methods: zero-shot, few-shot, chain-of-thought, self-consistency, ReAct, and more. Here is what each one is, when to reach for it, and the honest 2026 twist: why reasoning models quietly changed the rules."
tags: [ai, prompting, prompt-engineering, chain-of-thought, llm, explainer]
---

<style>
.pm-fig{margin:2.5rem 0;}
.pm-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* same model, different ask */
.pm-same{max-width:560px;margin:0 auto;display:flex;flex-direction:column;gap:.6rem;}
.pm-same .row{display:flex;gap:.7rem;align-items:stretch;}
.pm-same .ask{flex:none;width:120px;font-family:var(--font-mono);font-size:.74rem;color:var(--text-2);display:flex;align-items:center;}
.pm-same .out{flex:1;border:1px solid var(--border);border-radius:8px;padding:.5rem .7rem;font-size:.83rem;color:var(--text-2);background:var(--surface);}
.pm-same .out.bad{border-color:var(--border-2);} .pm-same .out.bad .m{color:var(--text-3);}
.pm-same .out.good{border-color:var(--accent);} .pm-same .out.good .m{color:var(--accent);}

/* zero vs few shot */
.pm-shot{display:grid;grid-template-columns:1fr 1fr;gap:.8rem;max-width:640px;margin:0 auto;}
@media(max-width:560px){.pm-shot{grid-template-columns:1fr;}}
.pm-shot .c{border:1px solid var(--border-2);border-radius:12px;padding:0;background:var(--surface);overflow:hidden;}
.pm-shot .c .hd{padding:.5rem .8rem;background:var(--surface-2);border-bottom:1px solid var(--border);font-family:var(--font-mono);font-size:.76rem;color:var(--text);}
.pm-shot .c.few .hd{color:var(--accent);}
.pm-shot .c .bd{padding:.7rem .8rem;font-family:var(--font-mono);font-size:.76rem;line-height:1.7;color:var(--text-2);white-space:pre-wrap;}
.pm-shot .c .bd .ex{color:var(--accent);}
.pm-shot .c .bd .q{color:var(--text);}

/* chain of thought reveal */
.pm-cot{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.pm-cot .line{font-family:var(--font-mono);font-size:.82rem;padding:.5rem .8rem;border-radius:8px;border:1px solid var(--border);background:var(--surface);opacity:0;transform:translateX(-6px);transition:opacity .45s ease,transform .45s ease;}
.pm-cot.go .line{opacity:1;transform:none;}
.pm-cot.go .line:nth-child(1){transition-delay:.1s}
.pm-cot.go .line:nth-child(2){transition-delay:.5s}
.pm-cot.go .line:nth-child(3){transition-delay:.9s}
.pm-cot.go .line:nth-child(4){transition-delay:1.3s}
.pm-cot.go .line:nth-child(5){transition-delay:1.7s}
.pm-cot .line.think{color:var(--text-2);}
.pm-cot .line.ans{border-color:var(--accent);color:var(--accent);}
.pm-cot .line .lbl{color:var(--text-3);}

/* accuracy jump bars */
.pm-bars{max-width:580px;margin:0 auto;display:flex;flex-direction:column;gap:.9rem;}
.pm-bars .row .lab{display:flex;justify-content:space-between;font-family:var(--font-mono);font-size:.78rem;margin-bottom:.3rem;}
.pm-bars .row .lab b{color:var(--text);} .pm-bars .row .lab span{color:var(--accent);}
.pm-bars .track{height:18px;border-radius:6px;background:var(--surface-2);border:1px solid var(--border);overflow:hidden;}
.pm-bars .fill{height:100%;width:0;transition:width 1.1s cubic-bezier(.2,.7,.2,1);}
.pm-bars.go .fill{width:var(--w);}
.pm-bars .fill.low{background:var(--surface-2);border-right:2px solid var(--text-3);}
.pm-bars .fill.high{background:var(--grad);}

/* self-consistency vote */
.pm-vote{max-width:600px;margin:0 auto;}
.pm-vote .paths{display:flex;flex-wrap:wrap;gap:.5rem;justify-content:center;margin-bottom:.8rem;}
.pm-vote .p{font-family:var(--font-mono);font-size:.76rem;padding:.4rem .7rem;border-radius:8px;border:1px solid var(--border-2);background:var(--surface);color:var(--text-2);opacity:0;transform:scale(.9);transition:opacity .35s ease,transform .35s ease;}
.pm-vote.go .p{opacity:1;transform:none;}
.pm-vote.go .p:nth-child(1){transition-delay:.1s} .pm-vote.go .p:nth-child(2){transition-delay:.25s}
.pm-vote.go .p:nth-child(3){transition-delay:.4s} .pm-vote.go .p:nth-child(4){transition-delay:.55s}
.pm-vote.go .p:nth-child(5){transition-delay:.7s}
.pm-vote .p.win{border-color:var(--accent);color:var(--accent);font-weight:600;}
.pm-vote .res{text-align:center;font-family:var(--font-mono);font-size:.8rem;color:var(--accent);opacity:0;transition:opacity .4s ease 1s;}
.pm-vote.go .res{opacity:1;}

/* method ladder / decision */
.pm-ladder{max-width:620px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.pm-rung{display:flex;align-items:center;gap:.9rem;border:1px solid var(--border);border-radius:10px;padding:.6rem .9rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.pm-ladder.go .pm-rung{opacity:1;transform:none;}
.pm-ladder.go .pm-rung:nth-child(1){transition-delay:.1s} .pm-ladder.go .pm-rung:nth-child(2){transition-delay:.3s}
.pm-ladder.go .pm-rung:nth-child(3){transition-delay:.5s} .pm-ladder.go .pm-rung:nth-child(4){transition-delay:.7s}
.pm-ladder.go .pm-rung:nth-child(5){transition-delay:.9s}
.pm-rung .nm{flex:none;width:130px;font-family:var(--font-mono);font-size:.78rem;color:var(--accent);font-weight:600;}
.pm-rung .ds{font-size:.85rem;color:var(--text-2);} .pm-rung .ds b{color:var(--text);}

/* table */
.pm-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.pm-tab th,.pm-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.pm-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.pm-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}

@media (prefers-reduced-motion: reduce){
  .pm-cot .line,.pm-bars .fill,.pm-vote .p,.pm-vote .res,.pm-ladder .pm-rung{transition:none !important;}
  .pm-cot .line{opacity:1 !important;transform:none !important;}
  .pm-bars .fill{width:var(--w) !important;}
  .pm-vote .p{opacity:1 !important;transform:none !important;} .pm-vote .res{opacity:1 !important;}
  .pm-ladder .pm-rung{opacity:1 !important;transform:none !important;}
}
</style>

Here's something a little unfair about AI: the exact same model, on the exact same question, will hand one person a lazy, wrong answer and hand another person a careful, correct one. Nothing changed about the model. The only difference was *how they asked*.

That gap is what "prompting" is about, and over the last few years it stopped being guesswork and became a real, named toolkit. There's a method for when you have no examples, one for when you have a few, one that makes the model *show its work* and suddenly get much smarter, one that asks the model the same thing several times and takes a vote, and one that lets it reason *and* act. Each has a name, an origin, and a moment when it's the right tool.

Let me walk you through them, from the simplest to the cleverest, with real examples. I'll keep it plain enough for a first-timer, and I'll end with the honest 2026 twist: the newest models quietly changed which of these you still need to do by hand.

## The whole point, in one picture

<figure class="pm-fig">
  <div class="pm-same">
    <div class="row">
      <div class="ask">Lazy ask</div>
      <div class="out bad">"Is 17 x 24 more than 400?" → <span class="m">"No."</span> (wrong, it guessed)</div>
    </div>
    <div class="row">
      <div class="ask">Careful ask</div>
      <div class="out good">"...think step by step." → <span class="m">"17 x 24 = 408, so yes."</span> (right)</div>
    </div>
  </div>
  <figcaption>Same model, same question, two different asks, two different answers. The careful version didn't make the model smarter; it gave the model room to actually reason instead of blurting a guess. Every prompting method below is a different way of shaping that room.</figcaption>
</figure>

## 1. Zero-shot: just ask

The simplest method, and the one you use without thinking. **Zero-shot** means you give the model a task with *no examples at all* and rely on everything it learned during training to figure out what you want. "Translate this to French." "Is this review positive or negative?" No demonstrations, just the instruction.

It works remarkably often, because a big model has seen so much during training that most everyday tasks are already familiar. Zero-shot is your default: quick, clean, no setup. You only reach for something fancier when zero-shot stumbles.

## 2. Few-shot: show, don't just tell

Sometimes an instruction isn't enough, the model needs to *see* the pattern you want. **Few-shot** prompting gives it a handful of examples right inside the prompt, then asks it to continue the pattern. The remarkable part is that the model learns the pattern *on the spot*, without any retraining. This is called in-context learning, and it's one of the genuinely surprising abilities of large models.

<figure class="pm-fig">
  <div class="pm-shot">
    <div class="c">
      <div class="hd">Zero-shot (no examples)</div>
      <div class="bd"><span class="q">Classify: "meh, it was fine."</span>
→ ?  (model has to guess your
   label style and format)</div>
    </div>
    <div class="c few">
      <div class="hd">Few-shot (a few examples)</div>
      <div class="bd"><span class="ex">"Loved it!" → positive</span>
<span class="ex">"Waste of money" → negative</span>
<span class="ex">"It's okay" → neutral</span>
<span class="q">"meh, it was fine." →</span> neutral</div>
    </div>
  </div>
  <figcaption>The examples do two jobs: they teach the <em>task</em> ("sort into positive/negative/neutral") and they lock in the <em>format</em> (a single lowercase word). Few-shot shines when you need a specific output shape, or the task is niche enough that a plain instruction leaves the model guessing. Even one to three examples usually does it.</figcaption>
</figure>

## 3. Chain-of-thought: make it show its work

This is the one that changed everything, and it came from a 2022 paper by Jason Wei and colleagues at Google. The insight is almost childishly simple: **if you want the model to reason correctly, make it reason out loud before answering.** Instead of jumping straight to the answer, it writes out the intermediate steps, exactly like a student showing their working in a math exam.

<figure class="pm-fig">
  <div class="pm-cot" id="cot">
    <div class="line think"><span class="lbl">Q:</span> A shop has 23 apples, sells 8, then gets 12 more. How many now?</div>
    <div class="line think"><span class="lbl">step:</span> Start with 23.</div>
    <div class="line think"><span class="lbl">step:</span> Sell 8 → 23 − 8 = 15.</div>
    <div class="line think"><span class="lbl">step:</span> Get 12 more → 15 + 12 = 27.</div>
    <div class="line ans"><span class="lbl">Answer:</span> 27.</div>
  </div>
  <figcaption>Chain-of-thought in action. By walking through the steps, the model catches itself where it would otherwise slip. The original paper showed this was dramatic: on a hard grade-school math benchmark (GSM8K), a large model given just eight worked examples with reasoning beat even a fine-tuned model. Accuracy on some reasoning tasks more than doubled.</figcaption>
</figure>

The results were striking enough to see as bars. Same model, same questions, the only change is whether it was asked to reason step by step first:

<figure class="pm-fig">
  <div class="pm-bars" id="bars">
    <div class="row"><div class="lab"><b>Answer straight away</b><span>guesses</span></div><div class="track"><div class="fill low" style="--w:33%"></div></div></div>
    <div class="row"><div class="lab"><b>Chain-of-thought (reason first)</b><span>much higher</span></div><div class="track"><div class="fill high" style="--w:80%"></div></div></div>
  </div>
  <figcaption>Illustrative of the paper's shape (exact numbers vary by task and model): giving the model room to reason step by step produced large jumps on multi-step reasoning. And here's the kicker the researchers found, this reasoning ability only really emerges in <em>large</em> models. Small models don't get the same boost. It's an ability that appears with scale.</figcaption>
</figure>

There's a lovely shortcut version too. You don't always need examples, sometimes just adding the words **"Let's think step by step"** to a plain zero-shot prompt is enough to switch on that same careful reasoning. A five-word phrase, measurable gains. That's zero-shot chain-of-thought.

## 4. Self-consistency: ask several times, take a vote

Chain-of-thought has one weakness: it produces *one* line of reasoning, and if that one path takes a wrong turn early, the whole answer is wrong. **Self-consistency** (from Wang and colleagues, 2022) fixes this with a beautifully simple idea borrowed from how humans check hard problems: **solve it several different ways, then go with the answer you land on most often.**

<figure class="pm-fig">
  <div class="pm-vote" id="vote">
    <div class="paths">
      <span class="p win">path A → 27</span>
      <span class="p">path B → 27</span>
      <span class="p">path C → 25</span>
      <span class="p win">path D → 27</span>
      <span class="p win">path E → 27</span>
    </div>
    <div class="res">majority vote → 27 ✓ (one bad path outvoted)</div>
  </div>
  <figcaption>Run the same chain-of-thought prompt several times (models are a little random each run), collect the different reasoning paths, and take the majority answer. Four paths said 27, one slipped to 25, so 27 wins and the mistake is outvoted. This adds a meaningful accuracy bump on top of plain chain-of-thought, with no retraining at all, just more attempts and a vote.</figcaption>
</figure>

The cost is obvious: you're paying to run the prompt several times instead of once. So you save self-consistency for problems where getting it *right* matters more than getting it *cheap*.

## 5. Role and system prompting: set the stage

A different lever entirely: instead of shaping the *reasoning*, you shape the *persona and rules*. **Role prompting** tells the model who to be ("You are a meticulous security reviewer"), which quietly pulls its answers toward that frame. **System prompting** sets the standing rules for the whole conversation, the constraints, the tone, the format, before the user even speaks. These are less about cleverness and more about *framing*: give the model a clear identity and clear boundaries, and its answers get more consistent and on-target.

## 6. ReAct: reason and act

The most advanced one, and if you read my post on how agents work, you already met it. **ReAct** (Yao and colleagues, 2022) interleaves *reasoning* with *actions*, the model thinks a step, then actually *does* something (searches, runs code, calls an API), sees the real result, and reasons again. It's the bridge from "a model that talks" to "an agent that acts," and in 2026 nearly every AI agent runs on a ReAct-style loop underneath. I won't re-teach it here; just know it belongs on this list as the method that turns prompting into *doing*.

## Putting it together: which method, when

Here's the whole toolkit as a ladder, from reach-for-it-first to save-it-for-when-you-need-it:

<figure class="pm-fig">
  <div class="pm-ladder" id="ladder">
    <div class="pm-rung"><div class="nm">Zero-shot</div><div class="ds"><b>Default.</b> Just ask. Most everyday tasks need nothing more.</div></div>
    <div class="pm-rung"><div class="nm">Few-shot</div><div class="ds"><b>Need a specific pattern or format?</b> Show 1 to 3 examples.</div></div>
    <div class="pm-rung"><div class="nm">Chain-of-thought</div><div class="ds"><b>Multi-step reasoning, math, logic?</b> Make it think step by step.</div></div>
    <div class="pm-rung"><div class="nm">Self-consistency</div><div class="ds"><b>Correctness really matters?</b> Run it several times, take the vote.</div></div>
    <div class="pm-rung"><div class="nm">ReAct</div><div class="ds"><b>Needs to use tools and act?</b> Interleave reasoning with actions.</div></div>
  </div>
  <figcaption>Climb the ladder only as far as the task demands. Each rung up costs more effort or money, so don't reach for self-consistency when a plain zero-shot ask would do. Start simple; escalate when the model stumbles.</figcaption>
</figure>

<figure class="pm-fig">
<table class="pm-tab">
<thead><tr><th>Method</th><th>What it does</th><th>Best for</th><th>Cost</th></tr></thead>
<tbody>
<tr><td>Zero-shot</td><td>Ask with no examples</td><td>Common, well-understood tasks</td><td>Lowest</td></tr>
<tr><td>Few-shot</td><td>Show a few examples</td><td>Specific formats, niche tasks</td><td>Low</td></tr>
<tr><td>Chain-of-thought</td><td>Reason step by step</td><td>Math, logic, multi-step problems</td><td>Low to medium</td></tr>
<tr><td>Self-consistency</td><td>Vote over many tries</td><td>High-stakes correctness</td><td>Higher (many runs)</td></tr>
<tr><td>Role / system</td><td>Set persona and rules</td><td>Tone, consistency, constraints</td><td>Lowest</td></tr>
<tr><td>ReAct</td><td>Reason + use tools</td><td>Agents that act in the world</td><td>Varies</td></tr>
</tbody>
</table>
  <figcaption>The full toolkit at a glance. Most real prompts combine a few: a system prompt for the rules, few-shot for the format, chain-of-thought for the hard step. They stack.</figcaption>
</figure>

## The honest 2026 twist

Here's the part most guides won't tell you, and it matters. The newest models changed the game. Claude's extended thinking, GPT-5's reasoning modes, and similar features **do chain-of-thought automatically**, internally, before they answer. You don't have to type "let's think step by step" anymore; the model already does, on its own.

So does that make these methods obsolete? No, but it *reshuffles* them. Two honest shifts:

- **Manual chain-of-thought matters less** on the strongest reasoning models, because they already reason internally. On smaller or older models, it still helps a lot.
- **Few-shot examples are quietly changing job.** Recent research on strong models found that adding worked examples often *doesn't* boost their reasoning anymore, instead, the examples mostly serve to lock in the *output format* you want. The example's job shifted from "teach the model to think" to "show the model exactly how to shape its answer."

The deeper lesson underneath all of it: **prompt engineering became writing clear specs.** State what you want, the constraints, the format, give structure and maybe an example, and let the model's own reasoning do the rest. Structure beats length. A short, precise, well-shaped ask beats a long rambling one every time.

## The takeaway

The same model can be dull or brilliant depending on how you talk to it, and now you know the vocabulary. Zero-shot to just ask. Few-shot to show the pattern. Chain-of-thought to make it reason. Self-consistency to vote away mistakes. Role and system prompts to set the stage. ReAct to let it act. Start at the simplest rung and climb only as high as the problem needs.

And as models keep getting better at reasoning on their own, the skill quietly shifts from *tricking* the model into thinking, toward *clearly telling it what good looks like*. Which, when you step back, is just good communication, the same thing that gets you a good answer from a sharp human colleague. Ask clearly, show what you mean, and give them room to think.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['cot','bars','vote','ladder'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
