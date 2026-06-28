---
title: "Skills: The Quiet Standard Every AI Coding Tool Agreed On"
date: 2026-06-28
excerpt: "A folder, a Markdown file, two lines of frontmatter. That is a Skill. Somehow it became the one thing Claude, Cursor, Bob, Codex, and a dozen other tools all read the same way. Here is how Skills actually work, why the design is clever, and where they earn their keep."
tags: [ai, skills, agents, developer-tools, explainer]
---

<style>
.sk-fig{margin:2.5rem 0;}
.sk-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* file card */
.sk-file{max-width:560px;margin:0 auto;background:var(--surface);border:1px solid var(--border-2);border-radius:14px;overflow:hidden;box-shadow:var(--glow);}
.sk-file-bar{display:flex;align-items:center;gap:.5rem;padding:.6rem .9rem;background:var(--surface-2);border-bottom:1px solid var(--border);}
.sk-file-bar .d{width:10px;height:10px;border-radius:50%;}
.sk-file-bar .nm{font-family:var(--font-mono);font-size:.78rem;color:var(--text-2);margin-left:.3rem;}
.sk-file-body{font-family:var(--font-mono);font-size:.84rem;line-height:1.85;padding:.95rem 1.1rem;color:var(--text-2);white-space:pre-wrap;}
.sk-file-body .k{color:var(--accent);}
.sk-file-body .c{color:var(--text-3);}
.sk-file-body .h{color:var(--text);font-weight:600;}

/* progressive disclosure animation */
.sk-pd{max-width:620px;margin:0 auto;}
.sk-pd-stage{display:flex;align-items:flex-start;gap:1rem;padding:.85rem 1rem;border:1px solid var(--border);border-radius:12px;background:var(--surface);margin-bottom:.7rem;opacity:.4;transform:translateY(4px);transition:opacity .5s ease,transform .5s ease,border-color .5s ease;}
.js .sk-pd-stage{opacity:.18;}
.sk-pd.go .sk-pd-stage{opacity:1;transform:none;}
.sk-pd.go .sk-pd-stage.s1{transition-delay:.1s;border-color:var(--border-2);}
.sk-pd.go .sk-pd-stage.s2{transition-delay:1.1s;border-color:var(--border-2);}
.sk-pd.go .sk-pd-stage.s3{transition-delay:2.1s;border-color:var(--accent);}
.sk-pd-lvl{flex:none;width:34px;height:34px;border-radius:50%;background:var(--grad);color:var(--accent-ink);font-family:var(--font-mono);font-weight:600;display:flex;align-items:center;justify-content:center;}
.sk-pd-txt{flex:1;}
.sk-pd-txt b{color:var(--text);display:block;font-size:.95rem;}
.sk-pd-txt span{font-size:.86rem;color:var(--text-2);}
.sk-pd-cost{flex:none;font-family:var(--font-mono);font-size:.74rem;color:var(--accent);text-align:right;white-space:nowrap;align-self:center;}

/* token bar chart */
.sk-bars{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:1rem;}
.sk-bar-row .lab{display:flex;justify-content:space-between;font-family:var(--font-mono);font-size:.8rem;color:var(--text-2);margin-bottom:.35rem;}
.sk-bar-row .lab b{color:var(--text);}
.sk-bar-track{height:18px;background:var(--surface-2);border:1px solid var(--border);border-radius:999px;overflow:hidden;}
.sk-bar-fill{height:100%;border-radius:999px;background:var(--grad);width:0;transition:width 1.1s cubic-bezier(.2,.7,.2,1);}
.sk-bars.go .sk-bar-fill{width:var(--w);}

/* loaded-vs-idle context window */
.sk-ctx{max-width:560px;margin:0 auto;display:flex;flex-wrap:wrap;gap:.4rem;justify-content:center;}
.sk-slot{font-family:var(--font-mono);font-size:.72rem;padding:.3rem .55rem;border-radius:7px;border:1px solid var(--border);background:var(--surface);color:var(--text-3);}
.sk-slot.on{border-color:var(--accent);color:var(--accent);background:transparent;font-weight:600;}

