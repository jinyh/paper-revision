---
name: paper-revision
version: 2.0.0
description: Systematically analyze peer review comments, develop revision strategies, and generate response letters for academic paper submissions. Use when handling reviewer feedback on submitted papers.
---

# Paper Revision Assistant

Help authors systematically process peer review feedback through a structured five-phase workflow:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Paper Revision Workflow                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐  │
│  │  Phase 1   │    │  Phase 2   │    │  Phase 3   │    │  Phase 4   │
│  │   Parse    │───▶│  Strategy  │───▶│  Summary   │───▶│  Verify    │
│  │  解析意见   │    │  讨论策略   │    │  生成文档   │    │  自查验证   │
│  └───────────┘    └───────────┘    └───────────┘    └───────────┘  │
│       │                │                │                │          │
│       ▼                ▼                ▼                ▼          │
│  revision-        User decides     revision-plan.md   Completeness │
│  overview.md      per comment      response-letter.md  + Scoring   │
│                                                         │          │
│                                                         ▼          │
│                                                    ┌───────────┐   │
│                                                    │  Phase 5   │   │
│                                                    │  Review    │   │
│                                                    │  用户审阅   │   │
│                                                    └───────────┘   │
│                                                         │          │
│                                                         ▼          │
│                                                       Done ✓       │
└─────────────────────────────────────────────────────────────────────┘
```

## When to Use This Skill

Trigger when user:
- Calls `/paper-revision`
- Mentions "审稿意见", "reviewer comments", "revision", "response letter"
- Has PDF files of a paper and review comments in the working directory

## Prerequisites

- Working directory contains PDF files (paper + review comments)
- Claude Code's PDF reading capability (Read tool with `pages` parameter)

## Phase 1: Parse — 解析审稿意见

### 1.1 Identify PDFs

Scan the current working directory for PDF files:

```
Glob pattern: *.pdf
```

Present the list to the user via `AskUserQuestion`:
- "以下是当前目录中的 PDF 文件，请确认哪个是论文原稿，哪个是审稿意见信？"
- Options should list each PDF filename, letting user assign roles

If only one PDF exists, ask if it contains both paper and reviews, or if the user needs to provide another file.

### 1.2 Extract Review Comments

Read the review comments PDF using the `Read` tool (use `pages` parameter for large PDFs — max 20 pages per request, read in batches if needed).

Extract and structure ALL comments from every reviewer. For each comment:
- **Reviewer ID**: Reviewer 1 / 2 / 3 / Editor
- **Comment number**: Sequential within each reviewer
- **Original text**: Exact quote
- **Category**: Major / Minor / Editorial (see classification rules below)
- **Related section**: Which part of the paper it refers to

### 1.3 Classification Rules

| Category | Criteria |
|----------|----------|
| **Major** | Requires new experiments, significant rewriting, methodological changes, or addresses fundamental flaws |
| **Minor** | Clarification, additional discussion, minor analysis, reference additions |
| **Editorial** | Typos, grammar, formatting, figure quality |

**Action type tags** (assign one or more to each comment for structured tracking):

| Action Type | Description |
|-------------|-------------|
| `clarification` | Rewrite existing text for clarity |
| `experiment` | Run new experiments, add results/figures/tables |
| `analysis` | Add ablations, statistical tests, or comparisons |
| `structural` | Move, merge, or split sections |
| `citation` | Add or correct references |

**Priority escalation rules:**
1. **Editor comments** → Always highest priority, process first
2. **Cross-reviewer concerns** (2+ reviewers raise the same issue) → Escalate to highest priority regardless of original category
3. Within same priority level: Major → Minor → Editorial

### 1.4 Generate `revision-overview.md`

Write the overview document in **Chinese** using this structure:

```markdown
# 审稿意见总览

## 基本信息
- 论文标题：[从论文中提取]
- 审稿人数量：[N]
- 意见总数：[N] (Major: X / Minor: Y / Editorial: Z)
- 编辑意见：[有/无]

## ⚠️ 共同关切点（最高优先级）
<!-- 2+ 位审稿人提到的相同问题 -->

| # | 关切点 | 提出者 | 类别 | 相关章节 |
|---|--------|--------|------|----------|
| 1 | ...    | R1, R3 | Major | §3.2 |

## 📋 编辑意见
<!-- 如有 -->

| # | 意见内容 | 类别 |
|---|----------|------|
| 1 | ...      | ...  |

## 审稿人 1
### Major
| # | 意见内容 | 相关章节 |
|---|----------|----------|
| 1 | ...      | ...      |

### Minor
...

### Editorial
...

