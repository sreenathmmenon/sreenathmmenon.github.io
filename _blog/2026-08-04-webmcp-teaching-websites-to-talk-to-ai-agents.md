---
title: "WebMCP: Teaching Your Website to Talk to AI Agents"
date: 2026-08-04
excerpt: "Right now, when an AI agent uses a website, it squints at the screen like a person would: reads the pixels, guesses which button is checkout, clicks, and hopes. It's slow, brittle, and it breaks the moment you move a div. WebMCP flips that around. Instead of the agent guessing what your site can do, your site just tells it, in plain, structured tools the agent can call directly. It's an early web standard, it's in Chrome right now behind a trial, and once you see it you'll want to add a tool to your own site in about ten minutes. Here's how it works and why it matters."
tags: [ai, webmcp, mcp, agents, web, browser]
---

<style>
.wm-fig{margin:2.5rem 0;}
.wm-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* scrape vs declare, animated */
.wm-vs{max-width:720px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:1rem;}
@media(max-width:600px){.wm-vs{grid-template-columns:1fr;}}
.wm-vs .col{border:1px solid var(--border);border-radius:14px;padding:1.1rem;background:var(--surface);opacity:0;transform:translateY(10px);transition:opacity .5s ease,transform .5s ease;}
.wm-vs.go .col{opacity:1;transform:none;} .wm-vs.go .col:nth-child(2){transition-delay:.2s;}
.wm-vs .col h4{margin:0 0 .7rem;font-size:.85rem;font-family:var(--font-mono);}
.wm-vs .col.old h4{color:var(--text-2);} .wm-vs .col.new{border-color:var(--accent);} .wm-vs .col.new h4{color:var(--accent);}
.wm-vs .step{font-size:.8rem;color:var(--text-2);line-height:1.4;padding:.35rem 0;border-top:1px solid var(--border);display:flex;gap:.5rem;align-items:baseline;}
.wm-vs .step:first-of-type{border-top:none;}
.wm-vs .step .m{font-family:var(--font-mono);font-size:.7rem;flex:none;}
.wm-vs .col.old .m{color:var(--text-3);} .wm-vs .col.new .m{color:var(--accent);}

/* how it works flow */
.wm-flow{max-width:820px;margin:0 auto;overflow-x:auto;}
.wm-frow{display:flex;flex-wrap:nowrap;gap:.4rem;align-items:stretch;min-width:660px;}
.wm-node{flex:1 1 0;min-width:120px;border:1px solid var(--border-2);border-radius:11px;background:var(--surface);padding:.75rem .7rem;opacity:0;transform:translateY(10px);transition:opacity .45s ease,transform .45s ease;}
.wm-frow.go .wm-node{opacity:1;transform:none;}
.wm-frow.go .wm-node:nth-child(1){transition-delay:.05s} .wm-frow.go .wm-node:nth-child(3){transition-delay:.2s}
.wm-frow.go .wm-node:nth-child(5){transition-delay:.35s} .wm-frow.go .wm-node:nth-child(7){transition-delay:.5s}
.wm-node.acc{border-color:var(--accent);}
.wm-node .st{font-family:var(--font-mono);font-size:.65rem;color:var(--accent);}
.wm-node b{display:block;color:var(--text);font-size:.81rem;margin:.2rem 0 .1rem;line-height:1.25;}
.wm-node span{font-size:.73rem;color:var(--text-3);line-height:1.35;}
.wm-arr{align-self:center;color:var(--text-3);font-family:var(--font-mono);flex:none;}

/* agent calling tool, animated typing */
.wm-demo{max-width:560px;margin:0 auto;border:1px solid var(--border-2);border-radius:12px;background:var(--code-bg);overflow:hidden;}
.wm-demo .bar{display:flex;gap:.4rem;padding:.6rem .8rem;border-bottom:1px solid var(--border);}
.wm-demo .bar i{width:10px;height:10px;border-radius:50%;background:var(--border-2);display:block;}
.wm-demo .body{padding:1rem 1.1rem;font-family:var(--font-mono);font-size:.8rem;line-height:1.7;}
.wm-line{opacity:0;transform:translateY(4px);transition:opacity .4s ease,transform .4s ease;}
.wm-demo.go .wm-line{opacity:1;transform:none;}
.wm-demo.go .wm-line:nth-child(1){transition-delay:.2s} .wm-demo.go .wm-line:nth-child(2){transition-delay:.8s}
.wm-demo.go .wm-line:nth-child(3){transition-delay:1.5s} .wm-demo.go .wm-line:nth-child(4){transition-delay:2.2s}
.wm-line .u{color:var(--text-3);} .wm-line .a{color:var(--accent);} .wm-line .t{color:var(--accent-2);} .wm-line .r{color:var(--green);}
.wm-cursor{display:inline-block;width:7px;height:1em;background:var(--accent);vertical-align:text-bottom;animation:wmblink 1s step-end infinite;}
@keyframes wmblink{50%{opacity:0;}}

