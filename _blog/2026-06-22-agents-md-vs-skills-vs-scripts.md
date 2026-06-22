---
title: "AGENTS.md vs Skills vs Plain Scripts: What Goes Where, and Why It Matters"
date: 2026-06-22
excerpt: "Three ways to teach an AI agent how to work in your project, and they are not interchangeable. Get the wrong one in the wrong place and your agent quietly gets worse, not better. Here is how to think about it."
tags: [ai, agents, agents-md, skills, developer-tools, explainer]
---

<style>
.post-fig{margin:2.5rem 0;}
.post-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* file card */
.fcard{max-width:560px;margin:0 auto;background:var(--surface);border:1px solid var(--border-2);border-radius:14px;overflow:hidden;box-shadow:var(--glow);}
.fcard-bar{display:flex;align-items:center;gap:.6rem;padding:.6rem .9rem;background:var(--surface-2);border-bottom:1px solid var(--border);}
.fcard-bar .dot{width:10px;height:10px;border-radius:50%;}
.fcard-bar .name{font-family:var(--font-mono);font-size:.78rem;color:var(--text-2);margin-left:.3rem;}
.fcard-body{font-family:var(--font-mono);font-size:.84rem;line-height:1.8;padding:.9rem 1.1rem;color:var(--text-2);}
.fcard-body .k{color:var(--accent);}
.fcard-body .c{color:var(--text-3);}

/* three-tier disclosure */
.tiers{display:flex;flex-direction:column;gap:.7rem;max-width:600px;margin:0 auto;}
.tier{display:flex;align-items:center;gap:1rem;background:var(--surface);border:1px solid var(--border);border-radius:12px;padding:.9rem 1.1rem;}
.tier .lvl{flex:none;width:34px;height:34px;border-radius:50%;background:var(--grad);color:var(--accent-ink);font-family:var(--font-mono);font-weight:600;display:flex;align-items:center;justify-content:center;}
.tier .body{flex:1;}
.tier .body b{color:var(--text);display:block;font-size:.95rem;}
.tier .body span{font-size:.85rem;color:var(--text-2);}
.tier .cost{flex:none;font-family:var(--font-mono);font-size:.74rem;color:var(--accent);text-align:right;white-space:nowrap;}
.tier.t1{border-color:var(--border);}
.tier.t2{border-color:var(--border-2);}
.tier.t3{border-color:var(--accent);}

/* decision flow */
.decide{display:flex;flex-direction:column;gap:.6rem;max-width:600px;margin:0 auto;}
.drow{display:flex;align-items:center;gap:.8rem;background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:.7rem .9rem;}
.drow .q{flex:1;color:var(--text-2);font-size:.92rem;}
.drow .q b{color:var(--text);}
.drow .a{flex:none;font-family:var(--font-mono);font-size:.78rem;font-weight:600;padding:.3rem .7rem;border-radius:999px;background:var(--surface-2);border:1px solid var(--accent);color:var(--accent);}

/* bad-info animation */
.drift{max-width:560px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.drift-line{display:flex;align-items:center;gap:.7rem;font-family:var(--font-mono);font-size:.84rem;}
.drift-said{flex:1;background:var(--surface);border:1px solid var(--border);border-radius:8px;padding:.5rem .7rem;color:var(--text-2);}
.drift-did{flex:none;font-size:.78rem;color:var(--text-3);}
.drift-bad{border-color:var(--accent);}
.drift-bad .x{color:var(--accent);font-weight:600;}

/* comparison table */
.ctab{width:100%;border-collapse:collapse;font-size:.9rem;}
.ctab th,.ctab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.ctab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.ctab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}
</style>

This week I gave an AI agent a small, wrong instruction. Not on purpose. I had a file in my project telling it how to behave, and one line in that file pointed at something that had since moved. The agent read the file, trusted it completely, and confidently did the wrong thing. No error. No warning. Just a quietly worse outcome that took me a minute to even notice.

That is the whole reason this post exists. We now have a few different ways to hand an AI agent instructions about our projects, and they look similar enough that people throw them around interchangeably. They are not interchangeable. Each one has a job, a cost, and a failure mode. Get them mixed up and you do not get an error message, you get an agent that is subtly worse than the one you started with.

