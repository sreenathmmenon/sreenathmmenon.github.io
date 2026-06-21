---
title: "What Tokens Actually Are (And Why They Cost You)"
date: 2026-06-21
excerpt: "Tokens are the hidden unit that every AI bill, every context limit, and every weird counting mistake comes down to. Here is what they really are, why providers count them differently, and why a model cannot tell you how many r's are in strawberry."
tags: [llm, tokens, tokenization, ai, explainer]
---

<style>
/* a couple of post-specific bits; the rest uses the shared .viz-* classes */
.post-fig{margin:2.5rem 0;}
.post-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* live tokenizer playground */
.tok-play{max-width:600px;margin:0 auto;background:var(--surface);border:1px solid var(--border-2);border-radius:16px;padding:1.4rem;box-shadow:var(--glow);}
.tok-play textarea{width:100%;box-sizing:border-box;font-family:var(--font-mono);font-size:.95rem;background:var(--bg-soft);color:var(--text);border:1px solid var(--border-2);border-radius:10px;padding:.8rem;resize:vertical;min-height:4.5rem;line-height:1.5;}
.tok-play textarea:focus{outline:none;border-color:var(--accent);}
.tok-out{display:flex;flex-wrap:wrap;gap:.3rem;margin-top:1rem;min-height:1rem;}
.tok-piece{font-family:var(--font-mono);font-size:.9rem;padding:.25rem .45rem;border-radius:6px;white-space:pre;background:var(--surface-2);border:1px solid var(--border-2);color:var(--text);}
.tok-piece:nth-child(odd){border-color:var(--accent);}
.tok-stats{display:flex;gap:1.5rem;margin-top:1.1rem;flex-wrap:wrap;}
.tok-stat{font-family:var(--font-mono);}
.tok-stat b{display:block;font-size:1.5rem;color:var(--accent);line-height:1;}
.tok-stat span{font-size:.74rem;color:var(--text-3);}
.tok-note{font-size:.78rem;color:var(--text-3);margin-top:.9rem;}

/* strawberry */
.berry{display:flex;flex-direction:column;gap:.7rem;max-width:520px;margin:0 auto;}
.berry-line{display:flex;align-items:center;gap:.5rem;flex-wrap:wrap;justify-content:center;}
.berry-tok{font-family:var(--font-mono);font-size:1.05rem;padding:.35rem .6rem;border-radius:8px;background:var(--surface-2);border:1px solid var(--border-2);color:var(--text);}
.berry-hi{border-color:var(--accent);color:var(--accent);font-weight:600;}

/* provider compare bars */
.cmp{display:flex;flex-direction:column;gap:.7rem;max-width:560px;margin:0 auto;}
.cmp-row{display:flex;align-items:center;gap:.9rem;}
.cmp-name{font-family:var(--font-mono);font-size:.85rem;width:5.5rem;flex:none;color:var(--text-2);}
.cmp-track{flex:1;background:var(--surface-2);border-radius:999px;height:26px;overflow:hidden;border:1px solid var(--border);}
.cmp-fill{height:100%;background:var(--grad);display:flex;align-items:center;justify-content:flex-end;padding-right:.6rem;color:var(--accent-ink);font-family:var(--font-mono);font-weight:600;font-size:.8rem;}
</style>

Here is a question that should be easy, and is not.

How many times does the letter "r" appear in the word "strawberry"?

You counted three. A child counts three. But for a long time, if you asked the most advanced AI models on earth, many of them said two. Confidently. The same models that can write working code and explain quantum physics could not count the letters in a fruit.

This is not a bug someone forgot to fix. It is a window into how these models actually read, and it all comes down to one idea that almost nobody outside the field talks about: the token.

Once you understand tokens, three things suddenly make sense. Why your AI bill is shaped the way it is. Why the model has a memory limit. And why it fumbles "strawberry." So let me walk you through it the way I wish someone had walked me through it.

## The model does not read letters. Or words.

When you type a sentence, you see letters grouped into words. The model never sees that. Before your text reaches the model, it gets chopped into pieces called tokens, and a token is usually a chunk of a word, not a whole word and definitely not a single letter.

Try it. Type anything into the box below and watch it break apart.

<figure class="post-fig">
  <div class="tok-play">
    <textarea id="tokInput" aria-label="Type text to see it split into tokens">Tokens are sneaky little things.</textarea>
    <div class="tok-out" id="tokOut" aria-hidden="true"></div>
    <div class="tok-stats">
      <div class="tok-stat"><b id="tokChars">0</b><span>characters</span></div>
      <div class="tok-stat"><b id="tokWords">0</b><span>words</span></div>
      <div class="tok-stat"><b id="tokCount">0</b><span>tokens (approx)</span></div>
    </div>
    <div class="tok-note">This is a close approximation, not a real tokenizer. The exact split differs by model. But the shape of the result, chunks of words plus spaces and punctuation, is exactly how it really works.</div>
  </div>
  <figcaption>Type and watch. Notice that common words stay whole, rare words get split, and the spaces and punctuation become pieces too.</figcaption>
</figure>

