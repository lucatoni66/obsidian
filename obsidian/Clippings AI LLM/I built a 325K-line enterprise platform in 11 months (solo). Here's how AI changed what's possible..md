---
title: "I built a 325K-line enterprise platform in 11 months (solo). Here's how AI changed what's possible."
source: "https://dev.to/traviticus/i-built-a-325k-line-enterprise-platform-in-11-months-solo-heres-how-ai-changed-whats-possible-2c2c"
author:
  - "[[Travis Wilson]]"
published: 2026-06-25
created: 2026-06-25
description: "I'm Travis, a staff engineer with ~20 years building software. Eleven months ago I started building..."
tags:
  - "clippings"
---
I'm Travis, a staff engineer with ~20 years building software. Eleven months ago I started building Flywheel — an AI workflow automation platform. Solo. Evenings and weekends.

Today the codebase is 325,000 lines of production code. 702 commits. Three services (Go backend, Next.js frontend, Python AI agent), 14 integrations, full CI/CD, Terraform infrastructure. Enterprise-grade auth, billing, monitoring — the works.

I didn't mass-generate code. I didn't skip tests. I wrote a real system that I'm shipping to real users. Here's how.

## The Stats

Let's get specific. These are real numbers, not estimates.

**Codebase:**

- 170K lines of Go (backend API, 14 provider integrations, distributed pipeline execution)
- 145K lines of TypeScript (visual workflow editor, React Flow canvas, 60+ component directories)
- 7.5K lines of Python (AI agent — FastAPI, Anthropic SDK, MCP tools)
- 2,500 lines of Terraform (GCP infrastructure as code)
- 847 test files across the stack

**Git churn:**

- 1.37 million lines inserted
- 716K lines deleted
- Only 325K survived
- For every line of code in the app, I wrote more than two

**Claude Code usage (all time):**

- 2,900 sessions across 144 active days (longest streak: 42 days straight)
- 94.7 million tokens generated — plus billions more served from prompt cache
- ~$1,200 in Claude Max subscription — usage that would've metered at $2,500+ on API pricing

## What I Actually Built

Flywheel automates recurring tasks. Not one-off AI magic — persistent workers that run on schedule, via webhook, or triggered by events.

The core insight: AI is great at reading messy data (emails, PDFs, Slack). Deterministic logic is great at routing, filtering, delivering. Combine them and you get workflows that handle jobs reliably.

**What the platform does today:**

- Visual drag-and-drop workflow builder on a React Flow canvas
- 10 data providers: Gmail, BigQuery, PostgreSQL, S3, DynamoDB, Firestore, GCS, Google Drive, Pub/Sub, Domo
- AI enrichment nodes: Claude and ChatGPT run mid-pipeline, reading data and producing structured output
- 8 processing transforms: filter, route, join, collect, flatten, split, apply template
- Human-in-the-loop: workflows pause for human review before continuing
- Distributed execution with chunked streaming, per-node error policies, failure retry
- Multi-org RBAC with entity-level permissions
- Stripe billing, monitoring dashboard, template marketplace

The workflow canvas isn't just where you build. It's where you run your operation. That's the 1.0 vision — you open the canvas and see live stats, reports, and action queues from your running workflows, not just the pipeline graph.

## Why the Fundamentals Post Matters More Now