There is a bigger shift underneath this. For years, the way we automated work was a folder full of little scripts. A `deploy.sh` here, a `seed-db.py` there, a dozen helpers that each did one thing. They worked beautifully right up until an API changed, a token expired, or the task needed one more condition the script did not anticipate. In 2026 the industry is steadily moving past that brittleness toward agents that read context, decide, and adapt. But "give it to an agent" is not one decision. It splits into several, and that is what trips people up.

Let me walk through the pieces you actually run into: a project's `AGENTS.md` file, Skills, the plain scripts you already have, and where MCP fits alongside all of it. By the end you will know exactly what belongs where, and why putting the wrong thing in the wrong place hurts.

## First, the thing nobody tells you

Every coding agent, before it touches your project, does the same thing a new hire does on their first day. It looks around. It reads the file tree, the package manifest, the README. The problem is that a README was written for a human. It explains what the project *is*. It does not explain how an agent should *work* in it: which build command with which flags, which files to never touch, what the house style is when it differs from the default.

That gap is what `AGENTS.md` fills. It is a plain Markdown file you drop at the root of your repo, and it has become a genuine open standard. As of 2026 it is read by more than thirty different agents, across tens of thousands of repositories. OpenAI's Codex, Cursor, GitHub Copilot, Gemini's CLI, and many more all look for it.

<figure class="post-fig">
  <div class="fcard">
    <div class="fcard-bar"><span class="dot" style="background:#F2555A"></span><span class="dot" style="background:#F5BD4F"></span><span class="dot" style="background:#5FD068"></span><span class="name">AGENTS.md</span></div>
    <div class="fcard-body">
<span class="c"># Build</span><br>
<span class="k">build:</span> npm run build<br>
<span class="k">test:</span> npm test -- --silent<br>
<br>
<span class="c"># Rules</span><br>
- never edit files in /vendor<br>
- commit messages: present tense<br>
- the API client lives in src/api, not src/lib
    </div>
  </div>
  <figcaption>An AGENTS.md is just Markdown. No required fields, no schema. The point is to write down the things a README leaves out: exact commands, hard boundaries, and the small rules that are obvious to you and invisible to a newcomer.</figcaption>
</figure>

I actually made one of these for this very website. It is not committed publicly, it sits on my machine, and it carries the rules I care about: never publish my resume, do not invent fake citations, keep a specific writing voice. When an agent works on the site, it reads that file first and stays inside the lines. That is `AGENTS.md` doing its job: persistent project context, loaded every time, shaping everything that follows.

## And here is the trap

If `AGENTS.md` is loaded every time and shapes everything, then a wrong line in it poisons everything. This is not hypothetical hand-wringing. There is real 2026 research on exactly this, looking at over a hundred real-world repositories.

The finding is uncomfortable and worth sitting with. Context files that were generated automatically by an LLM actually *reduced* how often agents succeeded at their tasks, while increasing cost by more than twenty percent. Human-written files did better, but only marginally, and only when they were short and precise. A bloated or slightly-stale instruction file is not neutral. It is a tax you pay on every single request, and sometimes it actively steers the agent wrong.

<figure class="post-fig">
  <div class="drift">
    <div class="drift-line"><div class="drift-said">"the API client lives in src/api"</div><div class="drift-did">&rarr; correct, agent edits the right file</div></div>
    <div class="drift-line drift-bad"><div class="drift-said">"the API client lives in src/api"</div><div class="drift-did"><span class="x">&rarr; but you moved it to src/core last month</span></div></div>
  </div>
  <figcaption>Same line, two different worlds. The agent has no way to know your file is stale. It trusts the instruction over the actual code, edits the wrong place, and reports success. The file did not get an error, it got obeyed.</figcaption>
</figure>

This is exactly what happened to me. The lesson is not "do not use `AGENTS.md`." It is the opposite of how most people treat it. Keep it short. Keep it true. Treat it like code that can rot, because it can. Every line in there runs on every task, so every stale line is a bug that fires every time. When mine bit me, the fix was not to add more instructions. It was to delete the wrong one.

## Skills: instructions that show up only when needed

Now, the second tool, and the one people find most confusing because it sounds like the first.