/* cross-tool grid */
.sk-tools{display:flex;flex-wrap:wrap;gap:.5rem;justify-content:center;max-width:640px;margin:0 auto;}
.sk-tool{font-family:var(--font-mono);font-size:.82rem;padding:.45rem .8rem;border-radius:999px;border:1px solid var(--border-2);background:var(--surface);color:var(--text-2);}
.sk-tool.hl{border-color:var(--accent);color:var(--accent);font-weight:600;}

/* write-once flow */
.sk-flow{max-width:600px;margin:0 auto;}
.sk-flow .one{text-align:center;font-family:var(--font-mono);font-size:.82rem;color:var(--accent);border:1px dashed var(--accent);border-radius:10px;padding:.6rem;margin-bottom:1rem;}
.sk-flow .fan{display:flex;flex-wrap:wrap;gap:.5rem;justify-content:center;}
.sk-flow .fan .leaf{opacity:0;transform:translateY(6px);transition:opacity .4s ease,transform .4s ease;font-family:var(--font-mono);font-size:.8rem;padding:.4rem .7rem;border:1px solid var(--border-2);border-radius:8px;background:var(--surface);color:var(--text-2);}
.sk-flow.go .fan .leaf{opacity:1;transform:none;}
.sk-flow.go .fan .leaf:nth-child(1){transition-delay:.1s}
.sk-flow.go .fan .leaf:nth-child(2){transition-delay:.25s}
.sk-flow.go .fan .leaf:nth-child(3){transition-delay:.4s}
.sk-flow.go .fan .leaf:nth-child(4){transition-delay:.55s}
.sk-flow.go .fan .leaf:nth-child(5){transition-delay:.7s}
.sk-flow.go .fan .leaf:nth-child(6){transition-delay:.85s}

/* skills vs rules toggle */
.sk-vs{display:grid;grid-template-columns:1fr 1fr;gap:.8rem;max-width:640px;margin:0 auto;}
@media(max-width:560px){.sk-vs{grid-template-columns:1fr;}}
.sk-vs .col{border:1px solid var(--border);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);}
.sk-vs .col h4{margin:0 0 .6rem;font-size:.9rem;color:var(--text);}
.sk-vs .col.rule{border-color:var(--border-2);}
.sk-vs .col.skill{border-color:var(--accent);}
.sk-vs .col ul{margin:0;padding-left:1.1rem;}
.sk-vs .col li{font-size:.85rem;color:var(--text-2);margin:.3rem 0;}
.sk-vs .col .tag{font-family:var(--font-mono);font-size:.72rem;color:var(--accent);}

/* comparison table */
.sk-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.sk-tab th,.sk-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.sk-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.sk-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}
.sk-tab code{font-size:.82rem;}

@media (prefers-reduced-motion: reduce){
  .sk-pd-stage,.sk-bar-fill,.sk-flow .fan .leaf{transition:none !important;}
  .sk-pd .sk-pd-stage{opacity:1 !important;transform:none !important;}
  .sk-bars .sk-bar-fill{width:var(--w) !important;}
  .sk-flow .fan .leaf{opacity:1 !important;transform:none !important;}
}
</style>

Here is a thing that almost never happens in this industry: a bunch of competing companies, all racing each other, all with their own opinions, quietly agreed on the same file format. No committee. No standards body sending out a 200-page PDF. Just a folder with a Markdown file in it, and one by one, the tools started reading it.

That file is called `SKILL.md`, and the folder it lives in is a Skill. By early 2026, Claude, Cursor, IBM Bob, OpenAI Codex, Gemini CLI, GitHub Copilot, JetBrains Junie, and more than a dozen others all read it. Same shape, same rules, mostly the same behaviour. If you write a Skill for one of them, it tends to just work in the others.

I want to walk through what a Skill actually is, the one design idea that makes the whole thing tick, and then the part nobody explains well: *where Skills earn their keep*, with real scenarios. No copied docs, no hand-waving. Let me show you the moving parts.

## A Skill is smaller than you think

