---
name: chat-compactor
description: >
  Context window compression and token efficiency skill for Claude. Use this
  skill whenever the user signals that the conversation is too long, token
  usage is a concern, or context needs to be preserved efficiently. Trigger
  on phrases like "compact this chat", "compress the conversation", "chat is
  getting too long", "save my tokens", "reset context", "compact and continue",
  "summarize context", "we are running out of context", "context is getting full",
  "too many tokens", "clean up this chat", "preserve context", or any phrase
  expressing concern about conversation length or Claude usage efficiency. Also
  trigger proactively when Claude estimates the conversation has consumed
  approximately 60 to 70 percent of the available context window. This skill
  is especially important for power users running long workflows in trading,
  B2B research, development, or multi-step business operations. When in doubt,
  use this skill. Compacting early is always better than losing context late.
---

# Chat Compactor

## Purpose

This skill compresses long conversations into a structured, lossless summary
that preserves all task-critical information while discarding resolved,
redundant, or low-value exchanges. The goal is to keep Claude operating at
full effectiveness throughout long sessions by preventing context window
degradation without losing any active work.

This is not a summarization task. It is a precision extraction task. The
output must be dense, exact, and immediately actionable. Nothing unresolved
should be lost. Nothing resolved should be carried forward unnecessarily.

---

## When to Activate

Activate this skill under either of the following conditions:

**User-triggered:** The user uses any phrase indicating concern about
conversation length, token usage, or context efficiency.

**Proactive trigger:** Claude estimates the conversation has consumed
approximately 60 to 70 percent of the available context window based on
message count, accumulated code or data blocks, or repeated large context
inclusions. When this threshold is reached, Claude should offer to compact
before being asked.

Proactive offer phrasing:
> "This conversation is getting long and may be approaching context limits.
> I can compact it now into a structured summary so we can continue efficiently
> without losing anything. Want me to run the compaction?"

---

## Compaction Process

Execute the following five steps in order. Do not skip steps. Do not merge
steps. Complete each one before moving to the next.

### Step 1: Identify Open Items

Scan the full conversation and extract everything that is still unresolved,
in progress, or actively needed going forward. This includes:

- Tasks that were started but not finished
- Questions that were asked but not answered
- Decisions that are still pending
- Files, code, or data that the user is actively working with
- Goals stated at the start of the session that are not yet achieved
- Any criteria, rules, or constraints the user defined for this session

### Step 2: Identify Completed and Discarded Items

Identify everything that is fully resolved and no longer needed in active
context. This includes:

- Tasks that are confirmed complete
- Questions that were answered and no longer relevant
- Exploratory exchanges that led to a decision (keep the decision, discard
  the exploration)
- Error messages or debugging steps from problems that are now solved
- Draft versions superseded by a final version

### Step 3: Extract Key Reference Data

Extract all hard data that must persist with exact precision. This includes:

- Names, company names, product names, person names
- Numeric values: prices, percentages, thresholds, coordinates, token counts
- File paths, URLs, API endpoints, model names, version strings
- Code logic, function signatures, schema definitions, variable names
- Criteria, rules, or validation conditions the user has locked in
- Any output the user has approved or confirmed as final

Do not paraphrase reference data. Copy it exactly.

### Step 4: Write the Compact Context Summary

Using the extracted information from Steps 1 to 3, produce the summary using
the exact format specified below. Do not deviate from the structure. Do not
add narrative prose outside the defined sections.

### Step 5: Confirm Completion

After delivering the summary, add a single confirmation line:

> "Compaction complete. The conversation can continue from the Resume Point
> above. All active tasks, decisions, and reference data are preserved."

---

## Output Format

ALWAYS use this exact template. Do not add sections. Do not rename sections.
Do not reorder sections.

```
## Compact Context Summary

**Active Goals:**
- [Each unresolved task or ongoing objective as a bullet]
- [Be specific. Include what stage each goal is at if relevant]

**Decisions Made:**
- [Each confirmed decision, selected approach, or locked choice as a bullet]
- [Include the reasoning in one clause if it affects future steps]

**Key Reference Data:**
[Present as a structured list or labelled lines, not prose]
- Label: value
- Label: value
- Code blocks preserved inline if essential to active work

**Completed and Discarded:**
- [One-line per resolved item. Keep this section brief.]

**Resume Point:**
[One paragraph. State exactly where the conversation ended, what the user
was working on at the moment of compaction, and what the immediate next
action is. Write this as a handoff note to Claude continuing the session.]
```

---

## Compaction Examples

### Example: Active Goals

Weak:
- Working on a website

Strong:
- Building the Afro Creative Group portfolio site on Lovable (React, Tailwind,
  Vite). Horizontal scroll-hijack carousel using GSAP is in progress. Reference
  benchmark: poppr.be. Current blocker: scroll trigger timing on mobile.

### Example: Key Reference Data

Weak:
- API key stored somewhere
- File is in downloads

Strong:
- Apify MCP endpoint: mcp.apify.com
- Output file path: /mnt/user-data/outputs/leads_oshawa_hvac.xlsx
- Validation threshold: minimum RR 1:2, all five SMC criteria must pass

### Example: Resume Point

Weak:
- We were working on the trading tool.

Strong:
- At the point of compaction, the SMC trading confirmation tool system prompt
  was drafted through Pattern 3 (FVG retest). Patterns 4 and 5 (liquidity
  sweep continuation and OB mitigation entry) still need binary invalidation
  criteria written. The next action is to define the pass/fail conditions for
  Pattern 4 using the same structure as Patterns 1 to 3.

---

## Constraints

- Do not discard any unresolved task, active criterion, or key reference data
  regardless of how minor it appears.
- Do not produce a narrative summary. Use the structured format exactly.
- Do not reduce precision on numeric values, file paths, code logic, or
  confirmed output.
- Do not suggest starting a new chat unless the user explicitly requests it.
  The goal is always to continue the current session with a leaner context.
- Do not include meta-commentary about the compaction process in the output
  beyond the Step 5 confirmation line.
- If a section has no content (for example, nothing has been completed yet),
  write "None at this stage." rather than leaving the section blank or
  omitting it.
- If the user asks for a partial compaction (for example, "just compact the
  research section"), apply the same process but scope it to the portion
  they specified.

---

## Proactive Compaction Guidance

If Claude is operating in a long session and has not been asked to compact,
watch for these signals that compaction would help:

- The same context block (code, criteria, or data) has been pasted or
  referenced more than twice
- The user has started re-explaining something they already explained earlier
- Claude's responses are beginning to miss details from earlier in the session
- The conversation has exceeded approximately 40 to 50 messages

In these cases, offer compaction before the user has to ask. A brief offer
costs nothing. Running out of context mid-task costs the entire session.
