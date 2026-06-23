---
title: "From Vibe Coding to Spec Kit: How to Tell an AI What to Build (and Get It Right)"
date: 2026-06-23
excerpt: "Letting an AI just wing it feels amazing for about three days. Then it starts guessing, and the guesses pile up. Spec-driven development, and GitHub's Spec Kit, is the calm, structured way out. Here is how it works, explained from scratch."
tags: [spec-kit, spec-driven-development, ai, developer-tools, vibe-coding, explainer]
---

<style>
.post-fig{margin:2.5rem 0;}
.post-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* two-path compare */
.paths{display:grid;grid-template-columns:1fr 1fr;gap:1rem;max-width:640px;margin:0 auto;}
@media(max-width:580px){.paths{grid-template-columns:1fr;}}
.path{background:var(--surface);border:1px solid var(--border-2);border-radius:14px;padding:1.1rem;}
.path h4{font-family:var(--font-mono);font-size:.78rem;text-transform:uppercase;letter-spacing:.06em;margin:0 0 .8rem;}
.path.vibe h4{color:var(--accent-2);}
.path.spec h4{color:var(--accent);}
.path .step{font-family:var(--font-mono);font-size:.82rem;color:var(--text-2);padding:.4rem .6rem;border-radius:7px;background:var(--surface-2);border:1px solid var(--border);margin-bottom:.4rem;}
.path .arrow{text-align:center;color:var(--text-3);font-size:.9rem;line-height:1;}
.path .loop{color:var(--accent-2);font-weight:600;}

/* the 3-month wall chart */
.wall{max-width:560px;margin:0 auto;}
.wall-bars{display:flex;align-items:flex-end;gap:.5rem;height:170px;padding:0 .4rem;}
.wall-bars .col{flex:1;display:flex;flex-direction:column;align-items:center;justify-content:flex-end;height:100%;}
.wall-bars .bar{width:100%;border-radius:5px 5px 0 0;transition:height .4s var(--ease);}
.wall-bars .bar.v{background:var(--accent-2);}
.wall-bars .bar.s{background:var(--accent);}
.wall-bars .lbl{font-family:var(--font-mono);font-size:.66rem;color:var(--text-3);margin-top:.4rem;}
.wall-key{display:flex;gap:1.3rem;justify-content:center;margin-top:1rem;font-family:var(--font-mono);font-size:.76rem;}
.wall-key span{display:flex;align-items:center;gap:.4rem;color:var(--text-2);}
.wall-key i{width:12px;height:12px;border-radius:3px;display:inline-block;}

/* the pipeline */
.pipe{display:flex;align-items:stretch;gap:.4rem;max-width:680px;margin:0 auto;flex-wrap:wrap;justify-content:center;}
.pstep{flex:1;min-width:110px;background:var(--surface);border:1px solid var(--border-2);border-radius:12px;padding:.9rem .6rem;text-align:center;}
.pstep .cmd{font-family:var(--font-mono);font-size:.74rem;color:var(--accent);display:block;margin-bottom:.4rem;font-weight:600;}
.pstep .what{font-size:.82rem;color:var(--text-2);}
.parrow{display:flex;align-items:center;color:var(--text-3);font-size:1.1rem;}
@media(max-width:600px){.parrow{transform:rotate(90deg);justify-content:center;}}

/* command reference table */
.ctab{width:100%;border-collapse:collapse;font-size:.9rem;}
.ctab th,.ctab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.ctab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.ctab td:first-child{font-family:var(--font-mono);color:var(--accent);white-space:nowrap;}

/* when-to-use scale */
.scale{max-width:600px;margin:0 auto;}
.scale-bar{height:14px;border-radius:999px;background:linear-gradient(90deg,var(--accent-2),var(--accent));margin:.6rem 0;}
.scale-ends{display:flex;justify-content:space-between;font-family:var(--font-mono);font-size:.74rem;color:var(--text-3);}
.scale-row{display:flex;gap:.8rem;align-items:center;margin-top:1rem;}
.scale-row .tag{flex:none;font-family:var(--font-mono);font-size:.72rem;font-weight:600;padding:.25rem .6rem;border-radius:999px;border:1px solid var(--border-2);}
.scale-row .tag.v{color:var(--accent-2);border-color:var(--accent-2);}
.scale-row .tag.s{color:var(--accent);border-color:var(--accent);}
.scale-row .ex{font-size:.88rem;color:var(--text-2);}
</style>

Picture your first week with an AI coding tool. You type "build me a login page," and a working login page appears. It feels like a magic trick. So you keep going. "Add a dark mode." Done. "Now connect it to a database." Done. You are flying.

