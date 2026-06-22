---
title: "Memory Providers: I tested them all"
source: "https://www.reddit.com/r/hermesagent/comments/1tms3g6/memory_providers_i_tested_them_all/"
author:
  - "[[Lorian0x7]]"
published: 2026-05-25
created: 2026-06-18
description: "Long story short: All the available memory providers kinda sucks for different reasons except one. Cloud providers sucks because"
tags:
  - "clippings hermes"
---
Long story short: All the available memory providers kinda sucks for different reasons except one.

Cloud providers sucks because they are cloud, Vendor lock-in and data retention is just not for me.

Hindsight is technically the best in terms of memory but it's too heavy to run, too many API calls, costly even within cheap models, hidden configuration settings, too much to deal with and with too many bugs.

OpenViking is a pain to setup, I dropped halfway the process.

Holographic, I liked the speed but quality was not there. I'm still unsure if it was doing something.

Hancho, Another one that was a pain to setup, pretty good at profiling, but suffering from the same issue of Hindsight.

Then I discovered Mnemosyne. It's not built in by default but it should! it's the easiest so setup, lightweight, fully local, and it be best balanced between quality and speed.

I'm essentially making this post because I think Mnemosyne it's not getting the attention it deserves.

It uses a a sqllite database with a fast embedding and a tiny local LLM to consolidate memories and its good enough, I swapped the default model with qwen 0.8b and it's even better, using bigger models is possible if you need maximum quality.

Try it, I'm curious to know what you think.

edit: link: [https://github.com/AxDSan/mnemosyne](https://github.com/AxDSan/mnemosyne)

---

## Comments

> **The1KrisRoB** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onptyso/) · 37 points
> 
> > Hancho, Another one that was a pain to setup...
> 
> I only pick Hancho because that's the only one I have experience with, but I really don't get the concept of any of these being a pain to set up?
> 
> You just tell your agent to set it up and then you go grab a sandwich.
> 
> The hardest part about the setup is deciding what you want on your sandwhich...
> 
> > **Lorian0x7** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onqzrxh/) · 4 points
> > 
> > that's true but it depends on the model you run, for me running a local qwen3.6 27b it was a pain, and my fridge was empty :')
> > 
> > **hometechgeek** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onqnjf5/) · 30 points
> > 
> > I was thinking the same thing. Every time the ai tells me what steps I need to run next, I'm like, no son, that's your job now ;)