`AGENTS.md` is always on. But most expertise is not needed most of the time. The detailed steps for filling out a PDF form are useless on a task about database migrations. If you stuffed every specialised workflow into your always-on context, you would drown the agent in irrelevant detail and pay for it on every request. That is precisely the bloat the research warned about.

Skills solve this with a beautifully simple idea: load the detail only when it is relevant. A Skill, in Anthropic's design, is just a folder with a file called `SKILL.md` inside it. That file starts with a tiny bit of structured metadata, called frontmatter, and only two fields are required: a `name` and a `description`.

<figure class="post-fig">
  <div class="fcard">
    <div class="fcard-bar"><span class="dot" style="background:#F2555A"></span><span class="dot" style="background:#F5BD4F"></span><span class="dot" style="background:#5FD068"></span><span class="name">pdf-forms/SKILL.md</span></div>
    <div class="fcard-body">
<span class="c">---</span><br>
<span class="k">name:</span> pdf-forms<br>
<span class="k">description:</span> Fill and extract fields from PDF forms.<br>
<span class="c">---</span><br>
<br>
<span class="c"># How to fill a PDF form</span><br>
<span class="c">(the full instructions live down here,</span><br>
<span class="c"> read only when the task actually needs them)</span>
    </div>
  </div>
  <figcaption>A Skill is a folder with a SKILL.md. The frontmatter up top (name and description, both required) is the only part the agent reads by default. The body below is loaded on demand.</figcaption>
</figure>

The clever part is how it loads, in three tiers. This is the idea called progressive disclosure, and once it clicks, you see why Skills scale where a giant instruction file does not.

<figure class="post-fig">
  <div class="tiers">
    <div class="tier t1">
      <div class="lvl">1</div>
      <div class="body"><b>At startup: just the name and description</b><span>Every installed Skill contributes only its one-line summary, so the agent knows what exists and when each might apply.</span></div>
      <div class="cost">~100 tokens<br>per skill</div>
    </div>
    <div class="tier t2">
      <div class="lvl">2</div>
      <div class="body"><b>On a match: the full SKILL.md</b><span>When your task matches a description, the agent reads that skill's full instructions. Nothing else loads.</span></div>
      <div class="cost">under<br>~5k tokens</div>
    </div>
    <div class="tier t3">
      <div class="lvl">3</div>
      <div class="body"><b>Only if needed: bundled files and scripts</b><span>Extra reference files or code the skill ships with are opened or run only at the exact moment they are required.</span></div>
      <div class="cost">loaded<br>on demand</div>
    </div>
  </div>
  <figcaption>Three tiers, like a manual with a table of contents, then chapters, then an appendix. Because tier one is so cheap, you can have fifty Skills installed and pay almost nothing for the forty-nine that are not relevant to what you are doing right now.</figcaption>
</figure>

That economy is the whole point. With an always-on file, everything you add is loaded forever. With Skills, the agent reads a one-line summary of each, and only opens the full thing for the one that matches the task in front of it. You get a big library of expertise without a big bill on every request.

## So how is a Skill different from just keeping a script?

This is the question I think is genuinely worth asking, because teams already have scripts. A `deploy.sh`, a `seed-db.py`, a folder of little helpers. If the agent can run a script, why wrap it in a Skill at all?

The difference is *discovery and judgement*, not execution.

A plain script sits in your repo doing nothing until a human decides to run it and knows which one and with what arguments. The agent will not reach for it unless you explicitly tell it to, every time. A script is a tool waiting for someone who already knows it exists.

A Skill is that same capability, but it announces itself. Its description sits in the agent's awareness from the start, so when a relevant task comes up, the agent recognises "this is a job for that" on its own, reads the how-to, and proceeds. And critically, a Skill can *bundle a script and run it without ever loading the script's code into the conversation.* The agent runs the tool, gets the result, and spends none of its limited attention reading the implementation. The script stays a black box that just works.

<figure class="post-fig">
  <table class="ctab">
    <thead><tr><th></th><th>A plain script</th><th>A Skill</th></tr></thead>
    <tbody>
      <tr><td>Discovery</td><td>Human must know it exists and invoke it</td><td>Agent notices it fits the task on its own</td></tr>
      <tr><td>Guidance</td><td>None, it is just a file</td><td>Carries instructions on when and how to use it</td></tr>
      <tr><td>Context cost</td><td>Zero until run, but invisible to the agent</td><td>~one line until needed, then loads in tiers</td></tr>
      <tr><td>Can bundle code</td><td>It is the code</td><td>Yes, and runs it without reading it into context</td></tr>
    </tbody>
  </table>
  <figcaption>A script is a capability. A Skill is a capability that knows when to volunteer itself, explains how it should be used, and can still run real code underneath. Skills do not replace scripts. They wrap them in discovery and judgement.</figcaption>
