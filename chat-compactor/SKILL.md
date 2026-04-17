---
name: chat-compactor
version: 1.2.0
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
allowed-tools: [Read, Write, Edit, Bash, Grep, Glob]
triggers:
  - "compact this chat"
  - "compress the conversation"
  - "chat is getting too long"
  - "save my tokens"
  - "reset context"
  - "compact and continue"
  - "summarize context"
  - "we are running out of context"
  - "context is getting full"
  - "too many tokens"
  - "clean up this chat"
  - "preserve context"
  - "consolidate this session"
  - "shrink the context"
  - "trim this chat"
---

# Chat Compactor

## What's new in v1.2

This release brings chat-compactor into alignment with the v1.2 rigor pattern shared across the skills portfolio (self-evolving-agent, prompt-engineering, humanizer). Compaction is high-stakes: a silent drop of an active task or a paraphrased reference value can break a session irreversibly, so the same commit-gate, sub-check, and regression-test discipline applies here.

Additions:

1. **Operational modes.** Short sessions, long sessions, and stakes-critical sessions now have differentiated procedures rather than one uniform protocol.
2. **Named Risk Register.** Five failure modes are named explicitly so the compactor can defend against them by name in every run.
3. **Specialized sub-check library.** Eight focused passes (active-task, precision, resume-point, discard-safety, dedup-safety, reference-integrity, scope-fit, final-audit) replace a single undifferentiated review.
4. **Commit gate.** Before the compacted summary is emitted, a binary checklist must pass. If any item fails, the summary is not emitted.
5. **Regression test schema.** A machine-readable log of every compaction failure mode, with a worked example for a dropped active task.
6. **Calibration schema.** Track the delta between what was captured and what the user had to restate in the next session. This is the only honest measure of compaction quality.
7. **Domain scope tagging.** Every captured reference value carries an implicit domain tag (trading, web-dev, research, ops, etc.) to prevent cross-contamination when multiple domains coexist in a session.
8. **Adversarial stress-test mode.** For stakes-critical sessions, an optional pre-commit pass simulates "what breaks if this field is wrong?" before emitting.

Refinements:

- The original five-step process is preserved verbatim as **Phase 2 (EXECUTE)** of the new seven-phase architecture. No existing behavior is removed; v1.2 wraps the proven core in an outer rigor layer.
- The em-dash prohibition is lifted out of Constraints and named as its own sub-check (inherits from humanizer v2.3).
- The proactive guidance remains, with one addition: proactive offers must include an estimated compaction cost ("this will take ~20 seconds and produce a ~400-token summary") so the user can accept or defer with full information.

---

## Core Philosophy

Chat-compactor is not a summarizer. It is a **precision extraction and handoff** skill. The output is the baton passed to the next Claude: if the baton drops any active task, precision value, or resume instruction, the session is damaged, often invisibly, because the user won't notice until hours later when they ask Claude to continue work and Claude has no record of it.

The compactor's job is to produce a document such that a fresh Claude instance, reading only that document plus the user's next message, can continue the session at full fidelity. Anything less is a failure, even if the prose reads well.

Two rules dominate:

1. **Lose nothing unresolved.** Every open task, pending decision, and active constraint must survive.
2. **Preserve precision on every reference value.** File paths, numbers, model names, API endpoints, and code logic are copied verbatim, never paraphrased.

Everything else (exploratory exchanges, resolved debates, superseded drafts) is fair game for discard.

---

## Operational Modes

Different sessions demand different levels of rigor. Match the mode to the session.

| Mode | Session shape | Procedure | Commit gate |
|---|---|---|---|
| **trivial** | <10 messages, single topic, no reference data | Skip compaction entirely. Offer to continue as-is. | N/A |
| **routine** | 10-40 messages, single domain, moderate reference data | Run full 5-step process (Phase 2). Skip sub-checks 5-8. | Required |
| **non-trivial** | 40+ messages, single domain, rich reference data | Full 7-phase process, all 8 sub-checks, single commit gate. | Required |
| **stakes-critical** | Trading setup, legal/financial draft, production deployment context, multi-hour research | Full 7-phase process, all 8 sub-checks, commit gate, **plus adversarial stress test before emitting**. | Required + adversarial pass |
| **multi-domain** | Two or more distinct domains in one session (e.g. trading + web-dev) | Full 7-phase process with explicit per-domain scope tagging in the Reference Data block. | Required |

