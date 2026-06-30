---
title: "MCP: The Port That Let AI Finally Touch the World"
date: 2026-06-30
excerpt: "For a while, the smartest models on earth were trapped behind glass. They could think, but they couldn't reach. MCP is the standard plug that changed that. Here is what it actually is, why it isn't just another REST API, the three things a server can offer, and how you build one yourself."
tags: [ai, mcp, agents, protocols, developer-tools, explainer]
---

<style>
.mc-fig{margin:2.5rem 0;}
.mc-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.9rem;text-align:center;line-height:1.5;}

/* the trapped-model / port visual */
.mc-port{max-width:560px;margin:0 auto;display:flex;align-items:center;justify-content:center;gap:0;}
.mc-port .box{border:1px solid var(--border-2);border-radius:12px;padding:1rem 1.2rem;background:var(--surface);text-align:center;min-width:120px;}
.mc-port .box b{display:block;color:var(--text);font-size:.95rem;}
.mc-port .box span{font-size:.78rem;color:var(--text-2);}
.mc-port .plug{font-family:var(--font-mono);font-size:.74rem;color:var(--accent);padding:0 .2rem;text-align:center;}
.mc-port .plug .line{height:2px;background:var(--accent);width:46px;margin:.35rem auto;border-radius:2px;}

/* N x M tangle vs hub */
.mc-nm{display:grid;grid-template-columns:1fr 1fr;gap:1rem;max-width:640px;margin:0 auto;}
@media(max-width:560px){.mc-nm{grid-template-columns:1fr;}}
.mc-nm .panel{border:1px solid var(--border);border-radius:12px;padding:1rem;background:var(--surface);}
.mc-nm .panel h4{margin:0 0 .7rem;font-size:.85rem;text-align:center;font-family:var(--font-mono);}
.mc-nm .panel.bad h4{color:var(--text-2);}
.mc-nm .panel.good h4{color:var(--accent);}
.mc-nm .grid{display:flex;flex-direction:column;gap:.4rem;font-family:var(--font-mono);font-size:.74rem;color:var(--text-2);}
.mc-nm .grid .l{display:flex;align-items:center;gap:.4rem;}
.mc-nm .grid .tag{padding:.2rem .45rem;border:1px solid var(--border-2);border-radius:6px;background:var(--surface-2);}
.mc-nm .grid .x{color:var(--text-3);}
.mc-nm .grid .hub{color:var(--accent);border-color:var(--accent);}

/* lifecycle flow animation */
.mc-flow{max-width:620px;margin:0 auto;}
.mc-step{display:flex;align-items:flex-start;gap:1rem;padding:.8rem 1rem;border:1px solid var(--border);border-radius:12px;background:var(--surface);margin-bottom:.65rem;opacity:.2;transform:translateX(-6px);transition:opacity .5s ease,transform .5s ease,border-color .5s ease;}
.mc-flow.go .mc-step{opacity:1;transform:none;}
.mc-flow.go .mc-step.f1{transition-delay:.1s;border-color:var(--border-2);}
.mc-flow.go .mc-step.f2{transition-delay:.8s;border-color:var(--border-2);}
.mc-flow.go .mc-step.f3{transition-delay:1.5s;border-color:var(--border-2);}
.mc-flow.go .mc-step.f4{transition-delay:2.2s;border-color:var(--accent);}
.mc-step .n{flex:none;width:30px;height:30px;border-radius:50%;background:var(--grad);color:var(--accent-ink);font-family:var(--font-mono);font-weight:600;font-size:.82rem;display:flex;align-items:center;justify-content:center;}
.mc-step .t{flex:1;}
.mc-step .t b{display:block;color:var(--text);font-size:.92rem;}
.mc-step .t span{font-size:.84rem;color:var(--text-2);}
.mc-step .t code{font-family:var(--font-mono);font-size:.76rem;color:var(--accent);}

/* three primitives cards */
.mc-prim{display:grid;grid-template-columns:repeat(3,1fr);gap:.8rem;max-width:680px;margin:0 auto;}
@media(max-width:600px){.mc-prim{grid-template-columns:1fr;}}
.mc-prim .p{border:1px solid var(--border-2);border-radius:12px;padding:1rem;background:var(--surface);}
.mc-prim .p .ic{font-family:var(--font-mono);font-size:.72rem;color:var(--accent);border:1px solid var(--accent);border-radius:6px;padding:.15rem .45rem;display:inline-block;margin-bottom:.6rem;}
.mc-prim .p b{display:block;color:var(--text);font-size:.95rem;margin-bottom:.3rem;}
.mc-prim .p span{font-size:.83rem;color:var(--text-2);line-height:1.5;}
.mc-prim .p em{display:block;font-style:normal;font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);margin-top:.5rem;}

