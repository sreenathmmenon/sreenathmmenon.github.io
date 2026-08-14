---
title: "I Was at Sarvam Epoch 2026"
date: 2026-08-14
excerpt: "A trillion-parameter model built from scratch in India. Inference that never leaves the country. A coding agent, a voice model that performs instead of reads, smart glasses that help a blind woman find her bus. I went to Sarvam's first big conference in Bengaluru expecting a product launch and left with something bigger: the feeling that India has stopped being the world's back office and started being one of its builders. Here's what I saw, what I honestly think of the bet, and why I couldn't stop being excited."
tags: [ai, sarvam, india, sovereign-ai, llm, conference]
---

<style>
.ep-fig{margin:2.4rem 0;}
.ep-fig img{width:100%;border-radius:14px;border:1px solid var(--border);display:block;}
.ep-fig figcaption{font-family:var(--font-mono);font-size:.8rem;color:var(--text-3);margin-top:.7rem;text-align:center;line-height:1.5;}
/* placeholder shown until a real photo is dropped in */
.ep-ph{border:1px dashed var(--border-2);border-radius:14px;background:var(--surface);min-height:190px;display:flex;align-items:center;justify-content:center;text-align:center;padding:1.5rem;color:var(--text-3);font-family:var(--font-mono);font-size:.82rem;line-height:1.6;}
/* merch gallery */
.ep-gallery{display:grid;grid-template-columns:repeat(3,1fr);gap:.7rem;}
@media(max-width:560px){.ep-gallery{grid-template-columns:1fr;}}
.ep-gallery img{width:100%;height:100%;object-fit:cover;aspect-ratio:3/4;border-radius:12px;border:1px solid var(--border);display:block;margin:0;}

/* pull-quote for the token-sovereignty idea */
.ep-quote{border-left:3px solid var(--accent);padding:.6rem 0 .6rem 1.2rem;margin:2rem 0;font-size:1.15rem;color:var(--text);line-height:1.5;font-weight:500;}
.ep-quote span{display:block;font-size:.8rem;color:var(--text-3);font-family:var(--font-mono);font-weight:400;margin-top:.5rem;}

/* product grid */
.ep-prods{display:grid;grid-template-columns:repeat(2,1fr);gap:.8rem;margin:1.6rem 0;}
@media(max-width:600px){.ep-prods{grid-template-columns:1fr;}}
.ep-prod{border:1px solid var(--border);border-radius:12px;background:var(--surface);padding:1rem 1.1rem;opacity:0;transform:translateY(10px);transition:opacity .5s var(--ease),transform .5s var(--ease);}
.ep-prods.go .ep-prod{opacity:1;transform:none;}
.ep-prods.go .ep-prod:nth-child(1){transition-delay:.05s} .ep-prods.go .ep-prod:nth-child(2){transition-delay:.13s} .ep-prods.go .ep-prod:nth-child(3){transition-delay:.21s} .ep-prods.go .ep-prod:nth-child(4){transition-delay:.29s} .ep-prods.go .ep-prod:nth-child(5){transition-delay:.37s} .ep-prods.go .ep-prod:nth-child(6){transition-delay:.45s}
.ep-prod .pn{font-family:var(--font-mono);font-size:.82rem;color:var(--accent);font-weight:600;margin-bottom:.25rem;}
.ep-prod .pd{font-size:.85rem;color:var(--text-2);line-height:1.5;}

/* the full-stack layers */
.ep-stack{max-width:560px;margin:1.6rem auto;display:flex;flex-direction:column;gap:.5rem;}
.ep-layer{display:flex;align-items:center;gap:.85rem;border:1px solid var(--border);border-radius:11px;background:var(--surface);padding:.7rem .95rem;opacity:0;transform:translateX(-10px);transition:opacity .5s var(--ease),transform .5s var(--ease);}
.ep-stack.go .ep-layer{opacity:1;transform:none;}
.ep-stack.go .ep-layer:nth-child(1){transition-delay:.08s} .ep-stack.go .ep-layer:nth-child(2){transition-delay:.2s} .ep-stack.go .ep-layer:nth-child(3){transition-delay:.32s} .ep-stack.go .ep-layer:nth-child(4){transition-delay:.44s}
.ep-layer .ln{flex:none;font-family:var(--font-mono);font-size:.68rem;color:var(--accent);border:1px solid var(--accent);border-radius:5px;padding:.15rem .45rem;}
.ep-layer .lt{font-size:.86rem;color:var(--text-2);} .ep-layer .lt b{color:var(--text);}