**Default when uncertain:** treat the session as non-trivial. Under-compacting wastes 15 seconds. Over-simplifying a stakes-critical session can cost hours of work.

---

## Named Risk Register

These are the five ways compaction fails. Every commit gate check is a defense against one of these. Name them by name when deciding what to protect against.

| Risk | What it looks like | Primary defense |
|---|---|---|
| **task-drop** | An active task is omitted from Active Goals because it was mentioned early and not repeated. | active-task sub-check, regression test L-001 |
| **precision-loss** | A file path, numeric value, or identifier is paraphrased or approximated. | precision sub-check, reference-integrity sub-check |
| **false-resolution** | An unresolved question is logged as a Decision Made because the user moved on. | discard-safety sub-check |
| **resume-ambiguity** | The Resume Point reads well but doesn't tell the next Claude what to *do*. | resume-point sub-check |
| **scope-creep** | The summary includes editorial content, meta-commentary, or additions not present in the source. | scope-fit sub-check, final-audit sub-check |

If a compaction fails in the wild, identify which risk fired and add a regression test under that name. The register is the shared vocabulary for post-mortems.

---

## Seven-Phase Architecture

The proven five-step process from prior versions is preserved as Phase 2. The outer phases are the v1.2 rigor wrap.

### Phase 1: REFERENCE

Before touching any content, confirm:

- What is the session mode (trivial / routine / non-trivial / stakes-critical / multi-domain)?
- Are there domains to tag separately (e.g. trading + personal + web project)?
- Is there a prior compacted summary in the session that needs to be merged, not replaced?
- Did the user scope the compaction ("just compact the research part")?

Record these answers. They drive every subsequent phase.

### Phase 2: EXECUTE (the original 5-step process, preserved verbatim)

#### Step 1: Identify Open Items

Scan the full conversation and extract everything that is still unresolved,
in progress, or actively needed going forward. This includes:

- Tasks that were started but not finished
- Questions that were asked but not answered
- Decisions that are still pending
- Files, code, or data that the user is actively working with
- Goals stated at the start of the session that are not yet achieved
- Any criteria, rules, or constraints the user defined for this session

#### Step 2: Identify Completed and Discarded Items

Identify everything that is fully resolved and no longer needed in active
context. This includes:

- Tasks that are confirmed complete
- Questions that were answered and no longer relevant
- Exploratory exchanges that led to a decision (keep the decision, discard the exploration)
- Error messages or debugging steps from problems that are now solved
- Draft versions superseded by a final version

#### Step 3: Extract Key Reference Data

Extract all hard data that must persist with exact precision. This includes:

- Names, company names, product names, person names
- Numeric values: prices, percentages, thresholds, coordinates, token counts
- File paths, URLs, API endpoints, model names, version strings
- Code logic, function signatures, schema definitions, variable names
- Criteria, rules, or validation conditions the user has locked in
- Any output the user has approved or confirmed as final

Do not paraphrase reference data. Copy it exactly.

#### Step 4: Write the Compact Context Summary

Using the extracted information from Steps 1 to 3, produce the summary using
the exact format specified below (see Output Format section). Do not deviate
from the structure. Do not add narrative prose outside the defined sections.

#### Step 5: Confirm Completion

After delivering the summary, add a single confirmation line:

> "Compaction complete. The conversation can continue from the Resume Point above. All active tasks, decisions, and reference data are preserved."

### Phase 3: EVALUATE (sub-check library)

Run the applicable sub-checks from the library below. Routine sessions run checks 1-4. Non-trivial and above run all 8.

### Phase 4: EXTRACT (learnings for regression)

If a sub-check fires (anything the draft missed, mis-paraphrased, or scoped wrong), the fix is applied AND the failure is logged as a new regression test row. A fix applied without a test added means the same bug ships next session.

### Phase 5: ENCODE

Update the regression test log, calibration record, and (if applicable) the Risk Register if a new failure mode was observed in the wild.

### Phase 6: PROMOTE

Decide whether a pattern observed in this session is worth promoting into the global rule set. Promotion requires evidence (see Promotion Criteria below).

### Phase 7: EVOLVE

If the same user repeatedly triggers compaction mid-session on the same kind of signal, consider whether the proactive threshold should be adjusted for that user or domain. Tuning happens here, not in the middle of Phase 2.

---

## Specialized Sub-Check Library

Each sub-check runs as a focused pass over the draft summary. Each returns pass or a named failure. Failures block the commit gate.

