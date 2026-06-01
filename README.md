<h1 align="center">research-log</h1>

<p align="center">
  <strong>v0.4.0 — your research advisor, not just a diary</strong><br/>
  <em>Surfaces anti-patterns before you commit, recalls prior solutions when stuck, flags drift from your goals in real time.</em>
</p>

<p align="center">
  <a href="https://github.com/Axect/research-log/stargazers"><img src="https://img.shields.io/github/stars/Axect/research-log?style=social" alt="GitHub Stars" /></a>&nbsp;
  <img src="https://img.shields.io/badge/claude--code-plugin-blueviolet" alt="Claude Code Plugin" />&nbsp;
  <img src="https://img.shields.io/badge/license-MIT-green" alt="License: MIT" />
</p>

---

## Why: diary → advisor

A research log that only records the past is half the value. `research-log` acts at three moments where passive logs stay silent:

| Moment | When it fires | What it does |
|--------|--------------|--------------|
| **M1 — decision guardrail** | You propose an architecture, method, or "let's do X" | Reads `rules.md`, surfaces matching anti-patterns *before* you commit — automatically, from conversation |
| **M3 — cross-project recall** | You are stuck or ask "have I solved this before?" | Semantic search (QMD) over journals + lessons across all registered projects |
| **M4 — real-time drift** | You're deep in implementation details | Compares your current work against the `← current focus` marker in `compass.md` and flags divergence |

M1 is a behavioral directive baked into the skill: it fires on any approach-choice in any registered project's working directory, without requiring an explicit command.

---

## Install

```bash
# Add the marketplace and install
claude plugin marketplace add Axect/research-log
claude plugin install research-log
```

Then register your first project inside Claude Code:

```
research-log: initialize my project
```

---

## Storage layout

All data lives in a single unified store at `~/.research/`:

```
~/.research/
├── dashboard.md                 # All projects at a glance (derived, regenerated at review)
├── rules.md                     # Promoted Rules — small, always-loaded M1 checklist
├── projects/
│   └── {slug}/
│       ├── compass.md           # Main Goal / Sub-goals % / Why / Biggest Risk / Core Documents
│       ├── state.md             # Session pointer (overwritten on each save)
│       └── journal.md           # Decision Log — append-only, newest-first, verbatim
├── lessons/
│   └── {lesson-id}.md           # First-class Lesson objects (one principle per file)
├── wiki/                        # Thesis narratives, concept definitions, method descriptions
└── .locks/
    └── {slug}.lock              # Per-project flock file (concurrency safety)
```

Each layer is read differently — design choices follow from that:

| Layer    | Size        | Lifetime        | How it is read                    |
|----------|-------------|-----------------|-----------------------------------|
| Compass  | small       | always current  | always loaded (steering + M4)     |
| State    | tiny        | overwritten     | session restore                   |
| Journal  | large       | append, verbatim| searched, not read whole          |
| Lessons  | small/file  | accumulating    | searched + matched (M1 / M3)      |
| Rules    | tiny        | accumulating    | always loaded (M1 checklist)      |

---

## How to use

`research-log` is a **single skill** that routes to the right workflow based on what you ask. There are no separate `/log-*` commands.

### Workflow triggers

| Intent | Example phrase |
|--------|---------------|
| **initialize** — register a new project | `research-log: set up a new project` |
| **record** — log an experiment or decision | `research-log: record today's experiment` |
| **save-state** — snapshot current session | `research-log: save state` |
| **recall** — find prior lessons or solutions (M3) | `research-log: have I solved this kind of instability before?` |
| **check** — explicitly test a plan against past mistakes (M1) | `research-log: check this approach against past errors` |
| **review** — weekly review, promote lessons, refresh dashboard | `research-log: weekly review` |

### The automatic M1 behavior

You do not need to call `check` for M1 to fire. Whenever you propose an approach in a registered project directory — an architecture choice, a loss function, an experimental design — the skill automatically:

1. Reads `~/.research/rules.md` (small; cheap).
2. Checks whether any Rule's trigger matches the proposed action.
3. If a match is found, surfaces it first: *"You are about to X — this bit you in [projects]: [Check]."*
4. If nothing matches, proceeds silently.

---

## Data model

### Compass (`projects/{slug}/compass.md`)

