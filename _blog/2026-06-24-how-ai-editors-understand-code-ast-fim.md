---
title: "How Your Editor Reads Your Mind: ASTs, Code Splitting, and the Trick Behind AI Suggestions"
date: 2026-06-24
excerpt: "Your editor finishes your code before you do. It feels like mind reading. It is not. It is two clever ideas working together: turning code into a tree, and learning to fill a gap from both sides. Here is the whole thing, from scratch."
tags: [ai, ast, code-completion, developer-tools, copilot, explainer]
---

<style>
.post-fig{margin:2.6rem 0;}
.post-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* tokenizer strip */
.tokrow{display:flex;flex-wrap:wrap;gap:.4rem;justify-content:center;max-width:560px;margin:0 auto;}
.tok{font-family:var(--font-mono);font-size:.95rem;padding:.35rem .6rem;border-radius:7px;background:var(--surface-2);border:1px solid var(--border-2);color:var(--text-2);}
.tok.op{border-color:var(--accent);color:var(--accent);font-weight:600;}
.tok.num{color:var(--text);}

/* AST tree (SVG, reliable alignment) */
.tree{max-width:360px;margin:0 auto;}
.tree svg{width:100%;height:auto;display:block;}
.tree text{font-family:var(--font-mono);font-weight:600;}
.tree .edge{stroke:var(--border-2);stroke-width:1.5;}

/* fixed vs ast chunk compare */
.chunk-cmp{display:grid;grid-template-columns:1fr 1fr;gap:1rem;max-width:640px;margin:0 auto;}
@media(max-width:600px){.chunk-cmp{grid-template-columns:1fr;}}
.chunk-col h4{font-family:var(--font-mono);font-size:.74rem;text-transform:uppercase;letter-spacing:.06em;margin:0 0 .7rem;}
.chunk-col.bad h4{color:var(--accent-2);}
.chunk-col.good h4{color:var(--accent);}
.codeblk{font-family:var(--font-mono);font-size:.8rem;line-height:1.7;background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:.7rem .9rem;color:var(--text-2);}
.codeblk .cut{display:block;border-top:2px dashed var(--accent-2);margin:.3rem -0.9rem;padding-top:.3rem;color:var(--accent-2);font-size:.7rem;}
.codeblk .keep{display:block;border:1px solid var(--accent);border-radius:6px;padding:.3rem .5rem;margin:.2rem 0;}

/* FIM cursor demo */
.fim{max-width:580px;margin:0 auto;}
.fim-editor{font-family:var(--font-mono);font-size:.9rem;line-height:1.9;background:var(--surface);border:1px solid var(--border-2);border-radius:12px;padding:1rem 1.2rem;box-shadow:var(--glow);}
.fim-pre{color:var(--text-2);}
.fim-gap{background:var(--grad);color:var(--accent-ink);border-radius:5px;padding:.05rem .4rem;font-weight:600;}
.fim-suf{color:var(--text-2);}
.fim-cursor{display:inline-block;width:2px;height:1.05em;background:var(--accent);vertical-align:-2px;animation:fimblink 1.05s steps(1) infinite;}
@keyframes fimblink{50%{opacity:0;}}
.fim-reorder{display:flex;gap:.5rem;justify-content:center;margin-top:1.1rem;flex-wrap:wrap;font-family:var(--font-mono);font-size:.8rem;}
.fim-tag{padding:.3rem .7rem;border-radius:999px;border:1px solid var(--border-2);color:var(--text-2);}
.fim-tag.pre{border-color:var(--accent);color:var(--accent);}
.fim-tag.suf{border-color:var(--accent);color:var(--accent);}
.fim-tag.mid{background:var(--grad);color:var(--accent-ink);border-color:transparent;font-weight:600;}
.fim-arrow{color:var(--text-3);}

/* prompt assembler stack */
.stack{max-width:560px;margin:0 auto;display:flex;flex-direction:column;gap:.4rem;}
.slab{display:flex;align-items:center;gap:.8rem;background:var(--surface);border:1px solid var(--border);border-radius:9px;padding:.6rem .9rem;}
.slab .pri{flex:none;font-family:var(--font-mono);font-size:.7rem;width:5.5rem;color:var(--accent);}
.slab .desc{flex:1;font-size:.88rem;color:var(--text-2);}
.slab.dim{opacity:.5;}
.slab.dim .pri{color:var(--text-3);}

