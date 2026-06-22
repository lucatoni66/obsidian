---
title: "My Hermes Journey"
source: "https://www.reddit.com/r/hermesagent/comments/1smgo1i/my_hermes_journey/?solution=542bcfaf344e00d4542bcfaf344e00d4&js_challenge=1&token=7afd7253fec22262ff1c52b1703fe9ece4d02edfa0700934225376a03240b0b9&jsc_orig_r="
author:
  - "[[hackrepair]]"
published: 2026-04-15
created: 2026-06-18
description: "Just a few notes for Hermes Agent users.Put in about 40 hours of setup and learned a few things along the way. These are my experiences, so"
tags:
  - "clippings hermes"
---
Just a few notes for Hermes Agent users.  
Put in about 40 hours of setup and learned a few things along the way. These are my experiences, so please be kind. Not perfect, just what I learned in my setup.

\- - Do not use Alibaba (qwen). A horror story for another time.  
\- Do not use local ChatGPT 5.4 Codex in Hermes Agent v0.9. Still too buggy.

\_\_\_  
I could spend an hour on this discussion, but it's more maddening than I'm willing to get into in this post. Just don't do it.  
\_\_\_

2.  
\- Set this as your Smart routing default (using OpenRouter):

Tier 1: Hermes (Gemini 3.1 Flash Lite)

Primary orchestrator and implementation workhorse. Tasks: Building from clear specs, scaffolding, migrations, wiring components, repetitive edits, standard feature implementation.  
Routing logic: Default for clear, mechanical, multi-file, or straightforward work.

Tier 2: Sonnet (claude-sonnet-4-7-2025)

High-judgment escalations and deep analysis.  
Tasks: Architectural refactors, security-sensitive changes, legacy cleanup, debugging obscure failures, performance tuning.  
Routing logic: Default for ambiguous, delicate, or high-risk tasks. I automatically escalate here if a Hermes task stalls or becomes brittle.

Tier 3: Minimax (minimax/minimax-m2.7)

Smart-routed low-overhead tasks>  
Tasks: Simple, cheap, one-off logic or data processing.  
Routing logic: Selected when task complexity is minimal to stay performant and cost-efficient.

\_\_\_  
Seriously, do this from day one, and you'll save about 10 hours of trial and error.  
\_\_\_

3.  
Run the minimax-cache-optimization skill.

4.  
Stop!  
For your first week, with the above setup, start using it for real-world tasks to see if it's what you need for your personal or business development.

\_\_\_  
I literally just saved you at least 10 hours of frustration, trial and error, and $40 in wasted tokens/credit. You're welcome.  
\_\_\_

5\. Use an IDE like Cursor, Codex, or Anitgravity to "fix" stubborn issues with Hermes settings. Attempting to fix problems with the Hermes Agent within Hermes Agent is a fool's errand. ;-)

Have fun with it!

---

## Comments

> **Birdinhandandbush** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogeazwr/) · 25 points
> 
> Currently running 100% locally with Qwen 3.5:4B and it's been excellent. I had been using the 9b version but 4b was recommended for a test and I haven't turned back. It's lightning fast and does everything I want. I'm not a coder, I don't need it to be super intelligent. It's a fantastic personal assistant for me
> 
> > **Several-Tax31** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogf6wrb/) · 10 points
> > 
> > What kind of things you do with it? Can you share more? I always admire people who uses smallest models for useful tasks. 

> **6donka9** · [2026-04-16](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogh1vxy/) · 8 points
> 
> Hey thanks for sharing! Just got a question regarding the minimax-cache-optimization skill, where can I find it? I tried to tell my Hermes agent to run it but it can't find it, where can I get the specific one you were talking about from?
> 
> > **hackrepair** · [2026-04-16](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogjotx4/) · 2 points
> > 
> > You know, now I suspect that I created that when I was doing my research. That is the funny part of using hermes agent, sometimes it creates some really nice skills and you don't even know it...

> **VonDenBerg** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/oge1ili/) · 7 points
> 
> on the contrary, I’ve been using qwen 3.6 with great success. Minimax is sonnets paint huffing cousin. GLM is opus lite, slightly drunk.
> 
> > **hackrepair** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/oge2g82/) · 4 points
> > 
> > No argument there. My implementation is more focused on getting everything up and running now, as quick as possible, at the lowest reasonable cost.

> **wolfgangamadeusme** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogeo8ry/) · 6 points
> 
> I’ve been running GPT 5.4 via codex on the subscription - the 20$ plus usage gets burned pretty quickly but I’ve gone up to the 5x for a month to try it.
> 
> FWIW I ran it on minimax m2.7 coding plan for the first few days and it was pretty good. Open AI feels better. It’s a little verbose but we’re working on that.
> 
> Whether the 5x on GPT is worth 8x minimax - we’ll see.
> 
> Overall it’s been pretty great and using obsidian + honcho and karpathy’s LLM wiki as a structural approach seems to be working well.
> 
> > **hackrepair** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogeuqvh/) · 3 points
> > 
> > I couldn't get GPT 5.4 codex to work reliably over time. After a while I started getting 429 errors

> **veganmaister** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogeehfa/) · 5 points
> 
> Are you using open router/auto?
> 
> How or where (hermes or oprnrouter) have you configured the smart routing?
> 
> > **hackrepair** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogek3ud/) · 2 points
> > 
> > You can copy my text to set up smart routing.

