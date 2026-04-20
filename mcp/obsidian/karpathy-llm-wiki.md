# karpathy's LLM wiki system

notes on the LLM Wiki pattern proposed by Andrej Karpathy (published April 4, 2026 as a GitHub gist). the original gist is designed to be copy-pasted into an AI agent so it builds the setup in collaboration with you.

gist: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

---

## the core idea

most people use LLMs with documents via RAG: upload files, retrieve relevant chunks at query time, generate an answer. the problem is that nothing accumulates. every question starts from scratch. complex questions that require synthesizing multiple sources require the LLM to re-discover relationships it found before.

the LLM wiki pattern is different. instead of retrieving from raw documents at query time, the LLM incrementally builds and maintains a persistent wiki: a structured, interlinked collection of markdown files that sits between you and the raw sources.

when you add a new source, the LLM doesn't just index it. it reads it, extracts key information, and integrates it into the existing wiki: updating entity pages, revising topic summaries, noting contradictions with existing claims, strengthening the synthesis. the knowledge is compiled once and kept current.

karpathy's framing: **"obsidian is the IDE. the LLM is the programmer. the wiki is the codebase."**

---

## why this is different from RAG

| | RAG | LLM wiki |
|---|---|---|
| knowledge model | retrieval at query time | compiled, persistent |
| cross-references | re-derived every query | already there |
| contradictions | may be missed | flagged during ingest |
| synthesis | happens on demand | already done |
| compounds over time | no | yes |

the key word is "compounding." every source you add makes the wiki richer. every question you ask can trigger updates. over time it becomes a structured picture of everything you've read and thought about on a topic.

---

## architecture: three layers

### 1. raw sources

your curated collection of source documents: articles, papers, PDFs, transcripts, images, data files. these are immutable. the LLM reads from them but never modifies them. this is your source of truth.

### 2. the wiki

a directory of LLM-generated markdown files. summaries, entity pages, concept pages, comparisons, an overview, a synthesis. the LLM owns this layer entirely. it creates pages, updates them when new sources arrive, maintains cross-references, and keeps everything consistent. you read it; the LLM writes it.

### 3. the schema

a configuration file (`CLAUDE.md` for Claude Code, `AGENTS.md` for Codex) that tells the LLM how the wiki is structured, what the conventions are, and what workflows to follow. this is what makes the LLM a disciplined wiki maintainer rather than a generic chatbot. you and the LLM evolve this over time.

---

## typical folder structure

```
vault/
├── raw/                    # immutable source documents
│   ├── articles/
│   ├── papers/
│   ├── transcripts/
│   └── assets/
├── wiki/
│   ├── sources/            # one summary page per source
│   ├── entities/           # people, orgs, products
│   ├── concepts/           # ideas, themes, frameworks
│   ├── index.md            # directory of everything in the wiki
│   └── log.md              # chronological record of all ingest operations
└── CLAUDE.md               # schema file
```

---

## operations

### ingest

you drop a new source into `raw/` and tell the LLM to process it. a typical flow:

1. LLM reads the source
2. discusses key takeaways with you
3. writes a summary page in `wiki/sources/`
4. updates the index
5. updates relevant entity and concept pages
6. appends an entry to the log

a single source might touch 10-15 wiki pages. you can ingest one at a time (more involvement, better quality) or batch-ingest many at once (less supervision, faster). document the approach you prefer in the schema.

### query

you ask questions against the wiki. the LLM searches relevant pages, reads them, synthesizes an answer with citations. answers can be a markdown page, a comparison table, a structured list, whatever fits the question. the wiki improves as a side effect of answering queries.

### maintain

periodic tasks: pruning outdated pages, resolving contradictions, updating cross-references, re-summarizing sections that have grown stale. you trigger these manually or set them up as scheduled workflows.

---

## use cases karpathy mentions

- **personal**: tracking goals, health, psychology, self-improvement. journal entries, articles, podcast notes compiled into a structured picture of yourself over time.
- **research**: going deep on a topic over weeks or months. building a comprehensive wiki with an evolving thesis.
- **business / team**: internal wiki maintained by LLMs, fed by Slack threads, meeting transcripts, project documents, customer calls.
- **competitive analysis, due diligence, trip planning, course notes, hobby deep-dives**: anything where you're accumulating knowledge over time.

---

## the schema file (CLAUDE.md)

this is the most important part. it defines:

- folder structure and conventions
- page types and their format (source summary vs entity page vs concept page)
- ingest workflow (step-by-step instructions the LLM follows)
- query workflow
- maintenance rules
- what to do with contradictions
- how to handle cross-references

without a good schema, the LLM will be inconsistent across sessions. the schema is what gives the system memory of how it's supposed to behave.

example frontmatter for a source page:

```markdown
---
title: "Article Title"
source: "https://..."
date: 2026-04-04
type: article
tags: [cortex, snowflake, AI]
---
```

---

## practical notes from implementations

- keep raw sources immutable. never let the LLM edit them. if a source is wrong, note it in the wiki.
- one schema file per wiki, not one per session. the LLM reads it at the start of every session.
- for ingest, prefer one source at a time when the topic is important. stay involved. read the summaries the LLM generates.
- for entity pages, the LLM should update them, not recreate them. each update should add information, not overwrite.
- the log file is useful for debugging. if something in the wiki looks wrong, the log shows when it was written and from what source.
- batch ingest is fine for archival or lower-stakes material. for anything you'll actually query later, do it properly.

---

## relationship to other PKM systems

this isn't Zettelkasten, PARA, or "building a second brain" in the Forte sense. those systems require you to do the connecting and maintaining. the wiki pattern delegates that work to the LLM. the bottleneck in most PKM systems isn't the method, it's the maintenance. this pattern removes that bottleneck.

---

## adapting to cortex code CLI

the original gist assumes Claude Code. with Cortex Code CLI the mechanics are slightly different:

- schema file: use `CLAUDE.md` in the project root (Cortex Code reads it by default, same as Claude Code)
- Obsidian connection: via MCP (Local REST API plugin), not direct file access
- Snowflake integration: Cortex Code can pull data from Snowflake and write summaries to the wiki directly

---

## references

- original gist: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- published: April 4, 2026