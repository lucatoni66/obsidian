---
title: "The Zero-Dependency AI Memory System"
source: "https://mnemosyne.site/"
author:
  - "[[Abdias J]]"
published:
created: 2026-06-18
description: "Mnemosyne is a native, sub-millisecond memory system for AI agents using SQLite. No HTTP, no servers, no API keys. 500x faster than cloud alternatives. Open source."
tags:
  - "clippings hermes"
---
## Memory for Her

The universal memory layer for any AI agent. SQLite-backed, sub-millisecond, zero dependencies. One pip install. That is all.

<1ms

Query latency

0

Dependencies

98.9%

LongMemEval

100%

Local & private

[![Mnemosyne - An open-source memory engine born from Hermes. Sub-ms recall | Product Hunt](https://api.producthunt.com/widgets/embed-image/v1/featured.svg?post_id=1159353&theme=light&t=1780314673437)](https://www.producthunt.com/products/mnemosyne-2?embed=true&utm_source=badge-featured&utm_medium=badge&utm_campaign=badge-mnemosyne-2)

![Mnemosyne memory engine visualization](https://mnemosyne.site/hero-demo.jpg)

### Latest Release

v3.10.0

- L3 persona layer — always-on behavioral rules tier that survives past the 24-hour working-memory TTL. New \`memoria\_persona\` SQLite table with tiered retention (\`permanent\` / \`long\_term\` / \`working\`). New tools: \`mnemosyne\_persona\_promote\`, \`mnemosyne\_persona\_demote\`, \`mnemosyne\_persona\_list\`, \`mnemosyne\_persona\_reinforce\`.
- Rule-based persona extractor (no LLM by default). Reads working\_memory and episodic\_memory, filters by source/importance, deduplicates by topic, renders Markdown grouped by topic. Deterministic and zero-cost.
- Auto-injection into system prompt via \`persona.md\`. Reads \`~/.hermes/memory/persona.md\` and includes it in the \`system\_prompt\_block()\` of the hermes provider. Feature-gated by \`MNEMOSYNE\_PERSONA\_ENABLED=true\` (default OFF). Mtime-cached for hot-path efficiency. Token cap enforced (\`MNEMOSYNE\_PERSONA\_TOKEN\_CAP\`, default 1500).

Features

## Everything you need. Nothing you do not.

Built from the ground up for AI agents that need fast, reliable, persistent memory.

Speed

## Numbers that speak

Measured on CPU with sqlite-vec + FTS5. No GPU required.

Write

0.81ms

56x faster

Read

0.076ms

500x faster

Search

1.2ms

43x faster

Cold Start

0ms

Instant

| Operation | Honcho | Zep | Mem0 | Mnemosyne |
| --- | --- | --- | --- | --- |
| Write | 45ms | 85ms | 50ms | 0.81ms |
| Read | 38ms | 62ms | 45ms | 0.076ms |
| Search | 52ms | 78ms | 60ms | 1.2ms |
| Cold Start | 500ms | 800ms | 300ms | 0ms |

### BEAM Benchmark (ICLR 2026)

End-to-end memory retrieval at scale. LLM-as-judge against published baselines.

100K Context

35.4%

Retrieval from 100K-token conversations

500K Context

19.3%

Retrieval from 500K-token conversations

1M Context

19.2%

Retrieval from 1M-token conversations

Compare

## Mnemosyne vs. cloud memory providers

See exactly what you gain — and what you trade — when you switch.

| Feature | Mnemosyne | Honcho | Zep | Mem0 |
| --- | --- | --- | --- | --- |
| Cost | Free forever | $$$ Paid (credits) | $$$ Paid (Flex+) | Freemium ($0-$249/mo) |
| Hosting | Local - your machine | Cloud only | Cloud / BYOC | Cloud only |
| Privacy | 100% local, zero exfil | External API calls | External API calls | External API calls |
| Offline mode | Yes - airplane mode | No | No | No |
| Setup | pip install | Docker + API keys | Docker + Postgres | API key + signup |
| Vector store | sqlite-vec (built-in) | pgvector (external) | pgvector (external) | pgvector (external) |
| Full-text search | FTS5 (built-in) | Separate service | Separate service | Separate service |
| Auth required | None | Supabase auth | OAuth / API key | API key |
| Rate limits | Unlimited | Plan-dependent | Credit-based | Plan-dependent |
| Data ownership | You own the SQLite file | Vendor-hosted | Vendor-hosted | Vendor-hosted |
| Export / import | One JSON file | Limited | Limited | Limited |
| Dependencies | Python stdlib + ONNX | Docker, Postgres | Docker, Postgres | pip + API key |
| Memory architecture | BEAM (3-tier) | Session + facts | Graph RAG + facts | Session + facts |
| Auto-consolidation | Sleep cycles built-in | Manual / paid | Manual | Manual |
| Temporal triples | Native with validity | No | No | No |
| LongMemEval | 98.9% Recall@All@5 | Not published | Not published | Not published |
| BEAM-100K | 35.4% / 19.3% / 19.2% | Not published | Not published | Not published |

### The bottom line

- ✓Speed: 43-500x faster than cloud alternatives — zero HTTP roundtrips.
- ✓Privacy: Data never leaves your machine. No API calls. No telemetry.
- ✓Cost: Zero ongoing cost. No credits. No tiers. No "contact sales."
- ✓Simplicity: One pip install. No Docker. No config. No signup.

Trade-off: You manage your own backup (one SQLite file). No web dashboard or team collaboration — Mnemosyne is built for individual developers and local agents.

Trusted