# Paper Revision v2.0.0

A Claude Code skill for systematically processing peer review feedback on academic paper submissions.

## What It Does

When you receive reviewer comments on a submitted paper, this skill guides you through a structured five-phase workflow:

```
┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐
│  Phase 1   │    │  Phase 2   │    │  Phase 3   │    │  Phase 4   │    │  Phase 5   │
│   Parse    │───▶│  Strategy  │───▶│  Summary   │───▶│  Verify    │───▶│  Review    │
│  解析意见   │    │  讨论策略   │    │  生成文档   │    │  自查验证   │    │  用户审阅   │
└───────────┘    └───────────┘    └───────────┘    └───────────┘    └───────────┘
```

1. **Parse** — Extract and classify all reviewer comments (Major / Minor / Editorial), identify cross-reviewer concerns, generate a structured overview
2. **Strategy** — Discuss each comment interactively, with suggested revision approaches and action type tags (`clarification` / `experiment` / `analysis` / `structural` / `citation`)
3. **Summary** — Generate a revision plan (Chinese) with LaTeX marking guide, and a point-by-point response letter (English)
4. **Verify** — Completeness & consistency checks, before/after self-scoring, risk alerts
5. **Review** — Refine the output documents based on your feedback

## Output Files

| File | Language | Purpose |
|------|----------|---------|
| `revision-overview.md` | Chinese | Structured summary of all reviewer comments with classification and priority |
| `revision-plan.md` | Chinese | Section-by-section revision checklist with LaTeX marking guide and action tags |
| `response-letter.md` | English | Standard academic response letter with point-by-point replies |

## Installation

```bash
npx skills add jinyh/paper-revision@paper-revision
```

Or manually:

```bash
mkdir -p ~/.claude/skills/paper-revision
cp SKILL.md ~/.claude/skills/paper-revision/skill.md
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