/* the debate, two columns */
.ep-debate{display:grid;grid-template-columns:1fr 1fr;gap:1rem;margin:1.6rem 0;}
@media(max-width:600px){.ep-debate{grid-template-columns:1fr;}}
.ep-side{border:1px solid var(--border);border-radius:13px;background:var(--surface);overflow:hidden;opacity:0;transform:translateY(10px);transition:opacity .5s var(--ease),transform .5s var(--ease);}
.ep-debate.go .ep-side{opacity:1;transform:none;} .ep-debate.go .ep-side:nth-child(2){transition-delay:.18s;}
.ep-side .sh{padding:.75rem 1rem;border-bottom:1px solid var(--border);font-family:var(--font-mono);font-size:.8rem;font-weight:600;}
.ep-side.buy .sh{color:var(--accent-2);} .ep-side.build .sh{color:var(--accent);}
.ep-side .sb{padding:.9rem 1rem;font-size:.85rem;color:var(--text-2);line-height:1.55;}
.ep-side .sb .who{font-family:var(--font-mono);font-size:.72rem;color:var(--text-3);margin-bottom:.4rem;}

/* the shift: services -> products */
.ep-shift{max-width:600px;margin:1.6rem auto;display:flex;align-items:center;gap:.8rem;flex-wrap:wrap;justify-content:center;}
.ep-shift .from,.ep-shift .to{flex:1;min-width:180px;border:1px solid var(--border);border-radius:12px;padding:1rem 1.1rem;background:var(--surface);}
.ep-shift .to{border-color:var(--accent);}
.ep-shift .k{font-family:var(--font-mono);font-size:.68rem;letter-spacing:.06em;text-transform:uppercase;margin-bottom:.4rem;}
.ep-shift .from .k{color:var(--text-3);} .ep-shift .to .k{color:var(--accent);}
.ep-shift b{color:var(--text);font-size:1rem;display:block;margin-bottom:.25rem;}
.ep-shift span{font-size:.83rem;color:var(--text-2);line-height:1.45;}
.ep-shift .arrow{flex:none;font-family:var(--font-mono);color:var(--accent);font-size:1.3rem;}
@media(max-width:560px){.ep-shift .arrow{transform:rotate(90deg);}}

@media (prefers-reduced-motion: reduce){
  .ep-prod,.ep-layer,.ep-side{opacity:1!important;transform:none!important;}
}
</style>

The moment I keep coming back to from two days at Sarvam Epoch wasn't the trillion-parameter model, or the pricing charts, or any of the twelve things they launched. It was a short video on the big screen, and then a woman walking up to the stage.

The video showed someone using a pair of AI smart glasses, Sarvam calls them Kaze, to do ordinary things that are not ordinary at all if you can't see: find and board the right bus, order a coffee. Then they invited the person from the video, who is blind, up on stage to talk about what it was actually like. The room went quiet in the way rooms do when something stops being a demo and starts being a person. I'm not going to pretend I remember every technical detail of how the glasses work. What I remember is the feeling, and the feeling was: oh, this is what all of it is for.

I went to Epoch 2026 in Bengaluru expecting an AI product launch. I left thinking about a much bigger question, one that's been argued about in Indian tech for two years now: **can a country build and own its own AI, and is that even the right thing to try?** Sarvam spent two days making the most complete argument I've seen that the answer to both is yes. This is what they showed, and what I honestly think about it.

<figure class="ep-fig">
  <img src="/assets/images/epoch-2026/epoch-entrance.jpg" alt="The Sarvam Epoch 2026 entrance backdrop in Bengaluru" loading="lazy">
  <figcaption>Sarvam Epoch 2026, Bengaluru. Two editions across two days: a builder day and an enterprise day. (My photo.)</figcaption>
</figure>

## First, the thing they most wanted you to hear

Standing on stage, co-founder Pratyush Kumar said the line the headlines ran with: they're building a **trillion-plus parameter foundation model, from scratch, in India,** and they expect it live within about six months.

Two words in that sentence are doing all the work. **"From scratch"** matters because it's a break from how Sarvam started: their early work adapted existing open models to Indian languages. Building a frontier model from zero means owning the entire pipeline, the training, the weights, the intellectual property, rather than renting capability from someone else's lab. **"In India"** matters because it's the whole thesis in two words.

I'll be honest about what this announcement is and isn't, because I think the honesty is the respectful thing. It's a **stated intention**, not a shipped model. There was no technical report, no benchmarks, no model card on stage, this is a roadmap item with an aggressive timeline, not something you can go use today. And notably, the framing wasn't "a model for Indian languages." It was aimed at coding, cybersecurity, simulation, scientific research, frontier-capability territory. Hold that thought, because it's the crack in the argument I want to come back to.

