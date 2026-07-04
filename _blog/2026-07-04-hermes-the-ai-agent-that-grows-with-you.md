---
title: "Hermes: The AI Agent That Learns From Itself"
date: 2026-07-04
excerpt: "Most AI agents wake up every morning with amnesia. You teach them something Monday, and Monday's lesson is gone by Tuesday. Hermes, from Nous Research, made forgetting optional: it writes its own skills from experience and gets better at what you actually do. Here is what it is, why people are talking about it, how it stacks up against OpenClaw, and where it genuinely falls short."
tags: [ai, agents, hermes, nous-research, open-source, explainer]
---

<style>
.hm-fig{margin:2.5rem 0;}
.hm-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* amnesia vs memory */
.hm-amn{display:grid;grid-template-columns:1fr 1fr;gap:.9rem;max-width:640px;margin:0 auto;}
@media(max-width:560px){.hm-amn{grid-template-columns:1fr;}}
.hm-amn .col{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.hm-amn .col h4{margin:0 0 .6rem;font-size:.88rem;}
.hm-amn .col.old{border-color:var(--border-2);} .hm-amn .col.old h4{color:var(--text-2);}
.hm-amn .col.new{border-color:var(--accent);} .hm-amn .col.new h4{color:var(--accent);}
.hm-amn .day{font-family:var(--font-mono);font-size:.76rem;color:var(--text-2);padding:.35rem .5rem;border-radius:6px;background:var(--surface-2);border:1px solid var(--border);margin-bottom:.35rem;}
.hm-amn .day .x{color:var(--text-3);}
.hm-amn .day .up{color:var(--accent);}

/* the self-improving loop */
.hm-loop{max-width:460px;margin:0 auto;}
.hm-loop svg{width:100%;height:300px;overflow:visible;}
.hm-lnode{fill:var(--surface);stroke:var(--border-2);stroke-width:1.5;}
.hm-lnode.acc{stroke:var(--accent);}
.hm-ltxt{fill:var(--text);font-family:var(--font-mono);font-size:12px;font-weight:600;text-anchor:middle;}
.hm-lsub{fill:var(--text-3);font-family:var(--font-mono);font-size:9px;text-anchor:middle;}
.hm-larrow{fill:none;stroke:var(--accent);stroke-width:2;marker-end:url(#hmh);}

/* skill accumulation over days */
.hm-days{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:.8rem;}
.hm-days .row{display:flex;align-items:center;gap:.8rem;}
.hm-days .row .d{flex:none;width:66px;font-family:var(--font-mono);font-size:.76rem;color:var(--text-2);}
.hm-days .row .bar{flex:1;height:22px;border-radius:6px;background:var(--surface-2);border:1px solid var(--border);overflow:hidden;}
.hm-days .row .fill{height:100%;width:0;background:var(--grad);transition:width 1s cubic-bezier(.2,.7,.2,1);}
.hm-days.go .row .fill{width:var(--w);}
.hm-days .row .n{flex:none;width:78px;font-family:var(--font-mono);font-size:.72rem;color:var(--accent);text-align:right;}
.hm-days.go .r1 .fill{transition-delay:.1s} .hm-days.go .r2 .fill{transition-delay:.35s}
.hm-days.go .r3 .fill{transition-delay:.6s} .hm-days.go .r4 .fill{transition-delay:.85s}

/* hermes vs openclaw philosophies */
.hm-vs{display:grid;grid-template-columns:1fr 1fr;gap:.8rem;max-width:660px;margin:0 auto;}
@media(max-width:560px){.hm-vs{grid-template-columns:1fr;}}
.hm-vs .p{border:1px solid var(--border-2);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);}
.hm-vs .p.h{border-color:var(--accent);}
.hm-vs .p .tag{font-family:var(--font-mono);font-size:.7rem;color:var(--accent);border:1px solid var(--accent);border-radius:5px;padding:.1rem .4rem;display:inline-block;margin-bottom:.5rem;}
.hm-vs .p.o .tag{color:var(--text-2);border-color:var(--border-2);}
.hm-vs .p b{display:block;color:var(--text);font-size:.95rem;margin-bottom:.3rem;}
.hm-vs .p .motto{font-size:.82rem;color:var(--text-2);font-style:italic;margin-bottom:.5rem;}
.hm-vs .p ul{margin:0;padding-left:1.05rem;}
.hm-vs .p li{font-size:.83rem;color:var(--text-2);margin:.25rem 0;}

/* pros cons */
.hm-pc{display:grid;grid-template-columns:1fr 1fr;gap:.8rem;max-width:680px;margin:0 auto;}
@media(max-width:560px){.hm-pc{grid-template-columns:1fr;}}
.hm-pc .side{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.hm-pc .side h4{margin:0 0 .6rem;font-size:.85rem;font-family:var(--font-mono);}
.hm-pc .side.pro{border-color:var(--accent);} .hm-pc .side.pro h4{color:var(--accent);}
.hm-pc .side.con h4{color:var(--text-2);}
.hm-pc .side .item{font-size:.84rem;color:var(--text-2);margin:.45rem 0;line-height:1.5;padding-left:1.1rem;position:relative;}
.hm-pc .side .item::before{position:absolute;left:0;font-family:var(--font-mono);}
.hm-pc .side.pro .item::before{content:"+";color:var(--accent);}
.hm-pc .side.con .item::before{content:"\2212";color:var(--text-3);}

/* example scenario cards */
.hm-ex{display:grid;grid-template-columns:repeat(3,1fr);gap:.7rem;max-width:720px;margin:0 auto;}
@media(max-width:640px){.hm-ex{grid-template-columns:1fr;}}
.hm-ex .c{border:1px solid var(--border-2);border-radius:12px;padding:.9rem;background:var(--surface);}
.hm-ex .c .who{font-family:var(--font-mono);font-size:.68rem;text-transform:uppercase;letter-spacing:.06em;color:var(--accent);}
.hm-ex .c b{display:block;color:var(--text);font-size:.9rem;margin:.15rem 0 .4rem;}
.hm-ex .c span{font-size:.83rem;color:var(--text-2);line-height:1.55;}

/* table */
.hm-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.hm-tab th,.hm-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.hm-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.hm-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}

@media (prefers-reduced-motion: reduce){
  .hm-days .row .fill{transition:none !important;width:var(--w) !important;}
}
</style>

Here is a strange thing about most AI agents: they are brilliant and they have amnesia at the same time.

You spend Monday teaching one how your project is laid out, which commands to run, the quirks of your setup. It nails the task. Then Tuesday comes, you open a fresh session, and it is a blank slate again. Everything you taught it, gone. You are onboarding the same new hire every single morning, forever. It is exhausting, and it is the quiet ceiling on how useful these things have been.

**Hermes**, an open-source agent from Nous Research, is one of the first serious attempts to knock that ceiling out. Its whole pitch is in the tagline: *the agent that grows with you.* Instead of forgetting, it *learns*, it writes down what worked as a reusable skill, remembers it across sessions, and gets faster and more reliable at the things you actually do. Lately it has been the agent everyone's arguing about, and it's usually argued about next to a rival called OpenClaw. So let me walk you through what it is, why it matters, how the two differ, and, honestly, where Hermes still stumbles. No hype, just what the research shows.

## The core idea: an agent with a memory that compounds

<figure class="hm-fig">
  <div class="hm-amn">
    <div class="col old">
      <h4>A normal agent</h4>
      <div class="day">Mon: learns your setup <span class="x">✓</span></div>
      <div class="day">Tue: <span class="x">forgot everything</span></div>
      <div class="day">Wed: <span class="x">forgot everything</span></div>
      <div class="day">Thu: <span class="x">forgot everything</span></div>
      <div style="font-size:.8rem;color:var(--text-3);margin-top:.5rem">Flat. Every day starts at zero.</div>
    </div>
    <div class="col new">
      <h4>Hermes</h4>
      <div class="day">Mon: learns, saves a skill <span class="up">✓</span></div>
      <div class="day">Tue: reuses it, refines it <span class="up">↑</span></div>
      <div class="day">Wed: faster still <span class="up">↑</span></div>
      <div class="day">Thu: near-instant <span class="up">↑</span></div>
      <div style="font-size:.8rem;color:var(--text-3);margin-top:.5rem">Compounding. Each day builds on the last.</div>
    </div>
  </div>
  <figcaption>The whole difference in one picture. A standard agent's competence is flat, reset to zero each session. Hermes's competence is a staircase, because it keeps what it learns. That shift from flat to compounding is the entire reason people are paying attention.</figcaption>
</figure>

If you've read my post on Skills, this will feel familiar, and that's the point. A Skill is a folder with instructions an agent can pull in when a task matches. The twist Hermes adds is: **the agent writes its own Skills.** You don't author them by hand. When Hermes finishes something tricky, it distills what worked into a reusable skill file and files it away. Next time a similar task shows up, it reaches for that skill instead of figuring it all out again.

## How the learning loop actually works

Under the hood it's a loop, and once you see the shape it's not mysterious at all. It sits *around* the normal agent loop (think, act, observe) and adds a fourth beat: *learn.*

<figure class="hm-fig">
  <div class="hm-loop">
    <svg viewBox="0 0 400 300">
      <defs><marker id="hmh" markerWidth="9" markerHeight="9" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" fill="var(--accent)"/></marker></defs>
      <rect class="hm-lnode" x="140" y="15" width="120" height="52" rx="12"/>
      <text class="hm-ltxt" x="200" y="37">DO THE TASK</text>
      <text class="hm-lsub" x="200" y="54">think, act, observe</text>
      <rect class="hm-lnode" x="255" y="120" width="125" height="52" rx="12"/>
      <text class="hm-ltxt" x="317" y="142">DISTILL A SKILL</text>
      <text class="hm-lsub" x="317" y="159">write what worked</text>
      <rect class="hm-lnode acc" x="140" y="230" width="120" height="52" rx="12"/>
      <text class="hm-ltxt" x="200" y="252">STORE IT</text>
      <text class="hm-lsub" x="200" y="269">memory + skill file</text>
      <rect class="hm-lnode" x="20" y="120" width="125" height="52" rx="12"/>
      <text class="hm-ltxt" x="82" y="142">REUSE + REFINE</text>
      <text class="hm-lsub" x="82" y="159">next similar task</text>
      <path class="hm-larrow" d="M258,58 Q330,80 320,116"/>
      <path class="hm-larrow" d="M300,174 Q270,215 262,232"/>
      <path class="hm-larrow" d="M140,258 Q70,240 82,176"/>
      <path class="hm-larrow" d="M90,116 Q120,70 142,58"/>
    </svg>
  </div>
  <figcaption>Hermes does a task, distills what worked into a skill, stores it in persistent memory, and reuses and sharpens it the next time. Round and round, the library of skills grows. Concretely: it writes reusable Markdown skill files and keeps outcomes in a searchable store (it uses SQLite full-text search plus LLM summarization to recall across sessions).</figcaption>
</figure>

So when you come back tomorrow, the skills are already there. Run something similar and it leans on what it learned, and executes faster. That "come back the next day and it remembers" behaviour is the headline feature, and it's baked into the architecture rather than bolted on.

## Why this matters now (and why it hit #1)

Watch what compounding does over even a short week. This is the intuition, a rough sketch, not a benchmark, of why a self-improving agent pulls ahead:

<figure class="hm-fig">
  <div class="hm-days" id="days">
    <div class="row r1"><div class="d">Day 1</div><div class="bar"><div class="fill" style="--w:22%"></div></div><div class="n">1 skill</div></div>
    <div class="row r2"><div class="d">Day 3</div><div class="bar"><div class="fill" style="--w:45%"></div></div><div class="n">6 skills</div></div>
    <div class="row r3"><div class="d">Day 7</div><div class="bar"><div class="fill" style="--w:72%"></div></div><div class="n">18 skills</div></div>
    <div class="row r4"><div class="d">Day 14</div><div class="bar"><div class="fill" style="--w:95%"></div></div><div class="n">40+ skills</div></div>
  </div>
  <figcaption>Illustrative, not measured: the point is the shape. A self-learning agent's usefulness curves upward as its skill library fills, while a forgetful one stays a flat line. This "depth over time" is exactly the bet that, per reports, pushed Hermes to the top of OpenRouter's daily-usage rankings in 2026, a meaningful slice of developers choosing depth of learning over sheer breadth of reach.</figcaption>
</figure>

## Hermes vs OpenClaw: two philosophies, not two products

You cannot read about Hermes without bumping into **OpenClaw**, its main rival, and the comparison is genuinely useful because they disagree at a deep level. They're not two versions of the same thing; they're two different *bets* about what a personal AI agent should be.

<figure class="hm-fig">
  <div class="hm-vs">
    <div class="p h">
      <span class="tag">Hermes · Nous Research</span>
      <b>Depth of learning</b>
      <div class="motto">"Grow with the user over time."</div>
      <ul>
        <li>Self-improving loop: writes its own skills</li>
        <li>Persistent, curated memory across sessions</li>
        <li>Gets faster at your recurring work</li>
        <li>Bet: a private assistant that compounds</li>
      </ul>
    </div>
    <div class="p o">
      <span class="tag">OpenClaw</span>
      <b>Breadth of reach</b>
      <div class="motto">"Be everywhere, do everything now."</div>
      <ul>
        <li>Central gateway wiring 50+ messaging channels</li>
        <li>Human-authored skills, batteries included</li>
        <li>Fast to deploy, tooling out of the box</li>
        <li>Bet: a hub that reaches every channel</li>
      </ul>
    </div>
  </div>
  <figcaption>The rivalry in one line: Hermes is built around a self-improving agent that learns; OpenClaw is built around a control-plane gateway that connects everywhere. Depth versus breadth. Neither is "right", they're optimized for different things.</figcaption>
</figure>

Put plainly: **if you want an agent that reaches you on twenty-five messaging channels and works out of the box, OpenClaw's breadth wins. If you want a private assistant that quietly gets better at your specific recurring work, Hermes's depth wins.** The clean way to choose is to ask whether you value *reach* or *learning* more for your use.

<figure class="hm-fig">
<table class="hm-tab">
<thead><tr><th>Dimension</th><th>Hermes</th><th>OpenClaw</th></tr></thead>
<tbody>
<tr><td>Core idea</td><td>Self-improving skill loop</td><td>Central gateway to many channels</td></tr>
<tr><td>Skills</td><td>Agent writes its own</td><td>Human-authored</td></tr>
<tr><td>Memory</td><td>Persistent, curated, cross-session</td><td>Session / gateway-centric</td></tr>
<tr><td>Strength</td><td>Depth: learns your work</td><td>Breadth: 50+ channels, fast setup</td></tr>
<tr><td>Setup time</td><td>Longer (2 to 4 hours)</td><td>Shorter (under 30 min)</td></tr>
<tr><td>License</td><td>Open source (MIT)</td><td>Open source</td></tr>
</tbody>
</table>
  <figcaption>A side-by-side, kept honest. Both are open source. The real fork is philosophy: learning depth vs channel breadth, and that flows into everything else including how long it takes to get going.</figcaption>
</figure>

## What it looks like in real life: three examples

Abstract talk of "self-improving" only lands with concrete scenes. Here's the same capability across three very different users:

<figure class="hm-fig">
  <div class="hm-ex">
    <div class="c">
      <div class="who">General</div>
      <b>Your personal assistant</b>
      <span>You ask it to plan trips a certain way, book with certain preferences. By week two it just *knows* your style, aisle seat, no red-eyes, and stops re-asking. It learned you.</span>
    </div>
    <div class="c">
      <div class="who">Technical</div>
      <b>A coding teammate</b>
      <span>First time, it fumbles your deploy process. It saves a skill for it. Next deploy, it runs the exact steps, your build flags, your test subset, without being re-taught. The tenth deploy is muscle memory.</span>
    </div>
    <div class="c">
      <div class="who">Marketing</div>
      <b>A content operator</b>
      <span>It drafts your newsletter, learns your voice and the sections you always want, remembers which subject-line style performed. Each issue costs you less editing than the last.</span>
    </div>
  </div>
  <figcaption>Same underlying mechanism, three worlds. A traveller, a developer, a marketer, each gets an agent that starts generic and becomes *theirs*. The value isn't any single task; it's the slope, the fact that next week is easier than this week.</figcaption>
</figure>

It runs about anywhere too, a $5 VPS, a GPU box, or serverless, and you reach it through the channels you already use (Telegram, Discord, Slack and more). It's model-agnostic (200+ models via Nous Portal, OpenRouter, OpenAI-compatible endpoints, or local Ollama), ships with 40-plus built-in tools, and speaks MCP, so it plugs into the same tool ecosystem I wrote about in the MCP post. In other words, it isn't an island; it's a learning loop wrapped around the agent ideas you already know.

## The honest part: where Hermes falls short

A teaching post that only sells you the upside isn't teaching, it's advertising. So here's the balanced ledger, straight from what practitioners report.

<figure class="hm-fig">
  <div class="hm-pc">
    <div class="side pro">
      <h4>Genuine strengths</h4>
      <div class="item">Memory compounds: recurring work gets faster and more reliable over time</div>
      <div class="item">Auto-generated skills, no hand-authoring each workflow</div>
      <div class="item">One-command install; runs on cheap hardware</div>
      <div class="item">Open source (MIT), model-agnostic, MCP-native, 40+ tools</div>
    </div>
    <div class="side con">
      <h4>Real limitations</h4>
      <div class="item">Self-learning is OFF by default; new users don't enable it and see a plain agent</div>
      <div class="item">Slow to set up: 2 to 4 hours vs under 30 min for OpenClaw</div>
      <div class="item">No managed cloud; you host it yourself</div>
      <div class="item">In fuzzy domains it can get confidently faster at the *wrong* thing, no ground truth to check against</div>
      <div class="item">Learning is domain-specific; a skill for "summarize a PR" won't transfer to "plan a DB migration"</div>
    </div>
  </div>
  <figcaption>The two that matter most are on the right. "Self-learning off by default" means many people try Hermes and never actually see its whole point. And "faster at the wrong thing" is the deep one: an agent that reinforces its own habits needs a way to know it's improving toward *correct*, not just toward *confident*.</figcaption>
</figure>

That last limitation is worth sitting with, because it's the real intellectual catch of self-improving agents in general. Learning from your own experience is powerful when there's a clear signal for "did that work?", tests passed, the deploy succeeded, the user said yes. In domains without that clear signal, a self-reinforcing agent can happily get more efficient at a mistake. Speed is not the same as correctness. Any agent that trains on itself inherits this, and it's exactly the kind of thing worth designing guardrails around, the same "keep a human in the loop for the ambiguous, irreversible calls" instinct that good agent design already demands.

## The takeaway

Hermes is a bet that the next leap in agents isn't a smarter model, it's a better *memory*. Make the agent write down what it learns, keep it across sessions, and let competence compound instead of resetting to zero every morning. That's a genuinely different shape from the forgetful assistants we've lived with, and it's why it climbed to the top of the usage charts.

It's not magic, and it's not free. It asks for setup patience, a flipped-on config switch most people miss, and a clear-eyed awareness that an agent improving on its own can drift confidently wrong where there's no ground truth. But the core idea, an agent that grows with you instead of forgetting you, is the right direction, and Hermes is one of the clearest, most open expressions of it yet. Whether you pick it or OpenClaw comes down to one honest question: do you want reach, or do you want an agent that learns you?

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['days'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
