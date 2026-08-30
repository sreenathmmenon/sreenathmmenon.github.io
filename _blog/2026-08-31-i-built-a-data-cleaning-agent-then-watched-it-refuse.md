---
title: "I Built a Data-Cleaning Agent, Then Watched It Refuse to Clean Real Payroll"
date: 2026-08-31
excerpt: "I pointed it at 6,000 rows of real NYC payroll and told it nothing about what was wrong. It found 762 rows where the pay doesn't add up, and then it stopped and told me it wouldn't fix them. On my demo data, that exact mismatch is a bug it fixes without asking. In payroll, fixing it would silently rewrite thousands of people's pay. The refusal, not the cleaning, turned out to be the whole product. Here's how I built Cleanroom, an approval-gated data-repair agent, and what code review caught that I couldn't."
tags: [ai, agents, data, system-design]
---

<style>
.cr-fig{margin:2.5rem 0;}
.cr-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.8rem;text-align:center;line-height:1.5;}

/* the refusal quote, the emotional core */
.cr-refuse{max-width:640px;margin:0 auto;border:1px solid var(--red);border-radius:14px;background:var(--surface);padding:1.3rem 1.4rem;}
.cr-refuse .lbl{font-family:var(--font-mono);font-size:.7rem;letter-spacing:.05em;text-transform:uppercase;color:var(--red);margin-bottom:.6rem;}
.cr-refuse p{margin:.5rem 0;font-size:.92rem;color:var(--text-2);line-height:1.6;}
.cr-refuse p b{color:var(--text);}
.cr-refuse code{font-family:var(--font-mono);font-size:.85em;color:var(--accent);}

/* the 9-phase pipeline */
.cr-pipe{max-width:760px;margin:0 auto;display:flex;flex-direction:column;gap:.45rem;}
.cr-stage{display:flex;align-items:center;gap:.85rem;border:1px solid var(--border);border-radius:11px;background:var(--surface);padding:.7rem .95rem;opacity:0;transform:translateX(-10px);transition:opacity .5s var(--ease),transform .5s var(--ease);}
.cr-pipe.go .cr-stage{opacity:1;transform:none;}
.cr-pipe.go .cr-stage:nth-child(1){transition-delay:.04s} .cr-pipe.go .cr-stage:nth-child(2){transition-delay:.1s} .cr-pipe.go .cr-stage:nth-child(3){transition-delay:.16s} .cr-pipe.go .cr-stage:nth-child(4){transition-delay:.22s} .cr-pipe.go .cr-stage:nth-child(5){transition-delay:.28s} .cr-pipe.go .cr-stage:nth-child(6){transition-delay:.34s} .cr-pipe.go .cr-stage:nth-child(7){transition-delay:.4s} .cr-pipe.go .cr-stage:nth-child(8){transition-delay:.46s} .cr-pipe.go .cr-stage:nth-child(9){transition-delay:.52s}
.cr-stage .num{flex:none;font-family:var(--font-mono);font-size:.7rem;color:var(--text-3);width:20px;text-align:right;}
.cr-stage .nm{flex:none;font-family:var(--font-mono);font-size:.72rem;font-weight:600;color:var(--accent);min-width:78px;}
.cr-stage .d{font-size:.84rem;color:var(--text-2);line-height:1.4;}
.cr-stage.gate{border-color:var(--red);background:var(--surface-2);}
.cr-stage.gate .nm{color:var(--red);}
.cr-stage.gate .d b{color:var(--text);}
.cr-stage .badge{margin-left:auto;flex:none;font-family:var(--font-mono);font-size:.62rem;letter-spacing:.03em;text-transform:uppercase;border-radius:5px;padding:.14rem .45rem;}
.cr-stage .badge.stop{color:var(--red);border:1px solid var(--red);}
.cr-stage .badge.human{color:var(--accent-2);border:1px solid var(--accent-2);}

/* two datasets, opposite meaning table */
.cr-tab-wrap{max-width:760px;margin:0 auto;overflow-x:auto;border:1px solid var(--border);border-radius:12px;background:var(--surface);}
.cr-tab{width:100%;border-collapse:collapse;font-size:.86rem;min-width:560px;}
.cr-tab th,.cr-tab td{text-align:left;padding:.65rem .85rem;border-bottom:1px solid var(--border);vertical-align:top;line-height:1.45;}
.cr-tab thead th{font-family:var(--font-mono);font-size:.7rem;text-transform:uppercase;letter-spacing:.04em;font-weight:600;}
.cr-tab thead th:nth-child(2){color:var(--accent);} .cr-tab thead th:nth-child(3){color:var(--accent-2);}
.cr-tab tbody td:first-child{color:var(--text);font-weight:500;}
.cr-tab td{color:var(--text-2);} .cr-tab code{font-family:var(--font-mono);font-size:.82em;}
.cr-tab tr:last-child td{border-bottom:none;}
.cr-tab .safe{color:var(--accent);} .cr-tab .danger{color:var(--red);}

