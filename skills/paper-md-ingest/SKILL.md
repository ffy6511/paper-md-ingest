---
name: paper-md-ingest
description: Convert research paper titles, arXiv IDs, official paper URLs, or research topics into an Obsidian-friendly `papers/` literature workspace. Use when adding papers, fetching arXiv HTML/source/PDF, generating agent-readable `paper.md`, initializing reading notes with source links and expanded sections, updating `PAPERS.md`, and maintaining Obsidian project maps with wikilinks, embeds, and backlinks under `projects/`.
---

# Paper MD Ingest

## Goal

Add research papers to an Obsidian-friendly `papers/` workspace as Markdown that agents and humans can read efficiently. Store stable paper materials under `papers/library/<paper-id>/`, and store human-facing Obsidian project maps under `papers/projects/`. Store `paper.md` as the primary full-text material and one reading-note `.md` file as the canonical Obsidian note for each paper. Do not store raw HTML, browser `_files/` folders, source archives, or PDFs unless explicitly requested.

This skill assumes Obsidian by default. Use wikilinks, aliases, embeds, backlinks, and lightweight frontmatter as first-class parts of the literature workflow.

## Workflow

1. Inspect the target `papers/` workspace before ingesting. If it is missing or immature, read `references/workspace.md` and initialize missing basics such as `AGENTS.md`, `PAPERS.md`, `library/`, `projects/`, and an optional project map.
2. If the user gives paper ids, titles, or official URLs, resolve each paper directly.
3. If the user only gives a topic, project goal, or research direction, first ask concise follow-up questions until the scope is clear enough to search. If the user already provided a concrete goal, infer reasonable search axes and proceed.
4. For topic-based discovery, search arXiv first. Unless the user specifies a count, curate 5-10 papers. Prefer a balanced set that covers core benchmarks, infrastructure/framework papers, methodology/survey papers, and project-specific risks.
5. Convert the selected papers into a manifest with concise summaries and project relevance in the user's working language before ingesting.
6. Prefer arXiv HTML. Convert it to `paper.md` and rewrite figures as remote Markdown image links.
7. If HTML is unavailable, use arXiv source and convert the TeX tree to `paper.md`.
8. If source is unavailable, extract text from the official PDF into `paper.md`.
9. Create or update one reading-note `.md` file under `library/<paper-id>/` in the user's working language as the canonical Obsidian note for the paper. `NOTES.md` is the default; a short title filename is allowed when it improves human navigation.
10. Add a `source` field in the reading note frontmatter pointing to the best human-readable official source, usually arXiv HTML or official PDF.
11. Update `PAPERS.md` manually after batch generation.
12. If the paper belongs to an active project, update the relevant `projects/<project>.md` map with Obsidian wikilinks and summary embeds.
13. Run `scripts/validate_papers_workspace.py` and fix every broken Markdown link, Obsidian wikilink/embed, missing heading target, missing `paper.md`, missing reading note, and missing `id/source` frontmatter field.

## Directory Shape

```text
papers/
  AGENTS.md
  PAPERS.md
  library/
    <paper-id>/
      <reading-note>.md
      paper.md
  projects/
    <project>.md
```

Use bare arXiv ids without version suffix for directory names, such as `library/2509.16779`. Use a stable slug for non-arXiv papers, such as `library/webaccessbench`. The default note filename is `NOTES.md`; a concise paper-title filename is also acceptable when it improves browsing. Whenever the note filename changes, update every project map and `PAPERS.md`, then run the validator.

For new or partially formed workspaces, use `references/workspace.md` as the initialization reference before ingesting papers.

## Obsidian Literature Workflow

Before creating or updating project maps, or when unsure about wikilinks, embeds, aliases, heading links, frontmatter, or backlinks, read `references/obsidian.md`.

Treat the non-`paper.md` reading note under `library/<paper-id>/` as the canonical Obsidian note for a paper. Treat `projects/<project>.md` as the human and agent reading entry for a project.

Use Obsidian Flavored Markdown in project maps:

```markdown
- [[papers/library/<paper-id>/<reading-note>|<short human title>]]
![[papers/library/<paper-id>/<reading-note>#<summary-heading>]]
```

Use wikilink aliases for readable titles. Use embeds for concise paper summaries. Rely on backlinks so a paper note can reveal which projects, research questions, or design decisions cite it. Use the actual summary heading present in the note, which depends on the user's working language. Do not copy `paper.md` or duplicate long single-paper analysis into project maps.

## Script

Use `scripts/ingest_papers.py` for deterministic fetching and conversion. Resolve the script from the installed or loaded `paper-md-ingest` skill directory; do not assume any fixed install root.

```bash
PY="${PY:-python3}"
PAPER_MD_INGEST_DIR="<path-to-installed-paper-md-ingest>"
ROOT="papers"
"$PY" "$PAPER_MD_INGEST_DIR/scripts/ingest_papers.py" --manifest batch.json --papers-root "$ROOT"
```

After every ingest, rename, or project-map edit, run:

```bash
PY="${PY:-python3}"
PAPER_MD_INGEST_DIR="<path-to-installed-paper-md-ingest>"
ROOT="papers"
"$PY" "$PAPER_MD_INGEST_DIR/scripts/validate_papers_workspace.py" --papers-root "$ROOT"
```

Manifest schema:

```json
[
  {
    "id": "2509.16779",
    "title": "Improving User Interface Generation Models from Designer Feedback",
    "authors": "Wu et al.",
    "year": 2025,
    "status": "key",
    "tags": ["feedback-learning", "designer-feedback", "ui-generation"],
    "source": "arxiv:2509.16779",
    "note_filename": "Improving UI Generation from Designer Feedback.md",
    "summary": [
      "这篇论文解决的问题是：...",
      "它使用的方法是：...",
      "主要效果或发现是：..."
    ],
    "notes": {
      "positioning": "这篇论文在相关方向中的定位：...",
      "background": "问题背景：...",
      "method": "核心方法：...",
      "contributions": [
        "贡献 1：...",
        "贡献 2：...",
        "贡献 3：..."
      ],
      "results": "实验与结果：...",
      "limitations": "局限与边界：..."
    },
    "relevance": "与 Critic Agent 的关系：..."
  }
]
```

For non-arXiv papers, set `source` to an official PDF URL and set `id` to a stable slug. `note_filename` is optional; omit it to use `NOTES.md`, or provide a concise safe filename when a short title improves browsing.

## Topic Discovery Requirements

When starting from a topic or project goal instead of known papers:

- Ask follow-up questions only when the goal is too broad, ambiguous, or missing the target domain.
- If the user has provided a concrete project goal, proceed with best-judgment arXiv search instead of blocking on questions.
- Default to 5-10 papers when the user does not specify a count.
- Record the search framing in the project map so future readers know why these papers were selected.
- Favor official arXiv ids and links. Use non-arXiv official PDFs only when a key source is not on arXiv.
- Do not treat arXiv search ranking as enough; curate for coverage across benchmark design, execution environment, evaluation fairness, reproducibility, transparency, and domain-specific benchmark examples.

## Reading Note Requirements

Each `library/<paper-id>/<reading-note>.md` note must have frontmatter:

```yaml
---
id: <paper-id>
title: "<title>"
year: <year>
status: inbox|to-read|reading|key|done
source: "<official html/pdf/source url>"
tags:
  - <tag>
---
```

The body must use the user's working language, inferred from the request and manifest. Keep the section structure equivalent even when translating headings.

```markdown
## 三句话摘要

这篇论文解决的问题是：...
它使用的方法是：...
主要效果或发现是：...

## 论文定位

这篇论文属于什么方向？它在相关工作中解决哪一类问题？

## 问题背景

作者为什么认为这个问题重要？现有方法、benchmark、系统或评测协议哪里不够？

## 核心方法

论文提出了什么系统、数据集、benchmark、算法、checklist 或评估流程？

## 主要贡献

- ...
- ...
- ...

## 实验与结果

它怎么验证自己的方法？用了哪些 benchmark、baselines、metrics 或案例？结果说明了什么？

## 局限与边界

哪些场景没有覆盖？关键假设是什么？可能有什么偏差、风险或后续待验证点？

## 与当前项目的关系

这篇论文能给当前项目提供什么设计、方法、checklist、反例或风险提示？
```

Keep the three-sentence summary section short and stable because project maps embed it via Obsidian double links. Use the expanded sections for single-paper reading and future agent handoff. If the paper has not been deeply read yet, write a conservative initial note and mark uncertain details in the user's working language rather than overclaiming.

## Project Map Requirements

When a batch is tied to a project, create or update `projects/<project>.md`. Project maps are the primary human/agent reading entry for project-specific literature. They should explain why the papers matter to the project, not duplicate single-paper notes.

Use Obsidian double links to the canonical notes:

```markdown
- [[papers/library/<paper-id>/<reading-note>|<short human title>]]
![[papers/library/<paper-id>/<reading-note>#<summary-heading>]]
```

Use the actual summary heading from the reading note, such as `三句话摘要` for Chinese notes or `Three-Sentence Summary` for English notes. See `references/obsidian.md` for Obsidian wikilink and embed syntax.

Required content:

- A project title and short project goal.
- A core reading path with ordered double links and one-line reasons.
- Project-specific groupings by problem, module, decision, or reading stage.
- Obsidian double links to every referenced paper note.

Strongly recommended content:

- Lightweight frontmatter with `type: note`, `status`, `visibility`, and `literature-map` tags.
- A search framing section when papers were discovered from a topic or project goal.
- Embedded summaries for the most important papers using `![[...#<summary-heading>]]`.
- A project design notes section for decisions, open questions, and follow-up paper gaps.

Do not duplicate `paper.md` or reading notes in project folders. Keep long single-paper analysis in `library/<paper-id>/<reading-note>.md`; keep project maps focused on navigation, synthesis, and project relevance.

## PAPERS.md Requirements

Keep `PAPERS.md` as a lightweight global inventory and audit list, not the primary project reading interface. Project-specific reading should happen in `projects/<project>.md`; `PAPERS.md` exists for de-duplication, quick whole-library scanning, and checking that every library item is indexed. Each entry should include:

```markdown
- [<paper-id> <title>](library/<paper-id>/<reading-note>.md)
  - Tags: `<tag>`, `<tag>`
  - Summary: <two or three concise sentences in the user's working language>
  - Source: <official source URL>
```

Prefer concise entries in `PAPERS.md`. Put richer reading paths, rationale, and synthesis in project maps.

Do not create `papers.jsonl` unless the user explicitly asks for a later indexing phase.

## Validation Requirements

Run the validator after every ingest, note rename, `PAPERS.md` edit, or project-map edit:

```bash
PY="${PY:-python3}"
PAPER_MD_INGEST_DIR="<path-to-installed-paper-md-ingest>"
ROOT="papers"
"$PY" "$PAPER_MD_INGEST_DIR/scripts/validate_papers_workspace.py" --papers-root "$ROOT"
```

The validator must pass before handing off. It checks library structure, reading-note frontmatter, `PAPERS.md` inventory coverage, Markdown links, Obsidian wikilinks/embeds, and embedded heading targets.
