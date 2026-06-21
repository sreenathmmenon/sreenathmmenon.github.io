---
title: "How LLMs Actually Work"
date: 2026-06-20
excerpt: "No math, no jargon walls. Just a plain, honest walk through what is really happening when you type a question into ChatGPT and words come back."
tags: [llm, ai, machine-learning, explainer, beginners]
---

<style>
/* visuals scoped to this post only */
.post-fig{margin:2.5rem 0;}
.post-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* phone keyboard demo */
.kbd-demo{max-width:340px;margin:0 auto;background:var(--surface);border:1px solid var(--border-2);border-radius:14px;padding:1.1rem;box-shadow:var(--glow);}
.kbd-typed{font-size:1.1rem;color:var(--text);padding:.4rem .2rem .9rem;border-bottom:1px solid var(--border);}
.kbd-cursor{display:inline-block;width:2px;height:1.1em;background:var(--accent);vertical-align:-2px;animation:kbd-blink 1.05s steps(1) infinite;}
@keyframes kbd-blink{50%{opacity:0;}}
.kbd-suggest{display:flex;gap:.5rem;padding-top:.9rem;justify-content:center;}
.kbd-chip{font-size:.85rem;padding:.35rem .8rem;border-radius:999px;background:var(--surface-2);border:1px solid var(--border);color:var(--text-2);}
.kbd-best{background:var(--grad);color:var(--accent-ink);border-color:transparent;font-weight:600;animation:kbd-pop 2.4s ease-in-out infinite;}
@keyframes kbd-pop{0%,70%,100%{transform:scale(1);}80%{transform:scale(1.08);}}

/* predict loop */
.loop-demo{display:flex;flex-direction:column;gap:.55rem;max-width:560px;margin:0 auto;}
.loop-row{display:flex;align-items:center;gap:.7rem;font-family:var(--font-mono);font-size:.82rem;background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:.6rem .8rem;}
.loop-in{flex:1;color:var(--text-2);}
.loop-arrow{color:var(--text-3);}
.loop-out{color:var(--accent);font-weight:600;background:var(--surface-2);border-radius:6px;padding:.1rem .5rem;}
.loop-dim{opacity:.55;}

/* temperature dial (static illustration) */
.dial-demo{display:flex;flex-direction:column;gap:.7rem;max-width:600px;margin:0 auto;}
.dial-row{display:flex;align-items:center;gap:.9rem;}
.dial-val{font-family:var(--font-mono);font-weight:600;color:var(--accent);width:2.4rem;flex:none;}
.dial-bar{flex:1;display:flex;gap:.4rem;flex-wrap:wrap;}
.dial-bar span{font-size:.82rem;padding:.3rem .6rem;border-radius:7px;background:var(--surface-2);border:1px solid var(--border);color:var(--text-2);}
.dial-bar .dial-pick{background:var(--grad);color:var(--accent-ink);border-color:transparent;font-weight:600;}
.dial-tag{font-size:.78rem;color:var(--text-3);flex:none;width:9.5rem;text-align:right;}

/* INTERACTIVE temperature playground */
.temp-play{max-width:600px;margin:0 auto;background:var(--surface);border:1px solid var(--border-2);border-radius:16px;padding:1.4rem 1.4rem 1.6rem;box-shadow:var(--glow);}
.temp-prompt{font-family:var(--font-mono);font-size:.85rem;color:var(--text-2);margin-bottom:1.1rem;}
.temp-prompt b{color:var(--accent);}
.temp-slider-wrap{display:flex;align-items:center;gap:1rem;margin-bottom:.4rem;}
.temp-slider-wrap input[type=range]{flex:1;accent-color:var(--accent);height:6px;cursor:pointer;}
.temp-readout{font-family:var(--font-mono);font-weight:700;font-size:1.2rem;color:var(--accent);width:3.2rem;text-align:right;}
.temp-scale{display:flex;justify-content:space-between;font-family:var(--font-mono);font-size:.7rem;color:var(--text-3);margin-bottom:1.2rem;}
.temp-mood{font-family:var(--font-mono);font-size:.78rem;letter-spacing:.06em;text-transform:uppercase;color:var(--accent);margin-bottom:.7rem;}
.temp-answer{background:var(--bg-soft);border:1px solid var(--border);border-radius:10px;padding:1rem 1.1rem;min-height:3.4rem;color:var(--text);line-height:1.6;font-size:1rem;transition:opacity .15s var(--ease);}
.temp-answer .swap{color:var(--accent);font-weight:600;}
.temp-hint{font-size:.8rem;color:var(--text-3);margin-top:.8rem;text-align:center;}