</figure>

So you do not throw away your scripts. The good pattern is often a Skill *around* a script: the script does the mechanical work, and the Skill is the thin layer that tells the agent this tool exists, when it applies, and how to call it.

## Where does MCP fit in all this?

If you have spent any time around AI agents lately, one more term keeps coming up, and it gets tangled with Skills constantly: MCP, the Model Context Protocol. People ask "should I build a Skill or an MCP server?" as if they are two answers to the same question. They are not. They answer different questions, and once you see the split it stops being confusing.

Here is the cleanest way I have heard it put, and it comes straight from Anthropic: **MCP connects the agent to your data. Skills teach the agent what to do with that data.**

Think about querying your company database. Before the agent can do anything, it needs to be able to *reach* the database at all, to open a connection, run a query, get rows back. That reaching-out, that plumbing to an external system, is MCP's job. It is about connectivity. Now, separately, there is the question of *how your team wants queries done*, always filter by date range first, never run an unbounded scan, format results a certain way. That know-how is a Skill. One gets the agent to the data, the other tells it how to behave once it is there.

<figure class="post-fig">
  <div class="decide">
    <div class="drow"><div class="q"><b>MCP</b> is about <b>connection</b>. "Let the agent reach this database / repo / tool."</div><div class="a">connect to</div></div>
    <div class="drow"><div class="q"><b>Skills</b> are about <b>procedure</b>. "Here is how we want that tool used."</div><div class="a">how to use</div></div>
    <div class="drow"><div class="q"><b>Scripts</b> are the <b>mechanical action</b> underneath either one.</div><div class="a">the doing</div></div>
  </div>
  <figcaption>Three different axes, not three competing choices. MCP opens the door to an external system. A Skill is the instruction manual for working inside it. A script is the actual mechanical step. The strongest setups use all three together: MCP for access, a Skill for judgement, a script for the work.</figcaption>
</figure>

So the honest answer to "Skill or MCP?" is usually *both*. MCP gives the agent its hands, the ability to touch external systems. Skills give it the training, the knowledge of how your team does the thing. I have built MCP servers myself, including one for OpenStack, and the mental model that finally stuck was exactly this: the MCP server is the wiring to the infrastructure, and everything about *how* to use it well is a separate, teachable layer on top.

## Putting it together: what goes where

Here is the mental model I have landed on. Three questions, three homes.

<figure class="post-fig">
  <div class="decide">
    <div class="drow"><div class="q">Is this <b>always true</b> about the project? (build commands, hard rules, boundaries)</div><div class="a">AGENTS.md</div></div>
    <div class="drow"><div class="q">Is this <b>expertise for a specific kind of task</b> the agent should reach for only sometimes?</div><div class="a">a Skill</div></div>
    <div class="drow"><div class="q">Is this <b>a mechanical action</b> with no judgement, that a human or a Skill triggers?</div><div class="a">a script</div></div>
  </div>
  <figcaption>The same litmus test, every time. Always-on truth goes in the always-on file. Sometimes-needed know-how becomes a Skill. Pure mechanical work stays a script, ideally wrapped by a Skill so the agent can find it.</figcaption>
</figure>

And one rule that sits above all three, because it is the one that actually bit me: **whatever you write down, keep it true and keep it small.** The research is blunt about it. More context is not better. A short, accurate `AGENTS.md` beats a long one. A handful of well-described Skills beats a sprawling pile. The cost of a wrong or bloated instruction is not paid once when you write it, it is paid on every request the agent ever makes, forever, until you notice and fix it.

I learned that this week, from one stale line, in a file I wrote myself. The agent did exactly what I told it. That was the problem. These tools are powerful precisely because the agent trusts them completely, which means the responsibility for keeping them honest is entirely yours.

Write less. Keep it true. Let the agent reach for the rest only when it needs it.
