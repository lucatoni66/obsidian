---
title: "Stop Trading Like It's 1999 — I Built an Autonomous, Vision-Capable Crypto Bot with Python 3.13"
source: "https://dev.to/qrak/-stop-trading-like-its-1999-an-llm-crypto-bot-that-sees-charts-remembers-trades-and-learns-1cd1"
author:
  - "[[qrak]]"
published: 2026-07-30
created: 2026-07-30
description: "Let me tell you a story about failure.  December 2024. I'm sitting in my Wrocław apartment after an... Tagged with ai, webdev, programming, crypto."
tags:
  - "clippings forex"
---
Let me tell you a story about failure.

December 2024. I'm sitting in my Wrocław apartment after an 8-hour warehouse shift. My back hurts. I've been trying to make a trading bot work for two weeks. Every night, same result: the LLM generates beautiful-sounding analysis. Confident. Articulate. Completely wrong.

"RSI is 28, oversold. BUY." Next morning: -4.2%.

The problem wasn't the LLM being stupid. It was me being lazy. I was dumping numbers into a prompt and hoping for magic — the exact thing every "LLM trading bot" tutorial on YouTube tells you to do.

I had three options: give up, buy a course, or do what nobody else was doing — read the actual research papers.

I chose option three. 77 papers later, I understood the real problem. And I built something that fixes it.

---

## The Research Rabbit Hole

Here's what 77 academic papers and industry reports taught me about LLM trading:

**TradingAgents** (UCLA/MIT, 95k GitHub stars) — brilliant framework. Multi-agent debate, specialized analyst roles. But three dealbreakers: stocks only (no crypto), zero chart vision (raw numbers only), and every analysis costs 5+ API calls. Beautiful architecture, expensive to run.

**Optiver's 2026 research** — LLMs pass their intern trading exam. They understand EV. They can explain market making. But when it's time to actually execute? They freeze. Too conservative. Leave money on the table. The gap between understanding EV and maximizing EV is where real trading lives — and LLMs fail there.

**Agentic Trading Survey** (arXiv 2026) — 77 studies reviewed. Only 2 had proper time-consistent data splits. Zero reached reproducibility level R3. Most academic LLM trading papers are architecturally impressive PowerPoint slides with cherry-picked backtests. Try running their code.

**Lopez-Lira at Bank of Finland** — LLM agents can trade. But they don't give a damn about losing money unless you explicitly program them to care. They'll follow losing instructions straight to zero, smiling in their probabilistic way the whole time.

So I asked: what if one bot fixed all of this? Chart vision for pattern recognition. EV computation from real history. Falsification checks that reject hallucinated signals. Vector memory that remembers every trade outcome. And bull/bear debate that costs zero extra API calls.

That's what I built. Here's exactly how.

---

## 1\. Chart Vision — The LLM Reads Candlestick Images

Every cycle, the bot generates a 4K PNG candlestick chart with 5 overlay indicators (SMA, RSI, Volume, CMF, OBV). This image goes directly to Google Gemini 3.6 Flash. Not as bytes. Not as a data URL. As a visual inference.

Why? Because I spent two weeks coding pattern detection algorithms — head and shoulders, wedges, trendlines, support/resistance. Then I compared the AI's visual reads against my code. The AI was better. By a lot.

I deleted 900 lines of pattern detection code. Never looked back.

## 2\. 50+ Indicators, JIT-Compiled Locally

No pandas-ta. No TA-Lib. I built the entire indicator engine from scratch using NumPy and Numba. Every EMA, SMA, MACD, ADX, Stochastic, Bollinger Band, TTM Squeeze, Ichimoku component, and 11 moving average variants compiles to machine code on first call and caches the result.

```
@njit(cache=True)
def _ema_numba(prices, period):
    alpha = 2.0 / (period + 1)
    result = np.empty_like(prices)
    result[0] = prices[0]
    for i in range(1, len(prices)):
        result[i] = alpha * prices[i] + (1 - alpha) * result[i - 1]
    return result
```

50 indicators. All custom. All JIT-compiled. All running in microseconds on a CPU. No CUDA. No cloud. Just a Ryzen 5700G on my desk.

## 3\. Bull vs Bear Debate — One Prompt, Two Sides, Zero Extra Cost

TradingAgents spawns separate agents for bull and bear analysis. Smart, but each one costs an API call. My bot gives the LLM a single instruction: argue the bullish case first (with evidence from the chart, indicators, and news), then argue the bearish case (with the same rigor). Then decide.

One prompt. Both perspectives. Zero extra tokens. The result is less biased because the model has to hold both arguments in context simultaneously — it can't "forget" the counter-argument halfway through.

## 4\. EV Framework — What Optiver Said LLMs Can't Do

Optiver found LLMs can explain expected value but can't execute it. My EV Framework computes:

- Win rate from trade history (filtered by similarity to current setup)
- Average win size vs average loss size
- Expected P&L = (win\_rate × avg\_win) — ((1 — win\_rate) × avg\_loss)
- Kelly fraction sizing based on account equity
- Risk/reward validation before signal acceptance

The LLM provides the reasoning. The EV Framework provides the math. Neither trusts the other blindly. If the LLM says "strong buy" but EV comes back negative — HOLD.

## 5\. Falsification Check — Name the Price That Proves You Wrong

Before any signal is accepted, the LLM must write one sentence: "This signal would be proven wrong if \[specific price level or indicator condition\] occurs."