/* code block */
.wm-code{max-width:640px;margin:0 auto;border:1px solid var(--border);border-radius:12px;background:var(--code-bg);overflow:hidden;}
.wm-code .hd{font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);padding:.6rem .9rem;border-bottom:1px solid var(--border);}
.wm-code pre{margin:0;padding:1rem 1.1rem;overflow-x:auto;font-family:var(--font-mono);font-size:.8rem;line-height:1.6;color:var(--text-2);}
.wm-code .k{color:var(--accent-2);} .wm-code .s{color:var(--green);} .wm-code .f{color:var(--accent);} .wm-code .c{color:var(--text-3);}

/* pros cons */
.wm-pc{max-width:720px;margin:0 auto;display:grid;grid-template-columns:1fr 1fr;gap:1rem;}
@media(max-width:600px){.wm-pc{grid-template-columns:1fr;}}
.wm-pc .side{border:1px solid var(--border);border-radius:14px;background:var(--surface);overflow:hidden;opacity:0;transform:translateY(10px);transition:opacity .5s ease,transform .5s ease;}
.wm-pc.go .side{opacity:1;transform:none;} .wm-pc.go .side:nth-child(2){transition-delay:.2s;}
.wm-pc .side .hd{padding:.8rem 1.1rem;border-bottom:1px solid var(--border);font-family:var(--font-mono);font-size:.82rem;font-weight:600;}
.wm-pc .side.pro .hd{color:var(--green);} .wm-pc .side.con .hd{color:var(--text-2);}
.wm-pc .side .li{padding:.55rem 1.1rem;font-size:.84rem;color:var(--text-2);line-height:1.45;border-top:1px solid var(--border);position:relative;padding-left:2rem;}
.wm-pc .side .li:first-of-type{border-top:none;}
.wm-pc .side .li::before{position:absolute;left:1.1rem;font-family:var(--font-mono);}
.wm-pc .side.pro .li::before{content:"+";color:var(--green);}
.wm-pc .side.con .li::before{content:"\2212";color:var(--text-3);}

/* use cases */
.wm-uses{max-width:760px;margin:0 auto;display:grid;grid-template-columns:repeat(3,1fr);gap:.8rem;}
@media(max-width:640px){.wm-uses{grid-template-columns:1fr;}}
.wm-use{border:1px solid var(--border-2);border-radius:12px;padding:1rem;background:var(--surface);opacity:0;transform:translateY(8px);transition:opacity .45s ease,transform .45s ease;}
.wm-uses.go .wm-use{opacity:1;transform:none;}
.wm-uses.go .wm-use:nth-child(1){transition-delay:.08s} .wm-uses.go .wm-use:nth-child(2){transition-delay:.16s}
.wm-uses.go .wm-use:nth-child(3){transition-delay:.24s} .wm-uses.go .wm-use:nth-child(4){transition-delay:.32s}
.wm-uses.go .wm-use:nth-child(5){transition-delay:.40s} .wm-uses.go .wm-use:nth-child(6){transition-delay:.48s}
.wm-use .t{font-family:var(--font-mono);font-size:.8rem;color:var(--accent);font-weight:600;margin-bottom:.3rem;}
.wm-use .d{font-size:.8rem;color:var(--text-2);line-height:1.45;}

/* status callout */
.wm-status{max-width:640px;margin:0 auto;border:1px solid var(--accent);border-radius:12px;background:var(--surface);padding:1.1rem 1.3rem;}
.wm-status .lb{font-family:var(--font-mono);font-size:.72rem;color:var(--accent);letter-spacing:.08em;text-transform:uppercase;margin-bottom:.5rem;}
.wm-status p{font-size:.86rem;color:var(--text-2);line-height:1.55;margin:0;}
.wm-status code{font-family:var(--font-mono);font-size:.8rem;background:var(--surface-2);border:1px solid var(--border);border-radius:5px;padding:.08em .4em;color:var(--text);}

