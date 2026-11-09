---
title: "How a WhatsApp AI Assistant Like Instinct Works"
date: 2026-11-09
excerpt: "You text a phone number 'book me a table for four Friday at 8' and an hour later it's done, no app, no dashboard, just a reply. That's Instinct, the invite-only assistant that hit a $10B valuation this year. The magic is that it has no special access to anything: it drives a real phone and a real computer the way you would. Here's what's actually public about how it works, and how you'd design a system like it yourself, kept honest about what's confirmed and what's a guess."
tags: [ai, agents, system-design]
---

<style>
.in-fig{margin:2.4rem 0;}
.in-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.8rem;text-align:center;line-height:1.5;}

/* chat flow */
.in-chat{max-width:460px;margin:0 auto;border:1px solid var(--border);border-radius:16px;background:var(--surface);padding:1rem;}
.in-chat .hd{font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);text-align:center;margin-bottom:.8rem;border-bottom:1px solid var(--border);padding-bottom:.6rem;}
.in-msg{max-width:80%;padding:.55rem .8rem;border-radius:14px;font-size:.88rem;line-height:1.45;margin:.4rem 0;opacity:0;transform:translateY(8px);transition:opacity .4s var(--ease),transform .4s var(--ease);}
.in-chat.go .in-msg{opacity:1;transform:none;}
.in-chat.go .in-msg:nth-child(2){transition-delay:.1s}.in-chat.go .in-msg:nth-child(3){transition-delay:.45s}.in-chat.go .in-msg:nth-child(4){transition-delay:.8s}.in-chat.go .in-msg:nth-child(5){transition-delay:1.15s}
.in-msg.you{background:var(--accent);color:var(--accent-ink);margin-left:auto;border-bottom-right-radius:5px;}
.in-msg.bot{background:var(--surface-2);color:var(--text);border:1px solid var(--border);border-bottom-left-radius:5px;}
.in-msg .work{font-family:var(--font-mono);font-size:.74rem;color:var(--text-3);}

