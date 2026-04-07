# Research Wiki Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build an LLM-maintained research wiki at `~/.research-wiki/` with three Claude Code skills (`/wiki-ingest`, `/wiki-query`, `/wiki-lint`) and qmd search integration.

**Architecture:** Claude Code plugin (`llm_wiki`) providing three skills that operate on a hierarchical markdown wiki with thesis backbone. qmd MCP server handles search. MAGI auto-suggest via global CLAUDE.md rule.

**Tech Stack:** Markdown (wiki pages), Claude Code plugin system (SKILL.md), qmd (MCP search), glow (terminal viewer)

**Spec:** `docs/superpowers/specs/2026-04-08-research-wiki-design.md`

---

## File Structure

### Plugin project (`~/Documents/Project/AI_Project/llm_wiki/`)

```
llm_wiki/
├── .claude-plugin/
│   └── plugin.json                    # Plugin metadata
├── skills/
│   ├── wiki-ingest/
│   │   └── SKILL.md                   # Ingest pipeline skill
│   ├── wiki-query/
│   │   └── SKILL.md                   # Query skill with qmd
│   └── wiki-lint/
│       └── SKILL.md                   # Lint/health-check skill
├── shared/
│   └── conventions.md                 # Shared page format conventions
└── docs/
    └── superpowers/
        ├── specs/
        │   └── 2026-04-08-research-wiki-design.md
        └── plans/
            └── 2026-04-08-research-wiki.md  (this file)
```

### Wiki data (`~/.research-wiki/`)

```
~/.research-wiki/
├── CLAUDE.md
├── index.md
├── log.md
├── thesis/
├── projects/
├── concepts/
├── methods/
├── findings/
├── papers/
└── raw/
    └── magi/ → ~/Documents/Project/AI_Project/magi-researchers/outputs/
```

---

## Task 1: Plugin Skeleton

**Files:**
- Create: `llm_wiki/.claude-plugin/plugin.json`
- Create: `llm_wiki/shared/conventions.md`

- [ ] **Step 1: Create plugin.json**

```json
{
  "name": "research-wiki",
  "version": "0.1.0",
  "description": "LLM-maintained research wiki with ingest, query, and lint skills",
  "author": {
    "name": "axect",
    "url": "https://github.com/axect"
  },
  "license": "MIT",
  "keywords": ["research", "wiki", "knowledge-base", "science"]
}
```

- [ ] **Step 2: Create shared conventions document**

Write `shared/conventions.md` — the canonical reference for page format that all three skills import. Content:

```markdown
# Wiki Page Conventions

## Wiki Root

`~/.research-wiki/`

## Directory Layout

| Directory | Type | Purpose |
|-----------|------|---------|
| `thesis/` | thesis | Research narrative, central claims, evidence tracking |
| `projects/` | project | Per-project status, know-how, findings |
| `concepts/` | concept | Physics & ML concepts |
| `methods/` | method | Techniques, architectures, what works/fails |
| `findings/` | finding | Experimental results, failures, lessons learned |
| `papers/` | paper | Literature interpretation (not metadata) |
| `raw/` | — | Immutable source symlinks (never modified) |

## Common Frontmatter

Every page has:

    ---
    title: "Page Title"
    type: concept | method | project | finding | paper | thesis
    created: YYYY-MM-DD
    updated: YYYY-MM-DD
    tags: [tag1, tag2]
    projects: [osprey, smeftml]
    sources: []
    ---

## Type-Specific Frontmatter

**project:** `status: active|paused|completed|archived`, `repo: <path>`, `target_venue: <journal>`
**finding:** `result: positive|negative|mixed`, `confidence: high|medium|low`, `experiment: "<desc>"`
**paper:** `arxiv: "<id>"`, `authors: [...]`, `year: <int>`, `relevance: high|medium|low`

## Required Body Sections

| Type | Sections |
|------|----------|
| project | Overview, Current Status, Key Findings, Know-how, Open Questions |
| finding | Context, What Happened, Why, Lesson, Related |
| thesis | Claim, Evidence For, Evidence Against, Implications, Next Steps |
| concept | Definition, Role in Research, Connections, References |
| method | Description, When to Use, Known Pitfalls, Project Usage, References |
| paper | Key Contribution, Relevance to My Research, Connections, Critical Notes |

## Cross-Referencing

- Use `[[type/slug]]` format: `[[projects/osprey]]`, `[[methods/deeponet]]`
- Within same directory, short form allowed: `[[msglaon-failure]]`
- Thesis pages always use full paths
- Only link to existing pages (no red links)

## index.md Format

- One entry per page, under 150 characters
- Grouped by type with count: `## Projects (3)`
- Entry format: `- [[slug]] — one-line description [status]`

