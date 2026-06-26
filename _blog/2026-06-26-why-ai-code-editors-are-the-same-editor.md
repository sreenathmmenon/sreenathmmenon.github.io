---
title: "Open the Hood: Why Almost Every AI Code Editor Is Secretly the Same Editor"
date: 2026-06-26
excerpt: "Cursor, Windsurf, Antigravity, IBM Bob. They look like rivals from different worlds. Pop the hood and you find the same engine in nearly all of them. Once you see it, the whole confusing landscape suddenly makes sense."
tags: [ai, code-editors, cursor, windsurf, developer-tools, explainer]
---

<style>
.post-fig{margin:2.6rem 0;}
.post-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* family tree (SVG-based, reliable) */
.fam{max-width:560px;margin:0 auto;}
.fam svg{width:100%;height:auto;display:block;}
.fam text{font-family:var(--font-mono);}

/* the "what's bolted on" stack */
.hood{max-width:440px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.layer{border-radius:11px;padding:.85rem 1rem;text-align:center;font-family:var(--font-mono);}
.layer .lt{font-size:.7rem;text-transform:uppercase;letter-spacing:.08em;opacity:.85;display:block;margin-bottom:.2rem;}
.layer .lb{font-size:.95rem;font-weight:600;}
.layer.l-ai{background:var(--grad);color:var(--accent-ink);}
.layer.l-glue{background:var(--surface-2);border:1px solid var(--accent);color:var(--accent);}
.layer.l-base{background:var(--surface);border:1px solid var(--border-2);color:var(--text-2);}
.hood .plus{text-align:center;color:var(--text-3);font-size:1.1rem;line-height:.5;}

/* autonomy spectrum (interactive) */
.spec{max-width:620px;margin:0 auto;background:var(--surface);border:1px solid var(--border-2);border-radius:16px;padding:1.4rem;box-shadow:var(--glow);}
.spec-track{position:relative;height:8px;border-radius:999px;background:linear-gradient(90deg,var(--accent-2),var(--accent));margin:1.4rem 0 .4rem;}
.spec input[type=range]{width:100%;accent-color:var(--accent);cursor:pointer;}
.spec-ends{display:flex;justify-content:space-between;font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);}
.spec-card{margin-top:1.2rem;border-top:1px solid var(--border);padding-top:1.1rem;}
.spec-card h4{font-family:var(--font-mono);font-size:.8rem;color:var(--accent);margin:0 0 .4rem;text-transform:uppercase;letter-spacing:.05em;}
.spec-card .body{color:var(--text-2);font-size:.95rem;line-height:1.6;}
.spec-card .body b{color:var(--text);}
.spec-eg{font-family:var(--font-mono);font-size:.78rem;color:var(--text-3);margin-top:.7rem;}

/* comparison table */
.ctab{width:100%;border-collapse:collapse;font-size:.88rem;}
.ctab th,.ctab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.ctab th{font-family:var(--font-mono);font-size:.7rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.ctab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}
.ctab .vs{font-family:var(--font-mono);color:var(--accent);font-size:.82rem;}

/* fork bars */
.share{max-width:520px;margin:0 auto;}
.share-row{display:flex;align-items:center;gap:.8rem;margin-bottom:.7rem;}
.share-name{flex:none;width:6.5rem;font-family:var(--font-mono);font-size:.82rem;color:var(--text);}
.share-bar{flex:1;height:26px;border-radius:7px;background:var(--surface-2);border:1px solid var(--border);overflow:hidden;display:flex;}
.share-vs{height:100%;background:var(--surface);border-right:2px solid var(--accent);display:flex;align-items:center;padding-left:.6rem;font-family:var(--font-mono);font-size:.68rem;color:var(--text-3);}
.share-ai{height:100%;background:var(--grad);flex:1;display:flex;align-items:center;justify-content:flex-end;padding-right:.6rem;font-family:var(--font-mono);font-size:.68rem;color:var(--accent-ink);font-weight:600;}
</style>

If you went looking for an AI code editor this year, you probably came back dizzy. Cursor. Windsurf. Google's Antigravity. IBM's Bob. A new one seems to launch every few weeks, each with its own logo, its own pitch, its own army of fans insisting it is the only real choice. It feels like walking into a phone shop where every box promises something completely different.