<script>
(function(){
  var input = document.getElementById('tokInput');
  if(!input) return;
  var out = document.getElementById('tokOut');
  var cChars = document.getElementById('tokChars');
  var cWords = document.getElementById('tokWords');
  var cCount = document.getElementById('tokCount');

  // a short list of common words / suffixes that real tokenizers tend to keep whole.
  // this is an approximation for teaching, not a real BPE model.
  var whole = ['the','and','are','you','that','this','with','for','was','not','have',
    'token','tokens','little','things','model','word','words','text','they'];
  var suffixes = ['ization','ing','ed','ly','tion','ness','ment','s'];

  function chunkWord(w){
    var lower = w.toLowerCase();
    if(w.length <= 4 || whole.indexOf(lower) !== -1) return [w];
    // peel a known suffix if present
    for(var i=0;i<suffixes.length;i++){
      var s = suffixes[i];
      if(lower.length > s.length + 2 && lower.slice(-s.length) === s){
        return [w.slice(0, w.length - s.length), w.slice(w.length - s.length)];
      }
    }
    // otherwise split long words into ~4-char chunks (rough stand-in for BPE)
    var parts = [], step = 4;
    for(var j=0;j<w.length;j+=step){ parts.push(w.slice(j, j+step)); }
    return parts;
  }

  function tokenize(text){
    // split keeping spaces and punctuation as their own pieces
    var raw = text.match(/\s+|[A-Za-z]+|[0-9]+|[^\sA-Za-z0-9]/g) || [];
    var toks = [];
    raw.forEach(function(piece){
      if(/^[A-Za-z]+$/.test(piece)){ chunkWord(piece).forEach(function(p){ toks.push(p); }); }
      else { toks.push(piece); }
    });
    return toks;
  }

  function render(){
    var text = input.value;
    var toks = tokenize(text);
    out.innerHTML = '';
    toks.forEach(function(t){
      var span = document.createElement('span');
      span.className = 'tok-piece';
      span.textContent = t === ' ' ? '·' : t.replace(/ /g,'·');
      out.appendChild(span);
    });
    cChars.textContent = text.length;
    cWords.textContent = (text.trim().match(/\S+/g) || []).length;
    cCount.textContent = toks.filter(function(t){ return t.trim() !== '' || true; }).length;
  }
  input.addEventListener('input', render);
  render();
})();
</script>

A few things you probably just noticed, and they are all real:

- Common words like "the" or "are" stay in one piece. The model saw them so many times that they earned their own token.
- Rarer or longer words get split. "Tokenization" might become "Token" plus "ization."
- Spaces and punctuation are not free. They are part of tokens too.

This chunking is done by a method called Byte Pair Encoding, or BPE. The short version: the tokenizer learned, from a mountain of text, which letter sequences show up together so often that it is worth giving them a single number. Frequent chunks get merged into one token. Rare ones stay split into many.

So a token is not a unit of meaning you chose. It is a unit of frequency the machine decided on, long before it ever met you.

## Now the strawberry makes sense

Back to our fruit. To you, "strawberry" is ten letters. To the model, it is something like three tokens.

