---
name: paper-md-ingest
description: Convert research paper titles, arXiv IDs, official paper URLs, or research topics into an Obsidian-friendly `papers/` literature workspace. Use when adding papers, extracting verified public HTML figures into reading notes, fetching arXiv HTML/source/PDF, generating agent-readable `paper.md`, initializing reading notes with source links and expanded sections, updating `PAPERS.md`, and maintaining project maps or cross-project topic indexes with Obsidian links, embeds, and backlinks.
---

# Paper MD Ingest

## Goal

Add research papers to an Obsidian-friendly `papers/` workspace as Markdown that agents and humans can read efficiently. Store stable paper materials under `papers/library/<paper-id>/`, project-specific reading maps under `papers/projects/`, and project-independent aggregation indexes under `papers/topics/`. Store `paper.md` as the primary full-text material and one reading-note `.md` file as the canonical Obsidian note for each paper. Do not store raw HTML, browser `_files/` folders, source archives, or PDFs unless explicitly requested.

This skill assumes Obsidian by default. Use wikilinks, aliases, embeds, backlinks, and lightweight frontmatter as first-class parts of the literature workflow.

## Workflow

1. Inspect the target `papers/` workspace before ingesting. If it is missing or immature, read `references/workspace.md` and initialize missing basics such as `AGENTS.md`, `PAPERS.md`, `library/`, `projects/`, `topics/`, and an optional project map or topic index.
2. If the user gives paper ids, titles, or official URLs, resolve each paper directly.
3. If the user only gives a topic, project goal, or research direction, first ask concise follow-up questions until the scope is clear enough to search. If the user already provided a concrete goal, infer reasonable search axes and proceed.
4. For topic-based discovery, search arXiv first. Unless the user specifies a count, curate 5-10 papers. Prefer a balanced set that covers core benchmarks, infrastructure/framework papers, methodology/survey papers, and project-specific risks.
5. Convert the selected papers into a manifest with concise summaries and project relevance in the user's working language before ingesting.
6. Prefer official arXiv HTML. Convert it to `paper.md` and retain figure images as remote Markdown links. If a legacy paper has no official HTML, a public HTML rendering mirror may supply figures, but the canonical note source remains the official arXiv/PDF URL.
7. If HTML is unavailable, use arXiv source and convert the TeX tree to `paper.md`.
8. If source is unavailable, extract text from the official PDF into `paper.md`.
9. Create or update one reading-note `.md` file under `library/<paper-id>/` in the user's working language as the canonical Obsidian note for the paper. `NOTES.md` is the default; a short title filename is allowed when it improves human navigation. When `paper.md` exposes usable HTML figures, select the one to three that explain the paper's mechanism, protocol, or main result and add them to the note with a concise reading cue; omit decorative or redundant figures.
10. Add a `source` field in the reading note frontmatter pointing to the best human-readable official source, usually arXiv HTML or official PDF.
11. Update `PAPERS.md` manually after batch generation.
12. If the paper belongs to an active project, update the relevant `projects/<project>.md` map with Obsidian wikilinks and summary embeds; then create or update its paired roadmap canvas.
13. If the paper belongs in a project-independent long-term grouping, create or update the relevant `topics/<topic>.md` index instead of inventing a project map.
14. Run `scripts/validate_papers_workspace.py` and fix every broken Markdown link, Obsidian wikilink/embed, missing heading target, missing `paper.md`, missing reading note, and missing `id/source` frontmatter field. For roadmap canvas edits, also validate that the `.canvas` file is parseable JSON and every edge references an existing node.

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
    <project> roadmap.canvas
  topics/
    <topic>.md