I want to let you in on the secret that makes all of this suddenly simple. If you pop the hood on almost every one of these editors, you find the exact same engine sitting inside. Once you know that, you stop seeing a dozen mysterious rivals and start seeing one familiar thing, dressed in different clothes. Let me show you the engine, and then show you what actually makes them different, because the real differences are more interesting than the marketing.

## The shared secret: it is all VS Code

Here is the engine. Microsoft makes a free, open-source code editor called Visual Studio Code, or VS Code for short. It is, by a huge margin, the most popular code editor in the world. Crucially, it is open source, which means anyone is allowed to take its entire codebase, make a copy, and build their own thing on top of it. That copy is called a fork.

And that is exactly what happened. Cursor is a fork of VS Code. Google's Antigravity is a fork of VS Code. IBM's Bob is built on VS Code. Even the tools that are not full forks, like Windsurf, plug into VS Code and editors like it. The familiar editor you already know is hiding inside nearly all of them.

<figure class="post-fig">
  <div class="fam">
    <svg viewBox="0 0 520 250" role="img" aria-label="Family tree showing VS Code as the root, with Cursor, Antigravity, IBM Bob as forks and Windsurf as a plugin">
      <!-- root -->
      <rect x="190" y="14" width="140" height="44" rx="10" fill="url(#fg)"/>
      <text x="260" y="34" text-anchor="middle" fill="#0A0B0D" font-size="14" font-weight="700">VS Code</text>
      <text x="260" y="50" text-anchor="middle" fill="#0A0B0D" font-size="9">Microsoft, open source</text>
      <!-- edges -->
      <line x1="260" y1="58" x2="70" y2="150" stroke="var(--border-2)" stroke-width="1.5"/>
      <line x1="260" y1="58" x2="190" y2="150" stroke="var(--border-2)" stroke-width="1.5"/>
      <line x1="260" y1="58" x2="330" y2="150" stroke="var(--border-2)" stroke-width="1.5"/>
      <line x1="260" y1="58" x2="450" y2="150" stroke="var(--border-2)" stroke-width="1.5" stroke-dasharray="4 3"/>
      <!-- children -->
      <rect x="20" y="150" width="100" height="40" rx="9" fill="var(--surface)" stroke="var(--border-2)"/>
      <text x="70" y="174" text-anchor="middle" fill="var(--text)" font-size="12" font-weight="600">Cursor</text>
      <rect x="140" y="150" width="100" height="40" rx="9" fill="var(--surface)" stroke="var(--border-2)"/>
      <text x="190" y="174" text-anchor="middle" fill="var(--text)" font-size="12" font-weight="600">Antigravity</text>
      <rect x="280" y="150" width="100" height="40" rx="9" fill="var(--surface)" stroke="var(--border-2)"/>
      <text x="330" y="174" text-anchor="middle" fill="var(--text)" font-size="12" font-weight="600">IBM Bob</text>
      <rect x="400" y="150" width="100" height="40" rx="9" fill="var(--surface)" stroke="var(--accent)" stroke-dasharray="4 3"/>
      <text x="450" y="170" text-anchor="middle" fill="var(--text)" font-size="12" font-weight="600">Windsurf</text>
      <text x="450" y="183" text-anchor="middle" fill="var(--text-3)" font-size="8">plugs in, not a fork</text>
      <defs><linearGradient id="fg" x1="0" y1="0" x2="1" y2="1"><stop offset="0" stop-color="#5EEAD4"/><stop offset="1" stop-color="#818CF8"/></linearGradient></defs>
    </svg>
  </div>
  <figcaption>The family tree. One open-source editor at the root, many descendants. The solid lines are full forks (they copied VS Code and changed it). The dashed line is the plugin approach (it adds itself into VS Code without forking). Different relationships, same ancestor.</figcaption>
</figure>

