# claude-skills

A small library of composable skills for Claude Code and Claude agents, focused on reliability, self-correction, advanced chain of thought, and working productively across long sessions.

Each skill is a self-contained Markdown file (`SKILL.md`) that Claude activates automatically when the conversation matches its triggers. Skills can be used individually or composed. 
---

## Why these exist

I use Claude daily across most of what I do: building and refining the skills in this library, personal projects, the operations of my businesses, and the research and benchmarking work I do on projects with Outlier, where I work as an Oracle-tier prompt engineer and reviewer evaluating AI training data and model outputs for correctness, reasoning quality, and failure modes for financial analysis.

The same categories of failure kept surfacing across all of it. Claude would hallucinate specific function, repeating the same mistakes. Long sessions would lose the thread of what we were actually working on. Prompts that worked yesterday would produce generic output today. Correction rules I'd established in one session wouldn't fire in the next. Consequential questions would get one fluent line of reasoning when three independent angles were needed.

Each skill in this library is a response to a specific failure class I kept hitting and wanted to stop hitting. None of them are speculative. Every one has been iterated against real work, and each has a version history that documents what the previous version didn't solve.

I'm publishing them because the failures they address aren't personal to me. They're structural to how LLMs behave.

---

## The skills

| Skill | Version | What it's for |
|-------|---------|---------------|
| [self-evolving-agent](self-evolving-agent/SKILL.md) | v1.2.0 | Catches systematic Claude failures and installs mechanical backstops. Corrections generate paired regression tests that run at a commit gate — a failing test blocks output whether or not the agent remembered to check. Five named risks on every non-trivial task; rules scoped to global, domain, or project so corrections don't bleed across contexts. |
| [prompt-engineering](prompt-engineering/SKILL.md) | v1.1.0 | Eight-component structure for prompts that work reliably. Every prompt is two contracts: behavioral (how Claude reasons) and output (what it delivers). Domain-agnostic. |
| [session-continuity](session-continuity/SKILL.md) | v1.2.0 | Restores working context when a session ends cold. Four layered query types instead of a single-query guess, synthesized into a working state. Five modes from quick-reload to stakes-critical. |
| [chat-compactor](chat-compactor/SKILL.md) | v1.2.0 | Compresses the context window mid-session around 60–70% usage. Keeps active tasks and decisions, drops resolved threads. Five modes. |
| [oracle-research](oracle-research/SKILL.md) | v1.0.0 | Per-claim source verification for research. Every claim traces to a primary source before output ships. Uncertainty registry for anything unverifiable — gaps stay visible, not papered over. |
| [superintelligence](superintelligence/SKILL.md) | v1.1.0 | Multi-lens reasoning across up to seven parallel registers. Steel-mans the opposition, falsifies before committing, calibrates confidence to evidence. Stakes-scaled from a one-line check to a full eight-phase protocol. |

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