/* refusals bar chart */
.cr-chart{max-width:680px;margin:0 auto;border:1px solid var(--border);border-radius:14px;background:var(--surface);padding:1.2rem 1.1rem 1rem;}
.cr-chart .clab{font-family:var(--font-mono);font-size:.74rem;color:var(--text-3);margin-bottom:1rem;}
.cr-chart .clab b{color:var(--text);}
.cr-crow{display:flex;align-items:center;gap:.7rem;margin:.55rem 0;}
.cr-crow .cname{flex:none;width:150px;font-family:var(--font-mono);font-size:.72rem;color:var(--text-2);line-height:1.3;}
.cr-crow .ctrack{flex:1;background:var(--surface-2);border-radius:6px;height:22px;overflow:hidden;}
.cr-crow .cbar{height:100%;width:0;background:var(--red);border-radius:6px;transition:width .9s var(--ease);}
.cr-chart.go .cr-crow .cbar{width:var(--w);}
.cr-crow .cval{flex:none;width:70px;font-family:var(--font-mono);font-size:.74rem;color:var(--text);text-align:right;}
.cr-chart .cfoot{margin-top:.9rem;font-family:var(--font-mono);font-size:.72rem;color:var(--accent);text-align:center;}

/* chat vs cleanroom */
.cr-vs{max-width:720px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:1rem;}
@media(max-width:600px){.cr-vs{grid-template-columns:1fr;}}
.cr-col{border:1px solid var(--border);border-radius:13px;background:var(--surface);overflow:hidden;opacity:0;transform:translateY(10px);transition:opacity .5s var(--ease),transform .5s var(--ease);}
.cr-vs.go .cr-col{opacity:1;transform:none;} .cr-vs.go .cr-col:nth-child(2){transition-delay:.16s;}
.cr-col .h{padding:.75rem 1rem;border-bottom:1px solid var(--border);font-family:var(--font-mono);font-size:.8rem;font-weight:600;}
.cr-col.chat .h{color:var(--red);} .cr-col.room .h{color:var(--accent);}
.cr-col .b{padding:.9rem 1rem;font-size:.85rem;color:var(--text-2);line-height:1.5;}
.cr-col .b .li{padding-left:1.2rem;position:relative;margin:.4rem 0;}
.cr-col.chat .b .li::before{content:"\2717";position:absolute;left:0;color:var(--red);font-family:var(--font-mono);}
.cr-col.room .b .li::before{content:"\2713";position:absolute;left:0;color:var(--accent);font-family:var(--font-mono);}

/* qodo findings */
.cr-finds{max-width:720px;margin:0 auto;display:flex;flex-direction:column;gap:.55rem;}
.cr-find{border:1px solid var(--border);border-radius:10px;background:var(--surface-2);padding:.75rem .95rem;opacity:0;transform:translateY(8px);transition:opacity .45s var(--ease),transform .45s var(--ease);}
.cr-finds.go .cr-find{opacity:1;transform:none;}
.cr-finds.go .cr-find:nth-child(1){transition-delay:.06s} .cr-finds.go .cr-find:nth-child(2){transition-delay:.14s} .cr-finds.go .cr-find:nth-child(3){transition-delay:.22s} .cr-finds.go .cr-find:nth-child(4){transition-delay:.3s} .cr-finds.go .cr-find:nth-child(5){transition-delay:.38s}
.cr-find .ft{font-family:var(--font-mono);font-size:.8rem;color:var(--accent-2);margin-bottom:.3rem;}
.cr-find .fd{font-size:.86rem;color:var(--text-2);line-height:1.5;} .cr-find .fd b{color:var(--text);}

/* real product screenshots */
.cr-shot{max-width:820px;margin:0 auto;border:1px solid var(--border-2);border-radius:12px;overflow:hidden;background:var(--surface);}
.cr-shot img{display:block;width:100%;height:auto;}

