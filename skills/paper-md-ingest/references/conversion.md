# Conversion Notes

Use this reference only when debugging conversion quality.

## HTML to Markdown

The arXiv HTML endpoint usually contains LaTeXML output with many `ltx_*` classes. Do not store the raw HTML in the papers workspace. Extract readable structure:

- title: `h1`
- sections: `h2`/`h3`/`h4`
- paragraphs: `p`
- figures: Markdown image links using the arXiv HTML page as base URL
- captions: text after each image
- tables: plain text fallback

Remote figure links are acceptable because the workspace is optimized for agent reading and small git diffs, not offline archival.

## TeX Fallback

When HTML is unavailable, download `https://arxiv.org/e-print/<id>`, extract the archive, locate `main.tex` or the most likely root TeX file, recursively inline `\input{}` and `\include{}` when possible, then strip common LaTeX commands into Markdown.

## PDF Fallback

For non-arXiv or PDF-only material, extract text into `paper.md`. Keep the official PDF URL in the `source` frontmatter field.