The project's goal tree: Main Goal, Sub-goals with completion percentages, Why This Approach, Biggest Risk, and exactly one `← current focus` marker that anchors M4 drift detection. The skill proposes percentage updates; you approve before any write.

### State (`projects/{slug}/state.md`)

An 8-field session snapshot (session timestamp, last session, Compass location, what you were working on, current status, blocker, next step, Compass link). Overwritten on each save — never appended.

### Journal (`projects/{slug}/journal.md`)

The Decision Log: append-only, newest-first, kept verbatim. Each entry has Context, Tried/Expected/Got, a Why analysis (root-cause chain + domain reasoning + literature), Conclusion, and a Lesson pointer. The skill drafts entries; you approve before any write.

### Lessons (`lessons/{id}.md`)

The first-class reusable unit extracted from journal entries. Each file captures one generalizable principle with structured frontmatter:

```yaml
---
id: kebab-case-stable-id
title: Short human title
kind: anti-pattern | principle | technique | convention
trigger:
  - phrase matched against proposed actions (M1) and recall queries (M3)
projects: [slug-a, slug-b]      # recurrence ledger; len >= 2 => promote to rule
status: lesson | rule
confidence: low | medium | high
created: YYYY-MM-DD
---
```

Lessons are never archived — they are the long-term cross-project asset.

### Rules (`rules.md`)

A compact projection of every Lesson with `status: rule`. A lesson is promoted to rule when it recurs across ≥ 2 projects. `rules.md` is the M1 engine — small by construction, loaded on every relevant turn:

```markdown
# Research Rules
> Promoted lessons (recurred across >= 2 projects). Consult before agreeing to any approach.

- **lesson-id** — One-line Check question. (project-a, project-b)
```

Regenerated automatically whenever a lesson is promoted (during `record` or `review`).

### Core Documents (optional, in `compass.md`)

An optional user-curated pointer layer to the project's research-frontier artifacts. Appended after the Biggest Risk section in `compass.md`. Two tiers:

- **★★★ Core** — actively cited by current paper substrate or canonical artifact
- **★★ Foundational** — architecture / training-data / methodology references kept for context

```markdown
## Core Documents
> Pointers to current research-frontier artifacts. Updated via skill prompts on user approval.
> Cap: ≤15 entries; demote stale items to ★★ before removal.

### ★★★ Core (current frontier)
- `path/to/artifact` — **role** (artifact inventory) · active canonical · last 2026-05-10

### ★★ Foundational (architecture / training-data / paper-substrate references)
- `path/to/ref` — role · foundational · last 2026-03-01
```

Fields are separated by ` · ` (U+00B7 middle dot) to avoid ambiguity with hyphens and em-dashes.

Maintenance rules:
- Cap of ≤15 entries total. Demote ★★★ → ★★ before removing.
- ★★★ entries untouched for 30+ days are flagged by `review` for demotion.
- User-curated, not derived: skills propose updates and wait for approval; `review` preserves the section intact during dashboard regeneration.
- Read by `recall` as authoritative "what matters now"; proposed updates surfaced by `record` (Step 7); staleness-checked by `review`.

---

## QMD setup and reindex

Semantic recall (M3) and lesson dedup use [QMD](https://github.com/Axect/qmd) to index `~/.research/`. Point a QMD collection at that path with pattern `**/*.md`.

After any significant write (record, review, lesson promotion):

```bash
qmd update && qmd embed
```

Verify the collection is healthy:

```bash
qmd status
```

If QMD is unavailable, the skill falls back to file-based grep over `lessons/` and `journal.md` files — slower but functional.

---

## Migration from 0.2.x / 0.3.x

The storage root moved from `~/.research-log/` to `~/.research/`, and the per-project single-file layout split into three files per project.

What changes:
- `~/.research-log/{slug}.md` → `~/.research/projects/{slug}/compass.md` + `state.md` + `journal.md`
- Decision Log content is preserved verbatim in `journal.md`
- The `## Core Documents` section (if present from 0.3.x) is carried into `compass.md` after Biggest Risk
- Lessons are extracted from journal entries into `lessons/{id}.md`
- The five `/log-*` skills are replaced by the single `research-log` skill

`~/.research-log/` is kept as a **read-only backup** until migration is confirmed. The skill never writes to it.

---

## License

MIT