/* who controls what bar */
.mc-ctrl{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:.6rem;}
.mc-ctrl .row{display:flex;align-items:center;gap:.8rem;}
.mc-ctrl .who{flex:none;width:130px;font-family:var(--font-mono);font-size:.78rem;color:var(--text-2);text-align:right;}
.mc-ctrl .track{flex:1;height:30px;border-radius:8px;border:1px solid var(--border);background:var(--surface-2);overflow:hidden;position:relative;}
.mc-ctrl .seg{height:100%;display:flex;align-items:center;justify-content:center;font-family:var(--font-mono);font-size:.72rem;color:var(--accent-ink);background:var(--grad);width:0;transition:width 1s cubic-bezier(.2,.7,.2,1);white-space:nowrap;}
.mc-ctrl.go .seg{width:var(--w);}

/* token/effort comparison bars */
.mc-cmp{max-width:600px;margin:0 auto;display:flex;flex-direction:column;gap:1.1rem;}
.mc-cmp .row .lab{display:flex;justify-content:space-between;font-family:var(--font-mono);font-size:.78rem;margin-bottom:.35rem;}
.mc-cmp .row .lab b{color:var(--text);}
.mc-cmp .row .lab span{color:var(--text-2);}
.mc-cmp .track{height:16px;border-radius:999px;background:var(--surface-2);border:1px solid var(--border);overflow:hidden;}
.mc-cmp .fill{height:100%;border-radius:999px;width:0;transition:width 1.1s ease;}
.mc-cmp.go .fill{width:var(--w);}
.mc-cmp .fill.rest{background:var(--surface-2);border-right:2px solid var(--text-3);}
.mc-cmp .fill.mcp{background:var(--grad);}

/* code card */
.mc-code{max-width:560px;margin:0 auto;background:var(--surface);border:1px solid var(--border-2);border-radius:14px;overflow:hidden;box-shadow:var(--glow);}
.mc-code-bar{display:flex;align-items:center;gap:.5rem;padding:.6rem .9rem;background:var(--surface-2);border-bottom:1px solid var(--border);}
.mc-code-bar .d{width:10px;height:10px;border-radius:50%;}
.mc-code-bar .nm{font-family:var(--font-mono);font-size:.76rem;color:var(--text-2);margin-left:.3rem;}
.mc-code-body{font-family:var(--font-mono);font-size:.8rem;line-height:1.8;padding:.95rem 1.1rem;color:var(--text-2);white-space:pre;overflow-x:auto;}
.mc-code-body .k{color:var(--accent);}
.mc-code-body .c{color:var(--text-3);}
.mc-code-body .s{color:var(--text);}

/* comparison + scenario table */
.mc-tab{width:100%;border-collapse:collapse;font-size:.9rem;}
.mc-tab th,.mc-tab td{text-align:left;padding:.6rem .7rem;border-bottom:1px solid var(--border);vertical-align:top;}
.mc-tab th{font-family:var(--font-mono);font-size:.72rem;text-transform:uppercase;letter-spacing:.06em;color:var(--text-3);}
.mc-tab td:first-child{color:var(--text);font-weight:500;}
.mc-tab code{font-size:.82rem;}

@media (prefers-reduced-motion: reduce){
  .mc-step,.mc-ctrl .seg,.mc-cmp .fill{transition:none !important;}
  .mc-flow .mc-step{opacity:1 !important;transform:none !important;}
  .mc-ctrl .seg,.mc-cmp .fill{width:var(--w) !important;}
}
</style>

For a while, we built the smartest things humanity had ever made and then left them sitting in a sealed room.

That is not a metaphor I am reaching for. It is close to literal. A frontier model in 2024 could reason about your codebase, draft your email, explain a legal clause, and it could do exactly none of it to your *actual* codebase, your *actual* inbox, your *actual* contract, unless someone hand-wired a custom bridge for that one task. The model could think. It could not reach. Every time you wanted it to touch one more real thing in the world, an engineer had to go build a one-off connector, by hand, again.