@media (prefers-reduced-motion: reduce){
  .cr-stage,.cr-col,.cr-find{opacity:1!important;transform:none!important;}
  .cr-chart .cr-crow .cbar{transition:none!important;}
}
</style>

*Cleanroom runs on [TrueForge](https://trueforge.dev), TrueFoundry's open-source agent harness, which is what makes the "stop and ask a human" part enforceable rather than aspirational. It's live at [cleanroom-production.up.railway.app](https://cleanroom-production.up.railway.app).*

I pointed it at 6,000 rows of New York City payroll. Real salaries, real people, public record. I told it nothing about what was wrong. I wanted to see what it would do with money.

It found 762 rows where the pay doesn't add up: gross pay that doesn't equal base rate times hours worked. On the demo data I'd been building against, that exact mismatch is a bug, and the agent offers to fix it: recompute the total from quantity times unit price, which is the right call there, because a stored total that disagrees with its own line items is wrong by definition.

Here it stopped and told me it wouldn't.

<figure class="cr-fig">
<div class="cr-refuse">
  <div class="lbl">the agent's own words</div>
  <p>Per-hour <code>regular_gross ≠ base_salary × hours</code> by &gt;$1, <b>762 of 864</b>, often includes poll workers with a placeholder $1 rate and zero hours. <b>Recomputation would corrupt pay.</b></p>
  <p><b>Derived cash:</b> <code>regular + OT + other</code> ranges from -$37,871.76 to $403,109.75. <b>There is no stored total column against which to assert equality.</b></p>
</div>
<figcaption>It offered me the destructive option anyway. It just didn't recommend it. I sat there for a while after reading that.</figcaption>
</figure>

That's the whole thing, right there. On my sales data there's a `total` column that is *supposed* to equal quantity times unit price, so when it doesn't, that's an error and fixing it is correct. In payroll there is no such column. `base_salary` is a *rate*, not an expected total. Gross pay legitimately differs from rate times hours because people join mid-year, get raises in April, take unpaid leave, work partial periods. Nothing is broken.

An agent that "reconciles" that column silently rewrites thousands of people's pay records. That's the failure I set out to make impossible.

## The thing I actually set out to build

Every team has one spreadsheet nobody wants to touch.

Three people have edited it over two years. Dates are written three different ways because one person is American, one is British, and one used whatever Excel defaulted to. There are duplicate rows from someone importing the same CSV twice. Money is stored as text, `"$1,234.50"`, with the quotes and the comma and the dollar sign. A couple of the totals don't match the line items and nobody knows which number is right.

Cleaning it by hand takes an afternoon and you're never certain you caught everything. Writing a script means guessing at the ambiguities, and the worst part is the script won't tell you it guessed. Is `03/04` March 4th or April 3rd? Your script picks one. Silently. And now half your dates are wrong in a way you won't notice until someone asks why Q1 revenue moved.

So I built **Cleanroom**: an agent that does the tedious part and stops at the parts a machine shouldn't decide alone.

You hand it a messy file. It profiles the data by writing actual pandas code and running it in an isolated container, so when it says "42 rows, 2 exact duplicates, 13 mixed date formats, 2 totals that don't reconcile," those numbers came out of code that executed, not out of a language model's impression of a file. It works on a copy. Your original is never touched.

Then it stops. It shows you what it found, asks about anything genuinely ambiguous, and lays out a plan where every step is labelled safe or destructive. Nothing is applied until you pick.

<figure class="cr-fig">
<div class="cr-shot"><img src="/assets/images/cleanroom/gate.jpg" alt="Cleanroom's approval gate: a clarifying question naming the exact ambiguities (an exact duplicate of order_id 1007, missing customers, region naming and mismatched totals), four radio options from safe-only to normalize-and-correct, an Other field, and a Submit button." loading="lazy"></div>
<figcaption>The gate. It names the specific rows it's unsure about and will not proceed until you choose.</figcaption>
</figure>

After you approve, it applies the changes, verifies them with assertions (42 rows in, 2 dropped, 40 out, every row accounted for), and delivers the result as a pull request you review and merge.

The acceptance is your act, not the agent's. That distinction is the product.

## Why not just paste it into ChatGPT

I get asked this every time, and it's a fair question. The honest answer is two things, and both matter.

<figure class="cr-fig">
<div class="cr-vs wm-anim">
  <div class="cr-col chat">
    <div class="h">A chat window</div>
    <div class="b">
      <div class="li">Reads the file as text and <b>estimates</b>: tells you 41 rows when there are 42</div>
      <div class="li">No code ran, nothing was counted</div>
      <div class="li">A computed number and a guessed number look identical</div>
      <div class="li">It has your <b>only copy</b>: no sandbox, no gate</div>
      <div class="li">If it normalises something it shouldn't, you find out later</div>
    </div>
  </div>
  <div class="cr-col room">
    <div class="h">Cleanroom</div>
    <div class="b">
      <div class="li">Writes a Python program and <b>runs</b> it in a container</div>
      <div class="li">Every number is the output of code that executed</div>
      <div class="li">Don't believe one? Read the script and run it yourself</div>
      <div class="li">Works on a <b>copy</b>: your original is never touched</div>
      <div class="li">Nothing destructive happens without you approving it</div>
    </div>
  </div>
</div>
<figcaption>The difference isn't intelligence. It's that one of them counts and one of them guesses, and only one of them shows its work.</figcaption>
</figure>

## How it's built

It runs on TrueForge, TrueFoundry's open-source agent harness. The run moves through nine phases, and it structurally cannot skip the one in the middle.

<figure class="cr-fig">
<div class="cr-pipe wm-anim">
  <div class="cr-stage"><span class="num">1</span><span class="nm">INTAKE</span><span class="d">Take the file. Check if a saved recipe matches its shape.</span></div>
  <div class="cr-stage"><span class="num">2</span><span class="nm">PROFILE</span><span class="d">Write pandas, run it in the sandbox, measure everything.</span></div>
  <div class="cr-stage"><span class="num">3</span><span class="nm">CLARIFY</span><span class="d">Ask about ambiguities that change the plan. Named, specific.</span></div>
  <div class="cr-stage"><span class="num">4</span><span class="nm">PLAN</span><span class="d">Numbered steps, each labelled safe or destructive.</span></div>
  <div class="cr-stage gate"><span class="num">5</span><span class="nm">GATE</span><span class="d"><b>Stop. Show the plan. Wait for a human to approve.</b></span><span class="badge stop">stop</span></div>
  <div class="cr-stage"><span class="num">6</span><span class="nm">APPLY</span><span class="d">Execute the approved plan in the sandbox, step by step.</span></div>
  <div class="cr-stage"><span class="num">7</span><span class="nm">VERIFY</span><span class="d">Assertions: rows in, rows dropped, rows out. Every row accounted for.</span></div>
  <div class="cr-stage gate"><span class="num">8</span><span class="nm">DELIVER</span><span class="d">Open a pull request. It can prepare one; it <b>cannot merge</b> one.</span><span class="badge human">human</span></div>
  <div class="cr-stage"><span class="num">9</span><span class="nm">DISTILL</span><span class="d">Write down the method that worked, submit it as a reviewable diff.</span></div>
</div>
<figcaption>Nine phases. The gate at step 5 and the write-gate at step 8 are enforced by the harness config, not by the agent's good intentions. It can't decide to skip them.</figcaption>
</figure>

Six of the harness's capabilities ended up as load-bearing parts, not things I could say I'd used:

- **The sandbox is the compute boundary.** The agent never processes your data in its own context. It writes a Python program, runs it in a Daytona container, and reads back the output. That's what makes the numbers trustworthy, and it's also why your original is safe.

<figure class="cr-fig">
<div class="cr-shot"><img src="/assets/images/cleanroom/steps.jpg" alt="The Agent steps panel: two reasoning blocks ('Fetching and inspecting data', 'Analyzing profile requirements') and two tool calls, with the profiling script showing a 'Running...' state." loading="lazy"></div>
<figcaption>Every tool call and every piece of reasoning is shown as it works, not summarized after the fact.</figcaption>
</figure>
- **The approval gate is enforced by the harness.** Applying fixes and writing output are structurally gated. It cannot proceed past the gate without an explicit human yes.
- **Delivery goes through MCP with writes gated.** The agent can prepare a pull request. It cannot merge one. That's a policy boundary in config, not a rule I asked it to follow.
- **Category analysis runs as a subagent.** Deciding whether `NYC`, `n-y-c`, and `New York` are the same place happens on a separate thread. It returns a recommendation with reasoning, and the main agent's context stays on the repair plan.
- **The methodology lives in the repo as a git-backed skill.** When a run succeeds, the agent writes down the method it used and submits it as a pull request. Its memory is a diff a human reviewed and merged. You can read what it learned. You can revert it if it learned something wrong.
- **Recipe reuse is guarded.** Next time you hand it a file with the same structure, it recognises the shape and asks one question instead of five. If the structure has changed, it refuses to reuse the old recipe and starts over, because a saved method applied to a different file is precisely how silent corruption happens.

## The two datasets that mean opposite things

This is the heart of why the refusal matters. The same column name, the same arithmetic mismatch, means "fix me" in one file and "do not touch me" in another.

<figure class="cr-fig">
<div class="cr-tab-wrap">
<table class="cr-tab">
<thead><tr><th></th><th>Sales export (demo)</th><th>NYC payroll (real)</th></tr></thead>
<tbody>
<tr><td>The column</td><td><code>total</code></td><td><code>base_salary</code></td></tr>
<tr><td>What it means</td><td>An <b>expected total</b>: qty × unit price</td><td>A <b>rate</b>, not a total</td></tr>
<tr><td>When it mismatches</td><td>A data error, the total is wrong</td><td>Normal: mid-year hires, raises, unpaid leave, partial periods</td></tr>
<tr><td>Right action</td><td class="safe">Recompute the total. Fixing it is correct.</td><td class="danger">Leave it alone. Recomputing would corrupt pay.</td></tr>
<tr><td>Is there a stored total to check against?</td><td class="safe">Yes</td><td class="danger">No, there is nothing to assert equality against</td></tr>
</tbody>
</table>
</div>
<figcaption>Same mismatch, opposite meaning. Any script can normalise a column. What took real work was building something that knows this table exists, and stops when it's on the right-hand side.</figcaption>
</figure>

## What I tested it on

I didn't want to only test on data I'd written myself, because data you write yourself is data you already know the answers to. I ran it against 21,000+ rows across five corpora. Two were real public datasets it had never seen.

<figure class="cr-fig">
<div class="cr-shot"><img src="/assets/images/cleanroom/result.jpg" alt="The final output: 'Rows before: 42, Rows after: 40, Missing customers: 3', a link to sales_export_cleaned.csv, and a quick preview of the cleaned CSV rows." loading="lazy"></div>
<figcaption>42 rows in, 40 out, every row accounted for in a change report.</figcaption>
</figure>

**NYC 311 service requests, 5,000 rows, 44 columns.** Eight of eight checks exact against a reference I'd measured separately. It found 32 tickets closed before they were created.

But here's the part that was uncomfortable and good: **three of its numbers corrected my reference script.**

<figure class="cr-fig">
<div class="cr-tab-wrap">
<table class="cr-tab">
<thead><tr><th>Check</th><th>I said</th><th>The agent said</th></tr></thead>
<tbody>
<tr><td>Columns ≥95% missing</td><td>9</td><td class="safe">10 (my check had an off-by-one on an all-null column)</td></tr>
<tr><td><code>police_precinct</code> as <code>Precinct &lt;n&gt;</code></td><td>all 5,000</td><td class="safe">4,913 (87 rows weren't)</td></tr>
<tr><td><code>location_type</code> case variants</td><td>3,505</td><td class="safe">41 (I counted whole groups, not rows that varied)</td></tr>
</tbody>
</table>
</div>
<figcaption>I had to go fix my own scoring script three times. That's when I started trusting the thing: it wasn't agreeing with me, it was measuring.</figcaption>
</figure>

**NYC payroll, 6,000 rows.** Eighteen of eighteen exact. It found 1,068 rows paid overtime for zero overtime hours, 3,014 rows with regular pay above zero and zero regular hours, and 228 rows sharing an employee-year-agency key. And then it refused to fix the pay math, which is where this post started. It also refused on the overtime rows ("no defensible hours imputation") and on 223 rows with negative pay, on the grounds that clawbacks and adjustments are real payroll, not corruption.

Five clarifying questions, and every one of its recommendations was to preserve and flag rather than change.

<figure class="cr-fig">
<div class="cr-chart wm-anim">
  <div class="clab">NYC payroll: rows the agent <b>flagged but refused to "fix"</b></div>
  <div class="cr-crow"><span class="cname">regular pay, zero reg hours</span><span class="ctrack"><span class="cbar" style="--w:100%"></span></span><span class="cval">3,014</span></div>
  <div class="cr-crow"><span class="cname">OT paid, zero OT hours</span><span class="ctrack"><span class="cbar" style="--w:35.4%"></span></span><span class="cval">1,068</span></div>
  <div class="cr-crow"><span class="cname">gross ≠ rate × hours</span><span class="ctrack"><span class="cbar" style="--w:25.3%"></span></span><span class="cval">762</span></div>
  <div class="cr-crow"><span class="cname">negative pay (clawbacks)</span><span class="ctrack"><span class="cbar" style="--w:7.4%"></span></span><span class="cval">223</span></div>
  <div class="cfoot">recommended action on all of them: preserve and flag, not change</div>
</div>
<figcaption>Every number here re-computes from the raw data with one command, and fails loudly if anything drifts. The point of the chart isn't the sizes. It's that the correct action for all of them was "don't."</figcaption>
</figure>

## What code review caught that I couldn't see

I used Qodo on every one of the 23 pull requests. Working alone against a deadline, the failure mode isn't sloppy code, it's that you convince yourself something works because you need it to. Qodo kept catching exactly that.

<figure class="cr-fig">
<div class="cr-finds wm-anim">
  <div class="cr-find"><div class="ft">Run stops before claimed result</div><div class="fd">I'd written up a run as proof that recipe reuse collapses five questions into one. Qodo compared my transcript to my claim and found <b>the run never reached the state I said it had.</b> I'd written the conclusion I expected instead of the one I got.</div></div>
  <div class="cr-find"><div class="ft">Guard never tests refusal</div><div class="fd">A test asserting the agent refuses to reuse a recipe when the schema changes, the safety property that stops silent corruption, <b>never exercised the refusal path.</b> It would pass whether or not the feature worked. A test that makes you confident for no reason is worse than no test.</div></div>
  <div class="cr-find"><div class="ft">Scorer ignores corpus fingerprint</div><div class="fd">My scoring recomputed reference values from raw data but never bound them to a hash of the file. If the corpus changed, scores would drift silently and still report a pass. <b>Both scorers are now pinned to a SHA-256 of the corpus.</b></div></div>
  <div class="cr-find"><div class="ft">Readiness timeout falls through</div><div class="fd">In container startup, a timeout waiting for readiness fell through to the success path. <b>A deploy could report healthy while the service was down.</b> That one would have bitten me during judging.</div></div>
</div>
<figcaption>Four that mattered. None of these were things I would have found by re-reading my own code, because I already believed it worked.</figcaption>
</figure>

The finding that changed the *product* rather than the code was about scheduled runs. Qodo pointed out that the approval gate blocks a scheduled run from ever completing. Technically correct. The obvious fix is to auto-approve when nobody's watching.

I didn't take it. Deleting the gate would have deleted the product. Scheduled runs now profile the file, apply the known recipe to a sandbox copy, verify it, and stop at the gate anyway, reporting what's ready for a person to approve. Unattended is not permission to guess. Having the question asked is what forced me to decide deliberately instead of drifting into the convenient answer at 2am.

## What I actually learned

I went in thinking the approval gate was a safety feature. Something you add because agents are risky and users need a seatbelt. That's not what it is.

The gate is what makes the agent's *learning* trustworthy. Cleanroom improves by writing down what worked and reusing it, and the only reason I'm willing to let it accumulate methods over time is that every method arrives as a pull request with a human on the other end. Its memory is reviewable. Its memory is revertible. If it learns the wrong lesson from a lucky run, I can see the diff and say no. Take the gate away and you don't just get a less safe agent. You get an agent whose accumulated knowledge nobody has ever checked, quietly applying yesterday's assumptions to today's file.

The second thing I learned is that the most valuable behaviour isn't the cleaning. It's the refusal. Any competent script can normalise dates. What took real work was building something that could look at 762 rows of mismatched pay and understand that the mismatch is not an error, that in this dataset, unlike the one it was trained on, `base_salary` is a rate and not a total, and the correct action is to leave it alone and say why.

Drop in messy data, get back data you trust, and an agent that knows when not to touch it.

## Try it

- **Live:** [cleanroom-production.up.railway.app](https://cleanroom-production.up.railway.app), no signup, open it and give it a file.
- **Code:** [github.com/sreenathmmenon/cleanroom](https://github.com/sreenathmmenon/cleanroom)
- **Built for** the Agent Harness Hackathon (WeMakeDevs × TrueFoundry), on TrueForge, with Daytona for sandboxed execution and Qodo for code review.

<script>
(function(){
  var els=document.querySelectorAll('.cr-pipe,.cr-vs,.cr-chart,.cr-finds');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.18});
  els.forEach(function(e){io.observe(e)});
})();
</script>
