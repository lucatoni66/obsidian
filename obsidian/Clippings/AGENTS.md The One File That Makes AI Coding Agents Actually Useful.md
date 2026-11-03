---
title: "AGENTS.md: The One File That Makes AI Coding Agents Actually Useful"
source: "https://dev.to/wolfejam/agentsmd-the-one-file-that-makes-ai-coding-agents-actually-useful-ckj"
author:
  - "[[wolfejam.dev]]"
published: 2026-06-30
created: 2026-06-30
description: "If you’ve used Claude Code, Cursor, Codex, Aider, Gemini CLI, GitHub Copilot, Grok, goose, or similar... Tagged with ai, agents, opensource, productivity."
tags:
  - "clippings ai llm"
---
If you’ve used Claude Code, Cursor, Codex, Aider, Gemini CLI, GitHub Copilot, Grok, goose, or similar tools, you’ve seen the same pattern: the agent’s first priority is context. What is this project? How do I build it? How do I run the tests? What conventions should I follow? What must I not break?

**AGENTS.md** is the open answer. It’s a single Markdown file placed at the root of your repository that serves as a dedicated, predictable briefing for AI coding agents. Think of it as a README written specifically for the agent instead of a human. The format is deliberately simple and tool-agnostic — one file that works across many agents.

AGENTS.md was pioneered by OpenAI's Codex and shaped alongside tools like Cursor, Amp, Factory, and Google's Jules. In December 2025 it was donated to the Agentic AI Foundation under the Linux Foundation, where it's now stewarded as an open standard.

Most AGENTS.md files fail in one of two ways: they’re either too thin to be useful or they’re long, rambling documents the agent skims and mostly ignores. A good AGENTS.md sits in the middle — short, current, specific, and actionable.

## README is for humans. AGENTS.md is for the agent.

This distinction determines everything that belongs in the file.

- **README.md** answers *what* and *why*: what the project is, why it exists, and how a human gets started.
- **AGENTS.md** answers *how to work here*: the exact commands, conventions, and guardrails an agent needs to make changes and verify them without asking questions.

Do not duplicate content from the README. If the agent doesn’t need it to act effectively, leave it out. Every non-essential line dilutes the signal.

## What actually belongs in AGENTS.md

The highest-value sections, in rough priority order.

**One-line orientation.** A single sentence telling the agent what the project is and what stack it uses — for example, “A TypeScript REST API on Node 20 with Postgres, deployed to Fly.io.”

**Setup and build commands.** The single most valuable section. Give the real, copy-pasteable commands:

```
npm install
npm run build
npm run dev
```

**How to run tests.** Even more important than build commands — tests are how the agent verifies its own work:

```
npm test          # all tests must pass before considering a change complete
npm run lint
npm run typecheck
```

**Where things live.** A short, focused map of key directories and entry points — not a full tree dump.

**Conventions.** Specific patterns you want followed: validation approach, error handling, naming, preferred libraries, architectural decisions. Vague statements like “write clean code” are useless; specific rules like “Validate all input with Zod at the route boundary” are useful.

**Commit and PR rules.** Message format, branch naming, changelog updates, and any other process requirements.

**Guardrails — what NOT to do.** The landmines that prevent expensive mistakes: don’t edit files in `/generated`, never run migrations against production config, and don’t commit directly to main.

## The anti-patterns that make it worse than nothing

A bad AGENTS.md doesn’t just fail to help — it actively misleads the agent.

- **The novel**: Too long. The agent skims and important instructions get lost. Cut anything that isn’t load-bearing.
- **The stale file**: An outdated command or path is worse than no file at all. The agent will confidently do the wrong thing.
- **Vagueness**: “Follow best practices” gives the agent nothing actionable.
- **Secrets**: Never put credentials, keys, or tokens in AGENTS.md. Point to where they come from (`.env`, secrets manager) instead.
- **README duplication**: Wasted tokens and maintenance burden.

## Keep it alive — treat AGENTS.md like code

AGENTS.md files rot when no one owns them. Treat the file the same way you treat production code:

- Review changes to it in pull requests.
- Update it the moment reality changes (new test command, new directory structure, new convention).
- Watch what the agent actually does. When it guesses, asks unnecessary questions, or touches something it shouldn’t, that’s usually a missing or stale instruction.

A useful habit: every time you find yourself giving the same instruction to an agent twice in chat, move that instruction into AGENTS.md.

## One file, many agents

The strength of AGENTS.md is that it isn’t tied to any single tool. Most major coding agents either read it directly or converge on the same root-level instruction pattern. Write plain Markdown with tool-agnostic instructions. Avoid “if you are using X tool” branching.

For monorepos, you can place additional `AGENTS.md` files in subdirectories. Most agents use the nearest file to the code they’re working on.

## A minimal, complete example

```
# AGENTS.md

A TypeScript REST API on Node 20, Postgres, deployed to Fly.io.

## Setup
npm install
cp .env.example .env   # fill in DATABASE_URL

## Build & run
npm run build
npm run dev            # http://localhost:3000

## Test — all must pass before a change is considered done
npm test
npm run lint
npm run typecheck

## Structure
- src/routes/     HTTP handlers
- src/db/         Postgres queries (use the query builder, no raw SQL)
- src/lib/        shared utilities
- test/           Vitest tests, mirroring src/ structure

## Conventions
- Validate all input with Zod at the route boundary
- Throw AppError (from src/lib/errors.ts), never a bare Error
- Match the style and patterns of the surrounding file

## Commits & PRs
- Use Conventional Commits (feat:, fix:, chore:)
- One logical change per PR
- Update CHANGELOG.md when relevant

## Don't
- Don't edit anything in src/generated/ (regenerated from schema)
- Don't commit directly to main — always branch and open a PR
- Never run db:reset against a non-local DATABASE_URL
```

An agent (or a new human teammate) can read this in under 30 seconds and start being productive.

## The real test of a good AGENTS.md

You’ll know it’s working when a fresh agent can land a correct, tested change in your repository without asking how to build, how to test, or which conventions to follow — and without touching the one thing you explicitly told it not to touch.

Short. Current. Specific. Actionable. That’s the entire job.

---

*This was the field guide — the why and the what.*

**Coming next in the series:** a step-by-step build-along, where we’ll write a complete AGENTS.md against a real repo, then point an agent at it and watch it build, test, and respect the guardrails without being asked. (A day or two out.)

[![Google article image](https://media2.dev.to/dynamic/image/width=775%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fi6mj0yymgm9gmhlz7l4y.png)](https://dev.to/googleai/building-capabilities-for-a-multi-agent-system-with-google-adk-mcp-and-cloud-run-ab9?bb=263437)

## Building Capabilities for a Multi-Agent System with Google ADK, MCP, and Cloud Run

My team's mission is to accelerate the developer journey from writing code to running secure AI workloads on Google Cloud. To help developers succeed, we focus on identifying their most pressing questions and building demos that provide straightforward, easy-to-implement solutions.