/* training steps */
.train-demo{display:flex;gap:.6rem;max-width:620px;margin:0 auto;flex-wrap:wrap;justify-content:center;}
.train-step{flex:1;min-width:130px;background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:.9rem;text-align:center;}
.train-step b{display:block;font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.08em;color:var(--accent);margin-bottom:.45rem;}
.train-step span{font-size:.85rem;color:var(--text-2);}
.train-tune{border-color:var(--accent);box-shadow:var(--glow);}

/* tokens: alternate a clearly tinted chip vs an accent-outlined chip so the
   chunks stay distinct in BOTH themes (no white-on-white in light) */
.token-demo{display:flex;flex-wrap:wrap;gap:.4rem;justify-content:center;}
.tok{font-family:var(--font-mono);font-size:.95rem;padding:.35rem .6rem;border-radius:7px;background:var(--surface-2);border:1px solid var(--border-2);color:var(--text-2);}
.tok:nth-child(odd){background:transparent;border-color:var(--accent);color:var(--accent);font-weight:600;}

/* flow */
.flow-demo{list-style:none;padding:0;max-width:560px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.flow-demo li{display:flex;align-items:center;gap:.9rem;background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:.7rem .9rem;color:var(--text-2);font-size:.95rem;}
.flow-n{flex:none;width:28px;height:28px;border-radius:50%;background:var(--grad);color:var(--accent-ink);font-family:var(--font-mono);font-weight:600;font-size:.85rem;display:flex;align-items:center;justify-content:center;}

@media (prefers-reduced-motion: reduce){
  .kbd-cursor,.kbd-best{animation:none;}
}
</style>

The first time I really understood how a language model works, I felt a little cheated.

Not because it was complicated. Because it was *simpler* than I expected. I had built a tool that switches between a dozen of these models. I had shipped AI features used by real people. And the actual core idea, the thing underneath all of it, fits in one sentence.

So let me give you that sentence up front, and then spend the rest of this post earning it.

> A large language model is a machine that, given some text, guesses the next word. That is all it does. Everything else is a consequence of doing that one thing extremely well.

That is it. It does not think. It does not look anything up. It does not understand you the way your friend does. It guesses the next word, then the next, then the next, faster than you can read, and somehow that turns into poetry, code, and answers to questions you did not know how to ask.

Sounds too small to be true. Let me show you why it works.

## Start with something you already trust

You have used this technology for years. Your phone keyboard.

You type "I'll be there in five" and it offers "minutes." You type "happy" and it offers "birthday." It is reading what you wrote and betting on what comes next.

<figure class="post-fig">
  <div class="kbd-demo" aria-hidden="true">
    <div class="kbd-typed">See you in five&nbsp;<span class="kbd-cursor"></span></div>
    <div class="kbd-suggest">
      <span class="kbd-chip kbd-best">minutes</span>
      <span class="kbd-chip">days</span>
      <span class="kbd-chip">years</span>
    </div>
  </div>
  <figcaption>Your phone already does a tiny version of this. It looks at your words and ranks the likely next ones.</figcaption>
</figure>

Your keyboard is bad at it. It only looks at the last word or two, so it has no idea what you are actually talking about. Ask it anything real and it falls apart.

A large language model is the same idea, taken to an absurd extreme. Instead of the last word or two, it looks at everything you wrote, plus everything before that, sometimes a whole book's worth of context. And instead of learning from your texts alone, it learned from a huge slice of everything humans have written down.

Same instinct. Wildly different scale. That gap is the whole story.

## The only trick: predict the next piece, then do it again

Here is the loop, and it really is this plain.

You give the model some words. It produces one piece of a word. It sticks that piece onto the end of your words. Then it feeds the whole thing back into itself and produces the next piece. Round and round, until it decides it is done.

<figure class="post-fig">
  <div class="loop-demo">
    <div class="loop-row"><span class="loop-in">The capital of France is</span><span class="loop-arrow">&rarr;</span><span class="loop-out">Paris</span></div>
    <div class="loop-row"><span class="loop-in">The capital of France is Paris</span><span class="loop-arrow">&rarr;</span><span class="loop-out">.</span></div>
    <div class="loop-row"><span class="loop-in">The capital of France is Paris.</span><span class="loop-arrow">&rarr;</span><span class="loop-out">It</span></div>
    <div class="loop-row loop-dim"><span class="loop-in">The capital of France is Paris. It</span><span class="loop-arrow">&rarr;</span><span class="loop-out">is</span></div>
  </div>
  <figcaption>One word out, glued back on, fed in again. The model never sees the future. It only ever guesses the very next step.</figcaption>
</figure>

That word "guess" is doing a lot of work, so let me be honest about it. The model does not pick one word. For every possible next word, it produces a number: how likely this word is, right here, right now. Thousands of words, each with a score. Then it usually picks from the top of that list.

This is why the same question can give you slightly different answers. There is a dial, often called temperature, that decides how adventurous the picking is. It is a real setting you can change when you call these models through their API, and it has a real number attached to it.

Here is the cleanest way I have found to picture it. Say you ask the model to finish "My favourite drink in the morning is." Behind the scenes it has scored every possible next word. "Coffee" sits near the top. "Tea" is right behind it. "Mango" is way down the list but not impossible. Temperature decides how far down that list the model is willing to wander.

<figure class="post-fig">
  <div class="dial-demo">
    <div class="dial-row">
      <span class="dial-val">0.0</span>
      <div class="dial-bar"><span class="dial-pick">coffee</span></div>
      <span class="dial-tag">focused, repeatable</span>
    </div>
    <div class="dial-row">
      <span class="dial-val">0.7</span>
      <div class="dial-bar"><span class="dial-pick">coffee</span><span>tea</span><span>chai</span></div>
      <span class="dial-tag">balanced, a little variety</span>
    </div>
    <div class="dial-row">
      <span class="dial-val">1.0</span>
      <div class="dial-bar"><span class="dial-pick">coffee</span><span>tea</span><span>chai</span><span>juice</span><span>mango lassi</span></div>
      <span class="dial-tag">creative, more surprising</span>
    </div>
  </div>
  <figcaption>Same prompt, same model, one dial. Low temperature keeps picking the safe favourite. Higher temperature lets less likely words into the running.</figcaption>
</figure>

Reading about it only gets you so far. Drag the slider below and watch what happens to the same question as you turn the heat up. Try the extremes. That feeling, of the answer going from reliable to playful to slightly off the rails, is the whole point.

<figure class="post-fig">
  <div class="temp-play">
    <div class="temp-prompt">prompt: <b>"Write me a one-line tagline for a coffee shop."</b></div>
    <div class="temp-slider-wrap">
      <input type="range" id="tempRange" min="0" max="20" value="7" aria-label="Temperature slider from 0 to 2">
      <span class="temp-readout" id="tempVal">0.7</span>
    </div>
    <div class="temp-scale"><span>0.0 cold</span><span>1.0</span><span>2.0 hot</span></div>
    <div class="temp-mood" id="tempMood">balanced</div>
    <div class="temp-answer" id="tempAnswer"></div>
    <div class="temp-hint">Same prompt every time. Only the temperature changed.</div>
  </div>
  <figcaption>A simulation of the effect, not a live model, but the behaviour is real: low temperature repeats the safe line, high temperature reaches for something stranger (and sometimes worse).</figcaption>
</figure>

<script>
(function(){
  var range = document.getElementById('tempRange');
  if(!range) return;
  var val = document.getElementById('tempVal');
  var mood = document.getElementById('tempAnswer');
  var moodLabel = document.getElementById('tempMood');
  var answer = document.getElementById('tempAnswer');
  var buckets = [
    { upTo:0.3, mood:'focused and repeatable', lines:[
      'Good coffee. Every day.' ]},
    { upTo:0.8, mood:'balanced', lines:[
      'Good coffee, made with care.',
      'Your daily cup, done right.',
      'Fresh coffee, friendly faces.' ]},
    { upTo:1.3, mood:'creative', lines:[
      'Mornings taste better here.',
      'Brewed slow, served warm.',
      'Where the day quietly begins.',
      'A small cup of calm.' ]},
    { upTo:2.0, mood:'wild, sometimes off', lines:[
      'Caffeine and tiny revolutions.',
      'Bean there, loved that.',
      'Espresso yourself, loudly.',
      'Liquid courage for soft mornings.',
      'We put the OW in cup of joe (sorry).' ]}
  ];
  function pick(t){
    for(var i=0;i<buckets.length;i++){
      if(t <= buckets[i].upTo){
        var lines = buckets[i].lines;
        // higher temperature = pull from anywhere in the bucket; low = first line
        var idx = t < 0.3 ? 0 : Math.floor(Math.random()*lines.length);
        return { mood: buckets[i].mood, text: lines[idx] };
      }
    }
    return { mood: buckets[3].mood, text: buckets[3].lines[0] };
  }
  function render(){
    var t = range.value/10;
    val.textContent = t.toFixed(1);
    var r = pick(t);
    moodLabel.textContent = r.mood;
    answer.style.opacity = 0;
    setTimeout(function(){ answer.innerHTML = '<span class="swap">"</span>'+r.text+'<span class="swap">"</span>'; answer.style.opacity = 1; }, 90);
  }
  range.addEventListener('input', render);
  render();
})();
</script>

So what are the actual numbers? Here is where it gets interesting, because they are <b>not the same everywhere</b>. This trips a lot of people up.

With OpenAI's API, temperature runs from <b>0 to 2</b> and defaults to <b>1</b>. With Anthropic's Claude API, it runs from <b>0 to 1</b>, also defaulting to <b>1</b>. So setting temperature to 1 does not mean the same thing on both. On Claude that is the top of the range, the wildest it goes. On OpenAI that is only the halfway point. Same number, different meaning. This is exactly the kind of small, real headache you hit when you build tools that talk to more than one provider, and it is a big part of why I built llmswap in the first place.

And it keeps moving. OpenAI's newer GPT-5 models removed the temperature knob altogether and just fix it internally. So the lesson is not "memorise the numbers." The lesson is: this is a setting that varies by provider and even by model, so when an answer feels too random or too robotic, the first thing to check is which dial you are actually turning.

A rough field guide I actually use:

- Around <b>0 to 0.3</b>: facts, code, data extraction, anything where you want the same answer twice. Boring on purpose.
- Around <b>0.7</b>: a good everyday middle. Helpful, with a little life to it. This is close to many defaults for a reason.
- Around <b>1 and above</b>: brainstorming, names, jokes, first drafts. You are asking for surprise, and you will get some duds along with the gems.

One honest catch, straight from Anthropic's own docs: even at temperature 0, the output is <b>not</b> guaranteed to be identical every single time. Lower temperature makes it far more consistent, but "deterministic" is a promise these systems quietly do not make. Good to know before you build something that assumes otherwise.

## But how does it know "Paris"?

Fair. Prediction is the loop. The real magic is *why* the prediction is any good. And the answer is the part that makes people uneasy, so I will say it plainly.

It read almost everything.

Books, websites, code, arguments on forums, recipes, instruction manuals, poems. An amount of text no human could finish in a thousand lifetimes. And during a long, expensive process called training, it played one game, billions of times: cover up the next word, guess it, check the real answer, adjust.

Imagine being handed every sentence ever written, with the last word hidden, and being asked to guess it. At first you would be hopeless. But do that enough times and you would start to notice patterns. "The capital of France is" tends to be followed by "Paris." "Once upon a" tends to be followed by "time." Code that opens a bracket tends to close it.

The model does not store these as facts in a list. It tunes billions of tiny internal dials so that the patterns are baked into how it reacts. When you later type "The capital of France is," the right answer lights up not because it looked it up, but because that path is worn smooth from a billion passes.

<figure class="post-fig">
  <div class="train-demo">
    <div class="train-step"><b>hide</b><span>Once upon a ___</span></div>
    <div class="train-step"><b>guess</b><span>"castle"? "dog"? "time"?</span></div>
    <div class="train-step"><b>check</b><span>real word was "time"</span></div>
    <div class="train-step train-tune"><b>adjust</b><span>nudge the dials, try again</span></div>
  </div>
  <figcaption>Training in four words: hide, guess, check, adjust. Repeat until the guesses stop being terrible. Then repeat a few billion more times.</figcaption>
</figure>

This is also why a model can be confidently wrong. It is not reciting truth. It is producing the most plausible continuation. Most of the time plausible and true line up. Sometimes they do not, and you get a sentence that sounds perfect and means nothing. People call this hallucination. I think of it as the model doing exactly its job, predicting believable text, with no separate sense of whether the text is real. It was never taught the difference. It was taught what sounds right.

Sit with that for a second, because it changes how you should trust these things. The fluency is not evidence of correctness. It is just evidence that the model has read a lot of fluent writing.

## Words are not really words

One more layer, and then you will understand more about this than most people who use it daily.

The model does not see letters or even whole words. Before anything happens, your text gets chopped into pieces called tokens. A token is usually a chunk of a word. "Running" might be "run" plus "ning." A space is part of a token. Common words are single tokens. Rare ones get split into several.

<figure class="post-fig">
  <div class="token-demo" aria-hidden="true">
    <span class="tok">I</span><span class="tok"> love</span><span class="tok"> build</span><span class="tok">ing</span><span class="tok"> tools</span><span class="tok">.</span>
  </div>
  <figcaption>One short sentence, six tokens. The model thinks in these chunks, not in letters or whole words. This is why it sometimes miscounts letters in a word: it never saw the letters.</figcaption>
</figure>

Each token gets turned into a long list of numbers. That sounds cold, but here is the beautiful part. These numbers are arranged so that tokens with related meanings end up near each other. "King" sits close to "queen." "Paris" sits close to "France." The model builds a kind of map of meaning, where direction and distance carry sense, all on their own, learned only from reading.

So when it predicts the next token, it is doing arithmetic on points in this map of meaning. It is not magic. It is geometry with a very good map.

## Putting the whole thing together

Let me stitch the pieces into one picture, start to finish, the moment you hit enter.

<figure class="post-fig">
  <ol class="flow-demo">
    <li><span class="flow-n">1</span><span>Your sentence gets chopped into tokens.</span></li>
    <li><span class="flow-n">2</span><span>Each token becomes numbers on the map of meaning.</span></li>
    <li><span class="flow-n">3</span><span>The model weighs everything you wrote and scores every possible next token.</span></li>
    <li><span class="flow-n">4</span><span>It picks one, based on the scores and the temperature dial.</span></li>
    <li><span class="flow-n">5</span><span>That token gets added on, and the whole thing loops back to step 3.</span></li>
    <li><span class="flow-n">6</span><span>It stops when it predicts a quiet "I am done" signal.</span></li>
  </ol>
  <figcaption>Six steps, on repeat, a few hundred times a second. That is the answer scrolling onto your screen.</figcaption>
</figure>

That is a language model. A next-word guesser that read almost everything, thinks in chunks, navigates a map of meaning, and runs its guess in a loop until it has said its piece.

## Why I find this comforting, not scary

When people hear "it just predicts the next word," some feel let down, like the trick has been spoiled. I felt the opposite. Knowing the shape of the thing made me trust it correctly. Not too much, not too little.

It is a phenomenal pattern matcher. Lean on it for that. Drafting, explaining, translating, turning your messy thought into clean words, sketching code, talking you through an idea at 1am when no one else is awake. It is genuinely good at all of it.

But it has no memory of being right, no inner check for truth, no stake in the answer. So when it matters, you check its work. You stay the human in the loop. That is not a limitation to resent. That is just knowing your tool.

I have spent years building on top of these models. The wonder has not worn off. If anything, understanding the simple engine underneath made it feel more impressive, not less, that something so plain, repeated at such scale, can sound so much like us.

It guesses the next word. And in doing that, billions of times over, it learned to keep us company.