Strip away the hype and a Skill is a folder. Inside it, one required file, `SKILL.md`. At the top of that file, a few lines of YAML with exactly two things that matter: a `name` and a `description`. Below that, plain instructions written for the agent. That is the whole minimum.

<figure class="sk-fig">
  <div class="sk-file">
    <div class="sk-file-bar"><span class="d" style="background:#F2555A"></span><span class="d" style="background:#F5BD4F"></span><span class="d" style="background:#5FD068"></span><span class="nm">pdf-processing/SKILL.md</span></div>
    <div class="sk-file-body"><span class="c">---</span>
<span class="k">name:</span> pdf-processing
<span class="k">description:</span> Extract text and tables from PDFs,
  fill forms, merge documents. Use when the
  user mentions PDFs, forms, or extraction.
<span class="c">---</span>

<span class="h"># PDF Processing</span>

<span class="c">## Quick start</span>
Use pdfplumber to pull text from a page.
For form filling, see <span class="k">FORMS.md</span>.
For the full field API, see <span class="k">REFERENCE.md</span>.</div>
  </div>
  <figcaption>The whole contract. A name, a description that says what it does AND when to use it, then instructions. The name is lowercase letters, numbers and hyphens, capped at 64 characters. The description maxes out around 1024 characters. Everything else is optional.</figcaption>
</figure>

That `description` line is doing more work than it looks. It is not decoration. It is the trigger. The agent reads every Skill's description up front and uses it to decide, later, whether this Skill is relevant to what you just asked. A vague description means the Skill never fires when it should. A sharp one ("use when the user mentions PDFs, forms, or extraction") means it fires at exactly the right moment. Writing that one line well is most of the skill of writing a Skill.

## The one idea that makes it work: progressive disclosure

Here is the problem Skills are quietly solving. You might have fifty Skills installed. If the agent loaded all fifty into its head every time you typed something, your context window would be full before you asked your first question. Useless.

So Skills don't load all at once. They load in stages, only as far as the task needs. This is the whole trick, and it is called progressive disclosure. Watch it happen:

<figure class="sk-fig">
  <div class="sk-pd" id="pd">
    <div class="sk-pd-stage s1">
      <div class="sk-pd-lvl">1</div>
      <div class="sk-pd-txt"><b>Metadata, always loaded</b><span>Just the name and description of every Skill. Enough to know it exists and when it might matter.</span></div>
      <div class="sk-pd-cost">~100 tokens<br>per skill</div>
    </div>
    <div class="sk-pd-stage s2">
      <div class="sk-pd-lvl">2</div>
      <div class="sk-pd-txt"><b>Instructions, loaded when triggered</b><span>You ask something that matches a description. Only now does the agent read the full SKILL.md body.</span></div>
      <div class="sk-pd-cost">under 5k<br>tokens</div>
    </div>
    <div class="sk-pd-stage s3">
      <div class="sk-pd-lvl">3</div>
      <div class="sk-pd-txt"><b>Resources and code, loaded as needed</b><span>FORMS.md, a schema, a script. Read or run only if this specific task touches them.</span></div>
      <div class="sk-pd-cost">effectively<br>unlimited</div>
    </div>
  </div>
  <figcaption>Three levels. The agent walks down them only as far as the job requires, like opening a manual to the table of contents first, then one chapter, then one appendix. Most tasks never reach level 3.</figcaption>
</figure>

The reason this matters becomes obvious the moment you put real numbers on it. The cost of *knowing a Skill exists* is tiny. The cost of *actually using it* only shows up when you use it. And the bundled files, the reference docs and scripts, cost nothing at all until the moment they are opened.

