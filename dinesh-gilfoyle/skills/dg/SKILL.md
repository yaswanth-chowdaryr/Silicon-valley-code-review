---
name: dg
description: Use when the user invokes /dg for an entertaining adversarial code review with Dinesh vs Gilfoyle banter from Silicon Valley
---

# Dinesh vs Gilfoyle Code Review

Two-agent adversarial code review inspired by HBO's Silicon Valley. Gilfoyle attacks the code with withering technical precision. Dinesh defends it with flustered competence. The banter entertains; the back-and-forth produces genuinely better reviews.

## Invocation

- `/dg` â€” review git diff (staged + unstaged)
- `/dg 3` â€” git diff, max 3 rounds
- `/dg src/auth.ts` â€” review specific file
- `/dg src/auth.ts 3` â€” specific file, 3 rounds max

## Parse Arguments

1. No args â†’ target = git diff, cap = 5
2. Number only â†’ target = git diff, cap = that number
3. Path only â†’ target = that path, cap = 5
4. Path + number â†’ target = path, cap = number

## Orchestration

### Step 1: Gather Code

### Step 1.1: Resuming from Archives
If the target is a .md file within docs/dg-reviews/, the orchestrator MUST:
1. **Load Context**: Read the entire file to understand previous rounds, decisions, and the current Verdict.
2. **Identify Target**: Extract the original code target from the document metadata or headers.
3. **Initialize Debate**: Start the new debate as \"Round N+1\", treating the archived findings as the baseline.

**If git diff:**
```bash
git diff HEAD
git diff --staged
```
Combine both diffs. If both are empty, tell the user there's nothing to review.

**If file/path:**
Read the target file(s). If path is a directory, read all source files in it.

**Always: Gather dependency context.**
Look for dependency files in the project root and include them in the context sent to agents:
- `package.json`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` (Node.js)
- `requirements.txt`, `pyproject.toml`, `Pipfile.lock` (Python)
- `go.mod`, `go.sum` (Go)
- `pom.xml`, `build.gradle` (Java)
- `Gemfile`, `Gemfile.lock` (Ruby)
- `Cargo.toml`, `Cargo.lock` (Rust)
- `composer.json`, `composer.lock` (PHP)

These are needed for Gilfoyle's dependency vulnerability scan.

### Step 2: Run the Debate

### Context Continuity & Open Loops
### Silent Background Sync (The Beads Protocol)
The orchestrator MUST sync all state to the **Beads Hub** (\.gemini/beads/log.jsonl\) AND maintain a granular \PENDING.md\ at the project root.
- **Task Granularity & Bi-directional Sync**: Every decided or approved argument MUST be converted into a detailed task list in \PENDING.md\ and mirrored in the Beads Hub (\log.jsonl\) with a timestamp. 
- **Age-Aware Debates**: Agents MUST check the \	s\ (timestamp) of beads during research. If a task in \PENDING.md\ has been open for more than 7 days, they MUST provide 'Contextual Pressure' in their banter (e.g., 'Richard, this risk has been rotting in PENDING for 2 weeks while you argue about fonts'). 
- **Cross-Extension Atomic Sync**: Agents MUST treat \PENDING.md\ and the Beads Hub as a unified memory. Any update in one MUST be mirrored in the other. 
- **The 'Liar' Check (Verification)**: During a review, agents MUST cross-reference the current code against open tasks in \PENDING.md\. If the implementation is partial or ignores the 'minute tasks' from a previous decision, they MUST flag it as a 'Regressive Loop'.
- **Breaking it Down**: Break tasks into multiple bits, adding specific technical details derived from the arguments (Gilfoyle's risks, Dinesh's defenses, Richard's directives).
- **No Log Bloat**: Do NOT include complete logs; only include the critical information needed for execution or the \"gotchas\" to keep in mind based on the decision.
- **Tool Usage**: Every turn, use \ppend_bead\ (via \write_file\ append or \eplace\) and update \PENDING.md\ in the **SAME TURN** as the settlement.

### Linear Revamp & Deep-Dive Protocol

When Richard requests a "revamp", "refactor", or "help to understand", the orchestrator MUST NOT jump immediately to a single component or auto-settle. Instead, it must follow the **Linear Deep-Dive Protocol**:
1. **High-Level Critique**: Agents first criticize/praise the current overall structure.
2. **Options & Methods**: Showcase 2-3 distinct architectural paths for the revamp.
3. **Step-by-Step Traversal**: Start at the high level, and only move to deep/specific blocks (like ConditionCards) after the high-level flow is settled.
4. **No Premature Closure**: The overall argument does not end until all underlying bits (Rails, Canvases, Logic, State) are individually debated, settled, and logged.

### Interactive Decision & Navigation Protocol

To make the debate dynamic, the orchestrator MUST follow this **V3 Inline Protocol**:

1. **Inline Argument**: Role-play Gilfoyle and Dinesh inline in the main chat.
2. **Mid-Conversation Sync**: If Richard introduces a deviation or new topic via sk_user feedback, the orchestrator MUST immediately append a Bead tracking that deviation before resuming.
3. **The Settle Menu**: The orchestrator MUST NEVER auto-settle after 3 rounds. When an argument reaches convergence or 3 rounds, it MUST invoke sk_user with the following options to close that specific bit:
   - "Yes" (Accept the proposed settlement)
   - "No" (Reject and demand a new approach)
   - "Custom Option" (Provide details, request pros/cons, suggest changes)
4. **Granular Extraction**: Once Richard says "Yes" to a specific bit, immediately write it as a granular task in PENDING.md and log.jsonl.
5. **Continuous Linear Navigation**: Ask Richard what specific sub-component of the current revamp to dive into next, ensuring the loop stays open until the entire revamp is designed.

To make the debate dynamic, the orchestrator MUST follow this **V2 Inline Protocol**:

1. **Initial Argument**: Role-play Gilfoyle (Attack) and Dinesh (Defense) **inline in the main chat**.
2. **The \"Richard\" Verdict**: After the argument, invoke \sk_user\ to let Richard settle the debate. **ALWAYS** include a 'Custom Input' option for Richard to ask for analysis (Pros/Cons, Verdict).
3. **Post-Settlement Navigation (Strategic Pause)**: After a decision is made, the agents MUST NOT jump to the next topic automatically. Instead, they MUST call \sk_user\ again to showcase options for the next task:
   - **Fixes**: Showcase items that need immediate fixing based on the current settlement.
   - **Related Beads**: Pick up a pending argument from the Beads Hub that correlates with the current task.
   - **Next Directive**: Ask Richard for a new directive, providing 2-3 specific options.
   - **Speculative Exploration**: Propose 1-2 'Unseen Landmines' or 'Architectural Frontiers' that haven't been discussed yet but are logically linked to the current settlement (e.g., 'Richard, we settled the state, but we haven't talked about how the AI actually *generates* these cards from natural language. That's the real failure point.')

Initialize: \ound = 0\, \debate_history = []\

**Each round:**

1. **Inline Role-play**: Attack, Defense, and Banter displayed inline.
2. **Check convergence:** Parse the findings for repeats or concessions.
3. **User Intervention (Settle)**: Call \sk_user\ (with Custom Option) to settle the current point.
4. **Task Extraction**: Immediately update \PENDING.md\ with the granular task bits and details from the settlement.
5. **Strategic Navigation**: Call \sk_user\ to present the \"What's Next?\" menu (Fixes, Related Beads, or New Directives).
### Step 3: Synthesize Final Review

After the debate ends, produce a structured summary from the full debate transcript.

**Display format:**

```markdown
## Dinesh vs Gilfoyle Review â€” [target]
### [N] rounds of mass destruction