If it can't name one — signal rejected. If the named condition triggers before the next cycle — position closed early. If the named condition is vague ("if the market turns bearish") — rejected, try again with specificity.

This single prompt addition improved signal quality more than any other change. Hallucinated signals can't produce falsifiable claims.

## 6\. ChromaDB Vector Memory — The Bot Remembers Its Mistakes

Every closed trade gets embedded as a vector with 15+ metadata fields:

symbol, direction, entry price, exit price, P&L percentage, max adverse excursion, timeframe, ADX at entry, trend strength, RSI, volume trend, pattern detected, news sentiment score, surprise ratio, and closed timestamp.

On every new analysis cycle, the bot queries: "Find me the 5 most similar trades to this current setup." These 5 trades are injected into the LLM prompt with their outcomes. The AI sees real results from similar past conditions.

Time-decay ensures recency matters: a trade from last week carries 3x more weight than one from two months ago, even if the indicators look identical.

## 7\. Seven AI Agents that Maintain Their Own Codebase

In the.ai directory you'll find seven specialized agent prompts. Each has a dedicated scope, a journal, and a personality. They've produced real commits.

- Bolt (performance): Found a sync file read in an async loop — 40ms latency fix
- Palette (UX): Added ARIA labels, toast notifications, collapsible panels
- Sentinel (security): Rate limiting middleware, CSP headers, API key isolation
- Refactor (clean code): Killed 27 isinstance chains, enforced DI pattern across 14 modules
- Concise (code reduction): Reduced 380 lines across the prompt builder
- Bugfixer (regressions): Caught 3 bugs before they hit main — verified by reading ALL journals
- Smoke Tests (pre-flight): Syntax check, import scan, and ruff lint before every commit

133 commits since the agent system was introduced. Every one of them tested and documented.

## 8\. Validation Pipeline — Never Trust Raw LLM Output

Every AI response passes through:

- TrendValidator: cross-checks LLM-reported trend strength against actual ADX calculations. Discrepancy over 15 points — computed value wins
- PatternQualityScorer: deterministic 0-100 score from actual pattern detection, not LLM self-reporting
- Falsification Check: reject signals that can't name their own failure condition
- Risk Guards: symbol whitelist, max position size, cooldown window, minimum risk/reward, SL/TP validity, wrong-side SL/TP detection

## 9\. Zero-Cost Sentiment — No API Keys, No Paid Services

Reddit JSON (free, no auth needed), X/Twitter through Nitter instances (public feeds), RSS from CoinDesk, CoinTelegraph, Decrypt, CryptoSlate. All processed through a RAG engine built on ChromaDB. No monthly bills. No API rate limits. Just free internet data, curated.

---

## The Test Suite (The Part Nobody Talks About)

1,324 tests. Fully mocked — no network calls, no ChromaDB, no LLM API. They run in about 60 seconds on any machine.

Coverage includes: LLM output corruption, async race conditions, rate limiting with exponential backoff, vector DB boundaries, friction reporting for all 6 guard types, closed-loop feedback injection, ticker retry with network timeout handling.

When Optiver says LLMs "fail to incorporate new information after acting" — my tests verify the bot doesn't do that. When the arXiv survey says reproducibility is the bottleneck — my tests are deterministic.

---

## What It Costs to Run

- Google Gemini 3.6 Flash: free tier (1,500 requests/day)
- Technical indicators: CPU (Numba JIT, microseconds)
- News and Sentiment: free (Reddit JSON, Nitter, RSS)
- Vector database: local ChromaDB (~50 MB disk, ~200 MB RAM)
- Hosting: home server (Ryzen 5700G, 32 GB RAM)
- Dashboard: Cloudflare Tunnel (free tier)

Month-to-month cost: exactly 0 zł. Not "almost free." Free.

---

## Quick Start

```
git clone https://github.com/qrak/LLM_trader.git
cd LLM_trader
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp keys.env.example keys.env
python start.py
```

Dashboard at [http://localhost:8000](http://localhost:8000/). Windows users: run scripts/experimental\_branch.ps1.

---

## Honest Limitations

What this bot IS: a research-grade, open-source LLM trading engine. Paper trading only. Fully transparent — every prompt, response, and trade log is inspectable. Self-improving through reflection engine.

What it's NOT: a get-rich-quick scheme, a production trading system (yet), or something you should trust with real money without extensive testing.

---

## Links

Source: github.com/qrak/LLM\_trader  
Live Dashboard: semanticsignal.qrak.org  
Landing Page: qrak.org  
Buy Me a Coffee: buymeacoffee.com/qrak  
Email: [contact@qrak.org](mailto:contact@qrak.org)

---

Built in Wroclaw, Poland. No degree. No bootcamp. Just Python, asyncio, and thousands of hours after warehouse shifts.

[![Google article image](https://media2.dev.to/dynamic/image/width=775%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1iux9rx86hrnk85o5g4p.png)](https://dev.to/gde/your-prompts-are-legacy-code-now-the-google-cloud-next-26-developer-breakdown-hc3?bb=263389)

## Your Prompts are Legacy Code Now: The Google Cloud Next '26 Developer Breakdown

Google's showcase centered on a multi-agent system coordinating a full city-scale marathon simulation—Planner, Evaluator, and Simulator agents working in concert.  
But the marathon wasn't the point. The architecture behind it is what you need to steal for your own systems to move from "vibes" to engineering rigor.