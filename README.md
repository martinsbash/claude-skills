# claude-skills

A small library of composable skills for Claude Code and Claude agents, focused on reliability, self-correction, and working productively across long sessions.

Each skill is a self-contained Markdown file (`SKILL.md`) that Claude activates automatically when the conversation matches its triggers. Skills can be used individually or composed. They're designed to degrade gracefully when the agent is tired, distracted, or running under time pressure, which is to say, most of the time.

---

## Why these exist

I use Claude daily across most of what I do: building and refining the skills in this library, personal projects, the operations of my ventures, and the research and benchmarking work I do on projects with Outlier. On those projects I work as an Oracle-tier benchmark engineer and reviewer, probing AI training data and model outputs for correctness, reasoning quality, and failure modes. My ventures include a creative agency, a trading workflow, a clothing brand, and a personal productivity system.

The same categories of failure kept surfacing across all of it. Claude would hallucinate specific function signatures under time pressure. Long sessions would lose the thread of what we were actually working on. Prompts that worked yesterday would produce generic output today. Correction rules I'd established in one session wouldn't fire in the next. Consequential questions would get one fluent line of reasoning when three independent angles were needed.

Each skill in this library is a response to a specific failure class I kept hitting and wanted to stop hitting. None of them are speculative. Every one has been iterated against real work, and each has a version history that documents what the previous version didn't solve.

I'm publishing them because the failures they address aren't personal to me. They're structural to how LLMs behave.

---

## The skills

| Skill | Version | What it's for |
|-------|---------|---------------|
| [self-evolving-agent](self-evolving-agent/SKILL.md) | v1.2.0 | A reliability framework for Claude. Most AI failures aren't random; they're systematic: hallucinated details, accepting coherent-looking work as correct, walking one reasoning path when three exist, giving generic advice when the situation called for specifics. This skill installs a seven-phase loop that identifies failure *categories* and builds mechanical backstops against each. Corrections spawn paired regression tests that run at a commit gate before any output ships. Five named risks get checked on every non-trivial task; seven specialized sub-checks fire where the task's risk zones demand them. Rules tag to global, domain, or project scope so finance corrections don't leak into creative writing. v1.2 converts the loop from behavioral to mechanical: a failing test blocks the output whether or not the agent remembered to check. |
| [prompt-engineering](prompt-engineering/SKILL.md) | v1.1.0 | A structured discipline for writing prompts that work reliably rather than by accident. Most prompts fail quietly because they skip one of the obvious components: audience, format, examples, what to avoid, how to handle edge cases. This skill treats prompt construction as an eight-component structure (role, context, task, constraints, format, examples, persistence, evaluation) and every prompt as two contracts at once: the behavioral contract (how Claude should reason) and the output contract (what the final artifact must look like). Domain-agnostic; the same components apply to a one-shot code prompt or a locked system prompt that persists across sessions. |
| [session-continuity](session-continuity/SKILL.md) | v1.2.0 | Context restoration across sessions. When a chat ends (context limit, crash, tab closed), the next one starts cold and projects Claude barely remembers. This skill runs a layered retrieval when you ask Claude to resume: four query types (recent chats, topic queries, reference queries, gap check) instead of the single-query guess most retrieval systems default to, then synthesizes the results into a working state rather than dumping snippets. Five operational modes, from quick-reload for short gaps to stakes-critical for long-running high-stakes projects. |
| [chat-compactor](chat-compactor/SKILL.md) | v1.2.0 | Context window compression for long sessions. When you're well into a chat, Claude's attention to the earlier parts degrades: it starts missing instructions set up at the top, mixing up which file you're on, or regressing on things it already resolved. This skill triggers around 60 to 70 percent context usage, preserves the active task, decisions, and live reference material, and drops the resolved back-and-forth. You stay in the same session, Claude's attention stays on what matters. Five modes depending on what's at stake in the compaction. |
| [oracle-research](oracle-research/SKILL.md) | v1.0.0 | Deep research with per-claim source verification. Hallucinated citations are the most confident-sounding failure mode and the hardest to catch after the fact: the output looks authoritative, the claim is wrong, and the reader has no way to tell. This skill installs a mandatory source-ledger protocol: every claim must trace back to a primary source before output is finalized. Multi-phase retrieval, credibility scoring, and an uncertainty registry for claims that can't be verified. The goal isn't to produce more citations; it's to make the provenance of every claim explicit so gaps are visible rather than papered over. |
| [superintelligence](superintelligence/SKILL.md) | v1.1.0 | A multi-lens reasoning protocol for hard questions. Most AI reasoning fails because the model walks one fluent line and never checks it against independent angles. This skill runs a Cognitive Operating Protocol across up to seven parallel registers (philosopher, scientist, mathematician, engineer, strategist, adversary, empath), requires steel-manning the strongest opposing position before committing, runs a falsification phase before any claim gets staked, and calibrates confidence to evidence rather than to fluency. Stakes-scaled: trivial questions get a one-line internal check; consequential decisions get the full eight-phase protocol. Composes with oracle-research for factual claims, self-evolving-agent for correction loops, and prompt-engineering for structured output. This is the cognitive substrate the other skills call when a hard thinking step appears. |

Version numbers reflect how many times I've rewritten the skill based on real failures. Three of the six (`self-evolving-agent`, `chat-compactor`, and `session-continuity`) have converged on a shared v1.2 architecture: seven phases, a named risk register, a binary commit gate before output, regression tests paired to each risk, and calibration tracking. That architecture emerged from closing structural failure gaps I kept hitting, and I've been retrofitting it to the other skills where it applies.

I update the skills roughly every two weeks. Each pass is driven by specific failures the current version didn't catch, not a scheduled release cadence.

---

## How to use

1. Clone this repo or download the `SKILL.md` file for the skill you want.
2. Add it to your Claude Code skills directory (typically `.claude/skills/<skill-name>/SKILL.md`) or paste it into a project's custom instructions.
3. The skill activates automatically when your messages match its triggers. No explicit invocation needed.

Skills compose. A long multi-session project might activate `session-continuity` to resume, `self-evolving-agent` to track and correct repeat errors, `prompt-engineering` when building a new workflow prompt, `oracle-research` when a task requires verified sourcing, `superintelligence` when a consequential question needs multiple independent reasoning angles, and `chat-compactor` when the session runs long. They don't conflict.

---

## Honest limits

A few things this library does not yet do, which I'd rather name than hide:

- **No held-out eval harness.** `self-evolving-agent` v1.2 does run regression tests mechanically at the commit gate, so corrections are tested before output ships. But that's internal to the skill, not an independent measure of whether the skill improves outputs against a benchmark set. A proper external eval pass is on the roadmap.
- **No user studies.** These have been tested mostly by me. If you find failure modes I haven't documented, I'd like to hear about them.
- **Not all failure classes are covered.** The skills address metacognition, context management, prompt construction, source verification, and multi-lens reasoning. They don't address multi-agent coordination, tool-use reliability, or alignment-style concerns. Each of those probably deserves its own skill; I haven't written them yet because I haven't hit the failures hard enough to know what the shape of the answer is.

---

## Background

Built by [Martins Bash](https://github.com/martinsbash). AI trainer and researcher; co-founder of [Afro Creative Group](https://afrocreativegroup.lovable.app). Longer-form writing on [Medium](https://medium.com/@martinsbash). The skills are refined in the course of that research and work, not as a separate project.

---

## License

MIT. Use, modify, and redistribute freely. A note back if something's useful is always welcome.
