---
name: session-continuity
version: 1.2.0
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
allowed-tools: [Read, Write, Edit, Grep, Glob, Bash, mcp__session_info__list_sessions, mcp__session_info__read_transcript]
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
  - continue where we left off
  - load context
  - restore session
  - the work we did
  - that thing we discussed
  - from our last chat
---

# Session Continuity

## What's new in v1.2

This release aligns session-continuity with the v1.2 rigor pattern used across the skills portfolio (self-evolving-agent, prompt-engineering, humanizer, chat-compactor). Context restoration sits at the same stakes level as compaction: a silent miss in retrieval propagates into every subsequent Claude response in the session, and the user usually doesn't notice until work has to be redone.

Additions:

1. **Operational modes.** One-topic resumptions, multi-thread projects, and long-gap resumptions now follow differentiated procedures instead of a uniform protocol.
2. **Named Risk Register.** Six failure modes are named so the restoration can defend against each by name.
3. **Specialized sub-check library.** Eight focused passes (completeness, precision, gap, resume, staleness, conflict-resolution, scope-fit, confirmation-readiness) replace a single vague review.
4. **Commit gate.** Before the restoration document is delivered to the user, a binary checklist must pass. If any item fails, the restoration is not delivered.
5. **Regression test schema.** Every observed retrieval failure becomes a named test row with a worked example.
6. **Calibration schema.** Track the delta between restored context and what the user had to re-state after restoration. This is the only honest measure of restoration quality.
7. **Domain scope tagging.** Multi-domain projects (e.g. trading + web-dev + outreach) are tagged separately in the restoration to prevent cross-contamination.
8. **Adversarial stress-test mode.** For long-gap or high-stakes resumptions, simulate "what breaks if this retrieval is incomplete?" before delivering.

Refinements:

- The original 4-layer retrieval protocol is preserved verbatim as **Phase 2 (EXECUTE)** of the new seven-phase architecture. Nothing existing is removed.
- The em-dash prohibition is lifted out of Constraints and named as its own sub-check item.
- Confirmation step now includes an explicit invitation for the user to add context that retrieval could not surface.

---

## Core Philosophy

Session-continuity is not a memory refresh. It is a **state reconstruction** skill. The output is the full working state of the user's prior sessions, expressed in a form that lets a fresh Claude operate as if the prior sessions never ended.

Two rules dominate:

1. **Retrieval must be deep, not default.** Claude's default retrieval behavior is shallow: it grabs what the user's current message keywords hit, stops there, and produces a plausible-looking but incomplete restoration. This skill exists specifically to break that default.
2. **Synthesis must be structured, not narrative.** A pile of snippets, however well-retrieved, does not restore context. The snippets must be assembled into the output template with deduplication, conflict resolution, and precision preservation.

If the user has to re-state anything that was captured in prior sessions, the restoration failed, even if the document reads well.

---

## Operational Modes

Different resumption scenarios demand different levels of rigor.

| Mode | Scenario | Procedure | Commit gate |
|---|---|---|---|
| **quick-reload** | User explicitly references a single recent thread ("the Hex dashboard we built yesterday") and mentions specific artifacts | Phase 2 layers 1-2 only. Skip sub-checks 5-8. | Required |
| **standard** | User asks to continue, project has 1-3 active work streams, gap of hours to days | Full 7-phase process, all 8 sub-checks | Required |
| **long-gap** | User hasn't worked with Claude in this project for >2 weeks, or references "from a while back" | Full 7-phase process, all 8 sub-checks, staleness sub-check tightened, explicit verification of each restored item | Required |
| **multi-thread** | Project has 4+ active work streams, or user is resuming mid-pivot | Full 7-phase process, all 8 sub-checks, per-thread scope tagging, per-thread resume points | Required |
| **stakes-critical** | Resumption is precursor to a live execution (trading setup, deployment, sending outreach, legal filing) | Full 7-phase process, all 8 sub-checks, commit gate, **plus adversarial stress test before delivering** | Required + adversarial pass |

**Default when uncertain:** treat as standard. Quick-reload is only appropriate when the user's trigger language is unambiguous about scope.

---

## Named Risk Register

These are the six ways restoration fails. Every commit gate check defends against one of these by name.