## log.md Format

- Reverse-chronological (newest first)
- Entry: `## [YYYY-MM-DD] <op> | <subject>` where op = ingest|query|lint|update
- List pages created/updated and total count
```

- [ ] **Step 3: Create skill directories**

Run:
```bash
mkdir -p ~/Documents/Project/AI_Project/llm_wiki/skills/{wiki-ingest,wiki-query,wiki-lint}
```

- [ ] **Step 4: Verify plugin structure**

Run:
```bash
find ~/Documents/Project/AI_Project/llm_wiki -type f | sort
```

Expected: plugin.json, conventions.md, and this plan + spec visible.

- [ ] **Step 5: Commit**

```bash
cd ~/Documents/Project/AI_Project/llm_wiki
git init
git add .claude-plugin/plugin.json shared/conventions.md
git commit -m "feat: initialize research-wiki plugin skeleton"
```

---

## Task 2: Wiki Directory + Schema

**Files:**
- Create: `~/.research-wiki/CLAUDE.md`
- Create: `~/.research-wiki/index.md`
- Create: `~/.research-wiki/log.md`

- [ ] **Step 1: Create wiki directory structure**

Run:
```bash
mkdir -p ~/.research-wiki/{thesis,projects,concepts,methods,findings,papers,raw}
```

- [ ] **Step 2: Create symlink to MAGI outputs**

Run:
```bash
ln -s ~/Documents/Project/AI_Project/magi-researchers/outputs ~/.research-wiki/raw/magi
```

Verify:
```bash
ls ~/.research-wiki/raw/magi/
```

Expected: List of MAGI output directories.

- [ ] **Step 3: Write CLAUDE.md schema**

Write `~/.research-wiki/CLAUDE.md`:

```markdown
# Research Wiki Schema

This wiki is maintained by Claude Code. The LLM writes and updates all pages.
The user reads, browses, and directs.

## Core Principles

1. **The LLM writes; the user reads.** Never ask the user to write wiki pages.
2. **Read index.md first.** Before any query or ingest, read index.md for current wiki state.
3. **One source touches many pages.** A MAGI report ingest typically updates 4-10 pages.
4. **Never hide contradictions.** Mark conflicts explicitly on both pages and update thesis.
5. **No red links.** Every `[[wikilink]]` must point to an existing page.
6. **Never delete pages.** Superseded pages get a notice and removal from index.md.

## Page Format

All pages: YAML frontmatter + markdown body.

Common frontmatter fields: title, type, created, updated, tags, projects, sources.
Type-specific: project adds status/repo/target_venue; finding adds result/confidence/experiment; paper adds arxiv/authors/year/relevance.

Cross-references use `[[type/slug]]` format (e.g., `[[projects/osprey]]`).
Within same directory, short form allowed (e.g., `[[msglaon-failure]]`).
Thesis pages always use full paths. Only link to existing pages.

For full format specification including required body sections per type,
see `~/Documents/Project/AI_Project/llm_wiki/shared/conventions.md`.

## Page Lifecycle

- **Create:** Fill frontmatter → write body → add to index.md → append to log.md
- **Update:** Bump `updated` date → modify body → append to log.md
- **Supersede:** Add `> Superseded by [[type/newer-page]]` → remove from index.md → log

## index.md Rules

- One entry per page, under 150 characters
- Grouped by type with count in heading
- Updated on every ingest/lint operation
- This is the LLM's primary navigation tool

## log.md Rules

- Append-only, reverse-chronological (newest entry at top)
- Format: `## [YYYY-MM-DD] <operation> | <subject>`
- Operations: ingest, query, lint, update
- Each entry lists pages created/updated with total count

## MAGI Auto-Suggest

After a MAGI `/research` pipeline completes and generates output in
`raw/magi/`, suggest running `/wiki-ingest magi <output-path>` to the user.
Wait for user approval before proceeding.
```

- [ ] **Step 4: Write empty index.md**

Write `~/.research-wiki/index.md`:

```markdown
# Research Wiki Index

## Thesis (0)

