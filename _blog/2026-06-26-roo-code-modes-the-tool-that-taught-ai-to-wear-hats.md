---
title: "Roo Code: The Tool That Taught AI to Wear Different Hats"
date: 2026-06-26
excerpt: "Roo Code hit three million installs, gave the AI in your editor a clever idea called modes, then bet its whole future against the very thing that made it famous. It is gone now, but the lessons inside it are the foundation of how agentic coding works today."
tags: [ai, roo-code, cline, coding-agents, developer-tools, explainer]
---

<style>
.post-fig{margin:2.6rem 0;}
.post-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* deprecation notice */
.notice{max-width:640px;margin:0 auto 2.4rem;background:var(--surface);border:1px solid var(--accent);border-left:4px solid var(--accent);border-radius:0 10px 10px 0;padding:.9rem 1.1rem;font-size:.92rem;color:var(--text-2);}
.notice b{color:var(--accent);}

/* family tree SVG */
.fam{max-width:540px;margin:0 auto;}
.fam svg{width:100%;height:auto;display:block;}
.fam text{font-family:var(--font-mono);}

/* interactive modes switcher */
.modes{max-width:640px;margin:0 auto;background:var(--surface);border:1px solid var(--border-2);border-radius:16px;padding:1.3rem;box-shadow:var(--glow);}
.modes-tabs{display:flex;flex-wrap:wrap;gap:.45rem;margin-bottom:1.1rem;}
.mode-tab{font-family:var(--font-mono);font-size:.82rem;padding:.4rem .8rem;border-radius:999px;background:var(--surface-2);border:1px solid var(--border-2);color:var(--text-2);cursor:pointer;transition:all .2s var(--ease);}
.mode-tab:hover{border-color:var(--accent);color:var(--accent);}
.mode-tab.on{background:var(--grad);color:var(--accent-ink);border-color:transparent;font-weight:600;}
.mode-body{border-top:1px solid var(--border);padding-top:1.1rem;}
.mode-role{color:var(--text);font-size:.96rem;margin-bottom:.9rem;}
.mode-perms{display:flex;flex-direction:column;gap:.4rem;}
.perm{display:flex;align-items:center;gap:.7rem;font-family:var(--font-mono);font-size:.82rem;}
.perm .ic{flex:none;width:20px;text-align:center;font-weight:700;}
.perm.yes .ic{color:var(--accent);}
.perm.no .ic{color:var(--text-3);}
.perm.no{color:var(--text-3);}

/* agent loop ring */
.loop{display:flex;flex-wrap:wrap;align-items:center;justify-content:center;gap:.4rem;max-width:660px;margin:0 auto;}
.lstep{background:var(--surface);border:1px solid var(--border-2);border-radius:11px;padding:.7rem .9rem;text-align:center;min-width:96px;}
.lstep .ln{font-family:var(--font-mono);font-size:.66rem;color:var(--accent);text-transform:uppercase;letter-spacing:.05em;display:block;margin-bottom:.3rem;}
.lstep .lt{font-size:.82rem;color:var(--text-2);}
.lstep.gate{border-color:var(--accent);}
.larrow{color:var(--text-3);font-size:1rem;}

/* boomerang */
.boom{max-width:560px;margin:0 auto;}
.boom svg{width:100%;height:auto;display:block;}
.boom text{font-family:var(--font-mono);}

/* timeline */
.tl{max-width:560px;margin:0 auto;display:flex;flex-direction:column;gap:0;}
.tl-row{display:flex;gap:1rem;}
.tl-when{flex:none;width:5.5rem;font-family:var(--font-mono);font-size:.78rem;color:var(--accent);padding-top:.1rem;text-align:right;}
.tl-line{flex:none;width:14px;display:flex;flex-direction:column;align-items:center;}
.tl-dot{width:11px;height:11px;border-radius:50%;background:var(--grad);margin-top:.3rem;}
.tl-bar{flex:1;width:2px;background:var(--border-2);}
.tl-what{flex:1;color:var(--text-2);font-size:.9rem;padding-bottom:1.3rem;}
.tl-what b{color:var(--text);}
.tl-row:last-child .tl-bar{display:none;}

