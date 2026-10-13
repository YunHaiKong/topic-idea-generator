---
name: topic-idea-generator
description: This skill should be used when a user wants to brainstorm content topics for a content creator account. Trigger phrases include "选题灵感", "选题策划", "给我 100 个选题", "今天写什么", "选题没思路", "topic ideas", and similar requests from self-media authors, brand operators, or content teams. The skill generates a deduplicated, ranked topic table covering 5 creative angles (pain point, curiosity, contrast, numbers, identity) with cross-platform title variants and viral-factor annotations.
agent_created: true
---

# Topic Idea Generator

## Overview

Generate a deduplicated, ranked list of content topics from a single domain input. Each topic ships with cross-platform title variants (公众号 / 小红书 / 抖音) and a one-line "why it might go viral" annotation. Built for self-media authors, brand operators, and content teams who need a starting topic pool in minutes rather than hours.

## When to Use This Skill

Activate this skill when the user wants to:
- Brainstorm topics for an upcoming content batch (daily / weekly / monthly)
- Find new angles when their current topic pool feels stale
- Pre-fill a content calendar with diverse, non-repetitive ideas
- Get cross-platform title variants from a single topic idea

Do **not** activate for: writing full articles, designing visuals, scheduling posts, or analyzing post-publish performance. Hand those off to other skills.

## Inputs

Collect these four details from the user before generating (ask in a single message):

1. **账号定位** — the content domain and niche (e.g., "职场效率工具分享，主攻 AI 工具应用")
2. **目标受众** — demographic + pain points (e.g., "25-35 岁职场人，想提升效率、探索副业")
3. **主攻平台** — 1-3 platforms from {公众号, 小红书, 抖音, B站, 知乎}
4. **数量** — how many topics to output (default 30)

If the user only gives one input (e.g., just a domain), fill the rest with sensible defaults and proceed.

## Workflow

### Step 1 — Parse and normalize inputs

Extract the domain keywords, audience pain keywords, and platform constraints. If anything is ambiguous, ask once before generating.

### Step 2 — Load the reference framework

Read `references/topic-frameworks.md` before generating. It defines:
- The 5 creative angles (痛点 / 好奇 / 反差 / 数字 / 身份)
- The 8 viral-factor genes (情绪共鸣 / 数字锚点 / 反差钩子 / 圈层身份 / 悬念反转 / 实用承诺 / 争议观点 / 热点借势)
- Cross-platform title adaptation rules
- Output table schema

Do not regenerate this framework in conversation; reference it directly.

### Step 3 — Generate topics per angle

For each of the 5 angles, generate 6-8 raw topic candidates. Total raw pool: ~30-40 topics.

Rules during generation:
- Each topic must be tagged with at least one angle.
- Each topic must carry 1-2 viral-factor genes in its annotation.
- Do **not** copy verbatim from any external list; the topics must be freshly composed from the user's inputs.

### Step 4 — Deduplicate and rank

- Drop topics whose core idea overlaps with another; keep the one covering the most angles.
- Rank survivors by: (a) angle coverage count, (b) viral-factor count, (c) platform coverage completeness.
- Output the top N topics per the user's requested count (default 30, capped at 50 to avoid bloat).

### Step 5 — Render the output table

Produce a Markdown table with these columns:

```
| # | 选题（核心标题） | 角度 | 公众号版 | 小红书版 | 抖音版 | 推荐理由 | 预估爆款度 |
```

- Only show columns for platforms the user selected.
- 预估爆款度 ∈ {高, 中高, 中, 低} based on the ranking weights.
- After the table, append a 3-bullet summary of the strongest patterns observed in this batch (e.g., "本批数字锚点选题占比最高，建议优先写第 3、7、12 条").

### Step 6 — Offer next-step hooks

End with a short prompt offering 1-2 follow-ups:
- "需要我挑 Top 5 直接展开成完整大纲吗？"
- "需要按这个选题表导出到 Excel 排期吗？"

Do not perform those steps unprompted.

## Constraints

- **No fabricated data**: do not invent specific statistics, named studies, or quotes in topics. Use placeholder shapes like "X 个工具" or "N 倍效率差" if a concrete number is needed.
- **No platform bias**: if the user names only one platform, still include the other two columns but mark them as "(未选平台，未生成变体)".
- **Diversity floor**: across the final batch, no single angle may exceed 40% of total topics.
- **Length cap**: each topic core title stays under 30 characters; platform variants under platform-specific limits defined in the reference.
- **No repetition of phrasing templates**: track sentence patterns used so far; force at least 4 distinct openers across the batch.

## Resources

### references/topic-frameworks.md

Load this file when generating any topic. It contains the 5 angles, 8 viral genes, cross-platform rules, and the initialization prompt. Do not duplicate its content into conversation; reference it.

### Initialization Prompt

When the skill first activates in a session, output this message verbatim:

```
你好，我是你的选题策划。
请告诉我：
1. 你的账号定位（领域 + 细分方向）
2. 目标受众（年龄/职业/痛点）
3. 主攻平台（公众号/小红书/抖音/B站/知乎，1-3 个）
4. 想要的选题数量（默认 30）

我会按"痛点/好奇/反差/数字/身份"5 个维度给你生成去重排序后的选题表，每条带跨平台标题变体和推荐理由。
```

Then wait for the user to reply before generating.

### Official Demo

See `assets/demo-report.md` for 4 rounds of sample runs covering: basic run, top-5 outline expansion, cross-domain rerun (母婴育儿), and style adjustment (seasonal + 口语化). Use it as the canonical reference for what this skill can output.

## Iteration

After the user reviews the batch, common refinement requests:
- "再来 20 条不同角度" → re-run from Step 3 with a different angle weighting.
- "把 Top 5 展开成大纲" → hand off to a writing/outline skill.
- "风格再口语一点" → re-run Step 5 with platform-style adjustments only.