## Projects (0)

## Concepts (0)

## Methods (0)

## Findings (0)

## Papers (0)
```

- [ ] **Step 5: Write empty log.md**

Write `~/.research-wiki/log.md`:

```markdown
# Research Wiki Log

<!-- Reverse-chronological. Newest entries at top. -->
<!-- Format: ## [YYYY-MM-DD] <op> | <subject> -->

## [2026-04-08] init | Wiki initialized
- Created: CLAUDE.md, index.md, log.md
- Directory structure: thesis/, projects/, concepts/, methods/, findings/, papers/, raw/
- Symlink: raw/magi/ → magi-researchers/outputs/
- Pages touched: 0
```

- [ ] **Step 6: Verify wiki structure**

Run:
```bash
find ~/.research-wiki -type f -o -type l | sort
```

Expected:
```
~/.research-wiki/CLAUDE.md
~/.research-wiki/index.md
~/.research-wiki/log.md
~/.research-wiki/raw/magi -> ~/Documents/Project/AI_Project/magi-researchers/outputs
```

- [ ] **Step 7: Commit**

```bash
cd ~/Documents/Project/AI_Project/llm_wiki
git add docs/
git commit -m "docs: add research wiki design spec and implementation plan"
```

Note: The wiki at `~/.research-wiki/` is outside this repo. It should be initialized as its own git repo:

```bash
cd ~/.research-wiki
git init
git add CLAUDE.md index.md log.md
git commit -m "init: research wiki skeleton"
```

---

## Task 3: /wiki-ingest Skill

**Files:**
- Create: `llm_wiki/skills/wiki-ingest/SKILL.md`

- [ ] **Step 1: Write SKILL.md**

Write `skills/wiki-ingest/SKILL.md`:

````markdown
# Wiki Ingest

Ingest a source into the research wiki at `~/.research-wiki/`.
Reads the source, plans page creation/updates, and executes on user approval.

## Usage

```
/wiki-ingest <mode> <target>
```

## Arguments

- `mode` (required): One of `magi`, `paper`, `finding`, `knowhow`
- `target` (required): Mode-dependent target
  - `magi <path>`: Path to a MAGI output directory (e.g., `outputs/topic_20260408_v1/`)
  - `paper <arxiv-id>`: arXiv paper ID (e.g., `2106.03456`)
  - `finding "<description>"`: Brief description of a finding to record
  - `knowhow <project-name>`: Project name to extract know-how from (e.g., `osprey`)

## Instructions

Follow these steps exactly. Read `~/.research-wiki/shared/conventions.md` equivalent
rules from the wiki's CLAUDE.md before proceeding.

### Step 0: Load Wiki State

1. Read `~/.research-wiki/CLAUDE.md` for operating rules
2. Read `~/.research-wiki/index.md` for current page inventory
3. Note existing pages to avoid duplicates and identify update targets

### Step 1: Read Source (mode-dependent)

**If mode = `magi`:**
1. Read `{target}/report.md` — the main synthesis
2. Read `{target}/brainstorm/synthesis.md` if it exists — cross-model ideas
3. Read `{target}/plan/research_plan.md` if it exists — methodology context
4. Read `{target}/plan/murder_board.md` if it exists — identified risks
5. Scan `{target}/results/` for key output files

**If mode = `paper`:**
1. If arXiv Explorer is available, query its SQLite DB for metadata
2. Otherwise, use web search to fetch paper title, authors, abstract
3. Read the paper content if a PDF/markdown is available in `raw/`

**If mode = `finding`:**
1. The `target` is the description itself
2. Gather context from the current conversation — what experiment was run, what happened, why

**If mode = `knowhow`:**
1. Find the project directory (check `~/Documents/Project/Research/{target}/`,
   `~/Documents/Project/AI_Project/{target}/`, `~/Documents/Project/MachineLearning/{target}/`)
2. Read the project's `CLAUDE.md` for conventions and context
3. Read memory files from `~/.claude/projects/` matching the project path
4. Focus on feedback-type memories (lessons learned, what worked/failed)

### Step 2: Plan

Determine which pages to create and which to update.

**Creation rules:**
- A new concept/method deserves its own page if it's referenced by 2+ other pages or is central to the source
- A new finding always gets its own page
- A paper always gets its own page
- Project pages are created once, then updated

**Update rules:**
- If a finding relates to a project → update that project's Key Findings and Know-how sections
- If new evidence supports/contradicts the thesis → update relevant thesis page
- If a method has new usage data → update that method's Project Usage and Known Pitfalls

Present the plan to the user:

```
Ingest plan for: {source description}