Then, somewhere around the third week, the cracks show. You ask for one small change and the AI quietly rewrites three files you did not want touched. It forgets a decision you made on day one. It builds a feature that almost matches what you meant, but not quite, and you cannot point to where the misunderstanding happened, because there is nothing to point *at*. Just a long, messy chat history.

This is the wall that everyone hits. And there is a name for the thing that gets you over it: spec-driven development. GitHub built a free toolkit for it called Spec Kit. By the end of this post, you will understand exactly what it is, why it works, and when to reach for it. No prior experience needed. Let me build it up from the ground.

## Two ways to work with an AI

There are basically two styles of building software with an AI today, and it helps to see them side by side.

The first is the one you started with. It even has a fun name now: vibe coding. You prompt, the AI generates, you eyeball it, you prompt again to fix what is off, and you go round and round until it looks right. It is fast, it is playful, and for small throwaway things it is genuinely the best way to work.

The second is more deliberate. You first write down, in plain language, *what* you want and *why*, before any code exists. Then the AI turns that into a plan, the plan into a checklist of tasks, and the tasks into code. The written description, not the chat, becomes the thing everyone trusts.

<figure class="post-fig">
  <div class="paths">
    <div class="path vibe">
      <h4>Vibe coding</h4>
      <div class="step">prompt: "build a login"</div>
      <div class="arrow">&darr;</div>
      <div class="step">AI writes code</div>
      <div class="arrow">&darr;</div>
      <div class="step">you spot a bug, prompt again</div>
      <div class="arrow">&darr;</div>
      <div class="step loop">&#8634; repeat until it looks right</div>
    </div>
    <div class="path spec">
      <h4>Spec-driven</h4>
      <div class="step">write the spec: what &amp; why</div>
      <div class="arrow">&darr;</div>
      <div class="step">AI makes a plan</div>
      <div class="arrow">&darr;</div>
      <div class="step">plan becomes tasks</div>
      <div class="arrow">&darr;</div>
      <div class="step">tasks become code you can trust</div>
    </div>
  </div>
  <figcaption>Same goal, two paths. Vibe coding is a fast conversational loop. Spec-driven development front-loads the thinking into a written document, then lets the agent execute against it. Neither is "better." They are good at different things.</figcaption>
</figure>

The key difference is where the truth lives. In vibe coding, the truth is scattered across a chat you will never read again. In spec-driven development, the truth lives in a written spec that you can read, review, and change on purpose. Someone smart called a spec "version control for your thinking," and that line stuck with me. It is not about writing more documents. It is about making your decisions visible instead of leaving them buried in a conversation.

## Why the vibe eventually breaks

Vibe coding is not wrong. It is just optimised for one thing: speed right now. The cost is hidden, and it shows up later.

Here is the pattern people keep reporting, and it is consistent enough to take seriously. Vibe-coded projects fly for the first stretch, then hit a wall somewhere around the three-month mark, where all the small unspoken decisions and quick patches pile into a tangle that is slow and painful to change. The early speed was real. So was the debt it quietly borrowed.

<figure class="post-fig">
  <div class="wall">
    <div class="wall-bars">
      <div class="col"><div class="bar v" style="height:35%"></div><div class="lbl">week 1</div></div>
      <div class="col"><div class="bar v" style="height:55%"></div><div class="lbl">week 2</div></div>
      <div class="col"><div class="bar v" style="height:80%"></div><div class="lbl">month 1</div></div>
      <div class="col"><div class="bar v" style="height:60%"></div><div class="lbl">month 2</div></div>
      <div class="col"><div class="bar v" style="height:28%"></div><div class="lbl">month 3</div></div>
      <div class="col"><div class="bar s" style="height:50%"></div><div class="lbl">week 1</div></div>
      <div class="col"><div class="bar s" style="height:62%"></div><div class="lbl">month 1</div></div>
      <div class="col"><div class="bar s" style="height:70%"></div><div class="lbl">month 2</div></div>
      <div class="col"><div class="bar s" style="height:78%"></div><div class="lbl">month 3</div></div>
    </div>
    <div class="wall-key">
      <span><i style="background:var(--accent-2)"></i> vibe coding pace</span>
      <span><i style="background:var(--accent)"></i> spec-driven pace</span>
    </div>
  </div>
  <figcaption>Illustrative, not a benchmark. Vibe coding starts faster, then sags as the unspoken decisions compound. Spec-driven development starts slower (you wrote a spec first) but holds its pace because the foundation is written down and stays true. The crossover is the whole argument.</figcaption>
</figure>