Six months ago I wrote about how [strong fundamentals + AI](https://dev.to/traviticus/how-strong-fundamentals-ai-helped-me-build-a-data-pipeline-platform-in-6-months-solo) multiplied my output. That post had 4,366 tests, and the core thesis was: AI multiplies whatever you already have. Clean codebase → clean AI output. Messy codebase → garbage.

That thesis has only gotten more true at scale. Here's what changed:

**Then (September 2025):** 4,366 tests. Felt like a lot.  
**Now (June 2026):** 8,000+ tests. 847 test files. 372 API endpoints.

**Then:** "Claude can generate endpoints that follow your patterns."  
**Now:** Claude reads CLAUDE.md files for context, follows registry patterns to add new providers, writes tests that match conventions, and refactors entire domains in a single session.

The fundamentals didn't just help me write code faster. They made it possible for AI to work on the codebase *without me present*.

## The Workflow That Scales

My process hasn't changed much. It's just faster.

**1\. Claude sketches, I refine.**

I let Claude take the first pass at a design. Then I highlight the parts that don't feel right and we iterate — back and forth, tightening the plan. Before implementation, I have Claude challenge its own plan to surface gaps I'm not thinking of. Once I'm satisfied, I let it ride.

**2\. Screenshots are worth a thousand tokens.**

I remapped macOS screenshot hotkeys to save directly into a `/screenshots` folder in my repo (gitignored). When I'm working with Claude Code, I drop a screenshot into the conversation and we're immediately on the same page — no describing layout issues, no explaining what "that button on the right" means. A picture is worth a thousand words, and it turns out it's worth even more when your pair programmer is an AI.

**3\. Tests after manual verification.**

I test manually first to iterate fast. Once I'm happy, Claude writes the tests to lock it in. Then `/check --run-all` runs the full suite before anything gets committed.

**4\. Continuous refactoring.**

Every feature that touches existing code is a refactoring opportunity. Small improvements. Better names. Clearer abstractions. This is what keeps the codebase AI-friendly at scale. Stop refactoring and the code rots — then AI suggestions start degrading too.

## Where This Is Going (and Why I'm Sharing)

The goal isn't just "solo dev ships product." It's something I find more interesting: **using Flywheel to help run Flywheel.**

I'm building workflows that take a first pass at the day-to-day, with me in the loop to review:

- Support — reading incoming tickets, routing them, and drafting replies to the common ones for me to approve
- Bug triage — spotting an edge case, reproducing it with a test, and opening a PR I can check before it merges
- Marketing — scheduling content, tracking engagement, and helping with cross-posting

A workflow platform only really earns trust if its own author leans on it, so I'm trying to live on it. The idea is to let Flywheel carry the repetitive load so my time goes to the harder problems. It's early — plenty doesn't work yet — but that's part of why I'm building it in the open.

After 11 months of solo building, the foundation feels like it's there.

## The Visual Journey

If you want to see how far the UI has come — the earliest version was a dark, flat, brutalist thing. Basic black backgrounds, no design system, functional but ugly. Today it's a polished visual workflow canvas with glass morphism, gradient provider icons, a two-level canvas architecture, and a real design system. Same codebase. Same one developer.

## Key Takeaways

1. **AI multiplies your codebase quality, not just your speed.** A clean, well-structured codebase gets better AI output than a messy one at any size.
2. **~$1,200 of Claude Max is not "cheating."** That's roughly $110 a month for usage that would meter at $2,500+ on API pricing. It's a new kind of labor — you still design, review, test, and ship. The tokens do the typing.
3. **Fundamentals compound.** IaC, event-driven architecture, service layer patterns, CLAUDE.md context files — every investment in structure pays dividends as the codebase grows.
4. **Fresh context > long context.** Even with 1M context windows, focused sessions produce better output than marathon ones.
5. **The solo dev ceiling just moved.** Production code with tests, CI/CD, infra, and 14 integrations. One person. Eleven months. That wasn't possible two years ago.

## What's Next

Flywheel 1.0 ships soon. Widgets turn the workflow canvas into a live operational dashboard — stats, reports, action queues alongside your pipeline cards. Then we start dogfooding hard: Flywheel workflows running Flywheel's support, development, and marketing.

If you're interested in the platform or the AI-assisted development approach: [flywheeletl.io](https://www.flywheeletl.io/). Want to see what a real workflow looks like? The [AI Marketing Worker for X](https://www.flywheeletl.io/explore/official-social-monitor) is three pipelines that find reply opportunities, draft responses for human approval, and track follower growth on a live dashboard — the same worker I use to run Flywheel's own marketing. (I also built a [Slack Court](https://www.flywheeletl.io/explore/official-slack-court) that puts petty team grievances on trial in a dedicated courtroom channel, complete with an AI judge — because dogfooding should be fun too.)

---

Has the ceiling for what one person can build actually changed? Or am I just in the honeymoon phase? Curious what others are seeing.[AWS](https://dev.to/aws)Promoted

[![Power smarter decisions with the cloud](https://media2.dev.to/dynamic/image/width=775%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fucarecdn.com%2F11127d15-483a-4e01-a4ee-6032a1253d81%2F)](https://aws-experience.com/amer/smb/events/series/IndustriesLive-AWSandAWSIndustriesPartnersShow?bb=263059)

## Power smarter decisions with the cloud

Join AWS experts and Partners to learn how cloud technology supports efficiency, agility, and growth. Watch live.