I felt this personally. I have built tools that wrap language models, and the part that hurt was never the intelligence. It was the plumbing. The model was ready. The world was right there. And between them sat a pile of bespoke glue code that broke every time an API shifted under it.

MCP is the thing that fixed the reaching. And once you understand it, a lot of the AI world stops looking like magic and starts looking like something you could have designed yourself. So let me take you through it slowly, the way I wish someone had taken me.

## The one-sentence version

MCP, the Model Context Protocol, is an open standard for connecting AI applications to the outside world: your files, your databases, your tools, your APIs.

The analogy the people who made it use is the right one, so I will keep it: **MCP is a USB-C port for AI.** Before USB-C, every device had its own charger, its own cable, its own little incompatible plug, and a drawer in your house was full of them. USB-C said: one shape, everything fits. MCP is that, for the connection between a model and the things it needs to touch.

<figure class="mc-fig">
  <div class="mc-port">
    <div class="box"><b>The Model</b><span>can think</span></div>
    <div class="plug">MCP<div class="line"></div>the plug</div>
    <div class="box"><b>The World</b><span>files, APIs, tools</span></div>
  </div>
  <figcaption>The whole point in one picture. The intelligence was never the bottleneck. The standard, reliable connection between intelligence and reality was. MCP is that connection.</figcaption>
</figure>

## "But we already had APIs. Why not just use REST?"

This is the question everyone asks, and it is a good one. I asked it too. The honest answer is that comparing MCP to REST is almost a category error, they live at different layers and were built for different *kinds of caller*. Let me make that concrete instead of hand-wavy.

A REST API was designed for a **human developer** writing **deterministic code**. You, the developer, read the docs once, at design time. You learn that `GET /users/42/orders` exists. You hard-code that call. The program runs the exact path you wrote, every time, forever. REST assumes the caller already knows the map before the journey starts.

An AI agent is a different kind of caller entirely. It is **probabilistic**. It does not have your code memorized; it figures out what to do at runtime, from intent. So it needs things REST never promised to give it:

<figure class="mc-fig">
<table class="mc-tab">
<thead><tr><th></th><th>REST API</th><th>MCP</th></tr></thead>
<tbody>
<tr><td>Built for</td><td>Human developers, fixed code</td><td>AI agents, runtime reasoning</td></tr>
<tr><td>Discovery</td><td>You read the docs at design time</td><td>The agent asks <code>tools/list</code> at runtime and gets a live menu</td></tr>
<tr><td>State</td><td>Stateless. Each call forgets the last.</td><td>Stateful session. Context carries across steps.</td></tr>
<tr><td>Schema</td><td>Described in docs for a human to read</td><td>Self-describing, machine-readable, handed to the model directly</td></tr>
<tr><td>Adding a feature</td><td>Ship new docs, hope clients update their code</td><td>Server announces it; agent discovers it instantly</td></tr>
</tbody>
</table>
  <figcaption>Not better or worse. Different layers. REST serves code that already knows what it wants. MCP serves a model that has to find out what it can do, while it is doing it.</figcaption>
</figure>

The cleanest way I can put it: **REST tells you what to do if you already read the manual. MCP hands the model the manual and lets it read on the spot.** That single shift, discovery at runtime instead of design time, is most of why MCP exists.

And here is the part people miss: MCP usually does not replace your REST API. Most MCP servers are thin translators sitting *on top of* an existing REST backend, adding the AI-friendly layer. Your API keeps doing its job. MCP just makes it legible to a model.

## The problem it really solves: the tangle

There is a deeper reason a standard had to exist, and it is the most convincing argument of all. Picture it honestly.

You have M different AI applications (Claude, Cursor, your own agent). You have N different systems you want them to reach (GitHub, Postgres, Slack, your internal API). Without a standard, connecting them means building a custom bridge for *every pair*. That is M times N pieces of fragile glue, each one a thing that can rot. People call it the N×M problem, and it is exactly the drawer full of incompatible chargers.