> **Jonathan\_Rivera** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onp2oud/) · 36 points
> 
> I know your a real person even though it totally looks AI generated. I'll try it out on my other box.
> 
> Is it really that good?
> 
> *Short answer: yes, seriously. Long answer below.*
> 
> *What it is: SQLite + sqlite-vec for vector search + FTS5 for text search. Fully local, zero external*  
> *dependencies, MIT license. The author is Abdias J (AxDSan), who publishes under his real name and has a*  
> *verifiable background.*
> 
> *What impressed me:*
> 
> *Architecture is genuinely thoughtful:*
> 
> *\- Polyphonic Recall — 4 parallel retrieval strategies (vector, graph, fact, temporal) fused with*  
> *deterministic re-ranking. This isn't just cosine similarity on embeddings. It's drawing from published*  
> *research (Hindsight's multi-strategy blog, Memanto's info-theoretic scoring from arXiv) and combining*  
> *them.*
> 
> *\- Veracity consolidation — Bayesian confidence scoring with tiers (stated > inferred > tool > imported).*  
> *Contradiction detection. Conflict resolution. This is the kind of thinking you want in a memory system —*  
> *not just "store and retrieve" but "store, weigh, conflict-resolve, and consolidate."*
> 
> *\- BEAM benchmark results — 65.2% at 100K scale, up from 35.4% in v2.5. That's a real jump. Hindsight's at*  
> *73.4% (SOTA) but Mnemosyne's using a different judge model, so not directly comparable. Still: competitive*  
> *with the best.*
> 
> *\- Hermes-native integration — has a hermes\_memory\_provider/ directory, a plugin.yaml, and a one-line*  
> *deploy script (curl ... | bash). This isn't a generic memory tool retrofitted; it was built for Hermes.*
> 
> *\- Fact engine (MEMORIA) — structured (subject, predicate, object) triples with deterministic SHA-256 IDs*  
> *(collision-safe). The commit history shows they fixed a silent data-loss bug where truncated fact IDs*  
> *caused PK collisions. They're paying attention.*
> 
> *Caveats:*
> 
> *\- Only 2 open issues and 365 stars — small community. If the author burns out, it's a bus-factor problem.*
> 
> *\- The local LLM consolidation (Qwen 0.8B) is optional. Without it, you get retrieval-only — still useful,*  
> *but the consolidation features are what make it special.*
> 
> *\- The local\_llm.py is 20KB of code — it's a real integration, not a stub. But it means you need*  
> *llama-cpp-python or ctransformers for the full experience.*
> 
> *Bottom line: This is the real deal. Lorian0x7 was right — it deserves more attention. It's not hype. The*  
> *architecture is well-researched, the code is serious (38KB* [*memory.py*](http://memory.py/)*, 37KB veracity consolidation, 37KB*  
> *polyphonic recall), and the BEAM benchmark shows measurable improvement. For a local-first Hermes memory backend, this is currently the best option I've seen.*
> 
> *Worth investigating as a built-in or at minimum a workshop-guide post. The one-liner deploy (curl | bash)*  
> *already fits your "zero terminal commands for the user" format perfectly.*
> 
> > **avadreams** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onpewtm/) · 7 points
> > 
> > Just did a fresh [Hermes](https://www.reddit.com/search/?q=Hermes+AI+agent+framework&cId=b783bde9-cd66-4694-b5c6-622315707168&iId=d114d24c-1099-4a03-9864-5b9b76924beb) for a new project. Is it as simple as providing the GitHub and telling Hermes to implement? I did that for obsidian and missed a heap because I didn't watch tutorials or understand it's true value and usecases first

> **Sjsamdrake** · [2026-05-24](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onp1c6n/) · 4 points
> 
> Got a link? My attempts to Google doe it fine other things that don't seem relevant
> 
> > **Lorian0x7** · [2026-05-24](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onp1jb5/) · 6 points
> > 
> > [https://github.com/AxDSan/mnemosyne](https://github.com/AxDSan/mnemosyne)
> > 
> > **WegoW** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onr0ls6/) · 8 points
> > 
> > I‘ll recommend to install the web dashboard too. Helps you to understand how Mnemo works and what is actually stored in the different memories. 
> > 
> > [https://github.com/wysie/mnemosyne-dashboard](https://github.com/wysie/mnemosyne-dashboard)

> **Sjsamdrake** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onual5n/) · 2 points
> 
> Thanks for this thread. I installed both mnemosyne and the dashboard into my Hermes and will report back in a week with updates. Some installation feedback for both:
> 
> For mnemosyne, the install was simple UNTIL it started asking me questions which the github page didn't mention. I SUPPOSE I should have taken all the defaults, but the 'make a separate db per profile' option seemed like the only logical one to start with. Should I have modified any other defaults? Please add at least some mention of the questions to the Quick Start.
> 
> edit: Oh, and after I installed it I asked Hermes if it was configured and happy, and Hermes pointed out that it'd be happier if I had sqlite-vec installed, which I didn't. The Installation and Prerequisites docs don't mention it, and they should.
> 
> For the dashboard, I guess it's obvious that I need to cd to the plugin directory before starting it, but stating that explicitly would be nice. 😄 Making it a user level systemd service like hermes itself would be nice too so it automatically restarts.
> 
> > **Lorian0x7** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onucb3a/) · 2 points
> > 
> > I have no affiliation or involvement with Mnemosyne, just sharing my honest opinion for a project that i think deserved some attention.
> > 
> > I partially had the same experience during the setup. multiple profile memory: True works fine, it's what I'm using. not sure about the dashboard, I'm not using it.
> > 
> > One thing I can suggest, let your llm configure the LLM for the consolidation, (sleep) it needs some extra dependency and you can setup a custom model

> **cg-mason** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onp95i5/) · 5 points
> 
> Mnemo is great. My agent set it up by itself.

> **Dizzy\_Car\_7198** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onpwmel/) · 4 points
> 
> Byterover; its light unlike Hindsight

> **Phoxerity** · [2026-05-26](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onx28l5/) · 4 points
> 
> What about Gbrain?

