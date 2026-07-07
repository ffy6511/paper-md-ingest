# Obsidian Literature Notes

Use this reference when creating or updating `papers/projects/*.md`, writing Obsidian links to paper notes, or deciding how a paper reading-note `.md` file should be referenced from project maps.

This workspace treats Obsidian features as part of the literature workflow, not as decoration. Use wikilinks to express relationships, embeds to surface short summaries, and backlinks to let a paper reveal which projects cite it.

## Wikilinks

Use wikilinks for files inside the vault:

```markdown
[[papers/library/<paper-id>/NOTES]]
[[papers/library/<paper-id>/<Short Title>]]
[[papers/library/<paper-id>/NOTES|<short human title>]]
[[papers/library/<paper-id>/<Short Title>#<heading>]]
```

Use aliases for readability. The file can remain `NOTES.md`, or it can use a concise short title filename. The project map should display a paper title or short name:

```markdown
[[papers/library/2408.04682/ToolSandbox|ToolSandbox]]
```

Use ordinary Markdown links only for external URLs:

```markdown
[arXiv HTML](https://arxiv.org/html/2408.04682)
```

## Embeds

Prefix a wikilink with `!` to embed note content inline:

```markdown
![[papers/library/<paper-id>/<reading-note>#^summary]]
```

Embed only compact sections in project maps. The normal target is the summary block of the reading note.

Use a fixed `^summary` block id at the end of the last sentence in the summary section. The project map already includes the file path and paper title, so paper-specific block ids are unnecessary.

```markdown
主要效果或发现是：... ^summary

- [[papers/library/2507.02825/Best Practices for Agentic Benchmarks|Agentic Benchmark Checklist]] - Use this first to define evaluation rigor![[papers/library/2507.02825/Best Practices for Agentic Benchmarks#^summary]]
```

Do not embed full `paper.md` into a project map.

## Paper Note Pattern

Each paper directory has one canonical Obsidian reading note:

```text
papers/library/<paper-id>/<reading-note>.md
papers/library/<paper-id>/paper.md
```

The reading note is for human and agent orientation. `paper.md` is the fuller converted paper body. `NOTES.md` is the default filename, but a concise paper-title filename is acceptable when it helps browsing. After renaming a note, update project maps and `PAPERS.md`, then run the validator.

Recommended frontmatter:

```yaml
---
id: <paper-id>
title: "<paper title>"
year: <year>
status: inbox|to-read|reading|key|done
source: "<official html/pdf/source url>"
tags:
  - <tag>
---
```

`source` should point to the best official human-readable source, usually arXiv HTML when available. This makes it easy to open the original source for browser annotation and close reading.

## Project Map Pattern

Each project map lives under:

```text
papers/projects/<project>.md
```

Project maps are reading maps, not duplicate paper notes. They should contain:

- project goal
- search framing when papers were discovered from a topic
- `主题聚合`: paper groups by problem, module, decision, or reading stage
- `推荐阅读路径`: ordered wikilinks with one-line reasons, placed after `主题聚合`
- inline block embeds for every paper listed under `主题聚合`
- `阅读关注点` only when useful, for close-reading cues, open questions, or follow-up paper gaps

Example:

```markdown
# <Project Title>

## Project Goal

...

## 主题聚合

### Evaluation Rigor

- [[papers/library/2507.02825/Best Practices for Agentic Benchmarks|Agentic Benchmark Checklist]] - Use this first to define evaluation rigor![[papers/library/2507.02825/Best Practices for Agentic Benchmarks#^summary]]

### Stateful Sandboxes

- [[papers/library/2408.04682/ToolSandbox|ToolSandbox]] - Read next for stateful sandbox design![[papers/library/2408.04682/ToolSandbox#^summary]]

## 推荐阅读路径

1. [[papers/library/2507.02825/Best Practices for Agentic Benchmarks|Agentic Benchmark Checklist]] - Use this first to define evaluation rigor.
2. [[papers/library/2408.04682/ToolSandbox|ToolSandbox]] - Read next for stateful sandbox design.
```

## Backlinks

Backlinks are the reason to prefer wikilinks over plain Markdown links for internal notes. When a project map links to a paper note, Obsidian can show that project in the paper note's backlinks.

This matters because one paper may support multiple projects or research questions. Do not force this relationship into directory structure by copying paper notes into project folders.

## Do Not

- Do not copy `paper.md` into project folders.
- Do not use ordinary Markdown links for internal paper notes.
- Do not embed long sections or full paper bodies in project maps.
- Do not hard-code one language for headings; use the user's working language and link to the actual heading present in the note.
- Do not rename a note without updating every wikilink, embed, and `PAPERS.md` entry, then running `scripts/validate_papers_workspace.py`.