## The word that ran through everything: sovereignty

The other co-founder, Vivek Raghavan, has a phrase for what Sarvam is actually chasing, and once you hear it, you notice it under every product they showed: **token sovereignty.**

<div class="ep-quote">
You can't bet a country's future on importing all of its "tokens" any more than you'd bet it on importing all of its manufacturing.
<span>the token-sovereignty argument, as Sarvam frames it</span>
</div>

The idea is simple and, I think, genuinely serious: a growing share of the AI "tokens" consumed inside India, every model call a bank, a hospital, or a government office makes, currently runs through infrastructure hosted abroad. If that's where your intelligence lives, that's where your data goes and where your dependence sits. Raghavan has put the stakes more bluntly elsewhere: India risks becoming, in his words, a "digital colony."

You can find this framing overheated if you want. But sitting in that room, watching the products stack up, I stopped hearing it as a slogan and started hearing it as an org chart. Because Sarvam isn't building one thing. They're building all four layers at once.

<figure class="ep-fig">
<div class="ep-stack">
  <div class="ep-layer"><span class="ln">layer 1</span><span class="lt"><b>Train</b> the models, from scratch, on Indian compute</span></div>
  <div class="ep-layer"><span class="ln">layer 2</span><span class="lt"><b>Serve</b> them on Indian infrastructure, so the data never leaves</span></div>
  <div class="ep-layer"><span class="ln">layer 3</span><span class="lt"><b>Build</b> end-user products: coding, voice, glasses, agents</span></div>
  <div class="ep-layer"><span class="ln">layer 4</span><span class="lt"><b>Sell</b> to enterprises and government who need all of the above in-country</span></div>
</div>
<figcaption>The full-stack bet. Most national-AI plays pick one layer. Sarvam is attempting all four, which means competing with the frontier labs and the cloud hyperscalers at the same time.</figcaption>
</figure>

That's the bet in one picture. It's also, I'll argue later, the risk.

## What they actually shipped (and it was a lot)

For all the talk about a model that doesn't exist yet, the striking thing about Epoch was how much *did* exist. They launched around a dozen products across two days. The ones that stuck with me:

<figure class="ep-fig">
<div class="ep-prods">
  <div class="ep-prod"><div class="pn">Sarvam Inference</div><div class="pd">India-hosted model serving. Runs Sarvam 105B, GLM 5.2, Gemma 4 on infrastructure inside the country. The literal expression of "token sovereignty": your inference, and the data in it, stays put.</div></div>
  <div class="ep-prod"><div class="pn">Sarvam Code</div><div class="pd">Their entry into the coding-agent race. Long-running tasks with checkpoints you can recover, a plan-then-execute structure. Invite-only at launch. The pitch: pay for completed work, not raw tokens.</div></div>
  <div class="ep-prod"><div class="pn">Bulbul V4</div><div class="pd">The voice model, and the demo was pointed. Instead of showing an accuracy chart, they played a ~113-second reel built around emotion and delivery. The claim isn't "fewer errors," it's "it can actually perform" across Indian languages.</div></div>
  <div class="ep-prod"><div class="pn">Saaras V4 + dubbing</div><div class="pd">Speech recognition across India's scheduled languages, with speaker separation. Paired with the voice work, this is the dubbing and translation stack, the unglamorous plumbing that makes multilingual India actually addressable.</div></div>
  <div class="ep-prod"><div class="pn">Sarvam Kaze</div><div class="pd">The smart glasses from that opening moment. An accessibility focus, built with the National Association for the Blind, multilingual, designed to run at the edge. The product I'd least expected and least forgot.</div></div>
  <div class="ep-prod"><div class="pn">The rest of the stack</div><div class="pd">Document intelligence for Indian scripts and handwriting, on-prem AI for government and defence, a training SDK, a knowledge-base retrieval tool, an agent platform tying it together. A full spread, not a single headline.</div></div>
</div>
<figcaption>A partial map of Epoch's launches. Treat specifics as announced; some shipped, some are early access. The pattern matters more than any one box: it's a company trying to cover every layer at once.</figcaption>
</figure>

The two I'll actually reach for as a builder are **Sarvam Inference** and the **voice stack**. Inference because I already use Sarvam through [llmswap](/blog/2025-08-30-hackathon-to-open-source-llmswap/), and India-hosted serving of open-weight models at their claimed prices (they put Sarvam 105B at a small fraction of the frontier APIs) is a real, concrete thing, not a roadmap. The voice work because "make it sound human in eleven Indian languages" is a problem the global labs simply aren't optimizing for, and it's exactly where an Indian lab should have an edge.