> **Icy-Trainer3302** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onppfnm/) · 10 points
> 
> Already using hindsight in agent mesh. It has been a thorny road to adoption, finicky to say the least but we got it nailed down through many iterations and local patch overlays. Asked hermes-agent/claude opus 4.6 to eval hindsight as it stands in our mesh vs all the ones mentioned in this thread, I found this useful so sharing for anyone else that might:  
> "Hindsight vs Mnemosyne vs [Signet AI](https://www.reddit.com/search/?q=Signet+AI+memory+provider&cId=d2103ce5-5587-4599-8b11-ceefe36342d2&iId=307b9f43-4b94-4226-b26d-2b9ef04656f6) vs [agentmemory](https://www.reddit.com/search/?q=agentmemory+AI+system&cId=17b88ee5-3fce-4557-81c4-78456967b9e5&iId=82ff4b93-d17f-46ab-828a-ca8f7451f74e) — Eval
> 
> Dimension: Architecture
> 
> Hindsight (ours): PostgreSQL + pgvector. Server on .11:8200. Semantic/graph/temporal banks.
> 
> Mnemosyne: SQLite + sqlite-vec. In-process. BEAM tiers (working/episodic/scratchpad).
> 
> Signet AI: Bun/Node daemon + SQLite. Workspace filesystem + semantic layer.
> 
> agentmemory: iii engine + SQLite. 4-tier consolidation (working/episodic/semantic/procedural).
> 
> ────────────────────────────────────────
> 
> Dimension: BEAM 100K score
> 
> Hindsight (ours): 73.4% (SOTA, Apr 2026)
> 
> Mnemosyne: 65.2%
> 
> Signet AI: N/A (97.6% LongMemEval, different bench)
> 
> agentmemory: N/A (95.2% R@5 LongMemEval-S)
> 
> ────────────────────────────────────────
> 
> Dimension: Search
> 
> Hindsight (ours): Semantic + BM25 + Graph + temporal. RRF rerank (FlashRank disabled).
> 
> Mnemosyne: Hybrid: 50% vector + 30% FTS5 + 20% importance.
> 
> Signet AI: FTS5 + vector + graph traversal.
> 
> agentmemory: BM25 + vector + KG. RRF k=60.
> 
> ────────────────────────────────────────
> 
> Dimension: Storage
> 
> Hindsight (ours): Postgres :5434 (184MB). External. Survives all clients dying.
> 
> Mnemosyne: Single SQLite file (~83MB). In-process.
> 
> Signet AI: SQLite + markdown workspace files.
> 
> agentmemory: iii KV state + SQLite. Requires iii-engine binary.
> 
> ────────────────────────────────────────
> 
> Dimension: Latency
> 
> Hindsight (ours): 20-200ms (network round-trip to .11)
> 
> Mnemosyne: 0.076ms read, 1.2ms search (in-process)
> 
> Signet AI: Not published
> 
> agentmemory: 14ms p50
> 
> ────────────────────────────────────────
> 
> Dimension: LLM cost per query
> 
> Hindsight (ours): ~5K tokens (recall with fact extraction + graph)
> 
> Mnemosyne: $0 default (local GGUF or host LLM)
> 
> Signet AI: Configurable (can use local)
> 
> agentmemory: $0 default (no LLM unless configured)
> 
> ────────────────────────────────────────
> 
> Dimension: Entity/KG
> 
> Hindsight (ours): Native. 119MB graph. Entity co-occurrence.
> 
> Mnemosyne: Temporal triples + entity extraction (regex + Levenshtein).
> 
> Signet AI: Knowledge graph + provenance.
> 
> agentmemory: Knowledge graph extraction (optional).
> 
> ────────────────────────────────────────
> 
> Dimension: Temporal knowledge
> 
> Hindsight (ours): Native. Banks have temporal windows.
> 
> Mnemosyne: Native triples with valid\_from/valid\_until + version chains.
> 
> Signet AI: Retention/decay/conflict handling.
> 
> agentmemory: Ebbinghaus decay curve. Auto-eviction.
> 
> ────────────────────────────────────────
> 
> Dimension: Multi-agent
> 
> Hindsight (ours): Per-bank isolation. Banks: ali, avery, rune, office.
> 
> Mnemosyne: Per-bank SQLite. DeltaSync between instances.
> 
> Signet AI: Multi-agent roster. Isolated/shared/group visibility. RBAC.
> 
> agentmemory: Leases + signals + actions + routines.
> 
> ────────────────────────────────────────
> 
> Dimension: Consolidation
> 
> Hindsight (ours): Reflect loop (currently disabled — burns credits).
> 
> Mnemosyne: Auto-sleep cycle. Summarize working→episodic.
> 
> Signet AI: Distillation layer (extraction→decision→graph→retention).
> 
> agentmemory: 4-tier: working→episodic→semantic→procedural.
> 
> ────────────────────────────────────────
> 
> Dimension: Privacy
> 
> Hindsight (ours): All local (LAN Postgres). No cloud.
> 
> Mnemosyne: 100% local. Zero network.
> 
> Signet AI: Local-first. Optional git sync.
> 
> agentmemory: Local-first. Optional cloud deploy.
> 
> ────────────────────────────────────────
> 
> Dimension: Framework lock
> 
> Hindsight (ours): Hindsight API (HTTP). Any client.
> 
> Mnemosyne: Hermes plugin native. MCP server.
> 
> Signet AI: Cross-harness (Claude Code, Codex, Gemini, OpenClaw, Hermes, etc).
> 
> agentmemory: Cross-agent (53 MCP tools, REST, any agent).
> 
> ────────────────────────────────────────
> 
> Dimension: Dependencies
> 
> Hindsight (ours): Postgres + pgvector + Ollama embeddings. External services.
> 
> Mnemosyne: pip install. Optional fastembed.
> 
> Signet AI: Bun/Node. Self-contained.
> 
> agentmemory: Node + iii-engine (Rust binary).
> 
> ────────────────────────────────────────
> 
> Dimension: Maturity
> 
> Hindsight (ours): Production. Running since early 2026. FlashRank leak = known issue.
> 
> Mnemosyne: v3.0.0. Active development. Benchmarked.
> 
> Signet AI: Active. Apache 2.0. Multi-platform.
> 
> agentmemory: v0.9.x. Very active (50+ issues, 950+ tests).
> 
> Verdict
> 
> Hindsight wins on:
> 
> \- BEAM 100K score (73.4% SOTA vs 65.2% Mnemosyne)
> 
> \- Graph depth (119MB memory\_links, entity co-occurrence)
> 
> \- Already deployed, integrated, running
> 
> \- Multi-bank isolation for agent-mesh
> 
> \- Postgres durability (survives all client crashes)
> 
> Hindsight loses on:
> 
> \- Latency (20-200ms vs <1ms Mnemosyne, 14ms agentmemory)
> 
> \- Cost per recall (5K tokens vs zero for local-only systems)
> 
> \- FlashRank ONNX leak (mitigated with RRF, not fixed)
> 
> \- Consolidation disabled (burns credits)
> 
> \- Setup complexity (Postgres + Ollama + systemd)
> 
> Mnemosyne interesting for:
> 
> \- Hermes-native plugin — drop-in Hermes memory provider
> 
> \- Temporal triples with version chains (matches memory-tree §3B idea)
> 
> \- Import FROM Hindsight (migration path exists)
> 
> \- Could replace Hermes built-in memory on each agent machine (local L1 cache)
> 
> \- BEAM benchmark directly comparable
> 
> Signet interesting for:
> 
> \- Cross-harness portability (if we ever run non-Hermes agents)
> 
> \- "Bring your own context" philosophy aligns with memory-tree design
> 
> \- Workspace + transcripts + semantic layers = our 3-tier model
> 
> \- RBAC/team controls for multi-agent (better than Hindsight bank isolation)
> 
> \- Dashboard + inspector for debugging memory behavior
> 
> agentmemory interesting for:
> 
> \- 4-tier consolidation matches our memory-tree design almost exactly
> 
> \- 53 MCP tools = richest agent-facing surface
> 
> \- Session replay (useful for debugging agent behavior)
> 
> \- Automatic hook capture (zero-ceremony memory)
> 
> \- iii-engine = interesting but yet-another-runtime dependency
> 
> Recommendation for Our System
> 
> Keep Hindsight as L2 warm tier. Best benchmark score, already deployed, graph data accumulated. Don't migrate — extend.
> 
> Consider Mnemosyne as local L1 hot tier (per-agent). Each agent machine (.11, .12, Mac) gets Mnemosyne as fast local cache. SQLite, sub-millisecond, zero-network. Hermes plugin = trivial integration. Temporal triples fill memory-tree §3B gap. Export/import to Hindsight handles L1→L2 promotion.
> 
> Steal ideas from:
> 
> \- agentmemory: 4-tier consolidation model, Ebbinghaus decay, auto-forgetting for stale memories
> 
> \- Signet: provenance tracking, memory repair tools, workspace filesystem as truth layer
> 
> \- Both: MCP server surface for external agent access to memory-tree
> 
> Don't adopt:
> 
> \- Signet (Bun/Node daemon = another runtime to manage, cross-harness portability irrelevant since we're Hermes-only)
> 
> \- agentmemory (iii-engine dependency = hard sell for infra we don't control, pinned to v0.11.2, fragile)"

> **z4ni** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/ontkwtn/) · 1 points
> 
> whats the benefit of using one of these providers over using a git-repo to store the memories and obsidian to read/write manually when needed?
> 
> > **Lorian0x7** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/ontn0gi/) · 3 points
> > 
> > LLM wiki style memory works great, probably even better, but its manual.. Mnemosyne inject the right memory at the start of each turn. The information to recall is already there for the LLM without the need to search it.
> > 
> > LLM wiki git repo are for knowledge bases. Memory systems recalls that you have a knowledge base on git.