| Risk | What it looks like | Primary defense |
|---|---|---|
| **shallow-retrieval** | Only the most recent thread is surfaced; older active work streams are missed. | Layered search (Phase 2), completeness sub-check |
| **stale-as-current** | A decision that was later revised appears in the restoration as the active version. | staleness sub-check, conflict-resolution sub-check |
| **precision-drift** | A file path, numeric value, or identifier is paraphrased instead of copied verbatim. | precision sub-check |
| **phantom-certainty** | Restoration states something as known when it was actually ambiguous or unretrieved. | gap sub-check, confirmation-readiness sub-check |
| **thread-collision** | Reference data from one domain is logged under another domain. | scope-fit sub-check, domain scope tagging |
| **resume-vagueness** | Resume Point reads well but doesn't tell the next Claude what to *do*. | resume sub-check |

If a restoration fails in a session, identify which risk fired and log a regression test row under that risk's name.

---

## Seven-Phase Architecture

The proven 4-layer retrieval protocol from prior versions is preserved as Phase 2. Phases 1, 3-7 are the v1.2 rigor wrap.

### Phase 1: REFERENCE

Before searching, confirm:

- What is the resumption mode (quick-reload / standard / long-gap / multi-thread / stakes-critical)?
- How long has the gap been since the user's last session in this project?
- Are there multiple domains active (trading, web-dev, outreach, etc.)?
- Did the user scope the resumption ("just pick up the dashboard work") or request full restoration?
- Is there a prior compacted summary (chat-compactor output) that should be the primary source?

Record these answers. They drive every subsequent phase.

### Phase 2: EXECUTE (the original 5-step restoration protocol, preserved verbatim)

#### Step 1: Acknowledge and Signal Depth

Before searching, acknowledge the continuation request and signal that you are
performing a comprehensive retrieval, not a shallow search. This sets
expectations.

Example:
> "Let me pull the full context from our previous sessions. I'll search thoroughly to make sure nothing active gets missed."

#### Step 2: Execute Multi-Query Search Strategy

Use the conversation_search and recent_chats tools to retrieve prior context.
Do NOT rely on a single search. Execute a layered search strategy:

**Layer 1: Recent temporal sweep**
Use recent_chats to retrieve the 5-10 most recent conversations in this project.
This catches the latest working state regardless of topic keywords.

**Layer 2: Topic-based queries**
Based on the user's request or known project themes, execute 3-5 targeted
conversation_search queries using specific content nouns. Examples:
- If working on a dashboard: search "dashboard", "React component", "data viz"
- If working on a trading tool: search "SMC criteria", "validation prompt", "pattern confirmation"
- If working on lead generation: search "lead list", "Apify scraper", "outreach"

**Layer 3: Reference data queries**
Search for specific artifacts that typically need to persist:
- File names mentioned in prior work
- API endpoints, credentials (references, not values)
- Code snippets, function names, schema definitions
- Decision criteria, validation rules, thresholds

**Layer 4: Gap check**
After initial retrieval, scan for references to items that were not retrieved.
If a prior conversation mentions "the spreadsheet from Tuesday" but no spreadsheet context was found, run an additional targeted search.

#### Step 3: Synthesize Retrieved Context

After retrieval, DO NOT dump raw search results. Synthesize them into a
structured restoration document using the format in the Output Format section.

Synthesis rules:
- Deduplicate: If the same information appears in multiple conversations, include it once.
- Resolve conflicts: If a decision was made in one conversation and revised in a later one, use the latest version and note it was updated.
- Separate active from resolved: Only carry forward what is still in progress or still needed as reference. Completed tasks can be noted briefly but do not need full detail.
- Preserve precision: Names, file paths, code, numeric thresholds, and criteria must be copied exactly. Do not paraphrase technical reference data.

#### Step 4: Identify Gaps and Clarify

After synthesis, explicitly check for gaps:
- Are there tasks mentioned as "in progress" whose current status is unclear?
- Are there file paths or artifacts referenced that could not be retrieved?
- Are there decisions where the reasoning was not captured?

If gaps exist, ask the user targeted clarifying questions BEFORE proceeding.
Example:
> "I found references to 'the outreach script' but couldn't retrieve its contents. Do you have it, or should I reconstruct it from what we discussed?"

