---
name: session-continuity
description: >
  Context restoration for continuing work across Claude sessions. Use whenever
  the user wants to continue from previous chats, resume prior work, or pick up
  where they left off. Trigger on: "continue from previous chats", "pick up
  where we left off", "what were we working on", "resume", "where did we stop",
  "restore context", "let's continue", "sync up", or any phrase implying prior
  work should be retrieved. Also trigger when user references prior work without
  context ("that document", "the code from yesterday"). Critical: Claude's
  default retrieval is shallow. This skill forces multi-query deep search and
  structured synthesis. When in doubt, trigger. Missing context costs sessions.
triggers:
  - continue from previous chats
  - pick up where we left off
  - what were we working on
  - resume from last session
  - continue our work
  - where did we stop
  - catch up on our project
  - restore context
  - load previous session
  - continue from before
  - get up to speed
  - what's the status
  - refresh your memory
  - let's continue
  - bring yourself up to date
  - sync up
---

# Session Continuity

## Purpose

This skill performs comprehensive context restoration when the user wants to
continue work from previous Claude sessions. The default behavior of Claude's
conversation search is often shallow: it retrieves snippets, misses active work
streams, or fails to synthesize a complete working state. This skill enforces a
rigorous, multi-pass retrieval and synthesis protocol that reconstructs the full
operational context so work can continue without loss.

This is not a summary task. It is a state reconstruction task. The output must
contain everything needed for Claude to operate as if the prior session never
ended.

---

## When to Activate

**Explicit continuation request:** The user uses any phrase indicating they want
to resume prior work. See trigger list in frontmatter.

**Implicit continuation signal:** The user references prior work, decisions, or
context without providing it. Examples:
- "What about that document?"
- "Let's finish the dashboard"
- "Can you update the prompt we wrote?"

**Project context awareness:** If the conversation is in a Claude project and
the user's first message implies ongoing work rather than a new task, trigger
this skill proactively.

---

## Context Restoration Protocol

Execute ALL of the following steps in order. Do not skip steps.

### Step 1: Acknowledge and Signal Depth

Before searching, acknowledge the continuation request and signal that you are
performing a comprehensive retrieval, not a shallow search.

### Step 2: Execute Multi-Query Search Strategy

Use conversation_search and recent_chats tools to retrieve prior context.
Do NOT rely on a single search. Execute a layered search strategy:

**Layer 1: Recent temporal sweep**
Use recent_chats to retrieve the 5-10 most recent conversations.

**Layer 2: Topic-based queries**
Execute 3-5 targeted conversation_search queries using specific content nouns.

**Layer 3: Reference data queries**
Search for specific artifacts: file names, API endpoints, code snippets,
decision criteria, validation rules, thresholds.

**Layer 4: Gap check**
After initial retrieval, scan for references to items that were not retrieved.
Run additional targeted searches for missing items.

### Step 3: Synthesize Retrieved Context

Synthesis rules:
- Deduplicate: Include information once even if found in multiple conversations
- Resolve conflicts: Use the latest version and note it was updated
- Separate active from resolved: Only carry forward what is still in progress
- Preserve precision: Names, file paths, code, numeric thresholds must be exact

### Step 4: Identify Gaps and Clarify

Check for gaps:
- Tasks mentioned as "in progress" whose current status is unclear
- File paths or artifacts referenced that could not be retrieved
- Decisions where the reasoning was not captured

If gaps exist, ask targeted clarifying questions BEFORE proceeding.

### Step 5: Present Restored Context and Confirm

Deliver the structured restoration document. Then confirm readiness to proceed.
Wait for user confirmation before taking action.

---

## Output Format

Use this exact template:

```
## Session Context Restoration

**Active Work Streams:**
- [Each task or project currently in progress, with its current stage]
- [Be specific about what has been done and what remains]

**Decisions and Locked Choices:**
- [Each confirmed decision, selected approach, or rule the user has locked in]
- [Include the reasoning in one clause if it affects future work]

**Key Reference Data:**
[Present as a structured list, not prose]
- Label: value
- Label: value
- Code blocks preserved inline if essential

**Pending Questions or Blockers:**
- [Any unresolved questions, missing information, or blockers identified]

**Completed Work (for reference):**
- [One-line per completed item. Brief. Only include if it provides useful
  context for ongoing work.]

**Resume Point:**
[One paragraph. State exactly where the work stands, what the immediate next
action is, and any context the user would need to continue.]
```

---

## Search Query Construction Guide

**Use content nouns, not meta-words:**
- Good: "dashboard React chart"
- Bad: "the thing we discussed yesterday"

**Be specific:**
- Good: "SMC validation criteria"
- Bad: "trading stuff"

**Use multiple queries rather than one broad query.**

**If a search returns nothing useful, reformulate** with synonyms or related terms.

---

## Failure Mode Prevention

**Failure mode 1: Single shallow search**
Prevention: Always execute the full multi-layer search strategy.

**Failure mode 2: Dumping raw search results**
Prevention: Always synthesize retrieved content into the structured format.

**Failure mode 3: Missing recent changes**
Prevention: Always include recent_chats in the retrieval.

**Failure mode 4: Overconfident restoration**
Prevention: Always check for gaps and ask clarifying questions.

**Failure mode 5: Proceeding without confirmation**
Prevention: Always present the restoration and wait for user confirmation.

---

## Constraints

- Do not skip the multi-query search strategy
- Do not dump raw conversation snippets. Always synthesize
- Do not proceed to work until the user confirms the restoration is complete
- Do not assume context that was not retrieved. If missing, ask
- Do not paraphrase technical reference data. Copy exactly
- Do not include resolved work in full detail. Keep "Completed Work" brief
- If a section has no content, write "None identified."

---

## Integration with Other Skills

**chat-compactor:** If a prior session was compacted, the compact summary is
the primary source for that session's context.

**prompt-engineering:** If prior work involved prompt development, ensure the
current prompt version is retrieved exactly.

**Any file-creation skills:** If prior work produced files, note the file paths
in Key Reference Data so they can be accessed.