## 审稿人 2
...
```

After generating, tell the user: "审稿意见解析完成，请查看 `revision-overview.md`。准备好后我们进入逐条讨论阶段。"

## Phase 2: Strategy — 逐条讨论修改策略

### 2.1 Read the Paper

Read the paper PDF using the `Read` tool (batch by pages if needed). Build understanding of:
- Paper structure (sections, key arguments, methodology)
- Figures and tables referenced by reviewers
- Current limitations acknowledged by authors

### 2.2 Interactive Discussion

Process comments in priority order:
1. Cross-reviewer concerns (共同关切点)
2. Editor comments
3. Major comments
4. Minor comments
5. Editorial comments (batch these, no individual discussion needed)

For each non-editorial comment, present to the user via `AskUserQuestion`:

**Display format:**
```
📌 [Reviewer X - Comment Y] (Category)

审稿意见：
> [Original comment text]

论文相关位置：
> [Quote or reference from paper, with section number]

建议修改方向：
[Your suggested revision approach based on understanding of the paper]
```

**Options:**
- "接受建议方向" — Adopt the suggested approach as-is
- "调整方向" — User wants to modify the approach (follow up to get their preferred direction)
- "跳过（稍后处理）" — Skip for now, revisit later

For editorial comments, present them as a batch:
- "以下 N 条编辑意见（拼写、语法、格式）将统一处理，无需逐条讨论。是否同意？"

### 2.3 Record Decisions

Maintain an internal record of each decision. Track:
- Comment ID
- Chosen strategy
- User's notes (if any)
- Skipped items (remind user at the end)

After all comments are discussed, if any were skipped:
- "以下意见被跳过，是否现在处理？" List skipped items.

## Phase 3: Summary — 生成修改计划与回复信

### 3.1 Generate `revision-plan.md` (Chinese)

Organize by paper sections. Write using the `Write` tool:

```markdown
# 修改计划

> 基于审稿意见讨论结果生成，用于追踪修改进度。

## LaTeX 修改标记
> 在修改论文时，使用以下 LaTeX 命令标记所有改动，方便审稿人快速定位：
> ```latex
> \usepackage{xcolor}
> \newcommand{\revised}[1]{{\color{blue}#1}}
> \newcommand{\added}[1]{{\color{blue}#1}}
> \newcommand{\deleted}[1]{{\color{red}\sout{#1}}}
> ```

## §1 Introduction
- [ ] [R1-C2, R3-C1] 补充研究动机的论述（共同关切）`clarification`
  - 策略：在第二段增加...
- [ ] [R2-C5] 修正文献引用格式 `citation`
  - 策略：统一为...

## §2 Methods
- [ ] [R1-C1] 补充实验细节（Major）`experiment`
  - 策略：添加...

## §3 Results
...

## 全局修改
- [ ] 英文润色（Editorial 类意见统一处理）
- [ ] 图表质量提升

## 跳过的意见
<!-- 用户选择跳过的意见，供后续参考 -->
- [R2-C3] ...（原因：...）
```

### 3.2 Generate `response-letter.md` (English)

Standard academic response letter format. Write using the `Write` tool:

```markdown
# Response to Reviewers

Dear Editor and Reviewers,

We sincerely thank you for your constructive comments and suggestions, which have significantly improved our manuscript. Below we provide a point-by-point response to each comment. Reviewer comments are shown in **bold**, and our responses follow.

---

## Response to Editor

**Editor Comment 1:** [Original comment]

[Response explaining what was done and why]

---

## Response to Reviewer 1

**Comment 1 (Major):** [Original comment]

We thank the reviewer for this insightful comment. [Response with specific changes made, referencing sections/pages/figures]

**Comment 2 (Minor):** [Original comment]

[Response]

---

## Response to Reviewer 2
...
```

**Response writing guidelines:**
- Always start with appreciation ("We thank the reviewer for...")
- Be specific: reference exact sections, page numbers, figure numbers
- For accepted changes: describe what was changed and where
- For partially accepted: explain what was adopted and why other parts were not
- For declined suggestions: provide respectful, evidence-based justification
- Use phrases like "We have revised...", "As suggested, we now...", "We respectfully note that..."
- Never be defensive or dismissive
- Cross-reference `revision-plan.md` checkbox items where applicable

After generating both files, tell the user:
"修改计划和回复信已生成。请查看：\n- `revision-plan.md` — 按章节组织的中文修改清单\n- `response-letter.md` — 英文逐条回复信\n\n准备好后进入审阅阶段。"

## Phase 4: Verify — 自查验证

在用户审阅之前，先进行系统性自查，确保修改计划的完整性和一致性。

### 4.1 完整性检查

逐条核对所有审稿意见是否都已在 `revision-plan.md` 和 `response-letter.md` 中得到回应：

生成检查清单并展示给用户：

```markdown
## 修改完整性检查