#### Step 5: Present Restored Context and Confirm

Deliver the structured restoration document. Then explicitly confirm readiness to proceed:

> "Context restored. I'm ready to continue from [specific resume point]. Does this look complete, or is there anything I missed?"

Wait for user confirmation before taking action. The user may have additional context not captured in prior conversations.

### Phase 3: EVALUATE (sub-check library)

Run the applicable sub-checks from the library below. Quick-reload runs checks 1-4. Standard and above run all 8.

### Phase 4: EXTRACT (learnings for regression)

If a sub-check fires, the fix is applied AND the failure is logged as a new regression test row. A fix without a test is a repeat bug.

### Phase 5: ENCODE

Update the regression test log, calibration record, and Risk Register (if new failure mode).

### Phase 6: PROMOTE

Decide whether a new retrieval heuristic or query pattern graduates to the global rule set. Requires evidence (see Promotion Criteria).

### Phase 7: EVOLVE

If certain query shapes repeatedly miss their targets, update the Search Query Construction Guide. Tuning happens here, not mid-restoration.

---

## Specialized Sub-Check Library

### 1. completeness sub-check
**Goal:** Every active work stream from the prior sessions is represented.
**Procedure:** Cross-reference the Active Work Streams block against every retrieved conversation's final state. A work stream mentioned in any retrieved session as unfinished must appear.
**Failure mode:** shallow-retrieval / thread-drop.

### 2. precision sub-check
**Goal:** Every Key Reference Data value matches its source verbatim.
**Procedure:** For each bullet in Key Reference Data, diff against the source occurrence.
**Failure mode:** precision-drift.

### 3. gap sub-check
**Goal:** Unretrieved references are flagged, not silently skipped.
**Procedure:** Scan retrieved material for phrases like "the script", "that document", "the file from Tuesday". For each, confirm the referent was actually retrieved. If not, add to Pending Questions or Blockers.
**Failure mode:** phantom-certainty.

### 4. resume sub-check
**Goal:** Resume Point tells the next Claude exactly what to *do*.
**Procedure:** Read only the Resume Point. Ask: "Could a fresh Claude type its next message without re-reading the whole restoration?" If no, rewrite.
**Failure mode:** resume-vagueness.

### 5. staleness sub-check (standard+)
**Goal:** No superseded decision, draft, or plan is presented as current.
**Procedure:** For each Decisions and Locked Choices entry, locate its most recent revision in the retrieved sessions. Confirm the entry reflects the latest state.
**Failure mode:** stale-as-current.

### 6. conflict-resolution sub-check (standard+)
**Goal:** Where two retrieved sessions disagree, the restoration uses the newer one and flags the change.
**Procedure:** For any decision or reference value, scan for retrieved evidence of a revision ("actually, let's…", "changed to…", "scratch that…"). If found, resolve to latest and note the change.
**Failure mode:** stale-as-current.

### 7. scope-fit sub-check (standard+, required for multi-thread)
**Goal:** No reference data is filed under the wrong domain.
**Procedure:** For each Key Reference Data item, verify the domain tag matches the source conversation's domain.
**Failure mode:** thread-collision.

### 8. confirmation-readiness sub-check (standard+)
**Goal:** The restoration ends with an explicit invitation for user correction, not a declaration of completeness.
**Procedure:** Check that the closing line asks rather than tells. It should be "Does this look complete, or is there anything I missed?" rather than "Context fully restored."
**Failure mode:** phantom-certainty via overconfident closure.

---

## Commit Gate

Before the restoration document is delivered, all applicable items must pass:

```
[ ] 1. completeness:    every active work stream is represented
[ ] 2. precision:       every reference value matches source verbatim
[ ] 3. gap:             unretrieved references are flagged, not silently skipped
[ ] 4. resume:          Resume Point tells next Claude what to DO
[ ] 5. staleness:       no superseded decision appears as current
[ ] 6. conflict-resolution: newer evidence wins, change is noted
[ ] 7. scope-fit:       every reference is under the correct domain
[ ] 8. confirmation-readiness: closing invites correction, doesn't declare completeness
[ ] 9. format:          template exact, no em dashes, no narrative prose outside sections
```