@media (prefers-reduced-motion: reduce){
  .wm-vs .col,.wm-node,.wm-line,.wm-pc .side,.wm-use{opacity:1!important;transform:none!important;}
  .wm-cursor{animation:none!important;}
}
</style>

Picture an AI agent trying to book you a table on a restaurant's website. Today, it works like a very patient, slightly confused intern. It loads the page, reads the raw HTML, tries to figure out which of the forty `<div>` elements is the date picker, guesses that the green button probably means "confirm," clicks it, waits, and re-reads the whole screen to see if anything happened. Move that button next week and the agent breaks. Rename a CSS class and it breaks. Add a cookie banner on top and it clicks the wrong thing entirely.

This is how almost all "agents using websites" works right now: screen-scraping and hoping. It's the automation equivalent of operating a computer by describing screenshots over the phone.

**WebMCP** proposes something much saner. Instead of the agent guessing what your site can do by staring at it, your site *declares* what it can do, as a set of clean, structured tools the agent can call directly. "Here's a `book_table` tool. It takes a date, a time, and a party size. Call it." No pixel-reading. No guessing. And the best part: it already runs in Chrome behind a trial, and adding your first tool takes about ten minutes.

Let me show you the whole thing.

## The core shift: from scraping to declaring

The entire idea fits in one comparison. Same task, two worlds.

<figure class="wm-fig">
<div class="wm-vs">
  <div class="col old">
    <h4>Today: the agent scrapes</h4>
    <div class="step"><span class="m">1</span>Read the entire DOM</div>
    <div class="step"><span class="m">2</span>Guess which element is the date field</div>
    <div class="step"><span class="m">3</span>Simulate typing and clicking</div>
    <div class="step"><span class="m">4</span>Re-read the whole page to check</div>
    <div class="step"><span class="m">5</span>Break when the layout changes</div>
  </div>
  <div class="col new">
    <h4>WebMCP: the site declares</h4>
    <div class="step"><span class="m">1</span>Page registers a <b>book_table</b> tool</div>
    <div class="step"><span class="m">2</span>Agent reads the tool's schema</div>
    <div class="step"><span class="m">3</span>Agent calls it with structured args</div>
    <div class="step"><span class="m">4</span>Tool runs your real JS, returns a result</div>
    <div class="step"><span class="m">5</span>Survives redesigns: the tool is the contract</div>
  </div>
</div>
<figcaption>The left column is brittle because the agent is reverse-engineering your UI every time. The right column is stable because you gave it a real interface. The layout can change freely underneath a tool whose name and schema stay the same.</figcaption>
</figure>

If you've read my earlier post on [MCP, the port that let AI touch the world](/blog/2026-06-30-mcp-the-port-that-let-ai-touch-the-world/), this will feel familiar, and it should. MCP gave AI a standard way to call tools on a server. WebMCP brings that same idea into the browser: the web *page itself* becomes a place that offers tools, running in the tab you already have open, with the session you're already logged into.

## What it actually is

WebMCP is a proposed web standard, developed jointly by Google (Chrome) and Microsoft (Edge) in the W3C Web Machine Learning Community Group, that gives a web page a small JavaScript API to register tools that an AI agent can discover and call. Google describes it plainly in the Chrome docs: a way to "build and expose structured tools for AI agents," where the site annotates its own features so agents "know exactly how to interact" with them. To be precise about maturity, it's a Community Group *draft*, not a finished W3C standard and not yet on the standards track, which is exactly why now is the moment to learn it and shape it.

Three things make it click into place:

- **Discovery.** A standard way for a page to say "I offer these tools," like `checkout` or `filter_results`, so an agent can list them.
- **Schemas.** Each tool declares its inputs and outputs as JSON Schema, so the agent knows exactly what to pass and there's far less room to hallucinate or misread.
- **State.** A shared understanding of what's on the page right now, so the agent knows what it can actually act on.

