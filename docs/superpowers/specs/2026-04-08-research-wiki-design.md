# Research Wiki Design Spec

**Author:** Tae-Geun Kim  
**Date:** 2026-04-08  
**Status:** Approved  

## 1. Problem Statement

Five interconnected problems that no existing tool solves together:

1. **Cross-project synthesis** — Connections between OSPREY, SMEFTML, VaTD (e.g., rank analysis ↔ recoverability, operator learning architecture comparisons) live only in conversation history and disappear between sessions.
2. **MAGI output management** — 18+ MAGI reports archived in `outputs/` but key findings are never extracted, connected, or made queryable. Re-reading raw reports is the only way to recall insights.
3. **Thesis tracking** — The central argument ("inverse problem recoverability is predictable from forward map structure") has evidence scattered across multiple projects with no unified view.
4. **Literature synthesis** — Papers tracked in arXiv_explorer by metadata, but interpretations ("what does this paper mean for my research") are not accumulated.
5. **Project know-how** — Architecture choices, hyperparameter lessons, data pipeline pitfalls, and failure records are partially captured in Claude Code memory files but not in a research-accessible form.

## 2. Solution Overview

An LLM-maintained markdown wiki at `~/.research-wiki/` following the Karpathy LLM Wiki pattern, with customizations for a physics+ML research workflow. The LLM writes and maintains all pages; the user reads, browses, and directs.

**Approach:** Existing tools (qmd for search) + custom Claude Code skills (`/wiki-ingest`, `/wiki-query`, `/wiki-lint`).

**Key design decisions:**
- Hierarchical directory structure with a thesis backbone (Approach B)
- Natural overlap with Claude Code memory allowed (memory = LLM behavior rules, wiki = research knowledge in full context)
- Semi-automatic MAGI ingest (auto-suggest after MAGI completion, execute on user approval)
- All wiki content in English for token efficiency
- Query priority: presentation/proposal > paper writing > experiment design > new project > literature review

## 3. Architecture

### 3.1 Directory Structure

```
~/.research-wiki/
├── CLAUDE.md              # Schema — LLM operating rules
├── index.md               # Content catalog (all pages, one-line each)
├── log.md                 # Chronological activity log (append-only)
├── thesis/                # Research narrative (most queried layer)
│   ├── overview.md        # "AI ↔ Physics" big picture
│   ├── recoverability.md  # Central conjecture + evidence tracker
│   └── open-questions.md  # Unresolved questions
├── projects/              # Per-project status + know-how
│   ├── osprey.md
│   ├── smeftml.md
│   └── vatd.md
├── concepts/              # Physics & ML concepts
│   ├── operator-learning.md
│   ├── hawking-radiation.md
│   └── inverse-problems.md
├── methods/               # Techniques, architectures, what works
│   ├── deeponet.md
│   ├── curriculum-learning.md
│   └── splus-optimizer.md
├── findings/              # Experimental results, failures, lessons
│   ├── msglaon-failure.md
│   ├── gbp-power-law-tails.md
│   └── riemann-17pct-normal.md
├── papers/                # Literature interpretation (not metadata)
└── raw/                   # Immutable sources (symlinks)
    └── magi/ → ~/Documents/Project/AI_Project/magi-researchers/outputs/
```

### 3.2 Page Format

All pages: YAML frontmatter + markdown body.

**Common frontmatter:**

```yaml
---
title: "Page Title"
type: concept | method | project | finding | paper | thesis
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [tag1, tag2]
projects: [osprey, smeftml]    # Related projects (optional)
sources: []                     # MAGI reports or papers this page derives from
---
```

**Type-specific additional fields:**

| Type | Additional Fields |
|------|-------------------|
| `project` | `status: active\|paused\|completed\|archived`, `repo: <path>`, `target_venue: <journal>` |
| `finding` | `result: positive\|negative\|mixed`, `confidence: high\|medium\|low`, `experiment: "<description>"` |
| `paper` | `arxiv: "<id>"`, `authors: [...]`, `year: <int>`, `relevance: high\|medium\|low` |

### 3.3 Body Structure by Type