Notice what the chart is *not* saying. It is not saying vibe coding is bad. It is saying the two curves cross. For a weekend hack you will never maintain, you want the left side of that picture, all speed, no ceremony. For something that has to keep working and keep changing, you want the right side. The skill is knowing which one you are doing.

## So what is Spec Kit, exactly?

Spec Kit is GitHub's free, open-source toolkit that makes the spec-driven path easy to actually follow. Without it, "write a spec first" is just good advice you will probably skip. Spec Kit turns it into a set of simple commands your AI agent already understands, with templates so you are not staring at a blank page.

You install a small command-line tool and point it at your project:

```
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
specify init my-project
```

Once it is set up, you do not work through a clunky app. You talk to your AI agent the way you already do, but now with a handful of special commands. And here is a detail worth pausing on: Spec Kit works with more than thirty different AI coding agents. Copilot, Claude, Gemini, Cursor, Codex, and many more. You are not locking yourself into one tool. The spec is the constant; you can swap the agent underneath it.

## The five moves

The whole method comes down to a short, ordered pipeline. Each step produces a plain Markdown file, and that file becomes the input to the next step. That is the trick that makes it reliable: the agent is never guessing from a vague prompt, it is always working from the document the previous step wrote down.

<figure class="post-fig">
  <div class="pipe">
    <div class="pstep"><span class="cmd">/constitution</span><span class="what">set the house rules</span></div>
    <div class="parrow">&rarr;</div>
    <div class="pstep"><span class="cmd">/specify</span><span class="what">what &amp; why</span></div>
    <div class="parrow">&rarr;</div>
    <div class="pstep"><span class="cmd">/plan</span><span class="what">how, the tech</span></div>
    <div class="parrow">&rarr;</div>
    <div class="pstep"><span class="cmd">/tasks</span><span class="what">the checklist</span></div>
    <div class="parrow">&rarr;</div>
    <div class="pstep"><span class="cmd">/implement</span><span class="what">build it</span></div>
  </div>
  <figcaption>The Spec Kit pipeline. Each command hands a written artifact to the next. You stay in the driver's seat at every handoff, reading and adjusting before you move on. The agent only writes code at the very end, once everything before it has been agreed on.</figcaption>
</figure>

Let me walk each one in plain terms, because the names are clearer than they sound.

<figure class="post-fig">
  <table class="ctab">
    <thead><tr><th>Command</th><th>What it does, in plain words</th></tr></thead>
    <tbody>
      <tr><td>/speckit.constitution</td><td>Writes down your project's non-negotiable rules once: "always write tests," "use Postgres," "keep it accessible." Every later step has to respect these.</td></tr>
      <tr><td>/speckit.specify</td><td>You describe what you want and why, in plain language. No tech choices yet. This is the heart of it: the spec.</td></tr>
      <tr><td>/speckit.clarify</td><td>The agent asks you about the fuzzy parts before it commits to anything. This is where misunderstandings get caught early, cheaply.</td></tr>
      <tr><td>/speckit.plan</td><td>Now the how. The agent proposes the tech: which framework, which database, how the pieces fit. You review it like a design doc.</td></tr>
      <tr><td>/speckit.tasks</td><td>The plan gets broken into a concrete, ordered checklist of small tasks the agent can tick off one by one.</td></tr>
      <tr><td>/speckit.implement</td><td>Finally, the agent writes the actual code, working through the task list, grounded in everything above.</td></tr>
    </tbody>
  </table>
  <figcaption>The commands, verified from GitHub's own Spec Kit documentation. There are a couple of extra optional quality checks too (an analyze step, a checklist step), but these six are the spine. You will feel in control because you are: nothing surprising happens, because every decision was written down before code existed.</figcaption>
</figure>

The `constitution` deserves a second look, because it is the part people skip and then regret. It is a small file of your project's unbreakable principles. Think of it as the rules of the house that every future feature has to live by. Once it exists, you never have to remind the agent "we always write tests" again, it is baked in. That is a quiet superpower over a long project.

## How is this different from an AGENTS.md file?

If you have read about AI coding setup before, you have probably met `AGENTS.md`, a file that tells an agent how to behave in your project. It is fair to ask how Spec Kit is different, because they sound related. They are, but they are not the same thing, and seeing the difference makes both click.

An `AGENTS.md` is *standing context*. It is always loaded, always true, and it describes the project in general: build commands, code style, boundaries. It is the briefing an agent reads before it does anything at all.

Spec Kit is *per-feature process*. It is not always-on background context. It is the structured path you walk *for one specific thing you are building right now*: this login system, this payment flow. The spec lives and dies with that feature.