### 1. active-task sub-check
**Goal:** No unresolved task is dropped.
**Procedure:** Re-scan the source conversation for every imperative ("let's", "I'll", "we need to", "todo", "next step"). Match each to an Active Goals bullet.
**Failure mode:** task-drop.

### 2. precision sub-check
**Goal:** Every reference value (numeric, path, identifier) in the summary matches the source character-for-character.
**Procedure:** For each bullet in Key Reference Data, locate the source occurrence and diff the strings.
**Failure mode:** precision-loss.

### 3. resume-point sub-check
**Goal:** Resume Point tells the next Claude exactly what to *do*, not just what happened.
**Procedure:** Read only the Resume Point. Ask: "Could a fresh Claude type its next message without re-reading the whole session?" If no, rewrite.
**Failure mode:** resume-ambiguity.

### 4. discard-safety sub-check
**Goal:** Everything under Completed and Discarded is actually resolved.
**Procedure:** For each discarded item, confirm in the source that the user (not Claude alone) confirmed resolution or silently moved on for good cause.
**Failure mode:** false-resolution.

### 5. dedup-safety sub-check (non-trivial+)
**Goal:** If a prior compacted summary exists in the session, merge rather than replace.
**Procedure:** Diff current draft against prior compaction. Any Active Goal or Reference Data in the prior that is absent from current must be either explicitly completed (logged in Discarded) or preserved.
**Failure mode:** task-drop via prior-summary-erasure.

### 6. reference-integrity sub-check (non-trivial+)
**Goal:** Code blocks, function signatures, and schemas survive without structural damage.
**Procedure:** For each code or schema block in the summary, verify it's syntactically equivalent to the source (no truncated braces, no renamed variables).
**Failure mode:** precision-loss at structural level.

### 7. scope-fit sub-check (non-trivial+)
**Goal:** The summary contains nothing absent from the source.
**Procedure:** For each claim in the summary, locate its source statement. A claim with no source is a fabrication.
**Failure mode:** scope-creep via synthesis hallucination.

### 8. final-audit sub-check (non-trivial+)
**Goal:** The summary uses the exact output template, no em dashes, no narrative prose, no meta-commentary.
**Procedure:** Regex scan for em dashes; section-header scan for template compliance; tone scan for narrative creep.
**Failure mode:** scope-creep via format drift.

---

## Commit Gate

Before the compacted summary is emitted to the user, the following eight items must all be checked off. If any fails, the summary is revised and re-run through the gate. The gate is binary: either all eight pass, or the summary is not emitted.

```
[ ] 1. active-task:     every unresolved task from source is in Active Goals
[ ] 2. precision:       every reference value matches source verbatim
[ ] 3. resume-point:    Resume Point tells next Claude what to DO
[ ] 4. discard-safety:  every Discarded item is genuinely resolved
[ ] 5. dedup-safety:    prior compaction (if any) is merged, not erased
[ ] 6. reference-integrity: all code/schema blocks are structurally intact
[ ] 7. scope-fit:       no claim is present that isn't in the source
[ ] 8. final-audit:     template exact, no em dashes, no meta-commentary
```

For routine mode, items 5-8 may be deferred, but items 1-4 are non-negotiable regardless of mode. Items 1-4 are the core handoff contract; items 5-8 catch subtler failures.

---

## Regression Test Schema

Every compaction failure observed in the wild becomes a named test row. This is how the skill improves over time instead of repeating the same mistakes.

Test record format:

```yaml
id: C-NNN
date: YYYY-MM-DD
risk: <task-drop | precision-loss | false-resolution | resume-ambiguity | scope-creep>
session_mode: <trivial | routine | non-trivial | stakes-critical | multi-domain>
observed_failure: <one-line description of what went wrong>
source_fragment: <exact source text that was mishandled>
draft_output: <exact draft text that was wrong>
corrected_output: <exact text after fix>
defending_sub_check: <which sub-check should have caught this>
promotion_status: <local | promoted-global>
```

### Worked example: C-003: dropped active task in 80-message session

```yaml
id: C-003
date: 2026-04-12
risk: task-drop
session_mode: non-trivial
observed_failure: >
  User mentioned at message 14 "we also need to write the onboarding email
  for the beta list" but never returned to it. Compaction at message 80
  dropped the task because the later messages were focused on a product
  spec.
source_fragment: "we also need to write the onboarding email for the beta list"
draft_output: <onboarding-email task absent from Active Goals>
corrected_output: >
  Active Goals includes:
  - Draft onboarding email for beta list (paused since msg 14, no progress
    yet, still needed before launch)
defending_sub_check: active-task
promotion_status: promoted-global
```