This is not a dirty secret, by the way. It is a smart, normal thing to do. Building a great code editor from scratch takes years: the syntax highlighting, the file explorer, the extensions, the git integration, the thousand tiny things you never think about until they are missing. Why rebuild all of that when an excellent, free, battle-tested version already exists? You take VS Code, you keep all the parts developers already love, and you spend your energy on the one part that is actually new: the AI.

## So what is actually different?

If the editor underneath is the same, then the differences must live somewhere else. They do, in two places that are bolted on top.

<figure class="post-fig">
  <div class="hood">
    <div class="layer l-ai"><span class="lt">the brain (the new part)</span><span class="lb">an AI model + how the tool uses it</span></div>
    <div class="plus">+</div>
    <div class="layer l-glue"><span class="lt">the wiring (the clever part)</span><span class="lb">how the AI reaches your files, terminal, browser</span></div>
    <div class="plus">+</div>
    <div class="layer l-base"><span class="lt">the body (the shared part)</span><span class="lb">VS Code: editing, files, extensions, git</span></div>
  </div>
  <figcaption>The three layers of basically every AI editor. The bottom is shared by almost all of them. The middle and top are where each tool makes its bets, and where all the personality lives. When people argue about which editor is best, they are really arguing about the top two layers.</figcaption>
</figure>

Think of it like cars. Underneath, a huge number of cars share the same chassis and engine block from the same supplier. What you actually feel as a driver is the steering, the seats, how much the car does for you versus how much you do yourself. AI editors are the same. The chassis is VS Code. The driving experience is the AI layer. And the single biggest difference between them is one question: how much is the AI allowed to drive?

## The one idea that explains all of them: how much does the AI drive?

This is the heart of it, so let me build it carefully. Every AI editor sits somewhere on a spectrum, from "the AI quietly suggests, you do everything" all the way to "you describe a goal and the AI goes off and does it." Drag the slider below and watch the style of help change.

<figure class="post-fig">
  <div class="spec">
    <div class="spec-ends"><span>AI suggests</span><span>AI assists</span><span>AI drives</span></div>
    <div class="spec-track"></div>
    <input type="range" id="specRange" min="0" max="2" step="1" value="0" aria-label="Autonomy level from suggest to drive">
    <div class="spec-card">
      <h4 id="specTitle"></h4>
      <div class="body" id="specBody"></div>
      <div class="spec-eg" id="specEg"></div>
    </div>
  </div>
  <figcaption>The autonomy spectrum. This single dial is the most useful way to tell AI editors apart. It is not about which is "best." It is about how much control you want to hand over, which depends entirely on what you are doing.</figcaption>
</figure>

<script>
(function(){
  var range=document.getElementById('specRange'); if(!range) return;
  var t=document.getElementById('specTitle'), b=document.getElementById('specBody'), e=document.getElementById('specEg');
  var levels=[
    {t:'AI suggests (autocomplete)', b:'You are fully in charge. As you type, the AI offers a grey ghost of the next line or block, and you accept it or ignore it. It never touches anything you did not ask for. This is the oldest and gentlest form of help, and for a lot of work it is still the most comfortable.', e:'feels like: a really good autocomplete that finishes your thought'},
    {t:'AI assists (agent in the loop)', b:'Now you give it a small task in plain words, and the AI reads your files, writes a change across several of them, and shows you a diff. But here is the key: <b>it waits for your approval before it does anything real.</b> You stay in the loop, approving each step. More power, but with a seatbelt.', e:'feels like: a junior developer who shows you every change before saving'},
    {t:'AI drives (agent first)', b:'You describe a goal, and the AI plans it, splits it into subtasks, edits files, runs the terminal, even opens a real browser to test the app it just built, all mostly on its own. You step back and <b>supervise the outcome</b> rather than each keystroke. Newer tools are built around this, with dashboards to watch several agents work at once.', e:'feels like: handing a project to a team and reviewing their work'}
  ];
  function render(){var L=levels[+range.value]; t.textContent=L.t; b.innerHTML=L.body; e.textContent=L.e;}
  range.addEventListener('input',render); render();
})();
</script>

Notice the trade as you slide right: you gain power and speed, but you give up fine control, and you have to trust more. Slide left and it flips: less power, but you see and shape everything. There is no universally correct spot on this dial. The right answer changes by the hour depending on whether you are carefully fixing a payment bug or quickly scaffolding a throwaway prototype.