Quick-reload mode may defer items 5-8. Items 1-4 and 9 are non-negotiable regardless of mode.

---

## Regression Test Schema

Every observed restoration failure becomes a named test row.

Test record format:

```yaml
id: S-NNN
date: YYYY-MM-DD
risk: <shallow-retrieval | stale-as-current | precision-drift | phantom-certainty | thread-collision | resume-vagueness>
resumption_mode: <quick-reload | standard | long-gap | multi-thread | stakes-critical>
observed_failure: <one-line description>
missed_query_shape: <what search query shape would have caught this>
source_fragment: <exact text from a prior conversation that should have surfaced>
corrective_action: <new query pattern, new sub-check tightening, or new regression row>
defending_sub_check: <which sub-check should have caught this>
promotion_status: <local | promoted-global>
```

### Worked example: S-005: stale-as-current on prompt version

```yaml
id: S-005
date: 2026-04-14
risk: stale-as-current
resumption_mode: long-gap
observed_failure: >
  User had two prior sessions on a trading prompt. Session 1 (three weeks ago)
  established 5 SMC criteria. Session 2 (two weeks ago) revised to 4 criteria
  after discovering criterion 3 was redundant. Restoration surfaced Session 1's
  5-criteria version because that was the more extensively-discussed thread.
missed_query_shape: >
  Did not search for "revised", "updated", "changed", "actually" within the
  trading prompt thread to find the later revision.
source_fragment: "actually let's drop criterion 3, it's redundant with criterion 1"
corrective_action: >
  Added "revision-scan" step to Phase 2 Layer 3: after retrieving reference data,
  search the same topic for revision markers ("revised", "updated", "changed",
  "actually", "scratch that"). Added regression row.
defending_sub_check: staleness
promotion_status: promoted-global
```

This row now lives in the test log. Every future standard+ restoration runs the staleness sub-check with the explicit question: *"Has any decision in this restoration been revised in a later retrieved session?"*

---

## Calibration Schema

Restoration quality is measured by what happens in the first 5 messages *after* restoration: did the user have to re-state, correct, or fill in anything the restoration claimed was covered?

Calibration record format:

```yaml
restoration_id: <date + project identifier>
restoration_date: YYYY-MM-DD
resumption_mode: <quick-reload | standard | long-gap | multi-thread | stakes-critical>
retrieval_depth:
  layer_1_conversations_retrieved: <count>
  layer_2_queries_executed: <count>
  layer_3_queries_executed: <count>
  layer_4_gap_follow_ups: <count>
user_corrections_first_5_messages:
  - type: <missed-work-stream | wrong-decision-version | paraphrased-reference | phantom-certainty | thread-mislabel>
    content: <what the user had to correct>
    root_cause: <shallow-retrieval | stale-as-current | precision-drift | phantom-certainty | thread-collision | resume-vagueness>
adjustments:
  - tightened sub-check / added regression row / updated query guide
```

Target: zero corrections across three consecutive restorations. One correction triggers a regression row. Three in a row triggers a skill-level audit.

---

## Domain Scope Tagging

When a project spans multiple domains, tag each Key Reference Data item:

```
**Key Reference Data:**
[trading]
- SMC validation: minimum RR 1:2, 4 criteria must pass (revised from 5, criterion 3 dropped as redundant)
- Prompt file: /sessions/.../trading_confirmation_v3.md

[lead-gen]
- Lead list: /mnt/user-data/outputs/leads_oshawa_hvac.xlsx
- Apify actor: apify/google-maps-scraper

[portfolio-site]
- Stack: Lovable, React, Tailwind, Vite
- Benchmark URL: poppr.be
```

Untagged references get `[shared]` or the domain is inferred from the user's current request.

---

## Adversarial Stress-Test Mode

For long-gap and stakes-critical resumptions only. Before delivering the restoration, run this pass:

**Step A: Imagine the retrieval was incomplete.** For each Active Work Stream, ask: "What's a high-probability active thread that my layered search would miss?" Run one more targeted query for each.

**Step B: Probe every precision value.** For each Key Reference Data item, ask: "If I have this wrong by one character, what breaks?" For high-cost values (file paths, prompts about to be deployed, financial thresholds), re-retrieve the source and diff.

