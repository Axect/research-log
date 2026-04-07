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

Follow these steps exactly. Read the wiki's CLAUDE.md before proceeding.

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