> **TheCientista** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onujhry/) · 1 points
> 
> I hear a load about using obsidian in hermes so I was surprised to find it was not one of Hermes 7 recommended providers. Neither indeed is Mnemosyne. So I guess I have two questions here. 1.) Is anyone using Obsdian for Hermes agent memory, is it good? And 2. ) why would we go off topic from Hermes 7 recommendations? Surely Nous studio know what is best for their own product?
> 
> > **Lorian0x7** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onuldpm/) · 3 points
> > 
> > Let's call it LLM wiki, Obsidian is just a markdown editor, used tonread the files. This comunity keeps calling it Obsidian makes me crazy XD.
> > 
> > The LLM wiki+ Obsidian is a good way to have a knowledge base. It's does need a skill already present inside Hermes, (called LLM-wiki in fact). It's not a memory system, It a manual way to store and retrieve information in a wiki manually. It's a memory but manual. That's why it's not listed
> > 
> > A memory system inject the right memory at the start of each conversation round automatically.
> > 
> > Mnemosyne it's new, I'm pretty sure that with the right visibility it will be added as a memory provider inside Hermes.

> **gogo101020** · [2026-05-26](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/ony8a9g/) · 1 points
> 
> how safe are these? 🤔

> **Asraf1el** · [2026-06-02](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/opcmu7b/) · 1 points
> 
> I believe people miss-understand why Hindsight is so superior to the rest.
> 
> The AI doesn't need to do any call to hindsight to get the data.. the data is injected in real time into the context. That thing alone is huge... The AI doesn't waste a single cycle into thinking on retrieving the data.. or reading/parsing through memory files, or calling an mcp endpoint.
> 
> Hindsight injects the relevant context automatically in real time inside the conversation. Been using it for a long time. and it's crazy effective.
> 
> Contrary to what you may believe that ends saving a lot of context. My Agents are so smart.. that i been using dumber models lately. because hindsight made them so effective that now i don't need the stupid frontier models for the agent to be effective.
> 
> It's not heavy to run if you choose a tiny model for it. The recommendation is chatgpt OSS 20B which is pretty inexpensive.
> 
> And they did some serious bugfixing in the last releases. i been really happy with it.