<figure class="wm-fig">
<div class="wm-status">
  <div class="lb">Where it stands today</div>
  <p>WebMCP is real and runnable, but early. It's available as a <b>Chrome origin trial from Chrome 149</b>, and you can switch it on locally with the flag <code>chrome://flags/#enable-webmcp-testing</code>. The proposal lives at <code>github.com/webmachinelearning/webmcp</code>, Angular already has experimental support, and Chrome ships demo sites (a pizza maker, travel search, a restaurant booking). Google's own words: it's "under active discussion and subject to change." So this is a "try it and shape it" moment, not a "ship it to production" one, and that's exactly why it's worth learning now.</p>
</div>
</figure>

## How a call actually flows

Here's the whole loop, page to agent and back. Nothing exotic happens: the page registers tools, the agent lists them, picks one, calls it with structured arguments, and your own JavaScript does the work in the page.

<figure class="wm-fig">
<div class="wm-flow">
<div class="wm-frow">
  <div class="wm-node"><span class="st">page</span><b>Register tools</b><span>Your JS declares book_table, search, etc.</span></div>
  <div class="wm-arr">&rarr;</div>
  <div class="wm-node"><span class="st">agent</span><b>Discover</b><span>Lists the page's tools and their schemas</span></div>
  <div class="wm-arr">&rarr;</div>
  <div class="wm-node acc"><span class="st">agent</span><b>Call with args</b><span>Structured JSON matching the schema</span></div>
  <div class="wm-arr">&rarr;</div>
  <div class="wm-node acc"><span class="st">page</span><b>execute() runs</b><span>Your real JS, in the logged-in page</span></div>
  <div class="wm-arr">&rarr;</div>
  <div class="wm-node"><span class="st">agent</span><b>Gets result</b><span>A structured answer, visibly, in the tab</span></div>
</div>
</div>
<figcaption>The tool's execute function runs inside your actual page, using your existing JavaScript, state, and the user's own logged-in session. It happens visibly in the tab, not in some invisible headless browser, so the user can watch it and trust it.</figcaption>
</figure>

That "runs in the page you're already logged into" detail is a big deal. The agent isn't a separate bot logging in with stolen credentials somewhere. It's calling a function in *your* open, authenticated tab, using the session you already have. The site keeps control of what it exposes, and the user can see it happen.

## Watch one call happen

Concretely, when you ask an in-browser agent to do something on a WebMCP-enabled site, it looks like this: your request, the agent picking the declared tool, the tool running, the result.

<figure class="wm-fig">
<div class="wm-demo">
  <div class="bar"><i></i><i></i><i></i></div>
  <div class="body">
    <div class="wm-line"><span class="u">you &rsaquo;</span> book a table for 4 tonight at 8</div>
    <div class="wm-line"><span class="a">agent &rsaquo;</span> found tool <span class="t">book_table</span> on this page</div>
    <div class="wm-line"><span class="a">agent &rsaquo;</span> calling <span class="t">book_table</span>({ date: "today", time: "20:00", party: 4 })</div>
    <div class="wm-line"><span class="r">page &rsaquo;</span> Booked. Table for 4 at 8:00 PM, confirmation #A17.<span class="wm-cursor"></span></div>
  </div>
</div>
<figcaption>No DOM guessing anywhere in that exchange. The agent called a named function with typed arguments, and the page did the rest with its own code. This is the difference between an agent operating your site and an agent operating a photograph of your site.</figcaption>
</figure>

## The code is genuinely tiny

This is the part that makes people want to try it. Registering a tool is one call. Using the current imperative API from the Chrome docs, a to-do site adding an "add item" tool looks essentially like this:

<figure class="wm-fig">
<div class="wm-code">
  <div class="hd">register a WebMCP tool (imperative API)</div>
<pre><span class="k">await</span> document.modelContext.<span class="f">registerTool</span>({
  name: <span class="s">'add_todo'</span>,
  description: <span class="s">'Add an item to the to-do list'</span>,
  inputSchema: {
    type: <span class="s">'object'</span>,
    properties: { text: { type: <span class="s">'string'</span> } },
    required: [<span class="s">'text'</span>]
  },
  execute: <span class="k">async</span> ({ text }) => {
    addTodoToPage(text);        <span class="c">// your own existing function</span>
    <span class="k">return</span> <span class="s">`Added to-do: ${text}`</span>;
  }
});</pre>
</div>
<figcaption>That's the whole thing. You give the tool a name, a description, an input schema, and an execute function that calls code you already wrote. The agent discovers it with getTools(), and you can pull a tool back with an AbortController if it stops being relevant. There's also a declarative flavor where you annotate an HTML form instead of writing JS.</figcaption>
</figure>