<figure class="sk-fig">
  <div class="sk-bars" id="bars">
    <div class="sk-bar-row">
      <div class="lab"><b>Level 1: metadata (per skill)</b><span>~100 tokens</span></div>
      <div class="sk-bar-track"><div class="sk-bar-fill" style="--w:3%"></div></div>
    </div>
    <div class="sk-bar-row">
      <div class="lab"><b>Level 2: SKILL.md body</b><span>under 5,000 tokens</span></div>
      <div class="sk-bar-track"><div class="sk-bar-fill" style="--w:33%"></div></div>
    </div>
    <div class="sk-bar-row">
      <div class="lab"><b>Level 3: bundled files, until opened</b><span>0 tokens</span></div>
      <div class="sk-bar-track"><div class="sk-bar-fill" style="--w:1%"></div></div>
    </div>
  </div>
  <figcaption>Why you can install dozens of Skills without paying for them. Level 1 is the only thing that is always present, and it is almost nothing. A Skill can bundle an entire API reference and a folder of scripts, and that bundle sits on disk at zero context cost until a task actually reaches for it.</figcaption>
</figure>

There is a second, sneakier win hiding in level 3. When a Skill bundles a script, say `validate_form.py`, the agent does not read the code into its head and reason about it. It just *runs* it and reads the output. "Validation passed." That is it. The script's hundred lines never touch the context window. For anything that needs to be exact and repeatable, a regex, a calculation, a file transform, this is both cheaper and far more reliable than asking the model to generate the equivalent on the fly.

So a Skill quietly hands the agent the right tool for each kind of work: prose instructions for judgement, reference files for facts, and scripts for the things that must be deterministic.

## What is actually sitting in context right now

Picture the agent's context window as a row of slots. With ten Skills installed, here is what is loaded before you have asked anything, and what changes the instant you ask about a PDF:

<figure class="sk-fig">
  <div class="sk-ctx">
    <span class="sk-slot">git-commit</span>
    <span class="sk-slot">pdf-processing</span>
    <span class="sk-slot">db-migration</span>
    <span class="sk-slot">react-forms</span>
    <span class="sk-slot">k8s-deploy</span>
    <span class="sk-slot">api-docs</span>
    <span class="sk-slot">test-writer</span>
    <span class="sk-slot">changelog</span>
    <span class="sk-slot">i18n</span>
    <span class="sk-slot">security-audit</span>
  </div>
  <figcaption>At rest: ten descriptions, one line each. The agent knows what every Skill is for and almost nothing else. Total cost, a rounding error.</figcaption>
</figure>

<figure class="sk-fig">
  <div class="sk-ctx">
    <span class="sk-slot">git-commit</span>
    <span class="sk-slot on">pdf-processing ← full SKILL.md loaded</span>
    <span class="sk-slot">db-migration</span>
    <span class="sk-slot">react-forms</span>
    <span class="sk-slot">k8s-deploy</span>
    <span class="sk-slot">api-docs</span>
    <span class="sk-slot">test-writer</span>
    <span class="sk-slot">changelog</span>
    <span class="sk-slot">i18n</span>
    <span class="sk-slot">security-audit</span>
  </div>
  <figcaption>You ask: "pull the text out of this invoice." One description matched. That one Skill expands to its full instructions; the other nine stay exactly as they were. Nothing else moved. This is progressive disclosure doing its quiet job.</figcaption>
</figure>

## The part that surprised me: it travels between tools

This is the bit that made me sit up. Skills are not a Claude feature that other tools cloned badly. They are close enough to a shared format that a Skill written for one tool is read by the next.

Cursor is the clearest example. It looks for Skills in its own folders, of course, but it *also* reads `.claude/skills/`, `.codex/skills/`, and `.agents/skills/`. So a Skill you wrote for Claude Code, sitting in `.claude/skills/`, is picked up by Cursor with no changes. Write once, and a whole row of tools can use it.

<figure class="sk-fig">
  <div class="sk-flow" id="flow">
    <div class="one">one folder: my-project/.claude/skills/deploy-runbook/SKILL.md</div>
    <div class="fan">
      <span class="leaf">Claude Code reads it</span>
      <span class="leaf">Cursor reads it</span>
      <span class="leaf">Codex reads it</span>
      <span class="leaf">Gemini CLI reads it</span>
      <span class="leaf">Bob reads its own copy</span>
      <span class="leaf">…and a dozen more</span>
    </div>
  </div>
  <figcaption>The same SKILL.md, discovered by multiple agents. Cursor explicitly reads .claude/skills/ and .codex/skills/ alongside its own .cursor/skills/, so cross-tool teams don't maintain three versions of the same runbook.</figcaption>