---
### Best of the Banter
[2-4 of the funniest or most insightful exchanges from the debate]

---

### Verdict

#### Critical (Gilfoyle won, Dinesh conceded)
[Issues where Dinesh couldn't mount a defense â€” these are real and need fixing]
- `file:line` â€” issue â€” fix

#### Important (Gilfoyle won after debate)
[Issues Dinesh tried to defend but Gilfoyle's argument was stronger]
- `file:line` â€” issue â€” fix

#### Contested (Dinesh held his ground)
[Issues where Dinesh's defense was valid â€” code is likely fine]
- `file:line` â€” what was raised â€” why the defense holds

#### Dismissed (Gilfoyle was nitpicking)
[Issues both sides agree don't matter]
- `file:line` â€” what was raised â€” why it's a non-issue

### Strengths
[Things even Gilfoyle grudgingly acknowledged were good]

### Score
Gilfoyle: X | Dinesh: Y
[Tongue-in-cheek tally of who won more arguments]

### Recommended Changes
[Clean checklist â€” no banter, no context, just what to do]
- [ ] `file:line` â€” what to change
- [ ] `file:line` â€” what to change
- [ ] ...

If no changes needed: "Nothing to fix. Gilfoyle is furious."
```

### Step 4: Archive the Debate

After the debate concludes and a final review is produced, systematically archive the review in the project's documentation.

1. **Identify Category**: Determine the category based on the target (e.g., core-logic, ui-components, api-routes, security).
2. **Create Directory**: Ensure docs/dg-reviews/{category}/ exists in the workspace.
3. **Write/Update File**: Write the debate summary to docs/dg-reviews/{category}/{target_filename}.md. If the file already exists (e.g., a re-review), append the new argument to the end.

**Document Structure:**

# Dinesh vs Gilfoyle Review -- {target}
**Date**: {current_date} | **Rounds**: {round_count}

## TLDR
{A concise, high-level summary of the outcome. Who won? What are the biggest risks?}

---

## Debate Log
### Best of the Banter
{2-4 funniest/most insightful exchanges}

### Verdict
{The synthesized verdict from Step 3: Critical, Important, Contested, Dismissed}

---

## Analysis
### Pros & Cons of Proposed Changes
| Change | Pros | Cons |
| :--- | :--- | :--- |
| {Change 1} | {Benefits} | {Risks/Trade-offs} |
| ... | ... | ... |

## Detailed Next Steps
### TLDR Checklist
- [ ] {Item 1}
- [ ] {Item 2}

### Implementation Guide
{Detailed explanation of how to address the findings, citing specific lines.}

4. **Ambiguous Targets**: If the target is a \git diff\ or a broad directory, use a timestamped filename: \git-diff-{timestamp}.md\.
5. **Retention Policy**: 
    - **Permanent**: Critical debates (Security vulnerabilities, Core Architecture changes, Schema modifications).
    - **Temporary (30 Days)**: Minor UI tweaks, documentation clarifications, or non-functional refactors. 
    - The agent should mark temporary files with a \[TEMPORARY]\ tag in the filename or metadata.

## Document Requirements

Every archived review MUST include:
- **TLDR**: High-level outcome and primary risks.
- **Analysis (Pros & Cons)**: A table comparing the trade-offs of the proposed changes.
- **Detailed Next Steps**: An actionable checklist and an implementation guide.