Notice what `execute` does: it calls `addTodoToPage`, a function that already exists on your site. WebMCP isn't asking you to rebuild anything. You're wrapping the actions your site can already do in a thin, declared interface so an agent can reach them cleanly. That's why the ten-minutes claim is real.

One accuracy note, because the API is young and moving: the entry point recently moved from `navigator.modelContext` (the original name, now deprecated) to `document.modelContext`, since tools really belong to a document, not the whole browser. If you follow an older tutorial showing `navigator`, that's why. A one-line shim (`const mc = document.modelContext || navigator.modelContext`) bridges both while the change rolls out. Expect a few more edges like this to shift; it's a draft.

## The trust model, because this is the scary part

The obvious worry: if a page can hand tools to an agent, can a malicious page trick the agent into doing something awful? The design takes this seriously, and it's worth knowing the guards.

- **It runs visibly, in the tab.** No headless, background execution. A browsing context has to be open, so actions happen where the user can see them.
- **Origin-isolated only.** Tools can only be registered in origin-isolated documents, and it's gated by a `tools` Permissions Policy that defaults to `self`, so a random cross-origin iframe can't quietly register tools.
- **Sensitive actions can demand a human.** For things like making a purchase, a tool can require an explicit user confirmation dialog before it proceeds. Human-in-the-loop is built into the pattern, not bolted on.
- **Untrusted content is flagged.** Tools carry annotation hints like `readOnlyHint` and `untrustedContentHint`, so the agent can treat a tool that returns third-party content with appropriate suspicion, which matters given everything we know about [prompt injection](/blog/2026-08-02-llm-security-prompt-injection-and-the-cost-of-defending/).

None of this makes it magically safe, the standard is young and the security model is still being worked out, but the shape is right: visible, same-origin, consent-gated, and honest about untrusted data.

## Honest pros and cons

<figure class="wm-fig">
<div class="wm-pc">
  <div class="side pro">
    <div class="hd">Why it's exciting</div>
    <div class="li">Structured tool calls instead of brittle DOM scraping</div>
    <div class="li">Runs in the user's real, logged-in session, no separate bot auth</div>
    <div class="li">The site stays in control of what it exposes</div>
    <div class="li">Survives redesigns: the tool contract outlives the layout</div>
    <div class="li">Reuses code you already have, tiny to adopt</div>
    <div class="li">A real standard direction, not one vendor's lock-in</div>
  </div>
  <div class="side con">
    <div class="hd">Why it's early</div>
    <div class="li">Experimental: origin trial only, subject to change</div>
    <div class="li">Chrome-first today; broad browser support isn't here yet</div>
    <div class="li">Needs site adoption to matter; agents can't call tools that don't exist</div>
    <div class="li">Discoverability gap: a client must visit a site to learn its tools</div>
    <div class="li">Security model still maturing (malicious tools, injection)</div>
    <div class="li">Complex sites may need real refactoring to expose clean tools</div>
  </div>
</div>
<figcaption>The cons are almost all "it's early," not "it's wrong." That's the profile of a promising standard in its incubation window: the idea is sound, the ecosystem hasn't caught up yet.</figcaption>
</figure>

## Where it fits: use cases

The pattern shines anywhere an agent needs to *do* something on a site, not just read it.

<figure class="wm-fig">
<div class="wm-uses">
  <div class="wm-use"><div class="t">E-commerce</div><div class="d">Expose search_products, add_to_cart, checkout. An agent shops your store through real tools, not by clicking around.</div></div>
  <div class="wm-use"><div class="t">Booking &amp; travel</div><div class="d">Multi-city, multi-passenger trips or restaurant tables, where the form is complex and scraping is painful.</div></div>
  <div class="wm-use"><div class="t">SaaS dashboards</div><div class="d">Let an agent run the actions your app already has: create a ticket, filter a report, update a record.</div></div>
  <div class="wm-use"><div class="t">Form filling</div><div class="d">Declare the form's fields and a submit tool; the agent maps data in cleanly instead of guessing inputs.</div></div>
  <div class="wm-use"><div class="t">Accessibility</div><div class="d">A declared, semantic tool surface is a gift for assistive agents, clearer intent than raw markup.</div></div>
  <div class="wm-use"><div class="t">Internal tools</div><div class="d">Wrap your admin panel's actions as tools so an internal assistant can drive them safely and visibly.</div></div>