<figure class="ep-fig">
  <img src="/assets/images/epoch-2026/sarvam-105b-benchmark.jpg" alt="Slide from the Epoch 2026 keynote comparing Sarvam 105B against GPT 5.4 Mini and Gemini 3.5 Flash" loading="lazy">
  <figcaption>A slide I photographed from the keynote: Sarvam 105B against GPT 5.4 Mini and Gemini 3.5 Flash on their own benchmark, built, as the slide says, on "hundreds of millions of minutes of Indian conversations." Sarvam leads on voice-call capabilities (82.1) and linguistic quality by a wide margin, with tool-calling roughly on par. These are Sarvam's own numbers, so read them as the maker's framing, but the point is real: on Indian-conversation tasks, a home-grown model going toe to toe with the frontier APIs is not nothing.</figcaption>
</figure>

## The honest part: is this the right bet?

Here's where I have to be fair, because the most interesting thing about Sarvam's plan is that serious people think it's partly wrong. And I don't think you can write an honest post about Epoch without sitting in that disagreement.

<figure class="ep-fig">
<div class="ep-debate">
  <div class="ep-side buy">
    <div class="sh">"Don't build frontier models"</div>
    <div class="sb">
      <div class="who">Nandan Nilekani's camp</div>
      If India has tens of billions to spend, spend it on compute, infrastructure, and AI cloud, the engines, not on training frontier models that commoditize fast. Use open models, build small ones quickly, and become the world's <b>use-case capital</b>. India's edge is applications, languages, and deployment at population scale, not pre-training.
    </div>
  </div>
  <div class="ep-side build">
    <div class="sh">"Build the capability"</div>
    <div class="sb">
      <div class="who">Aravind Srinivas's camp</div>
      A country that can't train its own models stays permanently dependent on Silicon Valley. The skill, the talent and the infrastructure to train, is strategic in itself. And DeepSeek showed frontier-ish work is possible on far less compute than the hyperscalers imply, so frugality can substitute for raw capital.
    </div>
  </div>
</div>
<figcaption>The real, ongoing debate in Indian AI, and it predates Epoch. Sarvam is planting its flag firmly in the "build the capability" camp.</figcaption>
</figure>

Both sides have a point, and here's the tension I couldn't shake walking out. Sarvam's *strongest* argument for sovereignty, data residency for regulated sectors, and doing India's many languages justice, is an argument for the **serving and adapting** layers. A bank in Mumbai genuinely cannot route sensitive data through a US endpoint; that need is real and doesn't go away by fine-tuning someone else's model on foreign servers. Sarvam Inference answers that need directly and today.

But their most attention-grabbing move, a from-scratch trillion-parameter model for coding and science, is precisely the layer the critics say India shouldn't prioritize. That model isn't primarily an Indic-language play; it's a bid for generic frontier capability against labs with far more compute and a head start. The sovereignty case is strongest exactly where the from-scratch-1T case is weakest, and vice versa.

So do I think the trillion-parameter model is the right move? Genuinely unsure, and I'd rather say that than pretend. If it's a forcing function, a way to build the compute, the pipeline, and the talent that the *serving* layer needs anyway, then it's smart even if the model itself underwhelms. If it's a vanity metric, a number to wave, on an unproven six-month timeline, against OpenAI and Google and the hyperscalers all at once, it's a heavy thing to carry. What I'm confident about is the *rest* of the stack. Inference, voice, document intelligence, the glasses, that's real, useful, and defensibly Indian. The 1T model is the bet on top of the business, not the business.

## And here's the part I can't be measured about: I'm just excited

I can do the sober analysis. But I'd be lying if I pretended the main thing I felt at Epoch was analysis. The main thing I felt was **pride**, and I want to say why plainly, because I think this is a moment worth being loud about.

For as long as I've been in this industry, the story about India in tech was "the back office of the world." We ran everyone else's systems. We wrote everyone else's software. We were the place work got *sent to*, not the place ideas got *built from*. That story was never the whole truth, but it was the loud part. And standing in that room in Bengaluru, watching a home-grown model go toe to toe with the frontier APIs on Indian tasks, I realized the loud part has quietly changed.

<figure class="ep-fig">
<div class="ep-shift">
  <div class="from">
    <div class="k">the old story</div>
    <b>The world's back office</b>
    <span>Run other people's systems. Write other people's software. Where work gets sent.</span>
  </div>
  <div class="arrow">&rarr;</div>
  <div class="to">
    <div class="k">what's happening now</div>
    <b>The world's builders</b>
    <span>Train frontier models. Ship products people abroad actually use. Where ideas get built.</span>
  </div>