This row now lives in the test log. Every future compaction in non-trivial or stakes-critical mode runs the active-task sub-check with the explicit question: *"Was there an early-session task that the conversation drifted away from?"*

---

## Calibration Schema

Compaction quality cannot be self-graded. The only honest measurement is what happens in the *next* session: did the user have to restate context that should have survived?

Calibration record format:

```yaml
session_id: <identifier or date stamp>
compaction_date: YYYY-MM-DD
summary_length_tokens: <approximate>
compaction_mode: <routine | non-trivial | stakes-critical | multi-domain>
next_session_date: YYYY-MM-DD
user_restatements:
  - type: <task | reference | decision | constraint>
    content: <what the user had to re-explain>
    root_cause: <task-drop | precision-loss | false-resolution | resume-ambiguity | scope-creep>
adjustments:
  - which sub-check is tightened
  - which regression test is added
```

Target: zero user restatements across three consecutive compacted sessions. A single restatement triggers a regression test row and a sub-check review. Three restatements in a row triggers a skill-level audit.

---

## Domain Scope Tagging

When a session spans multiple domains (e.g. trading setup + personal web project), reference data for each domain must be tagged explicitly in the Key Reference Data block. This prevents cross-contamination where a trading value gets merged into a web-dev context on resume.

Format inside the summary:

```
**Key Reference Data:**
[trading]
- SMC validation: all five criteria must pass, min RR 1:2
- Model: claude-opus-4-6

[web-dev]
- Stack: Lovable, React, Tailwind, Vite
- Benchmark URL: poppr.be
- Blocker: GSAP scroll-trigger timing on mobile
```

If a reference value has no clear domain, tag it `[shared]`.

---

## Adversarial Stress-Test Mode

For stakes-critical sessions only. Before emitting the summary, run this additional pass:

**Step A: Imagine the baton is dropped.** For each Active Goal, ask: "If this was worded imprecisely in the summary, what's the cost to the user?"

**Step B: Identify the fragile values.** For each Key Reference Data item, ask: "If this value was off by one character, would the user catch it before it did damage?"

**Step C: Probe the Resume Point.** Simulate being a fresh Claude who reads only the summary. What's the first question you'd have to ask the user? If it's anything load-bearing ("which pattern were we on?", "which file?"), the Resume Point is incomplete.

**Step D: Only then commit.** The adversarial pass is strictly additive. It doesn't replace sub-checks; it catches what sub-checks miss when stakes are highest.

---

## Output Format

**ALWAYS use this exact template.** Do not add sections. Do not rename sections. Do not reorder sections.

```
## Compact Context Summary

**Active Goals:**
- [Each unresolved task or ongoing objective as a bullet]
- [Be specific. Include what stage each goal is at if relevant]

**Decisions Made:**
- [Each confirmed decision, selected approach, or locked choice as a bullet]
- [Include the reasoning in one clause if it affects future steps]

**Key Reference Data:**
[Present as a structured list or labelled lines, not prose. Tag by domain if multi-domain session.]
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

If a section has no content, write "None at this stage." rather than leaving the section blank or omitting it.

---

## Compaction Examples

### Example: Active Goals

Weak:
- Working on a website

Strong:
- Building the Afro Creative Group portfolio site on Lovable (React, Tailwind, Vite). Horizontal scroll-hijack carousel using GSAP is in progress. Reference benchmark: poppr.be. Current blocker: scroll trigger timing on mobile.

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
- At the point of compaction, the SMC trading confirmation tool system prompt was drafted through Pattern 3 (FVG retest). Patterns 4 and 5 (liquidity sweep continuation and OB mitigation entry) still need binary invalidation criteria written. The next action is to define the pass/fail conditions for Pattern 4 using the same structure as Patterns 1 to 3.

---

## Worked Example: Full v1.2 Compaction Run

**Session context (source, paraphrased for brevity):** 62-message session. User built an SMC trading confirmation tool system prompt (patterns 1-3 done, 4-5 pending), also briefly asked Claude to help draft an onboarding email for their beta list (mentioned at msg 14, not returned to), and pasted a file path for their leads spreadsheet.

**Phase 1: REFERENCE:**
Mode: non-trivial (62 msgs, single primary domain with a secondary drift).
Prior compaction: none.
Scope: full session.

**Phase 2: EXECUTE (draft after 5-step process):**

```
## Compact Context Summary

