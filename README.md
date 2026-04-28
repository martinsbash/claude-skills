# Claude Skills


Custom AI skills and automation workflows built for Claude.


These skills extend Claude's capabilities with specialized behaviors for prompt engineering, self-improvement, context management, and more. Each skill is a standalone `.md` file that can be loaded into Claude to activate the workflow.

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
| [Oracle Research](oracle-research/SKILL.md) | Elite, domain-agnostic research and verification skill for Claude. 11-phase protocol with evidence tiering, triangulation, adversarial review, and calibrated confidence bands so every claim is traceable to a primary source. Inline memo + interactive source-verifier HTML|


## How to Use


1. Copy the `SKILL.md` file for the skill you want
2. Add it to your Claude project's custom instructions or skill directory
3. The skill activates automatically when relevant triggers are detected in conversation


## Author


Built by [Martins Bash](https://github.com/martinsbash) — Entrepreneur & Builder, Founder of Afro Creative Group.