> **bidyutm** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onp63cg/) · 3 points
> 
> I've found SignetAI to be better than every option out there during my automated tests involving storing, indexing, and retrieval. Have you tested that?
> 
> [https://github.com/Signet-AI/signetai](https://github.com/Signet-AI/signetai)

> **Bulky\_Magician** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onpbnur/) · 3 points
> 
> how do i switch models? mnenonsyne works great as is but I'd love to give it a better model.

> **Almarma** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onpic7j/) · 3 points
> 
> I just have tried holographic (a disaster) and now Hindsight, which I found quite finicky and delicate to keep up (it can silently fails and then your memories are not processed for days unless you notice it and do something about it). Once it grows, it gets messy and bombards your agent with context that not always is right or appropriate. How's Mnemosyne in that regard? Once it grows, does it becomes messy?

> **pisa\_p** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onqjtp4/) · 3 points
> 
> I tried hindsight and mnemosyne. Hindsight has good integration but I also had the impression that there are too many calls, but the biggest problem for me is the rerank that it puts on my VPS a 100% CPU strain on the embedded model. Menmosyne does a very good job in all aspects. For various reasons, I decided to create a native plugin for graphiti (shares memory with Openwebui) and so far graphiti is very fast and works very well. No problems and in my tests it performed better than hindsight in terms of speed and accuracy. I've only done empirical testing, not benchmarks. Menmosyne is much faster, but I need to get it to work properly on openwebui too, and I still have to find a solution.
> 
> If anyone's interested in the hermes-graphiti plugin, you can find a native version I built for personal use here: [https://github.com/p1s4/hermes-graphiti-plugin](https://github.com/p1s4/hermes-graphiti-plugin). It's just a test and my first time publishing anything, so it's bound to have some bugs or rough edges. That said, I've been using it in my current setup for two weeks now and I'm really happy with it!

> **ivanzhaowy** · [2026-05-25](https://reddit.com/r/hermesagent/comments/1tms3g6/comment/onqw6tn/) · 3 points
> 
> Thanks for this comprehensive breakdown! I've been hesitant about Mnemosyne because of the smaller community, but your detailed comparison really helps. The fact that it's built specifically for Hermes and has that one-liner deploy is huge. I'm definitely going to test it out on my setup. The SQLite approach seems much more practical than some of the heavier alternatives.