| 意见编号 | 审稿人 | 类别 | revision-plan | response-letter | 状态 |
|----------|--------|------|:---:|:---:|------|
| R1-M1 | R1 | Major | ✅ | ✅ | 已覆盖 |
| R1-M2 | R1 | Major | ✅ | ✅ | 已覆盖 |
| ... | ... | ... | ... | ... | ... |
```

如有遗漏，立即补充。

### 4.2 一致性检查

确认两份文档之间的一致性：
- `revision-plan.md` 中的每条策略是否与 `response-letter.md` 中的对应回复匹配
- 回复信中引用的章节号、图表号是否与修改计划一致
- 是否有矛盾的承诺（如一处说"已添加"，另一处说"将在未来工作中处理"）

### 4.3 修改前后自评对比

对论文的关键维度进行修改前后的自评打分（1-5 分），帮助用户量化改进效果：

```markdown
## 修改前后自评对比

| 维度 | 修改前 | 修改后（预期） | 说明 |
|------|:---:|:---:|------|
| 方法论清晰度 | 3 | 4 | 补充实现细节和数学公式 |
| 实验充分性 | 2 | 4 | 新增真实实验和基线对比 |
| 数据可信度 | 2 | 4 | 扩大验证样本、补全开源资源 |
| 写作质量 | 3 | 4 | 统一术语、修正拼写 |
| 可复现性 | 1 | 4 | 补全代码和数据集 |
```

### 4.4 潜在风险提示

检查修改是否可能引入新问题：
- 新增内容是否可能超出页数限制
- 新实验结果是否可能与已有结论矛盾
- 修改是否影响论文的核心贡献声明

将检查结果展示给用户，确认无误后进入审阅阶段。

## Phase 5: Review — 用户审阅与修改

### 5.1 User Review

After presenting the documents, wait for user feedback. The user may:
- Request changes to specific response letter entries
- Adjust revision strategies
- Add missing comments that were not captured
- Change the tone or wording of responses

### 5.2 Handling Modifications

When user requests changes:
1. Identify which document(s) need updating (`revision-plan.md` and/or `response-letter.md`)
2. Use the `Edit` tool to make targeted changes — do not regenerate entire files
3. Keep both documents in sync (a strategy change in the plan should reflect in the response letter)
4. Show the user the specific changes made

### 5.3 Completion

When the user is satisfied:
- Confirm all three output files are finalized
- Remind user: "修改计划中的复选框可以在实际修改论文时用来追踪进度。祝修改顺利！"

## Edge Cases

| Situation | Handling |
|-----------|----------|
| Single PDF contains both paper and reviews | Ask user to identify page ranges for each |
| Reviews in non-English language | Process in original language, but `response-letter.md` always in English |
| No editor comments | Skip editor section, note in overview |
| Only 1 reviewer | Skip cross-reviewer analysis, note in overview |
| Very long review (>20 pages) | Read in batches using `pages` parameter |
| Reviewer comments lack clear numbering | Assign sequential numbers and note the mapping |
| User wants to add their own context | Accept via `AskUserQuestion` and incorporate into strategy |
| PDF is scanned/image-based | Inform user that text extraction may be limited; suggest providing a text version |

## Decision Flow

```
/paper-revision triggered
        │
        ▼
┌─── Phase 1: Parse ────────────────────────────────┐
│  Scan PDFs → User confirms roles → Extract &       │
│  classify comments → Generate revision-overview.md  │
└───────────────────────┬────────────────────────────┘
                        ▼
┌─── Phase 2: Strategy ─────────────────────────────┐
│  Read paper → Discuss each comment with user       │
│  (priority order) → Record decisions               │
└───────────────────────┬────────────────────────────┘
                        ▼
┌─── Phase 3: Summary ──────────────────────────────┐
│  Generate revision-plan.md (Chinese)               │
│  Generate response-letter.md (English)             │
└───────────────────────┬────────────────────────────┘
                        ▼
┌─── Phase 4: Verify ───────────────────────────────┐
│  Completeness check → Consistency check →          │
│  Before/after scoring → Risk alerts                │
└───────────────────────┬────────────────────────────┘
                        ▼
┌─── Phase 5: Review ───────────────────────────────┐
│  User reviews → Edit as needed → Done              │
└───────────────────────────────────────────────────┘
```

## Key Rules

- Address every reviewer concern without exception — no comment should be left unresponded
- Preserve paper structure unless explicitly needed otherwise
- Base new results on actual experiments, not fabricated data
- Clearly mark all revised text in LaTeX for reviewer visibility (use `\revised{}` / `\added{}` / `\deleted{}`)
- Language policy: `revision-overview.md` and `revision-plan.md` in Chinese; `response-letter.md` in English
- Compare scores before and after revision to quantify improvement

## Related Skills

This skill works well in combination with:
- **self-review**: Run before submission to catch issues early
- **rebuttal-writing**: For conference-style rebuttals (shorter format than journal response letters)
- **paper-compilation**: For LaTeX compilation and formatting checks after revisions