</figure>

By early 2026 the list of tools that understand Skills had grown past sixteen. Here is the rough shape of who is in:

<figure class="sk-fig">
  <div class="sk-tools">
    <span class="sk-tool hl">Claude Code</span>
    <span class="sk-tool hl">Claude (web)</span>
    <span class="sk-tool hl">Cursor</span>
    <span class="sk-tool hl">IBM Bob</span>
    <span class="sk-tool">OpenAI Codex</span>
    <span class="sk-tool">Gemini CLI</span>
    <span class="sk-tool">GitHub Copilot</span>
    <span class="sk-tool">VS Code</span>
    <span class="sk-tool">Junie (JetBrains)</span>
    <span class="sk-tool">OpenCode</span>
    <span class="sk-tool">Amp</span>
    <span class="sk-tool">Goose</span>
    <span class="sk-tool">OpenHands</span>
    <span class="sk-tool">…and more</span>
  </div>
  <figcaption>A partial map of the ecosystem. The point is not the exact count, which keeps moving. The point is that for the first time, the instructions you write for an agent are not locked to the agent you wrote them for.</figcaption>
</figure>

Now, "mostly compatible" is not "identical," and the differences are worth knowing. Same idea, slightly different dialects:

<figure class="sk-fig">
<table class="sk-tab">
<thead><tr><th>Tool</th><th>Where Skills live</th><th>Worth knowing</th></tr></thead>
<tbody>
<tr><td>Claude Code</td><td><code>~/.claude/skills/</code> (personal), <code>.claude/skills/</code> (project)</td><td>Filesystem based, no upload. Custom Skills only.</td></tr>
<tr><td>claude.ai</td><td>Uploaded as a zip in Settings</td><td>Per-user, does not sync to the API or to Claude Code. Each surface is separate.</td></tr>
<tr><td>Cursor</td><td><code>.cursor/skills/</code>, <code>~/.cursor/skills/</code>, plus <code>.claude/</code> &amp; <code>.codex/</code></td><td>Reads other tools' folders too. Extra frontmatter: <code>paths</code>, <code>disable-model-invocation</code>.</td></tr>
<tr><td>IBM Bob</td><td><code>.bob/skills/</code> (project), <code>~/.bob/skills/</code> (global)</td><td>Asks approval before activating a Skill by default. Available in Advanced mode.</td></tr>
</tbody>
</table>
  <figcaption>Same SKILL.md contract, small local accents. The one that bites people: on Claude's hosted surfaces, Skills do not sync between claude.ai, the API, and Claude Code. Upload to one, you still have to add it to the others.</figcaption>
</figure>

## Skills are not Rules. Don't mix them up.

If you use Cursor you have probably written Rules, the always-on instructions in `.cursor/rules/`. When Skills arrived, a lot of people asked whether Rules were now dead. They are not. They do different jobs, and the difference is clean once you see it.

<figure class="sk-fig">
  <div class="sk-vs">
    <div class="col rule">
      <h4>Rules <span class="tag">always on</span></h4>
      <ul>
        <li>Loaded into every conversation, every time.</li>
        <li>Best for hard constraints: "always use tabs," "never call the legacy API."</li>
        <li>Declarative. They state how things must be.</li>
        <li>Cost context on every single turn, used or not.</li>
      </ul>
    </div>
    <div class="col skill">
      <h4>Skills <span class="tag">on demand</span></h4>
      <ul>
        <li>Loaded only when the task matches the description.</li>
        <li>Best for procedures: "here is how to cut a release," "here is our migration runbook."</li>
        <li>Procedural. They teach a multi-step how-to.</li>
        <li>Cost almost nothing until the moment they fire.</li>
      </ul>
    </div>
  </div>
  <figcaption>Rules are your hard boundaries; Skills are your workflows. In Cursor, when the two conflict, the Rule wins by design, because constraints are supposed to be non-negotiable and workflows are supposed to be flexible.</figcaption>