**Step C: Probe the Resume Point.** Simulate being a fresh Claude who reads only the restoration. What's the first question you'd need to ask the user to start work? If it's anything load-bearing ("which file?", "which version?", "what's the goal?"), the Resume Point is incomplete.

**Step D: Only then deliver.** Adversarial mode is strictly additive; it catches what sub-checks miss when the cost of being wrong is highest.

---

## Output Format

Use this exact template. Do not add sections. Do not rename sections. Do not reorder sections.

```
## Session Context Restoration

**Active Work Streams:**
- [Each task or project currently in progress, with its current stage]
- [Be specific about what has been done and what remains]

**Decisions and Locked Choices:**
- [Each confirmed decision, selected approach, or rule the user has locked in]
- [Include the reasoning in one clause if it affects future work]

**Key Reference Data:**
[Present as a structured list, not prose. Tag by domain if multi-domain project.]
- Label: value
- Label: value
- Code blocks preserved inline if essential

**Pending Questions or Blockers:**
- [Any unresolved questions, missing information, or blockers identified]

**Completed Work (for reference):**
- [One-line per completed item. Brief. Only include if it provides useful context for ongoing work.]

**Resume Point:**
[One paragraph. State exactly where the work stands, what the immediate next
action is, and any context the user would need to continue. Write this as a
handoff note to Claude continuing the session.]
```

---

## Search Query Construction Guide

The conversation_search tool is a text match system. Queries must use words that actually appeared in prior conversations.

**Use content nouns, not meta-words:**
- Good: "dashboard React chart"
- Bad: "the thing we discussed yesterday"

**Be specific:**
- Good: "SMC validation criteria"
- Bad: "trading stuff"

**Use multiple queries rather than one broad query:**
- Instead of one search for "project work", run separate searches for each known work stream: "lead generation Oshawa", "portfolio site GSAP", "trading prompt"

**If a search returns nothing useful, reformulate:**
- Try synonyms or related terms
- Try shorter, more specific phrases
- Try names of files, tools, or artifacts mentioned

**v1.2 addition: revision-scan queries:**
After retrieving the main thread for a topic, run a second query on that topic paired with revision markers: "<topic> revised", "<topic> updated", "<topic> actually let's", "<topic> scratch that". This is the defense against stale-as-current.

---

## Failure Mode Prevention

**Failure mode 1: Single shallow search.**
Prevention: Always execute the full multi-layer search strategy. A single search almost never captures the complete working state.

**Failure mode 2: Dumping raw search results.**
Prevention: Always synthesize retrieved content into the structured format. The user needs an actionable restoration, not a pile of snippets.

**Failure mode 3: Missing recent changes.**
Prevention: Always include recent_chats in the retrieval. Topic searches may miss conversations where the topic was discussed briefly or where work pivoted.

**Failure mode 4: Overconfident restoration.**
Prevention: Always check for gaps and ask clarifying questions. If something seems incomplete, say so rather than assuming.

**Failure mode 5: Proceeding without confirmation.**
Prevention: Always present the restoration and wait for user confirmation before taking action. The user may have context you did not retrieve.

**Failure mode 6 (new in v1.2): Surfacing the most-discussed version instead of the latest.**
Prevention: Always run revision-scan queries (Phase 2 Layer 3 addition) and the staleness sub-check.

---

## Worked Example: Full v1.2 Restoration Run

**Resumption context:** User opens a new session three weeks after last working with Claude on this project. First message: "let's continue the trading work, oh and also where did we land on the Oshawa leads?"

**Phase 1: REFERENCE:**
Mode: long-gap (three week gap) and multi-thread (two distinct domains in the opening message).
Primary domains: trading, lead-gen.

**Phase 2: EXECUTE:**

Layer 1: recent_chats: retrieved 8 conversations from the project.
Layer 2: topic searches:
- "SMC criteria" → found trading prompt threads.
- "liquidity sweep" → found pattern 4 discussion.
- "Oshawa HVAC" → found lead gen thread.
- "outreach template" → found email draft thread.

Layer 3: reference data + revision-scan:
- "leads_oshawa_hvac.xlsx" → confirmed file path.
- "SMC criteria revised" → **found**: the 5-criteria → 4-criteria revision.
- "outreach template updated" → no hits, template unchanged.