Create:
  - findings/new-finding-slug.md — "one-line description"
  - methods/new-method.md — "one-line description"

Update:
  - projects/osprey.md — add to Key Findings, update Know-how
  - thesis/recoverability.md — new evidence in Evidence For
  - concepts/operator-learning.md — add connection to new method

Pages touched: N
```

### Step 3: Approve

Wait for user approval. The user may:
- Approve as-is → proceed to Step 4
- Adjust emphasis → modify plan accordingly
- Reject → stop

### Step 4: Execute

For each page in the plan:

**Creating a new page:**
1. Determine the correct directory from the page type
2. Write the file with full frontmatter (all fields from conventions)
3. Write the body following the required sections for that type
4. Use `[[type/slug]]` wikilinks to connect to existing pages
5. Only link to pages that exist in index.md

**Updating an existing page:**
1. Read the current page
2. Add new content to the appropriate section
3. Bump the `updated` date in frontmatter
4. Add new wikilinks if the source introduces connections
5. If new content contradicts existing content, mark the contradiction explicitly

### Step 5: Update Index

Read the current `index.md`, then:
1. Add entries for newly created pages under the correct type heading
2. Update the count in each type heading: `## Type (N)`
3. Ensure entries are under 150 characters
4. Format: `- [[slug]] — one-line description [status/relevance]`

### Step 6: Update Log

Prepend a new entry to `log.md` (after the header comments, before existing entries):

```
## [YYYY-MM-DD] ingest | {source description}
- Source: {mode} {target}
- Created: {list of new pages}
- Updated: {list of updated pages}
- Pages touched: {total count}
```

### Step 7: Summary

Report to user:
- Pages created (with links)
- Pages updated (with what changed)
- Any contradictions discovered
- Suggested follow-up actions (e.g., "Consider running /wiki-lint to check cross-references")
````

- [ ] **Step 2: Verify skill file exists**

Run:
```bash
cat ~/Documents/Project/AI_Project/llm_wiki/skills/wiki-ingest/SKILL.md | head -5
```

Expected: `# Wiki Ingest` header visible.

- [ ] **Step 3: Commit**

```bash
cd ~/Documents/Project/AI_Project/llm_wiki
git add skills/wiki-ingest/SKILL.md
git commit -m "feat: add /wiki-ingest skill"
```

---

## Task 4: /wiki-query Skill

**Files:**
- Create: `llm_wiki/skills/wiki-query/SKILL.md`

- [ ] **Step 1: Write SKILL.md**

Write `skills/wiki-query/SKILL.md`:

````markdown
# Wiki Query

Search the research wiki and synthesize an answer from relevant pages.
Optionally file the answer back into the wiki as a new page.

## Usage

```
/wiki-query [--for <mode>] "<question>"
```

## Arguments

- `question` (required): Natural language question about the research
- `--for` (optional): Output mode. One of:
  - `proposal` — presentation/proposal-ready narrative using thesis backbone
  - `paper` — evidence-focused output suitable for paper drafts
  - (default) — general answer

## Instructions

### Step 0: Load Wiki State

1. Read `~/.research-wiki/CLAUDE.md` for operating rules
2. Read `~/.research-wiki/index.md` for page inventory

### Step 1: Parse Question

Extract from the question:
- Key concepts and entities
- Related projects (if any)
- Target page types (e.g., a "what went wrong" question targets findings)
- Whether this is a narrative, comparison, lookup, or evidence-collection question

### Step 2: Find Relevant Pages

**From index.md:** Scan entries for keyword matches and related pages.

**From qmd (if available):** Call MCP tool `qmd_query` with the question as search query.
Merge results with index.md findings, deduplicate.

If qmd is not available, rely on index.md scanning only.
This is acceptable — qmd enhances recall but is not required.

### Step 3: Read Pages

Read the relevant pages identified in Step 2.
Limit to 10-15 pages to stay within context.
Prioritize by:
1. Pages directly matching the question's subject
2. Thesis pages (if `--for proposal` or `--for paper`)
3. Project pages related to the question
4. Finding/method pages that provide evidence

### Step 4: Synthesize Answer

**Default mode:** Choose format based on question type:

| Question Type | Format |
|---------------|--------|
| Narrative summary | Markdown prose with section headers |
| Comparison | Table with pros/cons/notes columns |
| Know-how lookup | Bullet list with context |
| Evidence collection | Structured list with `[[page]]` citations |

**`--for proposal` mode:**
1. Start from `thesis/overview.md` as the narrative backbone
2. Weave in evidence from project and finding pages
3. Structure as: Context → Research Program → Key Results → Impact → Next Steps
4. Output is compatible with `/claude-typst-slides` schema input

**`--for paper` mode:**
1. Focus on evidence and citations
2. Structure as: Background → Method → Results → Discussion
3. Include `[[page]]` references that can be expanded into proper citations

### Step 5: Cite Sources

Every claim in the answer must reference at least one wiki page using `[[type/slug]]`.
If a claim cannot be supported by any wiki page, flag it as unsupported.

### Step 6: Offer Filing

If the answer has reuse value (comparison tables, evidence compilations,
narrative summaries), offer to file it:

> "This answer could be useful as a wiki page. Save as `concepts/deeponet-vs-fno.md`?"

If user approves:
1. Write the page with proper frontmatter
2. Update index.md
3. Append to log.md with operation `query`

### Step 7: Log

Append to log.md:

```
## [YYYY-MM-DD] query | {question summary}
- Read: {list of pages consulted}
- Filed: {page path if filed, otherwise "inline answer"}
```
````

- [ ] **Step 2: Verify skill file**

Run:
```bash
cat ~/Documents/Project/AI_Project/llm_wiki/skills/wiki-query/SKILL.md | head -5
```

Expected: `# Wiki Query` header visible.

- [ ] **Step 3: Commit**

```bash
cd ~/Documents/Project/AI_Project/llm_wiki
git add skills/wiki-query/SKILL.md
git commit -m "feat: add /wiki-query skill"
```

---

## Task 5: /wiki-lint Skill

**Files:**
- Create: `llm_wiki/skills/wiki-lint/SKILL.md`

- [ ] **Step 1: Write SKILL.md**

Write `skills/wiki-lint/SKILL.md`:

````markdown
# Wiki Lint

Health-check the research wiki. Scans all pages for structural issues,
reports problems, and fixes them on user approval.

## Usage

```
/wiki-lint [--fix] [--check <check-name>]
```

## Arguments

- `--fix` (optional): Auto-fix issues without individual approval (user still sees report)
- `--check` (optional): Run only a specific check. One of:
  `broken-links`, `orphans`, `stale`, `missing`, `contradictions`, `thesis-gaps`, `index-sync`

## Instructions

### Step 0: Load Wiki State

1. Read `~/.research-wiki/CLAUDE.md`
2. Read `~/.research-wiki/index.md`

### Step 1: Scan All Pages

Use Glob to find all `.md` files under `~/.research-wiki/` (excluding `raw/` and `CLAUDE.md`).
Read each page's frontmatter and body. Build an in-memory map:
- `pages`: Map of slug → {type, title, links_to[], linked_from[], updated, frontmatter}
- `index_entries`: Set of slugs listed in index.md

### Step 2: Run Checks

Run all checks (or the single check specified by `--check`):

**Check 1: broken-links**
For each `[[target]]` wikilink in every page, verify that the target file exists.
- Short-form links `[[slug]]` resolve within the same directory
- Full-form links `[[type/slug]]` resolve from wiki root
- Report: list of `{page} → [[broken-target]]`

**Check 2: orphans**
Find pages that are:
- Not listed in index.md AND
- Not linked from any other page
- Exclude: CLAUDE.md, index.md, log.md
- Report: list of orphan page paths

**Check 3: stale**
Find project pages where:
- `status` is `active` AND
- `updated` date is more than 90 days old
- Report: list of stale pages with last update date

**Check 4: missing**
Find concepts/methods/entities that are:
- Mentioned (as text, not wikilink) in 3 or more pages BUT
- Have no dedicated page
- Report: list of candidate page titles with mention count

**Check 5: contradictions**
Scan for pages that make conflicting claims about the same subject:
- Look for finding pages with opposite `result` values for related experiments
- Look for method pages recommending different approaches for the same problem
- Look for thesis evidence that appears in both "Evidence For" and "Evidence Against"
- Report: list of potential contradiction pairs with excerpts