| Type | Required Sections |
|------|-------------------|
| `project` | Overview → Current Status → Key Findings → Know-how → Open Questions |
| `finding` | Context → What Happened → Why → Lesson → Related |
| `thesis` | Claim → Evidence For → Evidence Against → Implications → Next Steps |
| `concept` | Definition → Role in Research → Connections → References |
| `method` | Description → When to Use → Known Pitfalls → Project Usage → References |
| `paper` | Key Contribution → Relevance to My Research → Connections → Critical Notes |

### 3.4 Cross-referencing

- Use `[[type/slug]]` wikilink format with relative path from wiki root (e.g., `[[projects/osprey]]`, `[[methods/deeponet]]`)
- Within the same directory, short form allowed: a finding page can link to `[[msglaon-failure]]` instead of `[[findings/msglaon-failure]]`
- Thesis pages use full paths since they reference across all types
- Only link to existing pages (no red links)
- If a link target should exist but doesn't, create the page or omit the link

## 4. Schema (CLAUDE.md Rules)

The `~/.research-wiki/CLAUDE.md` governs LLM behavior when operating on the wiki.

### 4.1 Core Principles

1. **The LLM writes; the user reads.** Do not ask the user to write wiki pages. The LLM owns all creation and maintenance.
2. **Read index.md first.** Before any query or ingest, read index.md to understand current wiki state.
3. **One source touches many pages.** A single MAGI report ingest typically creates/updates 4-10 pages across multiple directories.
4. **Never hide contradictions.** When new findings conflict with existing claims, mark contradictions explicitly on both pages and update thesis pages.
5. **No red links.** Every `[[wikilink]]` must point to an existing page. Create the target page or omit the link.
6. **Never delete pages.** If content is superseded, add `> ⚠️ Superseded by [[newer-page]]` and remove from index.md.

### 4.2 index.md Convention

- One entry per page, under 150 characters
- Grouped by type with count: `## Projects (3)`
- Each entry: `- [[slug]] — one-line description [status/relevance]`
- Updated on every ingest operation

### 4.3 log.md Convention

- Append-only, reverse-chronological (newest first)
- Entry format: `## [YYYY-MM-DD] <operation> | <subject>`
- Operations: `ingest`, `query`, `lint`, `update`
- Each entry lists pages created/updated and total pages touched
- Parseable with `grep "^## \[" log.md`

### 4.4 Page Lifecycle

- **Create:** Fill all frontmatter fields → write body → add to index.md → append to log.md
- **Update:** Bump `updated` date → modify body → append to log.md
- **Supersede:** Add superseded notice → remove from index.md → append to log.md (never delete the file)

## 5. Workflows

### 5.1 Ingest Workflow

**Trigger:** Semi-automatic. After MAGI `/research` pipeline completes, the system auto-suggests ingest. Also manually invocable.

**Steps:**

```
1. READ    — Read MAGI report (report.md + brainstorm/synthesis.md)
2. SCAN    — Read index.md for current wiki state
3. PLAN    — Determine pages to create/update, present summary to user
             Example: "Create: findings/gbp-tail-fitting.md
                       Update: projects/osprey.md, methods/curriculum-learning.md
                       Pages touched: 3"
4. APPROVE — Wait for user approval (user may adjust emphasis)
5. EXECUTE — Create/update pages
6. INDEX   — Update index.md
7. LOG     — Append to log.md
```

**MAGI artifact → wiki mapping:**

| MAGI Artifact | Wiki Target |
|---------------|-------------|
| `brainstorm/synthesis.md` | New concept/method page candidates |
| `plan/research_plan.md` | Experiment design context → project know-how |
| `plan/murder_board.md` | Risks/limitations → findings or open_questions |
| `results/` | Experimental results → finding pages |
| `report.md` | Key claims → thesis updates; synthesis → project pages |

**Non-MAGI ingest variants:**

| Command | Source | Action |
|---------|--------|--------|
| `/wiki-ingest paper <arxiv-id>` | arXiv paper | Create paper page, update related concepts/thesis |
| `/wiki-ingest finding "desc"` | Conversation | Interactively create finding page |
| `/wiki-ingest knowhow <project>` | Project CLAUDE.md + memory | Extract research insights into project page know-how section |

### 5.2 Query Workflow

**Trigger:** Manual via `/wiki-query "<question>"`.

**Steps:**