/* full pipeline */
.pipe{display:flex;align-items:stretch;gap:.4rem;max-width:700px;margin:0 auto;flex-wrap:wrap;justify-content:center;}
.pstep{flex:1;min-width:96px;background:var(--surface);border:1px solid var(--border-2);border-radius:11px;padding:.8rem .5rem;text-align:center;}
.pstep .n{font-family:var(--font-mono);font-size:.68rem;color:var(--accent);display:block;text-transform:uppercase;letter-spacing:.05em;margin-bottom:.35rem;}
.pstep .t{font-size:.8rem;color:var(--text-2);}
.parrow{display:flex;align-items:center;color:var(--text-3);}
@media(max-width:620px){.parrow{transform:rotate(90deg);justify-content:center;}}
</style>

You are typing a function. You get as far as `function calculateTotal(items) {` and pause. Before your fingers reach the next key, faded grey text appears, suggesting the entire body, and it is roughly what you were about to write. You press Tab. It was right.

That moment feels like the editor read your mind. I want to take that feeling apart, gently, until there is no magic left, only a couple of genuinely clever ideas that you will understand completely by the end of this. There are two of them. The first is how a computer turns your messy text into something it actually understands. The second is the surprisingly elegant trick that lets an AI fill a gap in the middle of your code. Let us build both from nothing.

## Code is just text, until it becomes a tree

Here is the uncomfortable truth a beginner rarely hears: to a computer, your beautiful code is, at first, a meaningless string of characters. `f`, `u`, `n`, `c`, a space, and so on. It has no idea that `function` is special or that a curly brace opens a block. Before anything smart can happen, the computer has to find the structure hiding in that text. It does this in two steps.

The first step is called tokenization. The computer scans your text left to right and groups the characters into meaningful chunks called tokens, throwing away the noise like extra spaces and comments. The word `function` becomes one token. A number becomes a token. A `+` becomes a token. Take a tiny expression, `2 + (z - 1)`, and watch it break apart.

<figure class="post-fig">
  <div class="tokrow">
    <span class="tok num">2</span>
    <span class="tok op">+</span>
    <span class="tok">(</span>
    <span class="tok">z</span>
    <span class="tok op">-</span>
    <span class="tok num">1</span>
    <span class="tok">)</span>
  </div>
  <figcaption>Tokenization: the raw text becomes a flat list of meaningful pieces. Each token knows what kind of thing it is (a number, an operator, a parenthesis). Whitespace and comments are dropped here, because they do not change the meaning.</figcaption>
</figure>

A flat list of tokens is better than raw text, but it is still flat. It does not capture that the `z - 1` part has to happen before the addition. So the second step, called parsing, arranges those tokens into a tree that captures the structure. This tree has a name you will hear constantly once you start noticing it: the Abstract Syntax Tree, or AST.

For `2 + (z - 1)`, the tree looks like this. The addition sits at the top. Its two branches are the number `2` and the whole subtraction. And here is the detail I love: the parentheses have vanished.

<figure class="post-fig">
  <div class="tree">
    <svg viewBox="0 0 320 200" role="img" aria-label="AST tree: a plus node with children 2 and a minus node, whose children are z and 1">
      <!-- edges -->
      <line class="edge" x1="160" y1="34" x2="80" y2="84"/>
      <line class="edge" x1="160" y1="34" x2="240" y2="84"/>
      <line class="edge" x1="240" y1="98" x2="190" y2="150"/>
      <line class="edge" x1="240" y1="98" x2="290" y2="150"/>
      <!-- plus (root) -->
      <circle cx="160" cy="22" r="18" fill="url(#tg)"/>
      <text x="160" y="28" text-anchor="middle" fill="#0A0B0D" font-size="18">+</text>
      <!-- 2 -->
      <circle cx="80" cy="96" r="18" fill="var(--surface)" stroke="var(--border-2)"/>
      <text x="80" y="102" text-anchor="middle" fill="var(--text)" font-size="16">2</text>
      <!-- minus -->
      <circle cx="240" cy="96" r="18" fill="url(#tg)"/>
      <text x="240" y="102" text-anchor="middle" fill="#0A0B0D" font-size="18">&minus;</text>
      <!-- z -->
      <circle cx="190" cy="164" r="18" fill="var(--surface)" stroke="var(--border-2)"/>
      <text x="190" y="170" text-anchor="middle" fill="var(--text)" font-size="16">z</text>
      <!-- 1 -->
      <circle cx="290" cy="164" r="18" fill="var(--surface)" stroke="var(--border-2)"/>
      <text x="290" y="170" text-anchor="middle" fill="var(--text)" font-size="16">1</text>
      <defs>
        <linearGradient id="tg" x1="0" y1="0" x2="1" y2="1">
          <stop offset="0" stop-color="#5EEAD4"/><stop offset="1" stop-color="#818CF8"/>
        </linearGradient>
      </defs>
    </svg>
  </div>
  <figcaption>The AST for <span class="mono">2 + (z - 1)</span>. The shape itself encodes the order of operations, so the parentheses are no longer needed. The tree does not care how you spaced things or where you put brackets. It captures only what the code <em>means</em>. That is the whole point of the word "abstract."</figcaption>
