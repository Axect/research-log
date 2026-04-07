---
name: wiki-query
description: Search the research wiki and synthesize answers from relevant pages, with proposal and paper modes
---

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