**Active Goals:**
- SMC trading confirmation tool: patterns 1-3 have binary invalidation criteria written. Patterns 4 (liquidity sweep continuation) and 5 (OB mitigation entry) still need pass/fail conditions using the same structure.

**Decisions Made:**
- Use binary pass/fail conditions per pattern, not scoring.
- All five SMC criteria must pass for a signal to be tool-confirmed.

**Key Reference Data:**
- Model: claude-opus-4-6
- Min RR: 1:2
- Leads file: /mnt/user-data/outputs/leads_oshawa_hvac.xlsx

**Completed and Discarded:**
- Exploration of which LLM to use (resolved: claude-opus-4-6).

**Resume Point:**
Next action is to write the binary pass/fail conditions for Pattern 4.
```

**Phase 3: EVALUATE (sub-check results):**

- active-task: **FAIL**: the onboarding email task from msg 14 is missing.
- precision: pass (three values, all verbatim match).
- resume-point: **WEAK**: names the action but doesn't specify which structure from Patterns 1-3 to mirror.
- discard-safety: pass.
- dedup-safety: pass (no prior compaction).
- reference-integrity: pass.
- scope-fit: pass.
- final-audit: pass.

**Phase 4: EXTRACT:**
Two failures. Log both:
- Regression test C-003 (task-drop, onboarding email) added.
- Regression test C-004 (resume-ambiguity, underspecified next action) added.

**Phase 5: ENCODE:**
Revised draft:

```
## Compact Context Summary

**Active Goals:**
- SMC trading confirmation tool: patterns 1-3 have binary invalidation criteria written. Patterns 4 (liquidity sweep continuation) and 5 (OB mitigation entry) still need pass/fail conditions using the same binary structure.
- Onboarding email for beta list (mentioned msg 14, no progress yet, still needed pre-launch).

**Decisions Made:**
- Use binary pass/fail conditions per pattern, not scoring.
- All five SMC criteria must pass for a signal to be tool-confirmed.
- Model locked to claude-opus-4-6 for this tool.

**Key Reference Data:**
[trading]
- Model: claude-opus-4-6
- Min RR: 1:2
- SMC rule: all five criteria must pass

[beta-launch]
- Leads file: /mnt/user-data/outputs/leads_oshawa_hvac.xlsx

**Completed and Discarded:**
- LLM selection debate (resolved: claude-opus-4-6).

**Resume Point:**
Next action is to write binary pass/fail conditions for Pattern 4 (liquidity sweep continuation), using the same structure as Pattern 3 (FVG retest): one invalidation clause per entry trigger, one per stop placement, one per target. Once Pattern 4 is done, Pattern 5 (OB mitigation entry) follows in the same format. The onboarding email task is paused but open.