</figure>

This is genuinely one of the most important ideas in all of programming, so let me say it plainly. An AST is your code's meaning, stripped of its appearance. Every node is either an operation (like add, or "call this function," or "loop over this") or a value (like a number, a variable, a piece of text). The connections between nodes say how they relate. Once your code is a tree, a computer can finally reason about it instead of just reading letters.

And this is not some niche AI thing. ASTs have quietly powered your tools for years. When your editor reformats your code on save, it parsed it into a tree and printed the tree back out neatly. When a linter warns you about an unused variable, it walked the tree and noticed a branch nothing points to. When you hit "rename symbol" and it correctly renames the right variable everywhere without touching a same-named one in a different scope, that is the tree knowing what belongs where. The AST is the unsung hero behind half the things you already take for granted.

## Why AI tools cut your code along the branches

Now we can connect this to AI. A language model has a limit on how much it can read at once, so a big codebase has to be chopped into pieces before the model can work with it. The naive way is to cut every fifty lines, like slicing a loaf without looking. The problem is obvious the moment you picture it: you will slice straight through the middle of a function, leaving its head in one piece and its body in another. Now neither piece makes sense on its own.

Because the AST knows where a function actually starts and ends, the smarter tools cut along the branches of the tree instead. A function stays whole. A class stays whole. Each piece is a complete, meaningful unit. Watch the difference.

<figure class="post-fig">
  <div class="chunk-cmp">
    <div class="chunk-col bad">
      <h4>Cutting by line count</h4>
      <div class="codeblk">
function getUser(id) {<br>
&nbsp;&nbsp;const row = db.find(id)<br>
<span class="cut">&#9986; cut at line 50</span>
&nbsp;&nbsp;return format(row)<br>
}
      </div>
    </div>
    <div class="chunk-col good">
      <h4>Cutting by the tree</h4>
      <div class="codeblk">
<span class="keep">function getUser(id) {<br>
&nbsp;&nbsp;const row = db.find(id)<br>
&nbsp;&nbsp;return format(row)<br>
}</span>
      </div>
    </div>
  </div>
  <figcaption>Same function, two ways to chop it for an AI. On the left, a blind line cut splits the function in half and ruins both pieces. On the right, AST-aware chunking keeps the whole function together because the tree knows where it begins and ends. If you read my <a href="/blog/2026-06-21-rag-part-2-chunking/">post on chunking</a>, this is that idea applied to code specifically, and it is why structure-aware splitting beats blind splitting every time.</figcaption>
</figure>

So when an AI assistant pulls in "relevant code" to help answer you, the good ones are pulling whole functions and classes that the AST identified, not random fifty-line windows. Better pieces in means better suggestions out.

## The real magic: filling the gap from both sides

Here is the part that genuinely surprised me when I learned it, and it is the heart of why completions feel like mind reading.

A normal language model writes left to right. It reads what came before and predicts what comes next. That is fine when your cursor is at the end of a line. But think about what you actually do when coding: very often your cursor is in the *middle* of existing code. There is code above it and code below it. A plain left-to-right model would ignore everything below your cursor, which is half the clue about what belongs in the gap.

The fix has a wonderfully descriptive name: Fill-in-the-Middle. The editor takes the code before your cursor, the code after your cursor, and the empty gap between them, and it asks the model to fill that gap using both sides as context.

<figure class="post-fig">
  <div class="fim">
    <div class="fim-editor">
      <div class="fim-pre">function calculateTotal(items) {</div>
      <div>&nbsp;&nbsp;<span class="fim-gap">return items.reduce((a, b) =&gt; a + b.price, 0)</span><span class="fim-cursor"></span></div>
      <div class="fim-suf">}</div>
    </div>
    <div class="fim-reorder">
      <span class="fim-tag pre">prefix (code above)</span>
      <span class="fim-arrow">+</span>
      <span class="fim-tag suf">suffix (code below)</span>
      <span class="fim-arrow">&rarr;</span>
      <span class="fim-tag mid">predict the middle</span>
    </div>
  </div>
  <figcaption>The accent text is what the model proposed for the gap. It could write it because it saw both the opening line above and the closing brace below, and reasoned about what fits between them. The editor knows the function is not finished, so it fills it in.</figcaption>