**Check 6: thesis-gaps**
For each thesis page, check:
- Evidence items that have no `sources` in frontmatter
- Claims that are not supported by any finding or paper page
- Report: list of unsupported claims per thesis page

**Check 7: index-sync**
Compare actual files vs index.md entries:
- Files that exist but are not in index.md
- index.md entries whose target files don't exist
- Incorrect counts in type headings
- Report: list of sync issues

### Step 3: Report

Present results grouped by check:

```
Wiki Lint Report
================

Broken Links (2 issues):
  - findings/msglaon-failure.md → [[methods/gated-adapter]] (file not found)
  - projects/osprey.md → [[concepts/pbh-spectrum]] (file not found)

Orphan Pages (1 issue):
  - methods/old-approach.md (not in index, no inbound links)

Stale Pages (0 issues): All clear

Missing Pages (1 suggestion):
  - "Gated Adapter" mentioned in 3 pages, no dedicated page

Contradictions (0 issues): All clear

Thesis Gaps (1 issue):
  - thesis/recoverability.md: "SVD spectrum predicts..." has no source

Index Sync (1 issue):
  - findings/new-finding.md exists but not in index.md

Total: 5 issues, 1 suggestion
```

### Step 4: Fix

If `--fix` was passed, fix all issues automatically.
Otherwise, present each fixable issue and ask for approval:

**Fixable issues and their fixes:**

| Issue | Fix |
|-------|-----|
| Broken link | Create a stub page for the target, or remove the link |
| Orphan page | Add to index.md |
| Stale page | Read project's current state and update the page |
| Missing page | Create a new page with basic content |
| Index sync (missing entry) | Add entry to index.md |
| Index sync (stale entry) | Remove entry from index.md |
| Index sync (wrong count) | Recalculate and update count |

**Non-fixable issues (report only):**
- Contradictions — require human judgment
- Thesis gaps — require research to fill

### Step 5: Log

Append to log.md:

```
## [YYYY-MM-DD] lint | Health check
- Checks run: {list or "all"}
- Issues found: {count}
- Issues fixed: {count}
- Details: {one-line per fix applied}
```
````

- [ ] **Step 2: Verify skill file**

Run:
```bash
cat ~/Documents/Project/AI_Project/llm_wiki/skills/wiki-lint/SKILL.md | head -5
```

Expected: `# Wiki Lint` header visible.

- [ ] **Step 3: Commit**

```bash
cd ~/Documents/Project/AI_Project/llm_wiki
git add skills/wiki-lint/SKILL.md
git commit -m "feat: add /wiki-lint skill"
```

---

## Task 6: Tool Integration

**Files:**
- Modify: `~/.claude/CLAUDE.md` (add MAGI auto-suggest rule)

- [ ] **Step 1: Install qmd**

Run:
```bash
claude marketplace add tobi/qmd
```

If marketplace install fails, fall back to manual MCP config.
Check if qmd binary exists after install:
```bash
which qmd || echo "qmd not found in PATH"
```

- [ ] **Step 2: Configure qmd for wiki**

After qmd is installed, it needs to know about the wiki directory.
Check qmd documentation for configuration:
```bash
qmd --help
```

The key configuration: point qmd at `~/.research-wiki/` as a collection,
excluding `raw/` directory.

- [ ] **Step 3: Install glow (terminal viewer)**

Check if glow is already installed:
```bash
which glow || echo "glow not installed"
```

If not installed:
```bash
# Arch Linux (user's OS)
sudo pacman -S glow
```

Note: This requires sudo. If unavailable, tell the user to run manually.

- [ ] **Step 4: Add shell alias**

Tell the user to add to their fish config (`~/.config/fish/config.fish`):

```fish
alias rw='glow ~/.research-wiki'
```

- [ ] **Step 5: Add MAGI auto-suggest rule to global CLAUDE.md**

Read `~/.claude/CLAUDE.md` first. Then append the following rule to the
appropriate section:

```markdown
## Research Wiki

- After completing a MAGI `/research` pipeline that generates output in `magi-researchers/outputs/`, suggest running `/wiki-ingest magi <output-path>` to ingest findings into the research wiki at `~/.research-wiki/`.
- Wait for user approval before ingesting.
```

- [ ] **Step 6: Register plugin in Claude Code**

The plugin needs to be registered so Claude Code discovers the skills.
Add to `~/.claude/settings.json` under `enabledPlugins`:

```json
"research-wiki@local": true
```

Or use the Claude Code plugin install mechanism:
```bash
claude plugin add ~/Documents/Project/AI_Project/llm_wiki
```

Verify skills are discoverable by checking Claude Code's skill list.

- [ ] **Step 7: Commit**

```bash
cd ~/Documents/Project/AI_Project/llm_wiki
git add -A
git commit -m "feat: complete plugin setup with all three skills"
```

---

## Task 7: Bootstrap — Wiki Initialization

This task populates the wiki with initial content from existing projects.
It uses the skills built in Tasks 3-5.

**Files:**
- Create: `~/.research-wiki/thesis/overview.md`
- Create: `~/.research-wiki/thesis/recoverability.md`
- Create: `~/.research-wiki/thesis/open-questions.md`
- Create: `~/.research-wiki/projects/osprey.md`
- Create: `~/.research-wiki/projects/smeftml.md`
- Create: `~/.research-wiki/projects/vatd.md`
- Update: `~/.research-wiki/index.md`
- Update: `~/.research-wiki/log.md`

- [ ] **Step 1: Bootstrap thesis layer**

This is an interactive step. Run `/wiki-ingest knowhow` conceptually, but since
the thesis layer requires conversation with the user, do this manually:

1. Read the user's research statement from:
   - `~/Documents/Project/Career/PostDoc/research_statement/`
   - `~/Documents/Project/Career/SPDR2027/`
2. Read the user's profile at:
   - `~/Documents/Project/Career/CareeCulum/career_docs/profile_comprehensive_en.md`
3. Draft `thesis/overview.md` covering the "AI ↔ Physics" research program
4. Draft `thesis/recoverability.md` with the central conjecture
5. Draft `thesis/open-questions.md` with current unknowns
6. Present drafts to user for review and refinement

Frontmatter for thesis/overview.md:
```yaml
---
title: "AI for Physics: Research Overview"
type: thesis
created: 2026-04-08
updated: 2026-04-08
tags: [ai-for-physics, operator-learning, inverse-problems]
projects: [osprey, smeftml, vatd]
sources: []
---
```

- [ ] **Step 2: Bootstrap project pages**

For each project (OSPREY, SMEFTML, VaTD), run the equivalent of
`/wiki-ingest knowhow <project>`:

1. Read the project's CLAUDE.md
2. Read memory files from `~/.claude/projects/` for that project
3. Generate the project page with all required sections
4. Focus on Know-how section — extract from memory/feedback files

Example frontmatter for projects/osprey.md:
```yaml
---
title: "OSPREY: PBH Hawking Radiation Neural Operator"
type: project
created: 2026-04-08
updated: 2026-04-08
tags: [pbh, hawking-radiation, operator-learning, deeponet]
projects: [osprey]
sources: []
status: active
repo: ~/Documents/Project/Research/OSPREY
target_venue: PRL/PRD
---
```

- [ ] **Step 3: Ingest 2-3 MAGI reports**

Select the most substantive MAGI reports from `~/.research-wiki/raw/magi/`.
Good candidates (based on earlier exploration):

1. `magi_methodology_evaluation_20260311_v1` — evaluates MAGI itself
2. `magi_harness_strengthening_review_20260326_v1` — improvement analysis

Run `/wiki-ingest magi <path>` for each selected report.
This will auto-generate concept, method, and finding pages.

- [ ] **Step 4: Update index.md**

After all bootstrap pages are created, verify index.md reflects all pages.
Run `/wiki-lint --check index-sync` to catch any mismatches.

- [ ] **Step 5: Run initial lint**

Run `/wiki-lint` to verify overall wiki health:
- All wikilinks resolve
- No orphan pages
- index.md counts are correct

- [ ] **Step 6: Commit wiki state**

```bash
cd ~/.research-wiki
git add -A
git commit -m "feat: bootstrap wiki with thesis, projects, and initial MAGI ingest"
```

---

## Execution Notes

- **Tasks 1-2** are independent and can run in parallel
- **Tasks 3-5** (skills) are independent and can run in parallel
- **Task 6** depends on Tasks 1-5 (needs plugin + skills to exist)
- **Task 7** depends on Task 6 (needs skills registered to run bootstrap)
- **Task 7, Step 1** is interactive — requires user conversation for thesis content
- **Task 7, Steps 3-5** use the built skills as their first real test
