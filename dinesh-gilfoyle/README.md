# Dinesh vs Gilfoyle: Adversarial Code Review Extension

An entertaining and technically rigorous two-agent adversarial code review skill for Gemini CLI, inspired by HBO's *Silicon Valley*.

## Overview

This extension introduces a unique code review workflow where two distinct personas—**Gilfoyle** and **Dinesh**—debate the quality, security, and architecture of your code.

- **Gilfoyle**: Attacks the code with withering technical precision, dry sarcasm, and a focus on security vulnerabilities, architectural rot, and "moral failings" in the implementation.
- **Dinesh**: Defends the code (as if he wrote it) with flustered competence, technical justifications, and occasional successful pushbacks against Gilfoyle's "nitpicks."

The result is a comprehensive review that covers security (OWASP, PII, CVEs), database performance, distributed systems, and general code quality, all while providing an entertaining back-and-forth.

## Installation

To use this extension in your Gemini CLI environment, ensure the files are located in your extensions directory:

1.  Place the `skills/dg` folder into your Gemini CLI extensions path (typically `~/.gemini/extensions/dinesh-gilfoyle/skills/dg`).
2.  The extension will be automatically discovered by the Gemini CLI.

## Usage

Invoke the adversarial review using the `/dg` command:

### Commands

- `/dg` — Review all current changes (staged + unstaged `git diff`).
- `/dg <number>` — Review current changes with a specific maximum number of debate rounds (e.g., `/dg 3`).
- `/dg <path>` — Review a specific file or directory (e.g., `/dg src/auth.ts`).
- `/dg <path> <number>` — Review a specific path with a set limit of rounds.

### What to Expect

1.  **Context Gathering**: The system gathers your code and relevant dependency files (`package.json`, `go.mod`, etc.).
2.  **The Attack**: Gilfoyle performs an initial scathing review, looking for security flaws, nits, and architectural weaknesses.
3.  **The Defense**: Dinesh responds to each point, either conceding (grudgingly), defending with technical evidence, or dismissing points as nitpicks.
4.  **The Debate**: The agents may go back and forth for several rounds until a verdict is reached or the round limit is hit.
5.  **The Richard Protocol (Interactive Synthesis)**: Once the debate concludes, the agents pivot to address *you* as **Richard** (the Founder/CEO). They will each present their case and ask you strategic questions to finalize the direction.
6.  **The Verdict**: A final summary of actionable improvements is provided only after Richard's approval.

## The Richard Protocol & State Tracking

This extension features a specialized interaction layer known as the **Richard Protocol**, backed by an automated state tracker.

### Richard Protocol (Founder Loop)
When the technical debate reaches a conclusion, the agents will address the user as "Richard," the CEO of Pied Piper.
- **Executive Authority**: As Richard, you have the final say on strategic decisions.
- **The "What About?" Rule**: Before synthesis, agents check for unresolved loops and proactively ask, *"Richard, what about [item]? We discussed this but haven't reached a conclusion yet."*
- **Interactive Q&A**: Dinesh simplifies concepts for your "product vision," while Gilfoyle uses dire analogies to explain the technical catastrophe of your choices.

### Silent Background Sync (Parallel Batching)
To ensure the debate remains fluid and context-rich without cluttering the chat:
- **PENDING.md Tracker**: The orchestrator maintains a `PENDING.md` file in the review's archive directory.
- **Parallel Batching**: Status updates to the tracker are executed in the **SAME TURN** as agent dispatches. This hides the file-system operations from the conversation while keeping the "Richard Protocol" perfectly synced.
- **Loop Tracking**: Every point raised is tracked as `[OPEN]` until it is explicitly `[CONCEDED]`, `[DEFENDED]` (with agreement), or `[RESOLVED]` by Richard.

## Technical Scope

The agents are configured to specifically look for:
- **Security**: Hardcoded secrets, PII leakage, Injection vulnerabilities, and known CVEs in dependencies.
- **Database**: N+1 queries, missing indexes, and connection pooling issues.
- **Architecture**: Distributed systems patterns, idempotency, and proper orchestration.

---
*Note: This is a community-created extension for entertainment and educational purposes. No actual gold-chain-wearing engineers were harmed in the making of this skill.*