> **Ion-manden** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogef2fx/) · 5 points
> 
> I am running 100% minimax m2.7 and it is working really well, I am amazed at how great it is at solving my tasks.
> 
> No direct development tasks, but have seen that it builds python script to solve some problems.
> 
> Only used it for a couple days but have had it do some very heavy research tasks with good results and today I paired with it to automate filling out a form for me so it is now able to do it for me.
> 
> > **hackrepair** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogetc32/) · 2 points
> > 
> > Automatic filling out forms. Clarify?

> **Excellent-Baker-1177** · [2026-04-16](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/oggwuia/) · 3 points
> 
> Ive tested just about every model available from 4b-30b in many quants/variants on llamacpp/vllm with my rtx3090 and hermes. Ive even tried hermes trained models.
> 
> BY FAR THE BEST is GLM4.7-Flash UD-K\_Q4\_XL via Llamacpp. Its been the most stable, doesnt fall apart in loops like Qwen, super fast!
> 
> Its such a sleeper. I have no clue why tbh but its better than gemma4, qwen3.5, nemotron, cascade2. But hey if it works it works.
> 
> > **hackrepair** · [2026-04-16](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogjmh30/) · 2 points
> > 
> > ok, this is interesting. Can I ask, what gpu you have running that allows this?

> **IcyOrdinary8042** · [2026-04-16](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogi474y/) · 3 points
> 
> Im using Gemma 4 modles. 31b Dense and the 26B model..if the numbers are wrong dont kill me but uk what im talking about. So far its great 👍 its smart and does it job.
> 
> My stack is paperclip ai+hermes agent+superpower skills+ gemma 4 models via google ai studio API key + composio MCP + tavily via composio and zep for memory 😀.
> 
> So far i didnt build any agents yet. Just researching and planning right now
> 
> > **hackrepair** · [2026-04-16](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogjpxdq/) · 2 points
> > 
> > superpower skills  
> > clarify a bit on this?

> **hackrepair** · [2026-04-16](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogjlvhc/) · 2 points
> 
> Hi all. I'm the original poster.  
> Many of you have asked how the smart routing is set up.
> 
> That's the cool thing about Hermes Agent. Once you find something that works, Hermes Agent will create/update a skill for you.
> 
> In your Hermes Agent setup, enter the question:  
> "What is my coding router setup?"
> 
> \_\_\_  
> An alternate I use, for example, which you could simply give to your coding router to update/tweak:
> 
> \- \*\*Smart Model Routing:\*\* Active (\`smart\_model\_routing: true\`), thresholds at 250 characters and 50 words for Tier 3.
> 
> \- \*\*Delegation:\*\* Pinned to \*\*Claude Sonnet 4.6\*\* for high-judgment work.
> 
> \- \*\*Fallback:\*\* \`openrouter\` used for \`minimax/minimax-m2.7\` if local Ollama fails.
> 
> \*\*The Tiers:\*\*
> 
> \* \*\*Tier 1 (Execution Layer):\*\* Uses \`minimax-m2.7:cloud\` (via local Ollama).
> 
> \- Scope: Implementation-heavy tasks (clear specs, scaffolding, migrations, wiring, repetitive edits, tests).
> 
> \* \*\*Tier 2 (Expert Layer):\*\* Uses \`claude-sonnet-4-6-2025\` (via Claude Code).
> 
> \- Scope: Architectural refactors, security audits, legacy cleanup, complex/ambiguous debugging.
> 
> \- Auto-revert: If stalled for >15 minutes, orchestration reverts to Tier 1 (unless the model is manually pinned).
> 
> \* \*\*Tier 3 (Cheap/Simple Layer):\*\* Uses \`gemini-3-flash-preview:cloud\` (via Ollama).
> 
> \- Scope: Low-overhead/one-off tasks.
> 
> \- Fallback: \`gemini-3.1-flash-lite-preview\` (via OpenRouter).
> 
> \*\*Rules:\*\*
> 
> \- Hermes retains primary orchestration of context and verification.
> 
> \- Manual user overrides always take absolute precedence.
> 
> \- If a task escalates to Tier 2 and stalls, the auto-revert rule applies.

> **infinitejennifer** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogeckzs/) · 9 points
> 
> You’re an excellent human for sharing with the group.

> **NukerX** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogeh4qi/) · 6 points
> 
> I've been using minimax2.7 through ollama
> 
> Cheapest way I found so far.

> **MercurialMadnessMan** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/oge7uc3/) · 1 points
> 
> Is smart routing a built-in feature or does it have to be added?
> 
> Where do I find the cache optimization skill?
> 
> > **hackrepair** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/oge8zdr/) · 1 points
> > 
> > See the skills list when it first loads and search for the word cache, it should appear there. No?

> **datewestwind** · [2026-04-15](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogetupi/) · 1 points
> 
> Hi thanks for sharing. What is the approximate cost with that smart routing?

> **Trade\_Manager** · [2026-04-16](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogfhq9q/) · 1 points
> 
> So I’m making a big mistake using kimi2.5? Like although it costs me nothing in $$ our time is most valuable. Should I just spend a little more and use a codex or Claude?
> 
> > **hackrepair** · [2026-04-16](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogfov18/) · 2 points
> > 
> > it's a decent Model, though it's rather outdated this week... ;\_)

> **nokafein** · [2026-04-16](https://reddit.com/r/hermesagent/comments/1smgo1i/comment/ogiaceh/) · 1 points
> 
> What does those tiers mean? I don't understand that part.