/* pipeline */
.in-pipe{max-width:760px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.in-stage{display:flex;align-items:center;gap:.85rem;border:1px solid var(--border);border-radius:11px;background:var(--surface);padding:.72rem .95rem;opacity:0;transform:translateX(-10px);transition:opacity .5s var(--ease),transform .5s var(--ease);}
.in-pipe.go .in-stage{opacity:1;transform:none;}
.in-pipe.go .in-stage:nth-child(1){transition-delay:.05s}.in-pipe.go .in-stage:nth-child(2){transition-delay:.12s}.in-pipe.go .in-stage:nth-child(3){transition-delay:.19s}.in-pipe.go .in-stage:nth-child(4){transition-delay:.26s}.in-pipe.go .in-stage:nth-child(5){transition-delay:.33s}.in-pipe.go .in-stage:nth-child(6){transition-delay:.4s}
.in-stage .nm{flex:none;font-family:var(--font-mono);font-size:.72rem;font-weight:600;color:var(--accent);min-width:92px;}
.in-stage.gate .nm{color:var(--red);} .in-stage.gate{border-color:var(--red);background:var(--surface-2);}
.in-stage .d{font-size:.84rem;color:var(--text-2);line-height:1.4;} .in-stage .d b{color:var(--text);}
.in-stage .badge{margin-left:auto;flex:none;font-family:var(--font-mono);font-size:.62rem;text-transform:uppercase;border:1px solid var(--red);color:var(--red);border-radius:5px;padding:.14rem .45rem;}

/* connect diagram */
.in-conn{max-width:720px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:.8rem;}
@media(max-width:560px){.in-conn{grid-template-columns:1fr;}}
.in-path{border:1px solid var(--border);border-radius:12px;background:var(--surface);padding:1rem;opacity:0;transform:translateY(8px);transition:opacity .45s var(--ease),transform .45s var(--ease);}
.in-conn.go .in-path{opacity:1;transform:none;}
.in-conn.go .in-path:nth-child(1){transition-delay:.06s}.in-conn.go .in-path:nth-child(2){transition-delay:.16s}.in-conn.go .in-path:nth-child(3){transition-delay:.26s}.in-conn.go .in-path:nth-child(4){transition-delay:.36s}
.in-path .t{font-family:var(--font-mono);font-size:.8rem;color:var(--accent);font-weight:600;margin-bottom:.3rem;}
.in-path .b{font-size:.83rem;color:var(--text-2);line-height:1.5;}

/* confirmed vs unknown table */
.in-tab-wrap{max-width:780px;margin:0 auto;overflow-x:auto;border:1px solid var(--border);border-radius:12px;background:var(--surface);}
.in-tab{width:100%;border-collapse:collapse;font-size:.85rem;min-width:560px;}
.in-tab th,.in-tab td{text-align:left;padding:.62rem .82rem;border-bottom:1px solid var(--border);vertical-align:top;line-height:1.4;}
.in-tab thead th{font-family:var(--font-mono);font-size:.68rem;text-transform:uppercase;letter-spacing:.04em;font-weight:600;}
.in-tab thead th:nth-child(2){color:var(--green);} .in-tab thead th:nth-child(3){color:var(--red);}
.in-tab tbody td:first-child{color:var(--text);font-weight:500;}
.in-tab td{color:var(--text-2);}
.in-tab tr:last-child td{border-bottom:none;}
.in-tab .y{color:var(--green);} .in-tab .n{color:var(--red);}

/* memory loop */
.in-loop{max-width:640px;margin:0 auto;display:flex;flex-direction:column;gap:.5rem;}
.in-step{display:flex;align-items:center;gap:.8rem;border:1px solid var(--border);border-radius:11px;background:var(--surface);padding:.7rem .9rem;opacity:0;transform:scale(.97);transition:opacity .45s var(--ease),transform .45s var(--ease);}
.in-loop.go .in-step{opacity:1;transform:none;}
.in-loop.go .in-step:nth-child(1){transition-delay:.06s}.in-loop.go .in-step:nth-child(2){transition-delay:.16s}.in-loop.go .in-step:nth-child(3){transition-delay:.26s}.in-loop.go .in-step:nth-child(4){transition-delay:.36s}
.in-step .n{flex:none;width:22px;height:22px;border-radius:50%;background:var(--surface-2);border:1px solid var(--accent);color:var(--accent);font-family:var(--font-mono);font-size:.72rem;display:flex;align-items:center;justify-content:center;}
.in-step .d{font-size:.85rem;color:var(--text-2);line-height:1.45;} .in-step .d b{color:var(--text);}

@media (prefers-reduced-motion: reduce){
  .in-msg,.in-stage,.in-path,.in-step{opacity:1!important;transform:none!important;}
}
</style>

There's no app. You don't log into a dashboard. You text a phone number like you'd text a friend, "rebook my Friday flight, I'll miss the connection", and some time later a message comes back: done. That's **Instinct**, the invite-only AI assistant that went from private beta to a $10 billion valuation in about a year, and it's the clearest example yet of where personal assistants are heading.

The interesting part isn't the chat. It's what happens in the gap between your message and the reply. Instinct has no magic API into the airline. It does the thing you'd do: it opens a browser, logs into your account, clicks through the rebooking flow, and if the website won't cooperate, it picks up a phone and calls them. Let me walk through what's actually public about how it works, then how you'd design a system like it.

## What it feels like

<figure class="in-fig">
<div class="in-chat wm-anim">
  <div class="hd">+1 (555) ··· · WhatsApp</div>
  <div class="in-msg you">book a table for 4 this Friday at 8, somewhere good near me</div>
  <div class="in-msg bot">On it. Checking spots near you with Friday availability.<span class="work"><br>· searching · reading menus · checking tables</span></div>
  <div class="in-msg bot">Found 3. I'd pick Osteria (your kind of place, 8:15 is open). Book it?</div>
  <div class="in-msg you">yes</div>
  <div class="in-msg bot">Booked. Table for 4, Friday 8:15. Confirmation in your email.</div>
</div>
<figcaption>One phone number, plain messages, a real result. No app to install, no forms to fill. Instinct reaches people over SMS, WhatsApp, iMessage, email, and voice calls, from its own dedicated number.</figcaption>
</figure>

## How it actually works (the confirmed parts)

Three facts from Instinct's own site and the reporting around it tell you almost everything:

**It has no new interface.** In its own words, "there are no new interfaces. You can text or call it. It can use a phone and computer the same way that you do." That last line is the whole design. Instead of integrating with a thousand services, it operates a real computer and a real phone like a person, so it works even on a website or a shop that has no API at all.

**It connects to your digital life to get the job done.** With your permission, it links to your email, messaging apps, and calendar. That's how it finishes real tasks: when a booking site texts a confirmation code, it reads the code from your inbox and types it in. When a login needs a six-digit authenticator code, it handles that too.

**When the web won't do, it calls.** Instinct added a "concierge" that literally phones businesses, to book the restaurant that only takes reservations over the phone, or chase a refund that needs a human on the line.

**And the assistants are starting to talk to each other.** It has a "trusted person network" where your assistant can coordinate plans with your friends' assistants, and it's wiring up to partners through proper tool protocols (for example, booking doctor visits through Zocdoc's [MCP](/blog/2026-06-30-mcp-the-port-that-let-ai-touch-the-world/) connection) rather than clicking through a browser when a clean API exists.

## How you'd design a system like it

Here's where I'll be honest: Instinct hasn't published its architecture, so the pipeline below is **how I'd build it**, grounded in what's confirmed, not a leak of their internals. But the shape is forced by the problem. If you want "text a number, it does real things," you need roughly these stages.

<figure class="in-fig">
<div class="in-pipe wm-anim">
  <div class="in-stage"><span class="nm">CHANNEL</span><span class="d"><b>Receive the message.</b> A webhook from the WhatsApp / SMS / iMessage / email provider lands your text on the server, tied to your account by phone number.</span></div>
  <div class="in-stage"><span class="nm">UNDERSTAND</span><span class="d"><b>Turn words into a goal.</b> The model reads your message plus your memory and decides what you actually want and what it still needs to ask.</span></div>
  <div class="in-stage"><span class="nm">PLAN</span><span class="d"><b>Break it into steps.</b> Find restaurants, check Friday availability, pick one, book. Each step is a concrete action, not a vibe.</span></div>
  <div class="in-stage"><span class="nm">ACT</span><span class="d"><b>Drive a real computer.</b> In an isolated cloud machine, it opens a browser, logs into your accounts, clicks, and types, reading codes from your email when a site asks.</span></div>
  <div class="in-stage gate"><span class="nm">GATE</span><span class="d"><b>Stop before anything binding.</b> Spending money or sending in your name waits for your "yes." A wrong paragraph is cheap; a wrong flight is not.</span><span class="badge">human</span></div>
  <div class="in-stage"><span class="nm">REPLY</span><span class="d"><b>Report back in the thread.</b> A plain message: done, here's the confirmation, or here's the one thing I need from you.</span></div>
</div>
<figcaption>How I'd architect it, not a confirmed Instinct diagram. The load-bearing idea is the ACT stage: a real, sandboxed computer the agent operates like a human, which is exactly why it works on services with no API.</figcaption>
</figure>

## How it connects, where the data comes from

The "no API needed" claim rests on four ways in. Most tasks use a mix.

<figure class="in-fig">
<div class="in-conn wm-anim">
  <div class="in-path"><div class="t">Your accounts</div><div class="b">You connect email, calendar, and messaging. This is the <b>context</b>: who you are, what's on your schedule, and the inbox it reads confirmation and login codes from.</div></div>
  <div class="in-path"><div class="t">A real browser</div><div class="b">For sites with no API, it drives a browser in a cloud machine, the same clicks and typing you'd do. This is the universal fallback that makes it work anywhere.</div></div>
  <div class="in-path"><div class="t">A real phone</div><div class="b">When there's no website at all, the concierge calls the business and talks to a human, so "book the place that only answers the phone" actually works.</div></div>
  <div class="in-path"><div class="t">Clean protocols</div><div class="b">Where a partner exposes proper tools (via MCP or an agent protocol), it uses those directly, faster and more reliable than clicking, no scraping needed.</div></div>
</div>
<figcaption>Four paths, in order of preference: a clean API if one exists, otherwise a browser, otherwise a phone call. The browser and the phone are what let it claim it can do anything a person can.</figcaption>
</figure>

## Where it stores your data, and why that's the scary part

To act for you, it has to *remember* you, your preferences, your history, the accounts it's connected to. This is both the feature and the risk, and Instinct's early stumble makes the lesson concrete.

<figure class="in-fig">
<div class="in-loop wm-anim">
  <div class="in-step"><span class="n">1</span><span class="d"><b>Index your connected data.</b> It reads and stores a searchable copy of your email and calendar so it can find that confirmation code or know you're free Friday.</span></div>
  <div class="in-step"><span class="n">2</span><span class="d"><b>Keep a preference memory.</b> The founder describes it as "learning your preferences, storing it in memory," then using that memory to book things exactly how you like next time.</span></div>
  <div class="in-step"><span class="n">3</span><span class="d"><b>Reuse it on the next task.</b> Over time it asks fewer questions, because it already knows your usual airline, your dietary limits, your home address.</span></div>
  <div class="in-step"><span class="n">4</span><span class="d"><b>And here's the catch.</b> Disconnecting a service does not automatically erase what it already indexed, you have to explicitly request deletion. Memory that persists is memory you have to be able to revoke.</span></div>
</div>
<figcaption>The memory is what makes it feel like magic, and what makes it dangerous. Instinct's original terms granted a sweeping "perpetual and irrevocable" license over your data; after a public backlash in August 2026 it revised them and added a deletion tool. The lesson: an assistant's memory must be inspectable and revocable, or trust resets to zero.</figcaption>
</figure>

There's a nice footnote here. Instinct's founder, Noah Shinn, wrote the well-known **Reflexion** paper, agents that improve by reflecting on their own feedback and storing the lesson in memory for next time. I can't confirm Instinct runs that exact technique, but its "learn your preferences, do better next time" behavior clearly rhymes with that work. The memory isn't a side feature; it's the whole personality.

## What's confirmed, and what I'm guessing

Because it's a closed product, keeping these straight matters:

<figure class="in-fig">
<div class="in-tab-wrap">
<table class="in-tab">
<thead><tr><th>Claim</th><th>Confirmed (public)</th><th>Unknown / my guess</th></tr></thead>
<tbody>
<tr><td>No app, text or call it</td><td class="y">Yes, from its own site</td><td class="n"></td></tr>
<tr><td>Drives a real browser + phone like a human</td><td class="y">Yes, stated and reported</td><td class="n"></td></tr>
<tr><td>Reads codes from your email to finish tasks</td><td class="y">Yes, reported</td><td class="n"></td></tr>
<tr><td>Indexes + retains your data, revised its terms</td><td class="y">Yes, well documented</td><td class="n"></td></tr>
<tr><td>Founder wrote the Reflexion paper</td><td class="y">Yes</td><td class="n"></td></tr>
<tr><td>The exact pipeline above</td><td class="y"></td><td class="n">My design, not theirs</td></tr>
<tr><td>Which AI model it runs</td><td class="y"></td><td class="n">Not public</td></tr>
<tr><td>The storage / infra details</td><td class="y"></td><td class="n">Not public</td></tr>
</tbody>
</table>
</div>
<figcaption>The confirmed column is enough to understand the product. The architecture is a sound reconstruction, and I'd rather tell you that than dress a guess up as a leak.</figcaption>
</figure>

## What I take from it

Two things. First, the winning interface for an assistant might be **no interface**, a phone number you already know how to use, backed by an agent that operates the rest of the world on your behalf. That's a bigger idea than any app.

Second, the hard part was never the chat. It's everything the pipeline above has to get right: operating real software without breaking, handling the login codes and phone calls, and above all, **gating the moment it spends your money or acts in your name, and keeping a memory you can actually see and delete.** Build an assistant that's fast and capable but can't be trusted with the "yes," and you've built a liability. Build one that does the boring 80% and stops for you on the 20% that matters, and you've built the thing everyone's racing toward. Instinct's early privacy stumble is the whole industry's lesson in miniature: capability is easy now, trust is the product.

<script>
(function(){
  var els=document.querySelectorAll('.in-chat,.in-pipe,.in-conn,.in-loop');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.18});
  els.forEach(function(e){io.observe(e)});
})();
</script>