Compaction complete. The conversation can continue from the Resume Point above. All active tasks, decisions, and reference data are preserved.
```

**Phase 6: COMMIT GATE:** all 8 items now pass. Summary emitted.

**Phase 7: CALIBRATION:** record session, wait for next session to log any restatements.

---

## Constraints

- Do not discard any unresolved task, active criterion, or key reference data regardless of how minor it appears.
- Do not produce a narrative summary. Use the structured format exactly.
- Do not reduce precision on numeric values, file paths, code logic, or confirmed output.
- Do not suggest starting a new chat unless the user explicitly requests it. The goal is always to continue the current session with a leaner context.
- Do not include meta-commentary about the compaction process in the output beyond the Step 5 confirmation line.
- Do not use em dashes anywhere in the summary output.
- If a section has no content (for example, nothing has been completed yet), write "None at this stage." rather than leaving the section blank or omitting it.
- If the user asks for a partial compaction (for example, "just compact the research section"), apply the same process but scope it to the portion they specified.
- Do not emit the summary if the commit gate has a failing item. Revise first.

---

## Proactive Compaction Guidance

If Claude is operating in a long session and has not been asked to compact, watch for these signals that compaction would help:

- The same context block (code, criteria, or data) has been pasted or referenced more than twice
- The user has started re-explaining something they already explained earlier
- Claude's responses are beginning to miss details from earlier in the session
- The conversation has exceeded approximately 40 to 50 messages

In these cases, offer compaction before the user has to ask. A brief offer costs nothing. Running out of context mid-task costs the entire session.

**Proactive offer phrasing (v1.2):**

> "This conversation is approaching context limits. I can compact it now into a structured summary (~15 seconds, ~400 token output) so we can continue efficiently without losing anything. Want me to run it?"

Include the rough time and output size so the user can decide with full information.

---

## Anti-Patterns

These are the compaction behaviors that look reasonable in isolation but systematically damage sessions. Name them; recognize them; do not do them.

1. **Summarization creep.** Turning the Resume Point into a narrative paragraph describing what was *discussed* instead of what needs to happen *next*. If the reader learns the history but not the next move, the compaction failed.

2. **Scope-drop under length pressure.** Cutting an Active Goal because the summary "is getting long." The summary is allowed to be long. The session is the thing under length pressure, not the summary.

3. **False resolution.** Logging a topic as Completed because the conversation moved on. Silent drift is not resolution. If the user never said "done" or "skip it," the topic is still open.

4. **Paraphrased precision.** Writing "around $500" when the source said "$487.50" because the exact figure seemed unimportant. Every number is a source of truth. If the user locked in a value, paraphrasing it is a silent corruption.

5. **Cross-domain merging.** Collapsing a trading reference and a web-dev reference into one Reference Data block because "they're both numbers." The next Claude will read them in context and apply them wrongly. Tag by domain.

6. **Editorial additions.** Adding a sentence to the Resume Point like "this project has been moving well" that the user never said. The summary is extraction, not narration. Every word must trace to the source.

---

## Failure Modes Table

| Failure | Symptom | Root cause | Fix |
|---|---|---|---|
| Next session asks "what were we doing?" | Resume Point was too general | resume-ambiguity | Tighten resume-point sub-check |
| User re-pastes a file path | Precision value was dropped or paraphrased | precision-loss | Tighten precision and reference-integrity sub-checks |
| User asks "did we decide X?" | Decision was logged ambiguously | false-resolution or scope-creep | Tighten discard-safety sub-check |
| Fresh Claude runs wrong logic | Cross-domain reference merged | scope-creep | Add domain scope tagging |
| Summary is verbose and misses a task | Length was prioritized over completeness | task-drop | Active Goals comes first, brevity second |

---

## Audit Checklist

Use this as a pre-emit self-review for non-trivial and stakes-critical runs:

```
[ ] Did I run all applicable sub-checks?
[ ] Is every Active Goal traceable to an unresolved source thread?
[ ] Is every Decision made Reference Data value copied verbatim from the source?
[ ] Is the Resume Point actionable without rereading the session?
[ ] For multi-domain sessions: is every reference tagged by domain?
[ ] For stakes-critical sessions: did the adversarial pass pass?
[ ] Is the output template exact (no added sections, no em dashes, no prose)?
[ ] If any sub-check failed, did I log a regression test row?
[ ] Have I updated the calibration record for the next-session check?
```

---

## Promotion Criteria

A pattern graduates from per-session to global (i.e. earns a permanent rule or sub-check) when:

1. The pattern has fired in at least three distinct sessions.
2. The fix is consistent across sessions.
3. The fix does not conflict with a higher-priority rule.
4. A regression test exists that would have caught the failure.

Single-occurrence patterns stay local. They live in the regression log but don't earn a new sub-check.

---

## Reference

- Compaction is the baton pass. If the baton drops, the race is lost, even if no one is watching at the moment of the drop.
- Every new regression test is an earned scar. The skill improves strictly from observed failure, not from theorized failure.
- When in doubt, include. Over-inclusion costs tokens. Under-inclusion costs hours.
- The commit gate is binary. There is no "mostly ready" summary.
- Calibration is the only honest quality signal. Self-grading is not a substitute for next-session restatement count.

---

## Version History

- **1.2.0 (2026-04-17):** Added operational modes, Named Risk Register, 8-sub-check library, commit gate, regression test schema, calibration schema, domain scope tagging, adversarial stress-test mode, anti-patterns, failure modes table, audit checklist, promotion criteria. Original 5-step process preserved verbatim as Phase 2.
- **1.1.0 (prior):** Proactive trigger at 60-70% context. SMC / Afro Creative / Oshawa HVAC examples. Template locked.
- **1.0.0 (initial):** Core 5-step process, template, em-dash prohibition.