<figure class="mc-fig">
  <div class="mc-nm">
    <div class="panel bad">
      <h4>Without a standard: N×M glue</h4>
      <div class="grid">
        <div class="l"><span class="tag">Claude</span><span class="x">→ custom →</span><span class="tag">GitHub</span></div>
        <div class="l"><span class="tag">Claude</span><span class="x">→ custom →</span><span class="tag">Slack</span></div>
        <div class="l"><span class="tag">Cursor</span><span class="x">→ custom →</span><span class="tag">GitHub</span></div>
        <div class="l"><span class="tag">Cursor</span><span class="x">→ custom →</span><span class="tag">Postgres</span></div>
        <div class="l"><span class="tag">Agent</span><span class="x">→ custom →</span><span class="tag">Slack</span></div>
        <div class="l"><span class="x">…and on, and on, each one breakable</span></div>
      </div>
    </div>
    <div class="panel good">
      <h4>With MCP: one plug each</h4>
      <div class="grid">
        <div class="l"><span class="tag">Claude</span><span class="x">→</span><span class="tag hub">MCP</span></div>
        <div class="l"><span class="tag">Cursor</span><span class="x">→</span><span class="tag hub">MCP</span></div>
        <div class="l"><span class="tag">Agent</span><span class="x">→</span><span class="tag hub">MCP</span></div>
        <div class="l"><span class="tag hub">MCP</span><span class="x">→</span><span class="tag">GitHub</span></div>
        <div class="l"><span class="tag hub">MCP</span><span class="x">→</span><span class="tag">Slack, Postgres…</span></div>
        <div class="l"><span class="x">build a server once, every client uses it</span></div>
      </div>
    </div>
  </div>
  <figcaption>Build an MCP server for GitHub once, and every MCP-speaking app can use it. Write a client once, and it can talk to every MCP server in existence. The tangle collapses into a hub.</figcaption>
</figure>

## The three players: host, client, server

Before the moving parts, the cast. This trips people up because the words sound interchangeable, so let me pin them down exactly.

- **The host** is the AI application you actually use. Claude Desktop, Cursor, VS Code, your own agent. It is the thing in charge.
- **The server** is a program that exposes some slice of the world: a filesystem, a database, the Sentry API. It can run locally on your machine or remotely on someone's platform.
- **The client** is the quiet middleman. The host spins up *one client per server*, and that client holds the dedicated connection to it. Two servers connected means two clients inside the host.

So when VS Code connects to a GitHub server and a Postgres server, it is running two clients, one married to each server. That one-client-per-server detail is the thing to hold onto.

## How a connection actually works, start to finish

Here is where it stops being abstract. Under the hood, MCP is just structured messages going back and forth in a format called JSON-RPC 2.0. Plain text, request and response. Watch one full handshake. This is the real sequence, simplified to its bones:

<figure class="mc-fig">
  <div class="mc-flow" id="flow">
    <div class="mc-step f1">
      <div class="n">1</div>
      <div class="t"><b>Initialize, the handshake</b><span>Client and server greet each other and negotiate what each can do. <code>initialize</code> → "I support tools and resources." This is capability negotiation; nobody assumes, everybody declares.</span></div>
    </div>
    <div class="mc-step f2">
      <div class="n">2</div>
      <div class="t"><b>Discover, "what can you do?"</b><span>The client asks <code>tools/list</code>. The server replies with a live menu: every tool, its description, and the exact shape of input it expects. The model now knows its options.</span></div>
    </div>
    <div class="mc-step f3">
      <div class="n">3</div>
      <div class="t"><b>Call, "do this one"</b><span>The model decides. The client sends <code>tools/call</code> with the tool name and arguments. The server runs the real work and returns the result as content the model can read.</span></div>
    </div>
    <div class="mc-step f4">
      <div class="n">4</div>
      <div class="t"><b>Notify, "things changed"</b><span>If the server's tools change mid-session, it can push <code>notifications/tools/list_changed</code>. No request needed. The client refreshes its menu. The connection stays alive and current.</span></div>
    </div>
  </div>
  <figcaption>Greet, discover, call, stay in sync. The magic step is the second one: the agent learns what it can do by asking, at runtime, not by you hard-coding it months earlier. That is the whole difference from REST, captured in one message.</figcaption>
</figure>

## The three things a server can offer

When you build an MCP server, you are not just exposing functions. The protocol gives you three distinct *kinds* of thing to offer, and choosing the right one is the actual craft. They are the three primitives.