</figure>

Now, the clever bit underneath. How do you make a left-to-right model fill a middle gap? You do not rebuild the model. You play a trick with the order. During training, code is split into three parts, prefix, middle, and suffix, and then reshuffled into the order prefix, then suffix, then middle. The model still does the only thing it knows how to do, predict the next token left to right, but because the middle now comes *last*, predicting "the next part" means predicting the gap, with both sides already in front of it.

It is a beautifully lazy idea. Rather than invent a new kind of model, you just rearrange the puzzle so that "fill the middle" becomes "continue from the end." And it works: GitHub reported that adding this Fill-in-the-Middle ability lifted the rate of suggestions people actually accepted by around ten percent. A reordering trick, ten percent more useful completions.

## How the editor actually builds the question

So your editor understands structure (AST) and knows how to fill a gap (FIM). The last piece is what it actually sends to the model each time, because it cannot send your whole project. It has a token budget, the same idea from my post on tokens, and it has to spend that budget wisely.

Behind the scenes there is a step often called the prompt assembler. Every time you pause typing, it quickly gathers the most useful context it can find, ranks it by importance, and packs as much as fits into the budget, dropping the least important things when space runs low.

<figure class="post-fig">
  <div class="stack">
    <div class="slab"><span class="pri">highest</span><span class="desc">the code right around your cursor (your prefix and suffix)</span></div>
    <div class="slab"><span class="pri">high</span><span class="desc">relevant snippets from your other open files and nearby code</span></div>
    <div class="slab"><span class="pri">medium</span><span class="desc">imports and key definitions, so it knows what is available</span></div>
    <div class="slab"><span class="pri">medium</span><span class="desc">your project's instruction file, if you have one</span></div>
    <div class="slab dim"><span class="pri">low</span><span class="desc">everything else, trimmed away first when the budget runs out</span></div>
  </div>
  <figcaption>The prompt assembler, simplified. It is a ranking-and-trimming game played against a token budget, every single time you pause. This is also why what you have open and how you have named things matters so much: it directly shapes what the model gets to see.</figcaption>
</figure>

## Putting the whole thing together

Let me stitch it into one picture, the journey from your keystrokes to that grey suggestion.

<figure class="post-fig">
  <div class="pipe">
    <div class="pstep"><span class="n">you type</span><span class="t">raw code text</span></div>
    <div class="parrow">&rarr;</div>
    <div class="pstep"><span class="n">tokenize</span><span class="t">split into tokens</span></div>
    <div class="parrow">&rarr;</div>
    <div class="pstep"><span class="n">parse</span><span class="t">build the AST</span></div>
    <div class="parrow">&rarr;</div>
    <div class="pstep"><span class="n">chunk</span><span class="t">whole functions, by the tree</span></div>
    <div class="parrow">&rarr;</div>
    <div class="pstep"><span class="n">assemble</span><span class="t">prefix + suffix + context, FIM</span></div>
    <div class="parrow">&rarr;</div>
    <div class="pstep"><span class="n">suggest</span><span class="t">the grey text you Tab</span></div>
  </div>
  <figcaption>The full path. Structure understanding (tokenize, parse, chunk) feeds context assembly (FIM prompt within a token budget), which produces the completion. No mind reading. Just a parser, a tree, a reordering trick, and a careful budget.</figcaption>
</figure>

## Why knowing this makes you better at using it

Here is the practical payoff, because understanding the machine changes how you drive it.

Once you know the editor leans on the code *around* your cursor and in your *open files*, you start helping it on purpose. You open the file with the function you are about to call, so its shape is in the context. You give variables clear, honest names, because those names are tokens the model reads and reasons about. You write a clear comment or function signature before the body, because that becomes the prefix the suggestion is built from. You are no longer hoping the AI guesses right. You are feeding it the clues it actually uses.

That is the quiet shift from being surprised by your tools to being in command of them. The grey text was never magic. It was a tree and a trick and a budget, and now you know all three. The next time a completion lands perfectly, you will not just press Tab. You will know exactly why it knew.
