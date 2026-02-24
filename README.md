# Paper Revision

A Claude Code skill for systematically processing peer review feedback on academic paper submissions.

## What It Does

When you receive reviewer comments on a submitted paper, this skill guides you through a structured four-phase workflow:

1. **Parse** — Extract and classify all reviewer comments (Major / Minor / Editorial), identify cross-reviewer concerns, generate a structured overview
2. **Strategy** — Discuss each comment interactively, with suggested revision approaches based on your paper's content
3. **Summary** — Generate a revision plan (Chinese) and a point-by-point response letter (English)
4. **Review** — Refine the output documents based on your feedback

## Output Files

| File | Language | Purpose |
|------|----------|---------|
| `revision-overview.md` | Chinese | Structured summary of all reviewer comments with classification and priority |
| `revision-plan.md` | Chinese | Section-by-section revision checklist with checkboxes for progress tracking |
| `response-letter.md` | English | Standard academic response letter with point-by-point replies |

## Installation

Copy the skill to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/paper-revision
cp SKILL.md ~/.claude/skills/paper-revision/
```

## Usage

1. Place your paper PDF and review comments PDF in a working directory
2. Open Claude Code in that directory
3. Run `/paper-revision`
4. Follow the interactive prompts

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- PDF files of your paper and reviewer comments in the working directory

## License

MIT