</div>
<figcaption>The through-line: any site that has "things you can do," not just "things you can read," is a candidate. The richer your site's actions, the more WebMCP gives you.</figcaption>
</figure>

## Where this is headed

Now the fun part, because the ceiling here is high.

The obvious next step is **standardization across browsers.** Right now it's a Chrome trial; the destination is a web standard every browser implements, the way fetch or the clipboard API are everywhere. When that lands, "does this site have an agent interface?" becomes as normal a question as "is this site mobile-friendly?"

Then there's the **agentic web** itself. Imagine sites shipping an agent interface alongside their visual one, on purpose, the way they ship a mobile layout today. Your site's UI is for humans; its declared tools are for agents; both are first-class. A site that's good at being operated by an agent gets used by more agents, which becomes a real reason to invest in the tool surface.

It gets more interesting when you **combine WebMCP with remote MCP.** WebMCP handles what lives in the browser (the page's own actions, the user's session), while remote MCP servers handle backend tools and data. An agent could fluidly use both: call a page's `add_to_cart` tool via WebMCP, then hit a remote inventory MCP server for stock, stitching client and server tools into one task.

And further out: **agent commerce.** If a store exposes clean purchase tools and an agent can call them within the user's authenticated, consenting session, you get a path to agents that actually complete transactions safely, with the human able to watch and confirm, rather than a scraper hammering a checkout flow. The same shape extends to booking, scheduling, support, anything transactional.

The big bet underneath all of it: the web was built for humans to read and click. The next version is built to *also* be operated by agents, cleanly and on the site's own terms. WebMCP is one of the first serious attempts to make that a standard instead of a hack.

## The takeaway, and go try it

Here's the whole thing in a sentence: **WebMCP lets your website hand an AI agent a clean set of tools instead of forcing it to reverse-engineer your buttons.** That single shift, declare instead of scrape, makes agent interactions reliable, keeps the site in control, runs in the user's real session, and survives your next redesign.

It's early, it's Chrome-first, and the standard will change. But the barrier to trying it is almost nothing: flip on `chrome://flags/#enable-webmcp-testing`, add a `registerTool` call wrapping a function your site already has, and watch an agent call it. Ten minutes, and you'll understand the agentic web better than most people reading about it. Then go read the proposal, poke the demos, and file the rough edges you hit, because right now, while it's still being shaped, your feedback actually moves it.

## References

Written from scratch after reading the official documentation. These are the primary, verified sources. Nothing here is copied from them; the code shape follows the documented API.

- Chrome for Developers, WebMCP overview: <https://developer.chrome.com/docs/ai/webmcp>
- Chrome for Developers, WebMCP imperative API: <https://developer.chrome.com/docs/ai/webmcp/imperative-api>
- Chrome for Developers, WebMCP declarative API: <https://developer.chrome.com/docs/ai/webmcp/declarative-api>
- WebMCP specification (Draft Community Group Report): <https://webmachinelearning.github.io/webmcp/>
- WebMCP proposal, explainer, and source (W3C Web Machine Learning Community Group): <https://github.com/webmachinelearning/webmcp>
- Chrome Platform Status, WebMCP feature: <https://chromestatus.com/feature/5117755740913664>
- WebMCP demo sites (Google Chrome Labs): <https://github.com/GoogleChromeLabs/webmcp-tools/tree/main/demos>
- Patrick Brosset (Microsoft Edge), WebMCP updates and clarifications: <https://patrickbrosset.com/articles/2026-02-23-webmcp-updates-clarifications-and-next-steps/>
- Model Context Protocol (MCP), for the underlying protocol: <https://modelcontextprotocol.io>

*Background reading: [MCP: The Port That Let AI Finally Touch the World](/blog/2026-06-30-mcp-the-port-that-let-ai-touch-the-world/) for the protocol WebMCP builds on, and [LLM security](/blog/2026-08-02-llm-security-prompt-injection-and-the-cost-of-defending/) for why the trust model here matters.*

<script>
(function(){
  var els=document.querySelectorAll('.wm-vs,.wm-frow,.wm-demo,.wm-pc,.wm-uses');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.18});
  els.forEach(function(e){io.observe(e)});
})();
</script>
