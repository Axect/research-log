---
name: wiki-init
description: Initialize the research wiki directory structure at ~/.research-wiki/ with CLAUDE.md, index.md, log.md, and all type directories
---

# Wiki Init

Create the research wiki directory structure at `~/.research-wiki/`.
Safe to run multiple times; skips anything that already exists.

## Usage

```
/wiki-init
```

## Arguments

None.

## Instructions

### Step 1: Check Existing State

Check if `~/.research-wiki/` already exists.

- If it exists AND has `CLAUDE.md`, `index.md`, and `log.md`: report that the wiki is already initialized, show current page count from index.md, and stop.
- If it partially exists (some files missing): proceed and fill in only what's missing.
- If it doesn't exist: proceed with full initialization.

### Step 2: Create Directory Structure

Create the following directories:

```
~/.research-wiki/
  thesis/
  projects/
  concepts/
  methods/
  findings/
  papers/
  raw/
```

### Step 3: Create CLAUDE.md

1. Read `shared/conventions.md` from this plugin's directory to get the page conventions.
2. Write `~/.research-wiki/CLAUDE.md` combining the operating rules below with the conventions content:

```markdown
# Research Wiki

Personal research knowledge base maintained by Claude Code.

## Rules

1. Never modify files in `raw/`. These are immutable source symlinks.
2. Every page must have complete YAML frontmatter (see conventions below).
3. Only link to pages that exist. No red links.
4. Update `index.md` and `log.md` after every operation.
5. When updating a page, always bump the `updated` date in frontmatter.

## Page Conventions

{paste the full content of shared/conventions.md here}
```

### Step 4: Create index.md

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

### Step 5: Create log.md

Write `~/.research-wiki/log.md`:

```markdown
# Research Wiki Log

<!-- Reverse-chronological operation log. Newest entries first. -->
```

### Step 6: Report

```
Wiki initialized at ~/.research-wiki/

Directories: thesis/ projects/ concepts/ methods/ findings/ papers/ raw/
Files: CLAUDE.md, index.md, log.md

Next steps:
  /wiki-ingest magi <path>       Ingest a MAGI report
  /wiki-ingest paper <arxiv-id>  Ingest a paper
  /wiki-query "<question>"       Search the wiki
```