```

Use bare arXiv ids without version suffix for directory names, such as `library/2509.16779`. Use a stable slug for non-arXiv papers, such as `library/webaccessbench`. The default note filename is `NOTES.md`; a concise paper-title filename is also acceptable when it improves browsing. Whenever the note filename changes, update every project map and `PAPERS.md`, then run the validator.

For new or partially formed workspaces, use `references/workspace.md` as the initialization reference before ingesting papers.

## Obsidian Literature Workflow

Before creating or updating project maps, topic indexes, reading-note figures, or when unsure about wikilinks, embeds, aliases, heading links, frontmatter, or backlinks, read `references/obsidian.md`. Before converting HTML or choosing figures, read `references/conversion.md`.

Treat the non-`paper.md` reading note under `library/<paper-id>/` as the canonical Obsidian note for a paper. Treat `projects/<project>.md` as the human and agent reading entry for a project; treat `topics/<topic>.md` as a cross-project navigation index, not a second paper store or a project map.

Use Obsidian Flavored Markdown in project maps:

```markdown
- [[papers/library/<paper-id>/<reading-note>|<short human title>]] - <one-line reason>![[papers/library/<paper-id>/<reading-note>#^summary]]
```

Use wikilink aliases for readable titles. Use inline block embeds for concise paper summaries directly on the same bullet as the paper link. In `主题聚合`, every paper bullet must include `![[...#^summary]]`; summary embeds are not optional there. Rely on backlinks so a paper note can reveal which projects, research questions, or decisions cite it. Prefer the stable summary block id `^summary` instead of embedding the whole summary heading section; the file path already identifies the paper, so paper-specific block ids are unnecessary. Do not copy `paper.md` or duplicate long single-paper analysis into project maps.

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

When a run adds or changes reading-note figures, also run the network-backed check:

```bash
"$PY" "$PAPER_MD_INGEST_DIR/scripts/validate_papers_workspace.py" --papers-root "$ROOT" --check-remote-images
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
    "figures": [
      {
        "url": "https://example.org/figure-1.png",
        "alt": "Figure 1: concise accessible description",
        "caption": "图 1：原图表达的机制或实验关系。",
        "takeaway": "阅读提示：这张图应如何帮助理解核心方法。"
      }
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
主要效果或发现是：... ^summary

## 论文定位

这篇论文属于什么方向？它在相关工作中解决哪一类问题？

## 问题背景

作者为什么认为这个问题重要？现有方法、benchmark、系统或评测协议哪里不够？

## 核心方法

论文提出了什么系统、数据集、benchmark、算法、checklist 或评估流程？

核心方法必须是扩展章节，而不是只写一段概括。开头先用 1-3 段概括整体方法、系统或评测流程；随后用 `###` 子标题拆解关键做法，让读者可以先扫总览，再按需深读机制细节。子标题应按论文实际贡献命名，例如 `### 任务实例 schema`、`### 执行式评测 pipeline`、`### 数据污染检测`、`### State-diff contract`、`### Scenario family 生成`。每个子标题下说明输入、处理步骤、输出/判分信号、隐藏信息或质量控制；如果论文涉及 contamination、cheating、资源预算、可复现环境、validator、judge 或 evaluator，这些机制必须在核心方法或紧邻方法章节中记录清楚，不要只放在实验/局限里一笔带过。

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

Keep the three-sentence summary section short and stable. End the last summary line with the fixed block id `^summary` so project maps can embed only the summary content via `![[...#^summary]]`. Use the expanded sections for single-paper reading and future agent handoff. If the paper has not been deeply read yet, write a conservative initial note and mark uncertain details in the user's working language rather than overclaiming.

For mature reading notes, the `## 核心方法` / `## Core Method` section is the most important handoff surface. Do not leave it as a flat one-paragraph summary when the paper's value lies in a concrete benchmark construction, execution protocol, system architecture, safety pipeline, contamination check, or evaluator design. Preserve the standard top-level section structure, but use method subheadings liberally inside this section.

### Key Figure Requirements

Use `## 关键图示` / `## Key Figures` when the paper's public HTML exposes a figure that materially improves understanding. Insert the section after `核心方法` and before contributions. Each selected figure must use a direct, public `https://` image URL from the paper's official HTML or a clearly identified public HTML rendering mirror; never use `data:` URLs, local files, screenshots, logos, UI decoration, or unverified search thumbnails. Verify that the URL returns an image before writing it. Keep the original figure number or caption in a short sentence, then add one Chinese/working-language sentence explaining what to inspect. A paper may have no suitable figure; do not add a placeholder or force an image merely for visual variety.

For manifest-driven generation, pass selected figures through the optional `figures` list. `ingest_papers.py` writes them into the canonical note, but does not choose their semantic relevance: inspect `paper.md` and its captions first. The usual target is one figure; use two or three only when they explain distinct necessary mechanisms.

## Project Map Requirements

When a batch is tied to a project, create or update `projects/<project>.md`. Project maps are the primary human/agent reading entry for project-specific literature. They should explain why the papers matter to the project, not duplicate single-paper notes.

Use Obsidian double links to the canonical notes:

```markdown
- [[papers/library/<paper-id>/<reading-note>|<short human title>]] - <one-line reason>![[papers/library/<paper-id>/<reading-note>#^summary]]
```

Use the fixed `^summary` block id from the reading note. Keep the paper title, one-line reason, and embedded summary on the same bullet. See `references/obsidian.md` for Obsidian wikilink and embed syntax.

Required content:

- A project title and short project goal.
- A paired roadmap canvas embedded near the top of the project map.
- A `主题聚合` section with project-specific groupings by problem, module, decision, or reading stage; every paper listed there must include an inline `![[...#^summary]]` block embed on the same bullet.
- A `推荐阅读路径` section after `主题聚合`, with ordered double links and one-line reasons.
- Obsidian double links to every referenced paper note.

Strongly recommended content:

- Lightweight frontmatter with `type: note`, `status`, `visibility`, and `literature-map` tags.
- A search framing section when papers were discovered from a topic or project goal.
- A `阅读关注点` section only when useful, for what to inspect during close reading, open questions, or follow-up paper gaps. Do not add a generic `设计笔记` section by default.

Do not duplicate `paper.md` or reading notes in project folders. Keep long single-paper analysis in `library/<paper-id>/<reading-note>.md`; keep project maps focused on navigation, synthesis, and project relevance.

## Topic Index Requirements

Use `papers/topics/<topic>.md` when the requested grouping is long-lived and independent of a specific project, such as classics, a method family, or a benchmark category. Do not create a project map merely to hold that grouping.

- Add lightweight frontmatter with `type: topic-index`, `status`, `visibility`, and topic tags.
- Link only to canonical reading notes in `library/`; use `![[...#^summary]]` for compact recall when helpful.
- Organize entries by conceptual dependency or reading order, and state why they belong in the same index.
- Do not create a roadmap canvas, duplicate paper note, or topic-specific raw-material directory.

## Project Roadmap Canvas

Each project map should have a paired roadmap canvas named `papers/projects/<project> roadmap.canvas`. Embed it near the top of the project map, usually after the project goal and before search framing:

```markdown
## Roadmap

![[papers/projects/<project> roadmap.canvas]]
```

Use a roadmap.sh-inspired layout for project roadmaps:

- Put project themes on the vertical center spine. These orange nodes should mirror the `主题聚合` subsections, such as problem areas, platform modules, design decisions, or reading stages. Do not make individual papers the main spine unless the project has only one linear reading list and no meaningful themes.
- Put concrete papers as blue branch nodes under the theme they support. Each paper node should use a vault-relative wikilink to the canonical reading note, with a short alias.
- Add a third-level explanation node for every paper node. The explanation node should state the paper's project role in one sentence, usually reusing the one-line reason from `主题聚合`.
- Use orange main nodes, preferably `#f6a800`; blue branch nodes, preferably `#b9dcff`; and blue branch lines such as `#2f7de1`. Keep top context blocks such as project goal or usage notes in neutral white nodes.
- Use pale blue `#eaf4ff` for third-level explanation nodes. Connect each paper node to its explanation node with a gray line such as `#9ca3af`, set `toEnd: "none"`, and align the two node centers vertically so the edge is horizontal and does not cross other edges.
- Preserve the project-map meaning: theme nodes answer "what design question or module is this cluster about?", and paper branches answer "which paper supplies evidence, patterns, or risks for this theme?"
- Make node widths large enough for the displayed title. Use at least 260-320px for theme nodes and widen paper nodes for Chinese or mixed Chinese/English aliases. If text does not fit, widen the node or split a long branch into multiple concise paper nodes.
- Keep all paths vault-relative. Do not use absolute paths inside `.canvas` or project maps.

## PAPERS.md Requirements

Keep `PAPERS.md` as a lightweight global inventory and audit list, not the primary reading interface. Project-specific reading belongs in `projects/<project>.md`, and cross-project curated paths belong in `topics/<topic>.md`; `PAPERS.md` exists for de-duplication, quick whole-library scanning, and checking that every library item is indexed. Each entry should include:

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

The validator must pass before handing off. It checks library structure, reading-note frontmatter, `PAPERS.md` inventory coverage, Markdown links, Obsidian wikilinks/embeds, and embedded heading targets. `--check-remote-images` additionally fetches selected figures and requires `image/*`; use it whenever figures change.

For roadmap canvas edits, also validate JSON parseability and edge integrity. At minimum, parse the `.canvas` file and check that every `fromNode` and `toNode` exists in `nodes`. When the canvas has third-level explanation nodes, verify their explanation edges are gray, use `toEnd: "none"`, and have matching vertical centers with their second-level paper nodes.