Layer 4: gap check:
- Retrieved thread mentions "the approved sender address": ran additional query, found decision locked in earlier session.

**Phase 3: EVALUATE (draft sub-check results):**

- completeness: pass (both domains represented, all active streams surfaced).
- precision: pass (file paths, URLs, criteria all verbatim).
- gap: pass (sender address gap caught and resolved in Layer 4).
- resume: **weak**: draft said "pick up Pattern 4", too general.
- staleness: **caught**: initial draft had 5 criteria, revised to 4 after revision-scan.
- conflict-resolution: pass (change noted in Decisions and Locked Choices).
- scope-fit: pass (trading vs lead-gen correctly tagged).
- confirmation-readiness: pass.
- format: pass.

**Phase 4: EXTRACT:**
Two sub-checks fired (staleness caught, resume weak). Log a regression row for resume-vagueness on Pattern 4 (underspecified next action).

**Phase 5: ENCODE:**
Revised restoration:

```
## Session Context Restoration

**Active Work Streams:**
[trading]
- SMC confirmation prompt: Patterns 1-3 are finalized with binary pass/fail criteria. Pattern 4 (liquidity sweep continuation) has criterion structure drafted but entry/stop/target conditions not yet written. Pattern 5 (OB mitigation entry) has not been started.

[lead-gen]
- Oshawa HVAC outreach: lead list of 47 leads generated 2026-03-23. Outreach email template drafted but not approved for send. Sender address decided (approved 2026-03-28) but template has not been updated with it.

**Decisions and Locked Choices:**
[trading]
- Binary pass/fail per criterion, not scoring.
- Updated 2026-03-31: 4 criteria must pass, not 5 (criterion 3 dropped as redundant with criterion 1).
- Minimum RR: 1:2.

[lead-gen]
- Outreach tone: professional but warm, no hard sell, emphasize speed and affordability.
- Approved sender: outreach@afrocreativegroup.com (locked 2026-03-28).

**Key Reference Data:**
[trading]
- Prompt working file: /sessions/.../trading_confirmation_v3.md
- Model: claude-opus-4-6

[lead-gen]
- Lead list: /mnt/user-data/outputs/leads_oshawa_hvac.xlsx (47 rows)
- Apify actor: apify/google-maps-scraper

**Pending Questions or Blockers:**
- Outreach template still has placeholder sender. Needs update to the approved address before first send.

**Completed Work (for reference):**
- SMC Patterns 1-3 criteria definition.
- Lead list generation.
- Sender address approval.

**Resume Point:**
Two active threads. Primary: Pattern 4 (liquidity sweep continuation). The next action is to write Pattern 4's entry, stop, and target conditions using the same structure as Pattern 3 (FVG retest): one binary pass/fail clause per condition. The 4-criteria update (from 5) must be reflected consistently when writing Pattern 4. Secondary: update the Oshawa outreach template with the approved sender address before sending: no other template changes were locked.

Context restored. Does this look complete, or is there anything I missed?
```

**Phase 6: COMMIT GATE:** all 9 items pass. Restoration delivered.

**Phase 7: CALIBRATION:** record session, measure user corrections in first 5 messages post-restoration.

---

## Constraints

- Do not skip the multi-query search strategy. Shallow retrieval is the primary cause of continuity failure.
- Do not dump raw conversation snippets. Always synthesize into the structured format.
- Do not proceed to work until the user confirms the restoration is complete.
- Do not assume context that was not retrieved. If information is missing, ask.
- Do not paraphrase technical reference data (file paths, code, thresholds, criteria). Copy exactly.
- Do not include resolved work in full detail. Keep the "Completed Work" section brief.
- Do not use em dashes anywhere in the output.
- If a section has no content, write "None identified." rather than omitting it.
- Do not deliver the restoration if the commit gate has a failing item. Revise first.

---

## Integration with Other Skills

This skill works alongside:

**chat-compactor:** If a prior session was compacted, the compact summary is the primary source for that session's context. Search for and prioritize compact summaries when they exist. Use compact-summary format as an authoritative source before re-deriving from raw conversation messages.

**prompt-engineering:** If prior work involved prompt development, ensure the current prompt version is retrieved exactly. Prompt wording is precision-critical. Run revision-scan (Phase 2 Layer 3 addition) specifically against prompt threads.