</div>
<figcaption>The shift I keep feeling. Not a hope for the future, a thing that's already visibly happening.</figcaption>
</figure>

Look at what's coming out of India right now. Sarvam is training foundation models from scratch and open-sourcing them at real scale. Krutrim is building an AI stack and its own compute. There are labs doing serious multilingual and voice work for a country with twenty-two official languages, a problem the global giants simply aren't going to solve for us because it isn't their market. There are teams building document intelligence for Indian scripts, agents for Indian government and healthcare, glasses that speak to a blind woman in her own language and help her find her bus. None of that is a demo of catching up. That's building things nobody else was going to build.

And the *people*. The talent leaving the big frontier labs abroad is starting to come home, or stay home, and build here. The developers in that room weren't waiting for permission or a foreign template. They were shipping. You could feel the difference between "we can do what they do" and "we are doing our own thing," and it was firmly the second one.

Is India going to out-compute Silicon Valley next year? No. Are there real gaps, in chips, in capital, in the sheer scale of frontier training? Of course, and the honest people here say so. But that's not the point of the excitement. The point is that a country of 1.4 billion people, with the languages and the engineers and now the *ambition*, has decided it's going to build its own AI future instead of importing it. I've wanted to feel this about Indian tech my whole career. At Epoch, for the first time, I did, without a single caveat I had to add for myself.

We are not the back office anymore. We're at the table, building. And it is a genuinely thrilling time to be an Indian engineer.

## The part the recaps won't tell you: it was just really well run

I want to say something that has nothing to do with strategy, because it's true and it's underrated. **Epoch was a genuinely well-organized event.** That sounds like a small thing. It isn't. A conference that runs on time, where the demos work, where you can actually find the room and the coffee and the people you came to meet, signals an organization that sweats the details, and detail-sweating is exactly what you want from a company promising to run your country's inference.

And the crowd. This is the part I'd tell a friend first. It was a genuine builder crowd, the people actually shipping Indian AI, in one place, and the energy of that is hard to fake and harder to buy.

The merch deserves its own sentence, because they clearly cared about it. Quality swag, and, my favorite touch, they were printing t-shirts on the spot. It's a tiny thing that tells you something: a company that gets the t-shirt right is a company that respects the people who showed up.

<figure class="ep-fig">
<div class="ep-gallery">
  <img src="/assets/images/epoch-2026/merch-art-tee.jpg" alt="Sarvam Epoch t-shirt: a robotic hand and a human hand reaching toward each other over binary and pixel-tapestry patterns" loading="lazy">
  <img src="/assets/images/epoch-2026/merch-tshirt.jpg" alt="Brown Sarvam Epoch t-shirt with the pixel-art epoch logo and colorful tapestry tiles" loading="lazy">
  <img src="/assets/images/epoch-2026/merch-tote.jpg" alt="Canvas Sarvam Epoch tote bag with a pixel-art floral print, and an epoch booklet" loading="lazy">
</div>
<figcaption>The swag I came home with. My favorite is the one on the left: a robot hand and a human hand reaching for each other over strands of binary and Indian tapestry patterns. Someone thought about that. (My photos.)</figcaption>
</figure>

## What I actually took home

Not the t-shirt. The question.

You can be skeptical about a trillion-parameter model that doesn't exist yet, and you should be, I am. But strip away the biggest headline and what's left is a company that has quietly built a real full stack: models you can serve inside the country, voices that speak India's languages with feeling, tools that read its scripts, and glasses that help a person who can't see find her bus. That last one is the tell. All the sovereignty talk in the world is abstract until it's a specific person doing a specific ordinary thing they couldn't do before.

Whether India *should* try to build its own frontier model from scratch is a real debate with smart people on both sides, and I honestly land somewhere in the uncomfortable middle. But whether India *can* build serious, useful, sovereign AI that matters to actual people, after two days at Epoch, that part I don't doubt anymore.

*I build with Indian models directly, that's [llmswap](/blog/2025-08-30-hackathon-to-open-source-llmswap/), my provider-agnostic SDK that includes Sarvam. If you want the wider frame on why open-weight models like the ones Sarvam serves matter, I wrote about [open weights](/blog/2026-07-19-open-weights-what-it-really-means/) separately.*

<script>
(function(){
  var els=document.querySelectorAll('.ep-prods,.ep-stack,.ep-debate');
  if(!('IntersectionObserver' in window)){els.forEach(function(e){e.classList.add('go')});return;}
  var io=new IntersectionObserver(function(en){en.forEach(function(x){if(x.isIntersecting){x.target.classList.add('go');io.unobserve(x.target)}})},{threshold:.2});
  els.forEach(function(e){io.observe(e)});
})();
</script>