/* tables */
.ctab{width:100%;border-collapse:collapse;font-size:.88rem;}
.ctab th,.ctab td{text-align:left;padding:.55rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.ctab th{font-family:var(--font-mono);font-size:.7rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.ctab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}
.ctab .num{font-family:var(--font-mono);color:var(--accent);}
</style>

<div class="notice"><b>Heads up:</b> Roo Code was shut down on May 15, 2026, and its repository is now archived (read-only). I am writing about it anyway, on purpose, because the ideas it introduced did not die with it. They became the standard way agentic coding tools work. The community carried the plugin forward (as Zoo Code), and its closest relatives, Cline and Kilo Code, are alive and well. Read this as a lesson, not a buying guide.</div>

Most software fades away quietly. Roo Code went out differently. It reached three million installs, became one of the most loved AI coding tools on the planet, and then its own creators looked at their success and said, more or less, "we think this entire category is about to be obsolete," and shut it down to chase something else. That is a wild story on its own. But the reason Roo Code is worth your time is not the dramatic ending. It is one genuinely clever idea it gave the AI living in your code editor, an idea that quietly spread everywhere. Let me teach you that idea, the story around it, and why it matters even now that the tool is gone.

## First, where Roo Code came from

To understand Roo Code, you have to meet its family, because it was never a lone invention. It was one branch of a fast-growing family tree, and that tree explains a lot.

It starts with a tool called Cline. In June 2024, just days after a powerful new AI model was released, a developer built a small VS Code extension that could actually *do* things in your project: edit files, run terminal commands, work through a task step by step, instead of only suggesting text. That was Cline, and it caught fire.

Then something very common in open-source happened. Because Cline was open source, other people copied its code to build their own version with different priorities. That copy is called a fork. Roo Code was a fork of Cline, made by people who wanted faster updates and more features. Later, Roo Code itself got forked into a tool called Kilo Code. Three tools, one bloodline, sharing actual code history.

<figure class="post-fig">
  <div class="fam">
    <svg viewBox="0 0 520 210" role="img" aria-label="Family tree: Cline became Roo Code which became Kilo Code">
      <!-- Cline -->
      <rect x="190" y="12" width="140" height="48" rx="10" fill="url(#rg)"/>
      <text x="260" y="32" text-anchor="middle" fill="#0A0B0D" font-size="14" font-weight="700">Cline</text>
      <text x="260" y="48" text-anchor="middle" fill="#0A0B0D" font-size="9">the original, June 2024</text>
      <line x1="260" y1="60" x2="260" y2="86" stroke="var(--border-2)" stroke-width="1.5"/>
      <!-- Roo -->
      <rect x="180" y="86" width="160" height="48" rx="10" fill="var(--surface)" stroke="var(--accent)" stroke-width="1.5"/>
      <text x="260" y="106" text-anchor="middle" fill="var(--text)" font-size="14" font-weight="700">Roo Code</text>
      <text x="260" y="122" text-anchor="middle" fill="var(--text-3)" font-size="9">forked Cline, 2025</text>
      <line x1="260" y1="134" x2="260" y2="160" stroke="var(--border-2)" stroke-width="1.5"/>
      <!-- Kilo -->
      <rect x="190" y="160" width="140" height="44" rx="10" fill="var(--surface)" stroke="var(--border-2)"/>
      <text x="260" y="180" text-anchor="middle" fill="var(--text)" font-size="13" font-weight="600">Kilo Code</text>
      <text x="260" y="195" text-anchor="middle" fill="var(--text-3)" font-size="9">forked Roo, late 2025</text>
      <!-- side labels -->
      <text x="40" y="110" fill="var(--text-3)" font-size="10">each fork</text>
      <text x="40" y="124" fill="var(--text-3)" font-size="10">copied the</text>
      <text x="40" y="138" fill="var(--text-3)" font-size="10">one above</text>
      <defs><linearGradient id="rg" x1="0" y1="0" x2="1" y2="1"><stop offset="0" stop-color="#5EEAD4"/><stop offset="1" stop-color="#818CF8"/></linearGradient></defs>
    </svg>
  </div>
  <figcaption>The bloodline: Cline, then Roo Code, then Kilo Code. Each one copied the code of the one before it and went its own way. They genuinely share git history, which is why a feature one of them invents tends to show up in the others. This is the quiet engine of open-source progress.</figcaption>
</figure>

Now, a question you might have, because I had it too. You may have heard that Roo Code "was used in everything." Let me be precise, because the truth is more interesting than the rumour. Roo Code was not secretly buried inside Cursor or Windsurf. But two things made it feel omnipresent. It was installed three million times, an enormous footprint. And more importantly, its big idea spread far beyond its own code, copied and adapted by tool after tool. Roo did not get embedded in everything. Its *ideas* did. And here is that idea.

## The big idea: give the AI different hats

Picture the problem Roo Code was solving. You have one AI assistant in your editor, and you ask it to do wildly different things. Sometimes you want it to carefully plan a big change without touching a single file. Other times you want it to aggressively write code and run commands. Other times you just want it to answer a question. If you give one assistant unlimited power for all of these, it gets dangerous and unfocused, like a single employee who is allowed to do anything at any time with no job description.

Roo Code's answer was elegant: give the AI named roles, called modes, and let each mode have its own personality and, crucially, its own set of permissions. Switching modes is like the AI putting on a different hat for a different job. Click through them below and watch how the permissions change. This is the heart of what made Roo special.

<figure class="post-fig">
  <div class="modes">
    <div class="modes-tabs" id="modeTabs"></div>
    <div class="mode-body">
      <div class="mode-role" id="modeRole"></div>
      <div class="mode-perms" id="modePerms"></div>
    </div>
  </div>
  <figcaption>Roo Code's modes, the idea that spread across the whole ecosystem. Each mode is the same underlying AI, but wearing a different hat with different powers. Notice how Architect mode literally cannot edit your code files: that restriction is the point, not a limitation. It keeps the planner from going rogue while it is supposed to be just planning.</figcaption>
</figure>

<script>
(function(){
  var modes=[
    {name:'🏗️ Architect', role:'Plans big changes, designs systems, writes specs. Deliberately held back from editing your code so it stays focused on thinking, not doing.', perms:[['read your code',1],['plan and discuss',1],['edit code files',0],['run terminal commands',0]]},
    {name:'💻 Code', role:'The everyday workhorse. Full power to edit files, run commands, and use external tools. This is where the real building happens.', perms:[['read your code',1],['edit code files',1],['run terminal commands',1],['use external tools (MCP)',1]]},
    {name:'❓ Ask', role:'Answers questions and explains the codebase. Pure information, makes no changes at all, so you can explore safely.', perms:[['read your code',1],['explain and answer',1],['edit code files',0],['run terminal commands',0]]},
    {name:'🪲 Debug', role:'Hunts bugs methodically. Adds logs, narrows down the cause, confirms with you before applying a fix rather than guessing wildly.', perms:[['read your code',1],['add diagnostic logs',1],['edit code files',1],['confirm before fixing',1]]},
    {name:'🪃 Orchestrator', role:'The manager. Does not do detailed work itself. Instead it breaks a big job into smaller pieces and hands each to the right specialist mode.', perms:[['break down big tasks',1],['delegate to other modes',1],['edit code directly',0],['stay focused on the plan',1]]}
  ];
  var tabs=document.getElementById('modeTabs'), role=document.getElementById('modeRole'), perms=document.getElementById('modePerms');
  if(!tabs) return;
  function render(i){
    Array.prototype.forEach.call(tabs.children,function(b,j){b.classList.toggle('on',j===i);});
    role.textContent=modes[i].role;
    perms.innerHTML=modes[i].perms.map(function(p){
      var yes=p[1]===1;
      return '<div class="perm '+(yes?'yes':'no')+'"><span class="ic">'+(yes?'✓':'✕')+'</span><span>'+p[0]+'</span></div>';
    }).join('');
  }
  modes.forEach(function(m,i){
    var b=document.createElement('button'); b.className='mode-tab'; b.textContent=m.name;
    b.addEventListener('click',function(){render(i);}); tabs.appendChild(b);
  });
  render(1);
})();
</script>

Why is this such a big deal? Because it solves the trust problem. The scariest thing about an AI agent is that it might do something destructive you did not intend. Modes put guardrails on the AI based on what job it is currently doing. When you are just planning, the AI literally cannot edit your files even if it wanted to. You get the AI's help with far less of the anxiety. That single design choice, named roles with scoped permissions, is now everywhere in agentic coding tools, and Roo Code is the one that made it famous.

## How a mode actually works under the hood

Let me pull back the curtain, because the mechanism is simpler than it sounds and worth understanding. When you pick a mode and give Roo a task, it builds one big instruction sheet, called a system prompt, and sends it to the AI model. That sheet is assembled fresh from a few ingredients.

<figure class="post-fig">
  <table class="ctab">
    <thead><tr><th>Ingredient</th><th>What it contributes</th></tr></thead>
    <tbody>
      <tr><td>Role definition</td><td>"You are an architect who plans but does not edit." The personality of the current mode.</td></tr>
      <tr><td>Tool descriptions</td><td>The exact list of tools this mode is allowed to use, and how to call them. Architect's list is short; Code's is long.</td></tr>
      <tr><td>Rules</td><td>Safety and behaviour rules, like asking before doing something risky.</td></tr>
      <tr><td>Your custom instructions</td><td>Anything you or your team added, such as "always write tests" or "edit only Markdown files."</td></tr>
      <tr><td>Workspace context</td><td>Relevant facts about your project so the AI is not working blind.</td></tr>
    </tbody>
  </table>
  <figcaption>How a mode becomes a prompt. Switching modes mostly changes the first two ingredients: the role and the allowed tools. The AI model is the same; what changes is the job description and the keys it is handed. This is why modes are powerful without needing a different AI for each one.</figcaption>
</figure>

Once that prompt is built, the agent runs a loop. It reads context, decides on one action, proposes it, waits for your approval, acts, and repeats. The approval step is the seatbelt.

<figure class="post-fig">
  <div class="loop">
    <div class="lstep"><span class="ln">1 read</span><span class="lt">look at the code and task</span></div>
    <div class="larrow">&rarr;</div>
    <div class="lstep"><span class="ln">2 decide</span><span class="lt">pick one tool to use</span></div>
    <div class="larrow">&rarr;</div>
    <div class="lstep gate"><span class="ln">3 ask you</span><span class="lt">show the change, wait for OK</span></div>
    <div class="larrow">&rarr;</div>
    <div class="lstep"><span class="ln">4 act</span><span class="lt">make the edit or run it</span></div>
    <div class="larrow">&#8635;</div>
    <div class="lstep"><span class="ln">repeat</span><span class="lt">until the task is done</span></div>
  </div>
  <figcaption>The agent loop. Read, decide, ask, act, repeat. Step three is the human-in-the-loop gate that made these tools trustworthy: the agent proposes, you approve. The mode you are in decides which actions are even on the menu at step two.</figcaption>
</figure>

## The cleverest trick: the boomerang

There is one more idea in Roo Code worth showing, because it is genuinely smart and it solves a real headache. What happens when a task is too big for one mode? Roo had a special mode, the Orchestrator, also nicknamed Boomerang, whose whole job was to manage other modes.

Here is the problem it solves. An AI has limited memory. If one agent tries to do a huge, ten-part task, its memory fills up with the details of part one and it gets confused by part ten. The Orchestrator avoids this by acting like a project manager. It breaks the big task into small subtasks and sends each one to a fresh specialist, who works in its own clean memory and reports back only a short summary. The detail stays with the worker; only the result comes back. Like a boomerang, the task is thrown out and returns.

<figure class="post-fig">
  <div class="boom">
    <svg viewBox="0 0 520 230" role="img" aria-label="Orchestrator delegating subtasks to specialist modes and getting summaries back">
      <!-- orchestrator -->
      <rect x="180" y="14" width="160" height="46" rx="10" fill="url(#bg)"/>
      <text x="260" y="34" text-anchor="middle" fill="#0A0B0D" font-size="13" font-weight="700">🪃 Orchestrator</text>
      <text x="260" y="50" text-anchor="middle" fill="#0A0B0D" font-size="9">holds the big plan</text>
      <!-- send arrows -->
      <line x1="200" y1="60" x2="90" y2="120" stroke="var(--accent)" stroke-width="1.5"/>
      <line x1="260" y1="60" x2="260" y2="120" stroke="var(--accent)" stroke-width="1.5"/>
      <line x1="320" y1="60" x2="430" y2="120" stroke="var(--accent)" stroke-width="1.5"/>
      <text x="135" y="92" fill="var(--text-3)" font-size="8.5">subtask &darr;</text>
      <text x="430" y="92" fill="var(--text-3)" font-size="8.5">subtask &darr;</text>
      <!-- specialists -->
      <rect x="20" y="120" width="140" height="44" rx="9" fill="var(--surface)" stroke="var(--border-2)"/>
      <text x="90" y="140" text-anchor="middle" fill="var(--text)" font-size="11" font-weight="600">🏗️ Architect</text>
      <text x="90" y="154" text-anchor="middle" fill="var(--text-3)" font-size="8.5">own clean memory</text>
      <rect x="190" y="120" width="140" height="44" rx="9" fill="var(--surface)" stroke="var(--border-2)"/>
      <text x="260" y="140" text-anchor="middle" fill="var(--text)" font-size="11" font-weight="600">💻 Code</text>
      <text x="260" y="154" text-anchor="middle" fill="var(--text-3)" font-size="8.5">own clean memory</text>
      <rect x="360" y="120" width="140" height="44" rx="9" fill="var(--surface)" stroke="var(--border-2)"/>
      <text x="430" y="140" text-anchor="middle" fill="var(--text)" font-size="11" font-weight="600">🪲 Debug</text>
      <text x="430" y="154" text-anchor="middle" fill="var(--text-3)" font-size="8.5">own clean memory</text>
      <!-- return arrows -->
      <path d="M90 164 Q90 195 250 198" fill="none" stroke="var(--border-2)" stroke-width="1.3" stroke-dasharray="4 3"/>
      <path d="M430 164 Q430 195 270 198" fill="none" stroke="var(--border-2)" stroke-width="1.3" stroke-dasharray="4 3"/>
      <text x="260" y="216" text-anchor="middle" fill="var(--text-3)" font-size="9">each returns only a short summary &uarr;</text>
      <defs><linearGradient id="bg" x1="0" y1="0" x2="1" y2="1"><stop offset="0" stop-color="#5EEAD4"/><stop offset="1" stop-color="#818CF8"/></linearGradient></defs>
    </svg>
  </div>
  <figcaption>Boomerang tasks. The Orchestrator keeps the master plan but does none of the detailed work. It throws each subtask to a specialist mode that works in its own fresh memory, then returns only a tidy summary. This keeps every agent focused and stops the big task from drowning in detail. It is multi-agent teamwork, built from the same modes idea.</figcaption>
</figure>

## The plot twist: it walked away from its own success

So here is the ending I promised. By early 2026, Roo Code had three million installs and was beloved. And the team chose to shut it down.

Their reasoning is the most thought-provoking part of the whole story. They came to believe that the era of AI plugins living inside your code editor is ending. Their bet is that the future of coding agents is not a tool inside your editor at all, but agents running in the cloud that you talk to from places like Slack, working on their own while you do other things. So they ended Roo Code to pour everything into that new bet. Whether they are right is one of the genuinely open questions in software right now.

<figure class="post-fig">
  <div class="tl">
    <div class="tl-row"><div class="tl-when">Jun 2024</div><div class="tl-line"><span class="tl-dot"></span><span class="tl-bar"></span></div><div class="tl-what"><b>Cline is born.</b> The first popular agent that actually edits files and runs commands inside VS Code.</div></div>
    <div class="tl-row"><div class="tl-when">2025</div><div class="tl-line"><span class="tl-dot"></span><span class="tl-bar"></span></div><div class="tl-what"><b>Roo Code forks it</b> and adds the modes idea, custom roles, and diff-based editing. It takes off.</div></div>
    <div class="tl-row"><div class="tl-when">Late 2025</div><div class="tl-line"><span class="tl-dot"></span><span class="tl-bar"></span></div><div class="tl-what"><b>Kilo Code forks Roo</b>, raising funding to combine the best of Cline and Roo into one polished tool.</div></div>
    <div class="tl-row"><div class="tl-when">Early 2026</div><div class="tl-line"><span class="tl-dot"></span><span class="tl-bar"></span></div><div class="tl-what">Roo Code crosses <b>3 million installs.</b> One of the most used AI coding tools in the world.</div></div>
    <div class="tl-row"><div class="tl-when">May 2026</div><div class="tl-line"><span class="tl-dot"></span><span class="tl-bar"></span></div><div class="tl-what"><b>Roo Code shuts down</b>, betting the company on a cloud-first agent instead. The community forks the plugin to keep it alive.</div></div>
  </div>
  <figcaption>The short, dramatic life of Roo Code. Notice how each step built on the last, and how the ending was a choice, not a collapse. The team did not fail. They decided the whole category was about to change and jumped early.</figcaption>
</figure>

## What to actually carry away

Roo Code is gone, but you just learned the three ideas that outlived it, and they are the ideas behind almost every serious coding agent today.

First, modes: give one AI different named roles, each with its own scoped permissions, so it is focused and safe instead of an all-powerful loose cannon. Second, the human-in-the-loop agent loop: the agent proposes an action and waits for your approval before doing anything real, which is what makes trusting it possible. Third, orchestration: when a job is too big, a manager agent splits it into clean subtasks handled by specialists, so nobody drowns in detail.

These are not Roo Code trivia. They are the working vocabulary of modern AI coding, and now they are yours. Tools will keep being born, forked, and retired. That churn is normal and even healthy. But the good ideas, like the ones Roo Code popularised, do not retire. They get inherited. The next time you use a coding tool that lets you switch between a planning mode and a building mode, you will know exactly where that came from, and why it was such a quietly brilliant idea.