</figure>

The rule of thumb I use: if it is a *standard you never want broken*, it is a Rule. If it is a *process you follow when a certain kind of task shows up*, it is a Skill. Coding style, error-handling conventions, banned dependencies, those are Rules. Cutting a release, onboarding a new service, filling a compliance form, those are Skills.

## Where Skills actually earn their keep

Concepts are easy to nod along to. Let me put four real scenarios on the table, the kind you actually hit, and show what a Skill changes in each.

<figure class="sk-fig">
  <div class="sk-pd">
    <div class="sk-pd-stage s1" style="opacity:1;border-color:var(--border-2)">
      <div class="sk-pd-lvl">A</div>
      <div class="sk-pd-txt"><b>The team release runbook</b><span>Twelve steps to cut a release: bump version, regenerate changelog, tag, run the right test subset, push. Everyone half-remembers it. A `release` Skill writes it down once, fires when you say "ship a release," and runs the deterministic steps via a bundled script.</span></div>
    </div>
    <div class="sk-pd-stage s2" style="opacity:1;border-color:var(--border-2)">
      <div class="sk-pd-lvl">B</div>
      <div class="sk-pd-txt"><b>The framework the model gets slightly wrong</b><span>Your house form library has its own validation quirks. Out of the box the agent writes plausible-but-wrong code. A Skill with the real patterns and a `REFERENCE.md` of the actual API turns "close enough" into correct, and only loads when you touch forms.</span></div>
    </div>
    <div class="sk-pd-stage s3" style="opacity:1;border-color:var(--accent)">
      <div class="sk-pd-lvl">C</div>
      <div class="sk-pd-txt"><b>The document chore</b><span>Filling the same PDF form, generating the same styled report, extracting fields from invoices. Bundle a script, let the agent run it for an exact result instead of hand-generating fragile code each time.</span></div>
    </div>
    <div class="sk-pd-stage s1" style="opacity:1;border-color:var(--border-2)">
      <div class="sk-pd-lvl">D</div>
      <div class="sk-pd-txt"><b>The cross-tool team</b><span>Half the team is on Cursor, half on Claude Code, one stubborn soul on Codex. Put the deploy runbook in `.claude/skills/` once. All three tools read it. One source of truth instead of three drifting copies.</span></div>
    </div>
  </div>
  <figcaption>Four shapes of the same payoff: a thing your team does repeatedly, written down once, loaded only when relevant, and shared across whatever tool each person happens to use.</figcaption>
</figure>

Notice the through-line. Skills are not for one-off requests, you would just type those. They are for the *repeated, shaped, slightly-fiddly* work: the procedure you do every sprint, the framework the model keeps fumbling, the chore that needs to be exact. That is the sweet spot.

## The catch, because there always is one

A Skill can bundle scripts, and the agent will run them. That is the power and the danger in the same sentence. A Skill from an untrusted source is, functionally, code you are about to execute with whatever access your agent has. The official guidance across these tools is blunt about it: treat installing a Skill like installing software. Audit the SKILL.md, audit the scripts, watch especially for anything that reaches out to an external URL, because fetched content can carry instructions of its own.

This is not paranoia, it is the same rule you already apply to npm packages and browser extensions. A Skill is convenient precisely because it can act. Convenience that can act is convenience you have to trust.

## The quiet lesson

What I keep coming back to is how *small* the winning design was. Not a grand framework. A folder, a Markdown file, two required lines, and one good idea about loading things only when you need them. That is it. And because it was small and obvious and easy to read, everyone could agree on it, and now the same Skill runs across Claude, Cursor, Bob, Codex, and the rest.

The next time you find yourself pasting the same multi-step instructions into an agent for the third time this week, that is the signal. That paragraph wants to be a Skill. Write the name, write the description that says *when* to use it, drop in the steps, and let progressive disclosure carry it from there, in whatever tool you happen to open tomorrow.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.25});
  ['pd','bars','flow'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