<figure class="mc-fig">
  <div class="mc-prim">
    <div class="p">
      <span class="ic">tools</span>
      <b>Tools</b>
      <span>Functions the model can call to <em style="display:inline;font-style:italic;color:inherit">do</em> something with a side effect. Query a database, send a message, create a file.</span>
      <em>"Take an action."</em>
    </div>
    <div class="p">
      <span class="ic">resources</span>
      <b>Resources</b>
      <span>Read-only data the model can pull in for context. A file's contents, a row, an API response, a schema. No side effects, just knowledge.</span>
      <em>"Here, read this."</em>
    </div>
    <div class="p">
      <span class="ic">prompts</span>
      <b>Prompts</b>
      <span>Reusable templates the server hands over: a structured code-review flow, a tuned query pattern. Pre-built ways to use the rest well.</span>
      <em>"Try it like this."</em>
    </div>
  </div>
  <figcaption>Tools act. Resources inform. Prompts guide. A database server, for instance, might expose a tool to run queries, a resource holding the schema, and a prompt with good example queries baked in.</figcaption>
</figure>

There is a quieter, more elegant half to this that almost nobody mentions: the *client* has primitives too. The server can ask the client to do things back. It can request **sampling** (ask the host's model to think about something, without the server needing its own model), **elicitation** (ask the human a clarifying question or for confirmation), and **logging**. So it is genuinely two-way. The server is not just a vending machine; it can tap the model and the user on the shoulder when it needs them. That bidirectionality is one of the prettiest design choices in the whole thing.

And notice who is in control of each piece, because it is deliberate:

<figure class="mc-fig">
  <div class="mc-ctrl" id="ctrl">
    <div class="row"><div class="who">Tools</div><div class="track"><div class="seg" style="--w:100%">the model decides when to call</div></div></div>
    <div class="row"><div class="who">Resources</div><div class="track"><div class="seg" style="--w:75%">the app picks what to load</div></div></div>
    <div class="row"><div class="who">Prompts</div><div class="track"><div class="seg" style="--w:55%">the user usually invokes</div></div></div>
    <div class="row"><div class="who">Elicitation</div><div class="track"><div class="seg" style="--w:40%">the human answers</div></div></div>
  </div>
  <figcaption>Control is spread on purpose. The model drives tool calls, the application governs context, the human stays in the loop for prompts and confirmations. No single party runs away with it.</figcaption>
</figure>

## How you actually build one

Here is the part that surprised me most: building a server is small. The SDKs (Python, TypeScript, Java, Kotlin, C#, Go, Ruby, and more) do the protocol grunt work, the JSON-RPC, the lifecycle, the message framing, so you write almost nothing but your actual logic.

In Python, a tool is, honestly, just a decorated function. You write a normal function with type hints and a docstring, and the SDK turns it into a fully described MCP tool, because the description and input schema the model needs are generated *from your hints and docstring*. You write a function; you get a tool.

<figure class="mc-fig">
  <div class="mc-code">
    <div class="mc-code-bar"><span class="d" style="background:#F2555A"></span><span class="d" style="background:#F5BD4F"></span><span class="d" style="background:#5FD068"></span><span class="nm">weather_server.py</span></div>
    <div class="mc-code-body"><span class="k">from</span> mcp.server.fastmcp <span class="k">import</span> FastMCP

mcp = FastMCP(<span class="s">"weather"</span>)

<span class="k">@mcp.tool()</span>
<span class="k">async def</span> <span class="s">get_forecast</span>(latitude: float, longitude: float) -> str:
    <span class="c">"""Get the weather forecast for a location."""</span>
    <span class="c"># your real logic: call an API, return text</span>
    <span class="k">return</span> fetch_forecast(latitude, longitude)

<span class="k">if</span> __name__ == <span class="s">"__main__"</span>:
    mcp.run(transport=<span class="s">"stdio"</span>)</div>
  </div>
  <figcaption>A working MCP server, near enough. The decorator registers the tool. The type hints become its input schema. The docstring becomes the description the model reads. The last line starts it talking over stdio. That is the shape, in any SDK, the names just change.</figcaption>
</figure>

That last line names the **transport**, the channel the messages travel over, and there are two that matter:

- **stdio**: the server runs as a local process on your own machine and talks through standard input and output. No network, no latency, perfect for "a server that reads my local files." This is the default for local servers in Claude Desktop and Claude Code.
- **Streamable HTTP**: the server runs remotely as a real web service, reachable over HTTP, optionally streaming results back. This is how a company exposes an official MCP server to the world, with proper authentication (OAuth) on top.

Same exact JSON-RPC messages either way. The transport is just the pipe; the conversation inside it is identical. That clean separation is why a server you wrote for local stdio can later be served remotely with barely a change.

## Where this actually shows up: real scenarios

Concepts settle once you see them carrying weight. Here is MCP doing real work:

<figure class="mc-fig">
<table class="mc-tab">
<thead><tr><th>Scenario</th><th>What the server exposes</th><th>What you get</th></tr></thead>
<tbody>
<tr><td>Coding assistant on your repo</td><td>Tools to read/edit files, run tests; a resource of the project structure</td><td>The agent works in your real codebase, not a copy you pasted in</td></tr>
<tr><td>Chat over your database</td><td>A query tool, the schema as a resource, example queries as a prompt</td><td>"Show me last quarter's churn" becomes a real, safe query</td></tr>
<tr><td>Design to code</td><td>A tool that reads a Figma file's structure</td><td>The model generates a web app from the actual design, not a screenshot</td></tr>
<tr><td>Personal assistant</td><td>Calendar and notes servers (Google Calendar, Notion)</td><td>"What's my week look like, and draft replies" against your real life</td></tr>
<tr><td>Ops and debugging</td><td>An error-tracking server (like Sentry), remote over HTTP</td><td>The agent pulls live incidents and reasons about them in context</td></tr>
</tbody>
</table>
  <figcaption>Every row is the same story: the model stops guessing from stale pasted text and starts working against the live thing. That is the difference between a clever chatbot and an agent that gets work done.</figcaption>
</figure>

There is a cost story here worth seeing too. With REST, an agent that needs a user's order status might call three endpoints (`get_user`, `get_orders`, `get_shipments`) and stitch them together, three round-trips, each one burning tokens and time. A well-designed MCP tool, `track_order(email)`, returns the whole answer in one call. For a human writing code, three calls is nothing. For an agent reasoning step by step, every extra call is real money and real latency:

<figure class="mc-fig">
  <div class="mc-cmp" id="cmp">
    <div class="row">
      <div class="lab"><b>Three granular REST calls, stitched by the agent</b><span>more steps, more tokens</span></div>
      <div class="track"><div class="fill rest" style="--w:90%"></div></div>
    </div>
    <div class="row">
      <div class="lab"><b>One outcome-shaped MCP tool</b><span>one round-trip</span></div>
      <div class="track"><div class="fill mcp" style="--w:30%"></div></div>
    </div>
  </div>
  <figcaption>A lesson the protocol quietly teaches: design tools around outcomes the agent wants, not around your database tables. Agentic iteration is expensive in a way that ordinary code is not, so fewer, smarter tools beat many tiny ones.</figcaption>
</figure>

## The part you must not skip: trust

I will be plain about this, because it matters. An MCP server is something you connect to your AI, and through that AI, to your data and your machine. A malicious or careless server can be handed real reach. Connecting a server you have not vetted is like running a program you downloaded from a stranger, because that is essentially what it is.

So: prefer official servers and ones you wrote yourself. Be especially wary of any server that pulls in content from the open internet, because that content can carry instructions of its own, the prompt-injection problem riding in through the side door. The convenience of MCP is exactly that it lets a model *act*. Anything that can act on your behalf is something you have to be able to trust. Treat installing a server with the same seriousness you treat installing software, because it is.

## Why this one is worth understanding deeply

Most things in the AI stack are getting more complicated. MCP is one of the rare ones that made things *simpler*, and it did it the way good standards always do: by picking a small, clear contract and getting everyone to agree on it. A host, a client, a server. Three primitives a server can offer. Messages in plain JSON-RPC. Discover at runtime, call when needed, stay in sync. That is the whole spine of it.

Once you hold that spine, the future stops looking like sorcery. The next time an AI app reaches into your calendar, edits your repo, or queries your database and hands you back the answer, you will know there is no magic in it. There is a model that learned to ask "what can I do here?", a server that answered honestly, and a small, well-designed plug between them, finally letting the thing that could always think, reach out and touch the world.

<script>
(function(){
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      if(e.isIntersecting){ e.target.classList.add('go'); io.unobserve(e.target); }
    });
  }, {threshold:.25});
  ['flow','ctrl','cmp'].forEach(function(id){
    var el = document.getElementById(id);
    if(el) io.observe(el);
  });
})();
</script>