<figure class="post-fig">
  <div class="berry">
    <div class="berry-line" style="opacity:.7">
      <span class="berry-tok">s</span><span class="berry-tok">t</span><span class="berry-tok">r</span><span class="berry-tok">a</span><span class="berry-tok">w</span><span class="berry-tok">b</span><span class="berry-tok">e</span><span class="berry-tok">r</span><span class="berry-tok">r</span><span class="berry-tok">y</span>
    </div>
    <div style="text-align:center;color:var(--text-3);font-family:var(--font-mono);font-size:.8rem">what you see (ten letters, three r's)</div>
    <div class="berry-line" style="margin-top:.6rem">
      <span class="berry-tok berry-hi">st</span><span class="berry-tok berry-hi">raw</span><span class="berry-tok berry-hi">berry</span>
    </div>
    <div style="text-align:center;color:var(--text-3);font-family:var(--font-mono);font-size:.8rem">what the model sees (three chunks, no loose letters)</div>
  </div>
  <figcaption>The model was handed "st", "raw", "berry". The individual r's are baked inside those chunks, invisible. Asking it to count letters is like asking you to count the bricks in a house by looking at a photo of the front door.</figcaption>
</figure>

The model never received the letters as separate things. It got three chunks, each one just a number to it. There is no "r" sitting in its view to count. So when it guesses an answer, it is pattern-matching on what people usually say, not actually counting, because it has nothing to count.

There is even a trick that proves the point. Type "s t r a w b e r r y" with spaces between every letter, and many models suddenly get it right. Why? Because the spaces force the tokenizer to break it into single letters. Now the model can finally see each one. Same word, same model, but you changed what it was allowed to look at.

I find this genuinely clarifying. The model is not stupid here. It is doing exactly what it can do with what it was given, and counting letters was never in the deal.

## Why this is also your bill

Here is where it gets practical, and a little expensive.

Every one of these AI services charges you by the token. Not by the word, not by the question, by the token. And it counts the tokens you send in plus the tokens it sends back. Both directions. So a chatty model that pads its answers is quietly costing you more.

There is a rough rule that is worth keeping in your head. On average, for ordinary English, one token works out to about four characters, which is roughly three quarters of a word. Flip that around and a hundred tokens is somewhere near seventy five words. So a page of writing, call it seven hundred and fifty words, lands around a thousand tokens. These are not exact figures, they are the kind of estimate you do on the back of a napkin, but they are close enough to sanity check a bill or a prompt length in your head.

<figure class="post-fig">
  <div class="viz-group">
    <span class="viz-chip viz-chip--solid">1 token</span>
    <span class="viz-chip">≈ 4 characters</span>
    <span class="viz-chip">≈ ¾ of a word</span>
    <span class="viz-chip">100 tokens ≈ 75 words</span>
  </div>
  <figcaption>A handy back-of-the-napkin conversion. Good enough to estimate a bill or check if your prompt will fit. Not exact, and never exact for code or other languages, which we will get to.</figcaption>
</figure>

That rule is your friend for quick maths. Need to know if a long document fits in a model's limit? Count the words, multiply by about 1.3, and you have a decent token estimate. Want to guess what a feature will cost at scale? Same trick.

But notice the rule keeps saying "English." That word is doing a lot of work, and it is where most people get surprised.

## Not all text costs the same

The four-characters rule holds for ordinary English prose. The moment you leave that lane, it falls apart, and your costs change with it.

- **Code** is heavier. Brackets, indentation, odd variable names, none of it was common in the same way plain English is, so it splits into more tokens per character.
- **Numbers and symbols** are heavier. A long number can become a surprising number of tokens.
- **Other languages are much heavier.** This one matters to a lot of us. Text in scripts the tokenizer saw less of, including many Indian languages, can run close to one token per character. That is roughly four times the token cost of the same meaning in English.

Sit with that last point for a second, because it is not just trivia. It means that, today, asking a question in Hindi or Tamil can cost several times more than asking the same question in English, purely because of how the tokenizer was built. The unfairness is baked into the unit itself. I think that is worth knowing, and worth pushing on.

## The part almost nobody mentions: everyone counts differently

You would think a token is a token. It is not. Each company built its own tokenizer, on its own data, with its own vocabulary. So the exact same sentence can cost a different number of tokens depending on who you send it to.

<figure class="post-fig">
  <div class="cmp">
    <div class="cmp-row"><span class="cmp-name">OpenAI</span><div class="cmp-track"><div class="cmp-fill" style="width:75%">15</div></div></div>
    <div class="cmp-row"><span class="cmp-name">Claude</span><div class="cmp-track"><div class="cmp-fill" style="width:85%">17</div></div></div>
    <div class="cmp-row"><span class="cmp-name">Gemini</span><div class="cmp-track"><div class="cmp-fill" style="width:78%">~16</div></div></div>
  </div>
  <figcaption>Illustrative, not a benchmark: the same sentence can land on different token counts across providers. For everyday prose the gap is usually small, a few percent. For code it can stretch to ten or twenty percent. The point is that there is no single true count.</figcaption>
</figure>

Here is the real, checkable picture of how the big three differ:

- **OpenAI** uses a library called tiktoken. Older models use an encoding called cl100k_base with a vocabulary around 100,000 chunks. Newer ones use o200k_base, roughly 200,000 chunks, with extra merges that handle code better. A bigger vocabulary means more text fits per token.
- **Anthropic** (Claude) uses its own tokenizer and, notably, does not publish it. The supported way to know your real cost is to ask their count-tokens endpoint before you send the full request.
- **Google** (Gemini) uses a different method again, called SentencePiece, with an even larger vocabulary.

So when you read that a model has a "200,000 token context window," that number does not mean the same amount of text everywhere. Two hundred thousand of Claude's tokens and two hundred thousand of OpenAI's tokens are not the same number of words. The unit looks standard and is not.

This is exactly the kind of quiet, real headache you hit when you build a tool that talks to more than one provider, and it is part of why I built [llmswap](https://llmswap.org). You stop being able to assume "a token" means one fixed thing.

## How to actually count them, when it matters

For a rough estimate, the four-characters rule is plenty. When you need the real number, do not guess. Use the tool the provider gives you:

- For OpenAI models, the tiktoken library counts exactly, offline, in your own code.
- For Claude, call the count-tokens endpoint, which returns the true input token total before you spend anything on the real call.
- For Gemini, there is a matching count-tokens call.

The pattern is the same everywhere. Estimate freely while you are sketching, and measure precisely before anything ships or anything gets billed.

## What I want you to take away

A token is a chunk of text, decided by frequency, that the model treats as a single number. It does not line up with letters, which is why "strawberry" trips it. It does not line up with words, which is why your bill is in this strange unit. And it does not line up across companies, which is why the same sentence has no single price.

None of this is magic, and none of it is arbitrary once you see the shape of it. It is just a side effect of teaching a machine to read by feeding it patterns instead of meaning.

The next time a model says something brilliant and then miscounts the r's in a fruit, you will not be confused. You will know it never saw the letters at all. It was only ever working with chunks, doing its best, the way it always is.
