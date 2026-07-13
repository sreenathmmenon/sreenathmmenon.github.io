---
title: "Understanding the Agent Harness, From Scratch"
date: 2026-07-13
excerpt: "A language model can only do one thing: read text and write text back. It can't open a file, run your code, or check a result. So how do AI agents actually do real work? The answer is a piece of software called the harness. This is a from-zero walkthrough of exactly what it is and how it works, built for anyone with just basic AI knowledge."
tags: [ai, harness, agents, beginner, llm, explainer]
---

<style>
.uh-fig{margin:2.5rem 0;}
.uh-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* model can only do one thing */
.uh-one{max-width:540px;margin:0 auto;text-align:center;border:1px solid var(--border-2);border-radius:12px;padding:1.2rem;background:var(--surface);}
.uh-one .flow{font-family:var(--font-mono);font-size:.9rem;color:var(--text-2);}
.uh-one .flow .in{color:var(--text);} .uh-one .flow .arr{color:var(--accent);padding:0 .5rem;} .uh-one .flow .out{color:var(--text);}
.uh-one .cant{margin-top:.8rem;font-size:.82rem;color:var(--text-3);}

/* brain in a jar analogy */
.uh-jar{max-width:600px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:.9rem;}
@media(max-width:520px){.uh-jar{grid-template-columns:1fr;}}
.uh-jar .side{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.uh-jar .side h4{margin:0 0 .5rem;font-size:.88rem;}
.uh-jar .side.brain{border-color:var(--border-2);} .uh-jar .side.brain h4{color:var(--text-2);}
.uh-jar .side.body{border-color:var(--accent);} .uh-jar .side.body h4{color:var(--accent);}
.uh-jar .side p{font-size:.84rem;color:var(--text-2);margin:0;line-height:1.55;}

/* the six-step loop, the centerpiece */
.uh-loop{max-width:640px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.uh-step{display:flex;align-items:center;gap:.9rem;border:1px solid var(--border);border-radius:10px;padding:.65rem .9rem;background:var(--surface);opacity:0;transform:translateX(-8px);transition:opacity .5s ease,transform .5s ease;}
.uh-loop.go .uh-step{opacity:1;transform:none;}
.uh-loop.go .uh-step:nth-child(1){transition-delay:.1s} .uh-loop.go .uh-step:nth-child(2){transition-delay:.3s}
.uh-loop.go .uh-step:nth-child(3){transition-delay:.5s} .uh-loop.go .uh-step:nth-child(4){transition-delay:.7s}
.uh-loop.go .uh-step:nth-child(5){transition-delay:.9s} .uh-loop.go .uh-step:nth-child(6){transition-delay:1.1s}
.uh-step .n{flex:none;width:28px;height:28px;border-radius:50%;background:var(--grad);color:var(--accent-ink);font-family:var(--font-mono);font-weight:600;font-size:.82rem;display:flex;align-items:center;justify-content:center;}
.uh-step .t{font-size:.86rem;color:var(--text-2);} .uh-step .t b{color:var(--text);}
.uh-step.loopback{border-color:var(--accent);border-style:dashed;} .uh-step.loopback .t b{color:var(--accent);}

/* worked example trace */
.uh-trace{max-width:620px;margin:0 auto;display:flex;flex-direction:column;gap:.45rem;}
.uh-tline{display:flex;align-items:flex-start;gap:.7rem;font-family:var(--font-mono);font-size:.79rem;border:1px solid var(--border);border-radius:8px;padding:.5rem .8rem;background:var(--surface);opacity:0;transform:translateX(-6px);transition:opacity .4s ease,transform .4s ease;}
.uh-trace.go .uh-tline{opacity:1;transform:none;}
.uh-trace.go .uh-tline:nth-child(1){transition-delay:.1s} .uh-trace.go .uh-tline:nth-child(2){transition-delay:.45s}
.uh-trace.go .uh-tline:nth-child(3){transition-delay:.8s} .uh-trace.go .uh-tline:nth-child(4){transition-delay:1.15s}
.uh-trace.go .uh-tline:nth-child(5){transition-delay:1.5s} .uh-trace.go .uh-tline:nth-child(6){transition-delay:1.85s}
.uh-tline .who{flex:none;width:86px;font-weight:600;}
.uh-tline.model .who{color:var(--accent);} .uh-tline.harn .who{color:var(--text-3);} .uh-tline.done .who{color:var(--accent);}
.uh-tline .msg{color:var(--text-2);}
.uh-tline.done{border-color:var(--accent);}

/* components grid */
.uh-comp{display:grid;grid-template-columns:repeat(3,1fr);gap:.7rem;max-width:700px;margin:0 auto;}
@media(max-width:600px){.uh-comp{grid-template-columns:1fr 1fr;}}
@media(max-width:400px){.uh-comp{grid-template-columns:1fr;}}
.uh-comp .c{border:1px solid var(--border-2);border-radius:12px;padding:.85rem;background:var(--surface);opacity:0;transform:translateY(8px);transition:opacity .45s ease,transform .45s ease;}
.uh-comp.go .c{opacity:1;transform:none;}
.uh-comp.go .c:nth-child(1){transition-delay:.1s} .uh-comp.go .c:nth-child(2){transition-delay:.2s} .uh-comp.go .c:nth-child(3){transition-delay:.3s}
.uh-comp.go .c:nth-child(4){transition-delay:.4s} .uh-comp.go .c:nth-child(5){transition-delay:.5s} .uh-comp.go .c:nth-child(6){transition-delay:.6s}
.uh-comp .c b{display:block;color:var(--text);font-size:.85rem;margin-bottom:.25rem;}
.uh-comp .c .role{font-family:var(--font-mono);font-size:.68rem;color:var(--accent);margin-bottom:.35rem;}
.uh-comp .c span{font-size:.78rem;color:var(--text-2);line-height:1.45;}

/* parse decision */
.uh-parse{max-width:560px;margin:0 auto;}
.uh-parse .q{text-align:center;font-family:var(--font-mono);font-size:.82rem;color:var(--text);border:1px solid var(--accent);border-radius:10px;padding:.6rem;margin-bottom:.8rem;}
.uh-parse .branches{display:grid;grid-template-columns:1fr 1fr;gap:.7rem;}
@media(max-width:480px){.uh-parse .branches{grid-template-columns:1fr;}}
.uh-parse .b{border:1px solid var(--border-2);border-radius:10px;padding:.8rem;text-align:center;background:var(--surface);}
.uh-parse .b .lab{font-family:var(--font-mono);font-size:.74rem;color:var(--accent);margin-bottom:.3rem;}
.uh-parse .b span{font-size:.8rem;color:var(--text-2);line-height:1.4;}

/* table */
.uh-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.uh-tab th,.uh-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.uh-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.uh-tab td:first-child{color:var(--text);font-weight:500;white-space:nowrap;}

/* extra reveals */
.uh-jar .side{opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.uh-jar.go .side{opacity:1;transform:none;}
.uh-jar.go .side:nth-child(1){transition-delay:.1s} .uh-jar.go .side:nth-child(2){transition-delay:.35s}

.uh-parse .q{opacity:0;transition:opacity .5s ease;}
.uh-parse.go .q{opacity:1;}
.uh-parse .b{opacity:0;transform:translateY(8px);transition:opacity .5s ease,transform .5s ease;}
.uh-parse.go .b{opacity:1;transform:none;}
.uh-parse.go .b:nth-child(1){transition-delay:.4s} .uh-parse.go .b:nth-child(2){transition-delay:.65s}

@media (prefers-reduced-motion: reduce){
  .uh-loop .uh-step,.uh-trace .uh-tline,.uh-comp .c,
  .uh-jar .side,.uh-parse .q,.uh-parse .b{transition:none !important;opacity:1 !important;transform:none !important;}
}
</style>

Let me start with a fact that surprises most people learning about AI: a language model, on its own, cannot actually *do* anything.

It can't open a file. It can't run your code. It can't search the web, check a result, or click a button. A raw model does exactly one thing, and only one thing: you give it text, and it gives you text back. That's the whole of it. It's astonishingly good at that one thing, good enough to write essays, explain ideas, and reason through problems, but it is still just text in, text out.

So here's the puzzle. If a model can only read and write text, how do "AI agents" fix bugs, book flights, run commands, and get real work done? Something else must be doing the actual *doing*. That something has a name, and it's the piece almost nobody explains clearly: the **harness**.

This post is a from-scratch walkthrough of exactly what a harness is and how it works. You only need basic AI knowledge to follow along. By the end, the whole thing will feel obvious, and you'll understand the machine that turns a text-predictor into an agent that acts.

## The one thing a model can do

First, let's really sit with the limitation, because the harness exists entirely to work around it.

<figure class="uh-fig">
  <div class="uh-one">
    <div class="flow"><span class="in">text in</span><span class="arr">→</span> the model <span class="arr">→</span> <span class="out">text out</span></div>
    <div class="cant">That's all. It cannot open files, run code, browse, or check anything in the real world.</div>
  </div>
  <figcaption>A model is a text-to-text machine. Powerful, but sealed off from the world. It can <em>describe</em> how to fix your code, but it cannot open your code, try the fix, or see if it worked. Every real action an "agent" takes happens outside the model.</figcaption>
</figure>

## The brain-in-a-jar picture

Here's the mental image that makes everything click. Picture the model as a **brain in a jar**: brilliant, full of knowledge, able to think and reason, but with no eyes, no hands, no way to touch the world. It can decide "I should open that door," but it has no arm to reach out and turn the handle.

The harness is the **body** you build around that brain. It's the eyes that show the brain what's happening, and the hands that carry out what the brain decides. The brain thinks; the body acts. Neither is useful alone. Together, they get things done.

<figure class="uh-fig">
  <div class="uh-jar" id="jar">
    <div class="side brain">
      <h4>The model (the brain)</h4>
      <p>Thinks, reasons, decides what should happen next. But it can only express that decision as text. It has no way to actually carry it out.</p>
    </div>
    <div class="side body">
      <h4>The harness (the body)</h4>
      <p>Reads the brain's decision, actually performs it in the real world, a file gets read, code gets run, then reports back what happened.</p>
    </div>
  </div>
  <figcaption>The model is the thinker; the harness is the doer. When you hear "AI agent," what you're really looking at is a smart brain (the model) wrapped in a capable body (the harness). The harness is all the software that isn't the model.</figcaption>
</figure>

## So what is a harness, in one line?

**A harness is a loop of software that runs around the model, letting it take actions and see the results, over and over, until a task is done.**

That word *loop* is the heart of it. The harness doesn't call the model just once. It calls it, does what the model asked, shows the model the result, and calls it again with that new information. Round and round. Let me show you exactly what happens in one trip around that loop.

## The loop, step by step

Here is the entire machine, in six steps. Read it once and you'll understand more about agents than most people do.

<figure class="uh-fig">
  <div class="uh-loop" id="loop">
    <div class="uh-step"><div class="n">1</div><div class="t"><b>Gather the input.</b> The harness bundles together the instructions, the list of actions the model is allowed to take, and everything that's happened so far. This whole bundle becomes the text sent to the model.</div></div>
    <div class="uh-step"><div class="n">2</div><div class="t"><b>Ask the model.</b> The harness sends that bundle to the model and gets text back. This is the "thinking" step.</div></div>
    <div class="uh-step"><div class="n">3</div><div class="t"><b>Read what the model wants.</b> The harness looks at the reply. Is it a final answer for the user? Or is it a request to take an action, like "read this file"?</div></div>
    <div class="uh-step"><div class="n">4</div><div class="t"><b>Do the action.</b> If the model asked to take an action, the harness actually performs it, safely, and captures the result.</div></div>
    <div class="uh-step"><div class="n">5</div><div class="t"><b>Show the model the result.</b> The harness adds that result to the running history, so next time the model can see what happened.</div></div>
    <div class="uh-step loopback"><div class="n">6</div><div class="t"><b>Loop or stop.</b> If the model gave a final answer, stop and show the user. Otherwise, go back to step 1 with the new information. Repeat until done.</div></div>
  </div>
  <figcaption>Six steps, and step six sends you right back to step one. That circle is the harness. Notice the model only ever <em>reads and writes text</em> (steps 2 and 3). Every actual action in the world (step 4) is done by the harness, not the model. That division is the entire trick.</figcaption>
</figure>

## The key decision: is it an answer, or an action?

Step 3 is the clever pivot of the whole loop, so let's zoom into it. Every time the model replies, the harness has to decide one thing: did the model just give a **final answer**, or is it **asking to use a tool**?

<figure class="uh-fig">
  <div class="uh-parse" id="parse">
    <div class="q">the model just replied... what kind of reply is it?</div>
    <div class="branches">
      <div class="b">
        <div class="lab">final answer</div>
        <span>Plain text meant for the user. The harness shows it and the loop ends. Done.</span>
      </div>
      <div class="b">
        <div class="lab">tool request</div>
        <span>The model is asking to take an action. The harness runs it, feeds back the result, and loops again.</span>
      </div>
    </div>
  </div>
  <figcaption>This one fork drives the whole loop. A final answer means "I'm finished, here's the result." A tool request means "I'm not done, please do this for me and tell me what happened." The harness reads every reply and routes it down one of these two paths.</figcaption>
</figure>

## Watch it work: fixing a broken test

Abstract steps are easier to believe once you see them play out. Say you ask an agent: *"my Python test is failing, fix it."* Here's the actual back-and-forth between the model and the harness, one loop at a time.

<figure class="uh-fig">
  <div class="uh-trace" id="trace">
    <div class="uh-tline model"><span class="who">model:</span><span class="msg">"I need to see the test first. Please read test_app.py."</span></div>
    <div class="uh-tline harn"><span class="who">harness:</span><span class="msg">reads the file, sends its contents back to the model</span></div>
    <div class="uh-tline model"><span class="who">model:</span><span class="msg">"I see it. Now run the tests so I can see the error."</span></div>
    <div class="uh-tline harn"><span class="who">harness:</span><span class="msg">runs pytest, feeds the error message back</span></div>
    <div class="uh-tline model"><span class="who">model:</span><span class="msg">"The bug is a missing 'await' on line 12. Editing it now."</span></div>
    <div class="uh-tline done"><span class="who">done:</span><span class="msg">harness applies the edit, reruns tests, all pass. Reports success.</span></div>
  </div>
  <figcaption>Every line where the model "does" something, it's really just asking; the harness does the actual work and reports back. The model never touched a file. It reasoned, requested, and reacted to real results. Without the harness, the model could only <em>guess</em> a fix and hope. With it, the model sees the real error and fixes the real bug.</figcaption>
</figure>

This is the sentence worth remembering: **the intelligence is in the loop, not in the tools.** The tools are simple (read a file, run a command). The magic is that the model gets to see each result and decide the next move, again and again.

## The parts that make up a real harness

The loop is the skeleton. A real, working harness adds a few more parts, each one solving a specific limitation of the bare model. Here they are in plain terms.

<figure class="uh-fig">
  <div class="uh-comp" id="comp">
    <div class="c"><div class="role">the rules</div><b>System prompt</b><span>The standing instructions: who the agent is, how it should behave, what it must never do. Set once, applies always.</span></div>
    <div class="c"><div class="role">the hands</div><b>Tools</b><span>The specific actions the agent is allowed to take: read a file, run code, search the web. The harness decides which exist.</span></div>
    <div class="c"><div class="role">the memory</div><b>Memory</b><span>Keeps track of what's happened so the agent doesn't forget earlier steps or repeat itself on long tasks.</span></div>
    <div class="c"><div class="role">the workspace</div><b>Execution environment</b><span>The safe place where tools actually run: a sandbox, a container. Without it, actions have nowhere to happen.</span></div>
    <div class="c"><div class="role">the safety</div><b>Guardrails</b><span>The limits: a cap on steps so it can't run forever, permission checks, a human sign-off before risky actions.</span></div>
    <div class="c"><div class="role">the black box</div><b>Observability</b><span>A record of every step, every action, every error, so a human can see what the agent did and why.</span></div>
  </div>
  <figcaption>Six parts, six jobs. The system prompt sets the rules, tools are the hands, memory is the notebook, the environment is the workshop, guardrails are the safety rails, and observability is the flight recorder. Stack these around the loop and you have a real, trustworthy agent.</figcaption>
</figure>

## When do you even need one?

Not every use of AI needs a harness, and it's worth knowing the line. If you just want the model to write something once, a summary, an email, an explanation, you don't need a harness at all. You ask, it answers, done.

You need a harness the moment the task requires the model to *act and react over time*:

<figure class="uh-fig">
<table class="uh-tab">
<thead><tr><th>You DON'T need a harness</th><th>You DO need a harness</th></tr></thead>
<tbody>
<tr><td>Write me an email</td><td>Read my inbox and reply to the urgent ones</td></tr>
<tr><td>Explain how sorting works</td><td>Fix the failing test in my project</td></tr>
<tr><td>Summarize this text</td><td>Research a topic across many web pages</td></tr>
<tr><td>One question, one answer</td><td>A goal that takes many steps and real feedback</td></tr>
</tbody>
</table>
  <figcaption>The rule of thumb: if the job is "text in, text out, once," the bare model is enough. If the job needs the model to take actions, see results, and adjust, again and again, you need the loop. That loop is the harness.</figcaption>
</figure>

## The takeaway

Here's the whole idea, tied up. A language model is a brilliant brain in a jar: it can think, but it can't touch the world. The harness is the body you build around it, a loop of software that takes the model's decisions, actually carries them out, shows the model what happened, and asks it again, over and over until the job is done. The model supplies the thinking. The harness supplies the doing.

So the next time you watch an AI agent fix a bug, book a trip, or work through a task step by step, you'll know the real shape of what's happening underneath. It isn't one magic super-brain. It's a plain, understandable loop, patiently letting a text-predictor act, look, and act again. That loop is the quiet engine behind every AI agent you'll ever meet, and now you understand it exactly.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.2});
  ['loop','trace','comp','jar','parse'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