**humanizer:** If prior work produced content that was humanized, the final humanized version is the source of truth, not intermediate drafts.

**Any file-creation skills:** If prior work produced files (docx, xlsx, code), note the file paths in Key Reference Data so they can be accessed.

---

## Anti-Patterns

1. **Default-shallow retrieval.** Running one keyword search, getting plausible results, and stopping. The layered strategy exists precisely to break this habit.

2. **Narrative synthesis.** Writing the restoration as prose ("We've been working on...") instead of using the structured template. Prose hides gaps; the template exposes them.

3. **Most-discussed wins.** Surfacing the older, more-discussed version of a decision because it had more conversation turns. The latest revision wins, regardless of discussion volume.

4. **Silent gap handling.** Encountering a reference that couldn't be retrieved, and omitting it rather than flagging it. Flag every gap; never silently skip.

5. **Overconfident closure.** Ending with "Context fully restored" instead of "Does this look complete?" The user always has context the retrieval missed. The closing line must invite correction.

6. **Cross-domain bleed.** Filing a trading decision under a web-dev Active Work Stream because both mentioned the word "component." Domain-tag every reference.

---

## Failure Modes Table

| Failure | Symptom | Root cause | Fix |
|---|---|---|---|
| User re-explains a task | Active Work Stream missed | shallow-retrieval | Add targeted topic query to the guide |
| User corrects a decision | Older version surfaced | stale-as-current | Run revision-scan queries, tighten staleness sub-check |
| User re-pastes a file path | Reference was paraphrased | precision-drift | Tighten precision sub-check |
| User asks "did we actually decide X?" | Decision stated with false confidence | phantom-certainty | Tighten gap sub-check, flag uncertain items explicitly |
| User pastes a value in the wrong domain | Cross-domain contamination | thread-collision | Enforce domain scope tagging |
| User asks "what should I do next?" | Resume Point too general | resume-vagueness | Tighten resume sub-check |

---

## Audit Checklist

Pre-delivery self-review for standard+ restorations:

```
[ ] Did I run all 4 retrieval layers, not just 1 or 2?
[ ] Did I run revision-scan queries for decision-critical threads?
[ ] Is every Active Work Stream traceable to a retrieved thread?
[ ] Is every reference value in Key Reference Data verbatim from source?
[ ] Are unretrieved references flagged under Pending Questions or Blockers?
[ ] For multi-domain sessions: is every item tagged by domain?
[ ] Is the Resume Point actionable without reading the whole restoration?
[ ] For long-gap / stakes-critical sessions: did the adversarial pass pass?
[ ] Is the closing line an invitation, not a declaration?
[ ] If any sub-check failed, did I log a regression test row?
```

---

## Promotion Criteria

A retrieval heuristic or query pattern graduates from local to global when:

1. The pattern has caught a real failure in at least three distinct sessions.
2. The fix is consistent across sessions.
3. The fix does not conflict with a higher-priority rule.
4. A regression test exists that would have caught the failure.

Revision-scan (the "revised", "updated", "actually" query pattern) is an example of a local heuristic that was promoted after repeated staleness failures.

---

## Reference

- Retrieval is deep by construction, not by hope. The layered strategy exists because shallow retrieval is Claude's default, not its exception.
- Every regression test is an earned scar. The skill improves strictly from observed failure, not theorized failure.
- When in doubt, run one more query. Over-retrieval costs seconds. Under-retrieval costs sessions.
- The commit gate is binary. There is no "mostly ready" restoration.
- The first 5 messages after restoration are the honest quality signal. Self-grading is not a substitute for zero-correction calibration.

---

## Version History

- **1.2.0 (2026-04-17):** Added operational modes, Named Risk Register, 8-sub-check library, commit gate, regression test schema, calibration schema, domain scope tagging, adversarial stress-test mode, revision-scan query pattern, anti-patterns, failure modes table, audit checklist, promotion criteria. Original 5-step protocol and worked example preserved.
- **1.1.0 (prior):** Added 4-layer search strategy, gap check, integration notes. Worked example in output.
- **1.0.0 (initial):** Core restoration protocol, output template, trigger list.
