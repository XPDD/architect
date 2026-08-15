---
name: exam-note-builder
description: Generate a complete exam study kit from courseware PDFs/讲义 (e.g. 软考/考证 chapters): 康奈尔笔记 (Cornell notes) HTML, interactive 真题 practice-quiz pages, and knowledge maps + E-R entity-relationship diagrams (offline SVG). Use when the user wants to (1) convert exam PDF/讲义 into study notes, (2) build practice-question/做题 pages from 真题, (3) create a knowledge-system or entity-relationship visualization for a chapter/topic, or (4) organize notes with routing/index links.
---

# Exam Note Builder

Build a reusable study kit from exam courseware PDFs. The workflow turns raw PDFs into: **康奈尔笔记** (Cornell notes) → **真题做题页** (interactive quiz) → **知识体系图 + E-R 图** (knowledge map), wired together with an index.

## Workflow

1. **Extract PDF content** with `pdftotext -layout file.pdf out.txt` (poppler). Check for embedded images with `pdfimages -list file.pdf` — banner/watermark images repeat on every page; unique-sized images are content (tables/diagrams that pdftotext misses).

2. **Recognize content images via vision MCP** (e.g. `understand_image`). Extract image files with `pdfimages -j`, resize large ones with `sips --resampleWidth 1000`, then ask the vision tool to extract tables/diagrams/真题. These often hold knowledge the text layer omits (e.g. 内聚/耦合分级表, 遗留系统评价框架).

3. **Generate 康奈尔笔记** per sub-topic. Use `assets/cornell-note.html` as the template. Structure: 线索栏 (questions) / 笔记栏 (numbered sections, tables, vs-blocks, cards) / 费曼白话 / 图示 / 总结栏. Keep each note focused on one sub-topic.

4. **Build the 真题 quiz page** from questions found in text + images. Use `assets/quiz.html`. Interactive: click-to-answer, correct/wrong highlight, 解析, score. Tag each question with a topic so notes can deep-link via `quiz.html#topic`.

5. **Create the knowledge map + E-R diagram** using `assets/knowledge-map.html` (pure inline SVG, no CDN). Structure: lifecycle main flow → sub-nodes → foundation/umbrella module → support modules.

6. **Wire routing** and verify: add note links to `index.html` + study-plan, and cross-links in each note footer. Run a broken-link scan over all HTML (check every `href`/`src` resolves) — fix any 404s.

## Conventions (hard-won lessons)

- **ASCII filenames only** (`se-testing.html`, not `软件测试.html`). Chinese filenames break `file://` relative-link navigation on macOS browsers (404).
- **Self-contained HTML**: inline `<style>` and `<script>`; no CDN for anything critical. Diagrams use pure SVG; fonts may use Google Fonts `@import` (graceful fallback to system fonts if offline).
- **Dark tech theme** for all pages: `--bg:#05080e`, `--card:#111620`, accent `--cyan:#00d4ff`, category colors `#34d399/#a78bfa/#fbbf24/#f87171/#4d7cff`.
- **Each note covers 是什么 + 为什么 + 口诀**: tables/cards for facts, a 费曼 block for plain-language intuition, a mnemonic (口诀) for lists/orderings.
- **真题 explanations cite the trap**: state why wrong options are wrong (e.g. "PAD 支持结构化而非原型化").
- **Verify facts against images**: MCP vision on the PDF's own diagrams beats recalling the mapping from memory (the 遗留系统 评价框架 mapping was wrong until the image was checked).

## Templates

- `assets/cornell-note.html` — 康奈尔笔记 template (full dark-tech CSS + structure; fill in cue questions, numbered sections, 费曼, 图示, summary, tags).
- `assets/quiz.html` — interactive quiz template (single question at a time, score, 解析, topic-based hash deep-linking).
- `assets/knowledge-map.html` — knowledge map + E-R diagram template (pure inline SVG, offline; nodes = entities, edges = relationships with labels).

Copy the relevant template, replace placeholder content, and rename to the topic's ASCII slug.
