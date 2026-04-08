---
name: wiki-lint
description: Health-check the research wiki for broken links, orphans, stale pages, contradictions, and index sync
---

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
- Contradictions: require human judgment
- Thesis gaps: require research to fill

### Step 5: Log

Append to log.md:

```
## [YYYY-MM-DD] lint | Health check
- Checks run: {list or "all"}
- Issues found: {count}
- Issues fixed: {count}
- Details: {one-line per fix applied}
```
