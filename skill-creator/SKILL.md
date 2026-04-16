---
name: skill-creator
description: Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to create a skill from scratch, edit, or optimize an existing skill, run evals to test a skill, benchmark skill performance with variance analysis, or optimize a skill's description for better triggering accuracy.
---

# Skill Creator

A skill for creating new skills and iteratively improving them.

At a high level, the process of creating a skill goes like this:

- Decide what you want the skill to do and roughly how it should do it
- Write a draft of the skill
- Create a few test prompts and run claude-with-access-to-the-skill on them
- Evaluate the results both qualitatively and quantitatively
- Rewrite the skill based on feedback
- Repeat until you're satisfied
- Expand the test set and try again at larger scale

## Capture Intent

Start by understanding the user's intent:

1. What should this skill enable Claude to do?
2. When should this skill trigger? (what user phrases/contexts)
3. What's the expected output format?
4. Should we set up test cases to verify the skill works?

## Interview and Research

Proactively ask questions about edge cases, input/output formats, example files, success criteria, and dependencies.

## Write the SKILL.md

Based on the user interview, fill in these components:

- **name**: Skill identifier
- **description**: When to trigger, what it does. This is the primary triggering mechanism.
- **compatibility**: Required tools, dependencies (optional)
- **the rest of the skill**

### Anatomy of a Skill

```
skill-name/
  SKILL.md (required)
    YAML frontmatter (name, description required)
    Markdown instructions
  Bundled Resources (optional)
    scripts/    - Executable code for deterministic tasks
    references/ - Docs loaded into context as needed
    assets/     - Files used in output (templates, icons, fonts)
```

### Progressive Disclosure

Skills use a three-level loading system:
1. **Metadata** (name + description) - Always in context (~100 words)
2. **SKILL.md body** - In context whenever skill triggers (<500 lines ideal)
3. **Bundled resources** - As needed (unlimited)

### Writing Patterns

- Keep SKILL.md under 500 lines
- Reference files clearly from SKILL.md with guidance on when to read them
- Prefer using the imperative form in instructions
- Explain the **why** behind instructions rather than relying on rigid MUSTs
- Include examples with concrete input/output pairs

## Test Cases

After writing the skill draft, come up with 2-3 realistic test prompts. Share them with the user for confirmation.

Save test cases to evals/evals.json:

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "User's task prompt",
      "expected_output": "Description of expected result",
      "files": []
    }
  ]
}
```

## Running and Evaluating Test Cases

For each test case, spawn two runs: one with the skill and one without (baseline). This helps measure the skill's actual impact.

### Evaluation Steps

1. **Run tests** - Spawn with-skill and baseline runs in parallel
2. **Draft assertions** - While runs execute, draft quantitative assertions
3. **Capture timing** - Record total_tokens and duration_ms from each run
4. **Grade and aggregate** - Evaluate assertions, generate benchmark data
5. **Launch viewer** - Present results for human review
6. **Read feedback** - Process user feedback from the review

## Improving the Skill

### How to think about improvements

1. **Generalize from feedback** - Don't overfit to specific test cases
2. **Keep the prompt lean** - Remove things that aren't pulling their weight
3. **Explain the why** - Help the model understand reasoning, not just rules
4. **Look for repeated work** - If all test cases write similar helper scripts, bundle them

### The Iteration Loop

1. Apply improvements to the skill
2. Rerun all test cases into a new iteration directory
3. Launch the reviewer with previous iteration comparison
4. Wait for user review
5. Read feedback, improve again, repeat

Keep going until the user is happy or you're not making meaningful progress.

## Description Optimization

The description field is the primary mechanism that determines whether Claude invokes a skill. After creating a skill, optimize the description for better triggering accuracy.

1. Generate 20 eval queries (mix of should-trigger and should-not-trigger)
2. Review with user
3. Run the optimization loop (splits 60% train / 40% test, iterates up to 5 times)
4. Apply the best description to the skill

## Core Loop Summary

- Figure out what the skill is about
- Draft or edit the skill
- Run claude-with-access-to-the-skill on test prompts
- Evaluate outputs qualitatively and quantitatively
- Repeat until satisfied
- Package the final skill and return it to the user