<figure class="post-fig">
  <div class="paths">
    <div class="path spec">
      <h4>AGENTS.md</h4>
      <div class="step">always loaded</div>
      <div class="step">describes the whole project</div>
      <div class="step">"how to behave here, in general"</div>
      <div class="step">the standing briefing</div>
    </div>
    <div class="path vibe">
      <h4>Spec Kit</h4>
      <div class="step">used per feature</div>
      <div class="step">describes one thing you're building</div>
      <div class="step">"what to build, step by step"</div>
      <div class="step">the working plan</div>
    </div>
  </div>
  <figcaption>Not rivals, teammates. AGENTS.md is the always-on house rules for the whole project. Spec Kit is the deliberate plan for one feature. In fact Spec Kit's own constitution file plays a role much like AGENTS.md does, the lasting rules, while the spec, plan, and tasks are fresh for each feature. You can absolutely use both at once.</figcaption>
</figure>

## How I have actually been using it

Let me get specific, because I have been trialing this on a real project, not just reading about it. We are migrating a bunch of our UI tables over to TanStack-based tables, and that is exactly the kind of work where spec-driven shines: it is repetitive across many pages, but each page has its own fiddly rules.

So instead of opening one page and vibe-coding it, I asked the agent to *research first*. For each page, before any code, it wrote up a plan: the full table, the validations, the listing behaviour, the conditions for when each button should show or hide, the role-based access rules for that page. All of it written down as a document I could read. Only then did we move to actually building, feeding it the relevant context in pieces as we went.

Here is the honest part, and I am leaving it in because I think it is useful: I am not certain I followed the textbook flow perfectly. I leaned heavily on the research-and-plan documents and was looser about the strict command sequence. And you know what? It still helped enormously, because the value was never really about ceremony. It was about forcing the messy decisions, the RBAC, the button conditions, the edge cases, out of my head and onto the page *before* the agent started writing. Once those were written down, the implementation stopped being a guessing game.

That is the thing I want a beginner to take away. You do not have to perform the method flawlessly to get most of its benefit. Even just "make the agent write the plan before it writes the code, and read that plan" gets you eighty percent of the way. Spec Kit makes that habit easy and repeatable, but the habit is the real win.

## When should you actually use it?

Here is the honest part, and it is the part that will make you trust the method rather than just follow it blindly. Spec Kit is not free. Writing and reviewing a spec takes time that vibe coding skips entirely. For a tiny script, that time is pure overhead with nothing to show for it. So the answer to "should I use this?" is genuinely "it depends," and here is how to think about it.

<figure class="post-fig">
  <div class="scale">
    <div class="scale-ends"><span>just needs to work once</span><span>has to last and keep changing</span></div>
    <div class="scale-bar"></div>
    <div class="scale-row"><span class="tag v">vibe it</span><span class="ex">a one-off script, a quick prototype, exploring an idea, a weekend hack you will throw away</span></div>
    <div class="scale-row"><span class="tag s">spec it</span><span class="ex">a real feature in a real product, anything a team shares, code that has to be correct, anything you will maintain for months</span></div>
  </div>
  <figcaption>It is a slider, not a switch. The more it matters that the thing is correct and survives change, the more a spec pays for itself. The more disposable it is, the more a spec is just friction. Most good developers live in both modes and switch on purpose.</figcaption>
</figure>

There is even a lovely middle path that the best teams use. You vibe-code first, fast and loose, just to *discover* what you actually want, treating it as a sketch. Then, once you understand the thing, you write the spec and build the real version properly. Exploration with vibes, construction with specs. You get the joy of the fast loop and the durability of the written plan, and you never have to feel guilty about either one.

## Why I think you will enjoy this

Most advice about "doing things properly" feels like eating your vegetables. This one does not, and that is what surprised me. The first time you watch an agent work through a clean task list it wrote from a spec you reviewed, calmly, in order, touching only what it should, something relaxes. You stop bracing for the AI to go rogue, because you already agreed on everything before it started.

That is the real gift here. Not more process for its own sake, but *confidence*. You hand off more to the AI, not less, precisely because you set the boundaries up front. The agent gets to move fast inside lines you drew. You get to stop babysitting every prompt.

So play with vibe coding, it is genuinely delightful, and it is the right tool more often than purists admit. But the next time you start something you actually care about, something you want to still be working in three months, try writing the spec first. Install Spec Kit, run `/speckit.specify`, and describe what you want like you are explaining it to a thoughtful friend. Then watch the agent build it from your words instead of its guesses.

Your thinking, written down, in order. It turns out that is most of the job.