## Where each editor sits, and why

Now the fun part. With that one dial in mind, the whole confusing landscape lines up neatly. Each tool picked a philosophy and built around it.

<figure class="post-fig">
  <table class="ctab">
    <thead><tr><th>Editor</th><th>Built on</th><th>Its bet</th></tr></thead>
    <tbody>
      <tr><td>Cursor</td><td class="vs">VS Code fork</td><td>Polished, fast, AI baked deep into the editor. Strong predictive edits plus an agent when you want it. A smooth all-rounder.</td></tr>
      <tr><td>Windsurf</td><td class="vs">plugin, not a fork</td><td>Works inside many editors instead of being one. The bet: meet developers where they already are, across 40-plus tools.</td></tr>
      <tr><td>Antigravity</td><td class="vs">VS Code fork</td><td>Agent first. A control room to spawn and watch multiple agents, which produce visible plans and screenshots so you can check their reasoning.</td></tr>
      <tr><td>IBM Bob</td><td class="vs">built on VS Code</td><td>Aimed at big companies and old codebases: legacy modernization, compliance, structured plan-code-review modes for serious enterprise work.</td></tr>
    </tbody>
  </table>
  <figcaption>Same body, different bets. Some chase polish, some chase reach, some chase autonomy, some chase the enterprise. None of them reinvented the editor, because they did not need to. They competed on the layers that matter.</figcaption>
</figure>

Look at how little of each tool is actually new. The vast majority of every one of these editors is just VS Code that someone else already built. The thin slice on top, the AI brain and its wiring, is the entire battleground.

<figure class="post-fig">
  <div class="share">
    <div class="share-row"><span class="share-name">Cursor</span><div class="share-bar"><span class="share-vs" style="flex:0 0 72%">shared VS Code base</span><span class="share-ai">their AI</span></div></div>
    <div class="share-row"><span class="share-name">Antigravity</span><div class="share-bar"><span class="share-vs" style="flex:0 0 70%">shared VS Code base</span><span class="share-ai">their AI</span></div></div>
    <div class="share-row"><span class="share-name">IBM Bob</span><div class="share-bar"><span class="share-vs" style="flex:0 0 74%">shared VS Code base</span><span class="share-ai">their AI</span></div></div>
  </div>
  <figcaption>Illustrative, not exact percentages. The grey is the editor everyone shares. The accent slice is the genuinely new part each company built. When you pick an editor, you are mostly choosing that thin accent slice, plus a philosophy about how much it should drive.</figcaption>
</figure>

## Why fork at all? And why did Windsurf not?

You might wonder: if VS Code already supports extensions, why copy the whole thing instead of just writing a plugin? It comes down to how deep you want to reach.

A plugin lives in a guest room. It can do a lot, but the house has rules, and some doors are locked. When a company forks VS Code, they own the whole house. They can wire the AI into the deepest parts of the editor, the parts a plugin is never allowed to touch, which lets them build smoother, more powerful features. That is why Cursor and Antigravity forked: they wanted those locked rooms.

Windsurf made the opposite bet on purpose. By staying a plugin, it gives up those locked rooms, but in exchange it can live inside dozens of different editors, not just one. Less depth, much more reach. Neither choice is wrong. They are answers to different questions, and seeing the trade-off is the whole point.

## What this means for you

So here is the payoff, the thing I hope you carry away. The next time a new AI editor launches with a slick video and bold claims, you will not feel lost. You will quietly ask three questions, and they will tell you almost everything.

First: is it built on VS Code? Almost certainly yes, which means the editing experience will feel familiar and you are really evaluating the AI, not the editor. Second: where does it sit on the autonomy dial, from gentle suggestions to a fully driving agent? That tells you how it will feel to use day to day. Third: what is its bet, polish, reach, autonomy, or the enterprise? That tells you who it is really for.

The flood of new editors stops being intimidating once you realise they are variations on a theme you already understand. Same engine, different drivers. You are not choosing between a dozen alien spaceships. You are choosing how much you want the AI to take the wheel, in an editor you already know how to drive.
