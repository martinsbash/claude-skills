# Claude Skills


Custom AI skills and automation workflows built for Claude.


A small library of composable skills for Claude Code and Claude agents, focused on reliability, self-correction, and working productively across long sessions.
Each skill is a self-contained Markdown file (SKILL.md) that Claude activates automatically when the conversation matches its triggers. Skills can be used individually or composed. They're designed to degrade gracefully when the agent is tired, distracted, or running under time pressure, which is to say, most of the time.

## Why these exist

I build with Claude daily, primarily as a research partner. On projects with Outlier I work as an Oracle-tier benchmark engineer and reviewer, probing AI training data and model outputs for correctness, reasoning quality, and failure modes. Claude runs alongside that work for reading papers, cross-referencing sources, synthesizing documentation, and pressure-testing my own reasoning before a review goes out. I also use Claude on personal research and on my ventures: a creative agency, a trading workflow, a clothing brand, and a personal productivity system.
The same categories of failure kept surfacing across all of it. Claude would hallucinate specific function signatures under time pressure. Long sessions would lose the thread of what we were actually working on. Prompts that worked yesterday would produce generic output today. Correction rules I'd established in one session wouldn't fire in the next.
Each skill in this library is a response to a specific failure class I kept hitting and wanted to stop hitting. None of them are speculative. Every one has been iterated against real work, and each has a version history that documents what the previous version didn't solve.
I'm publishing them because the failures they address aren't personal to me. They're structural to how LLMs behave.


## Skills


| Skill | Description |
|-------|-------------|
| [Prompt Engineering](prompt-engineering/SKILL.md) | Production-grade prompt writing, auditing, and optimization for any domain |
| [Self-Evolving Agent](self-evolving-agent/SKILL.md) | Recursive self-improvement loops — Claude learns from mistakes and gets better over time |
| [Chat Compactor](chat-compactor/SKILL.md) | Context window compression to keep long conversations efficient |
| [Session Continuity](session-continuity/SKILL.md) | Context restoration for picking up work seamlessly across sessions |
| [Oracle Research](oracle-research/SKILL.md) | Elite, domain-agnostic research and verification skill for Claude. 11-phase protocol with evidence tiering, triangulation, adversarial review, with every claim traceable to a primary source. Inline memo + interactive source-verifier HTML|


## How to Use


1. Copy the `SKILL.md` file for the skill you want
2. Add it to your Claude project's custom instructions or skill directory
3. The skill activates automatically when relevant triggers are detected in conversation

Each skill has trigger phrases built in. When your message matches one, the skill fires automatically. For example:

Type "learn from this mistake" or "don't make that error again" → self-evolving-agent kicks in
Type "compact this chat" or "context is getting long" → chat-compactor runs
Type "continue from last session" or "pick up where we left off (mention the particular chat name/title)" → session-continuity restores context
Type "research this" or "verify this claim" → oracle-research runs a sourced investigation

You can also call any skill directly by typing /skill-name in the chat — for example /self-evolving-agent or /oracle-research. Useful when the trigger phrases don't quite fit what you're doing but you know which skill you need.

## Honest limits

A few things this library does not yet do, which I'd rather name than hide:

1. No held-out eval harness. self-evolving-agent v1.2 does run regression tests mechanically at the commit gate, so corrections are tested before output ships. But that's internal to the skill, not an independent measure     of whether the skill improves outputs against a benchmark set. A proper external eval pass is on the roadmap.
2. No user studies. These have been tested mostly by me. If you find failure modes I haven't documented, I'd like to hear about them.
3. Not all failure classes are covered. The skills address metacognition, context management, prompt construction, and source verification. They don't address multi-agent coordination, tool-use reliability, or alignment-     style concerns. Each of those probably deserves its own skill; I haven't written them yet because I haven't hit the failures hard enough to know what the shape of the answer is.

## Author


Built by [Martins Bash](https://github.com/martinsbash) — Entrepreneur & Builder, Founder of Afro Creative Group.