```
1. PARSE   — Extract keywords, related projects, type hints from question
2. INDEX   — Read index.md to identify candidate pages
3. SEARCH  — Supplement with qmd MCP search (catches what index missed)
4. READ    — Read relevant pages (max 10-15)
5. ANSWER  — Synthesize answer with source citations
6. FILE?   — If answer has reuse value, suggest filing as wiki page
```

**Output format by question type:**

| Question Type | Format | Example |
|---------------|--------|---------|
| Narrative summary | Markdown prose | "Summarize my overall research" |
| Comparison | Table | "DeepONet vs FNO pros/cons" |
| Know-how lookup | Bullet list | "Curriculum learning pitfalls" |
| Evidence collection | Structured list with citations | "Gather recoverability evidence" |

**Proposal/presentation mode:**

`/wiki-query --for proposal "<topic>"` uses thesis/overview.md as backbone, collects evidence from projects/findings, and outputs a presentation-ready narrative compatible with `/claude-typst-slides`.

### 5.3 Lint Workflow

**Trigger:** Manual via `/wiki-lint`, or periodic.

**Steps:**

```
1. SCAN    — Traverse all pages
2. CHECK   — Run all checks (see table below)
3. REPORT  — Display issues with suggested fixes
4. FIX?    — Execute fixes on user approval
```

**Checks:**

| Check | Description |
|-------|-------------|
| Broken links | `[[target]]` points to non-existent page |
| Orphan pages | Page not in index.md and not linked from anywhere |
| Stale pages | Active project page with `updated` > 90 days old |
| Missing pages | Concept mentioned in 3+ pages but has no own page |
| Contradictions | Pages making conflicting claims on same topic |
| Thesis gaps | Evidence items in thesis pages with empty `sources` |
| Index sync | Mismatch between actual files and index.md entries |

## 6. Tool Integration

### 6.1 qmd (Search)

Install as Claude Code plugin:
```bash
claude marketplace add tobi/qmd
```

Configure collection for wiki:
```yaml
collections:
  - name: research-wiki
    globs:
      - "~/.research-wiki/**/*.md"
    exclude:
      - "~/.research-wiki/raw/**"
```

Used internally by `/wiki-query` skill via MCP tools: `query`, `get`, `multi_get`.

### 6.2 MAGI Integration

Auto-suggest via Claude Code hook after `/research` skill completion. Does not modify magi-researchers codebase. The hook triggers a `/wiki-ingest magi <output-path>` suggestion.

### 6.3 arXiv Explorer Integration

`/wiki-ingest paper <arxiv-id>` fetches metadata from arXiv Explorer's SQLite DB to auto-fill frontmatter. Wiki paper pages focus on interpretation and connections, not metadata (which stays in arXiv Explorer).

### 6.4 Terminal Viewer

`glow` for terminal rendering:
```bash
glow ~/.research-wiki/thesis/overview.md
```

Optional shell alias:
```bash
alias rw='glow ~/.research-wiki'
```

Obsidian as optional GUI viewer — point at `~/.research-wiki/` as vault. Not required.

## 7. Custom Skills

Three Claude Code skills to build:

| Skill | Trigger | Core Action |
|-------|---------|-------------|
| `/wiki-ingest` | Auto-suggest after MAGI / manual | Source → create/update wiki pages |
| `/wiki-query` | Manual | Search wiki → synthesize answer → optional filing |
| `/wiki-lint` | Manual / periodic | Health check → report issues → fix on approval |

## 8. Bootstrap Plan

### Phase 1 — Skeleton (manual, one-time)
- Create directory structure
- Write CLAUDE.md schema
- Create empty index.md and log.md
- Symlink raw/magi/ → magi-researchers/outputs/

### Phase 2 — Thesis layer (interactive)
- Collaborative conversation to draft thesis/overview.md
- Write recoverability.md and open-questions.md

### Phase 3 — Project pages (semi-automatic)
- Extract research context from each project's CLAUDE.md + memory files
- Generate projects/osprey.md, smeftml.md, vatd.md
- Migrate know-how from existing memory/feedback files

### Phase 4 — MAGI output ingest (automatic)
- Ingest 2-3 key MAGI reports from outputs/
- Auto-generate concept, method, finding pages
- Establish initial cross-references

### Phase 5 — Verification
- Run `/wiki-lint` for initial health check
- Verify index.md consistency
- Review graph of connections
