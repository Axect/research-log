---
name: log-query
description: Search research logs and answer questions about project state, history, and cross-project patterns
---

# Log Query

Search research logs and synthesize answers.
Reads project files, archive files, and optionally uses QMD for semantic search.

## Usage

```
/log-query "<question>"
```

## Arguments

- `question` (required): Natural language question about research projects, decisions, or history.

## Instructions

Read `shared/conventions.md` from this plugin directory for format specifications.

### Step 1: Parse Question

Determine the scope and type:

- **Project-specific**: mentions a project name → search that project's files
- **Cross-project**: asks about patterns, comparisons, or "all projects" → search all files
- **Timeline**: asks about "recent", "last month", time ranges → filter by date
- **Why/decision**: asks why something was done → target Decision Log entries
- **Status**: asks about current state → target State and Compass sections

### Step 2: Gather Sources

1. Read `~/.research-log/dashboard.md` to identify all projects
2. Based on scope from Step 1:
   - **Project-specific**: read `{slug}.md` + all `{slug}-decisions-*.md` archive files
   - **Cross-project**: read all project files + their archives
   - **Timeline**: read relevant project files, filter entries by date
3. If QMD plugin is available (`mcp__plugin_qmd_qmd__query`):
   - Run a semantic search query against the research-log collection
   - Merge QMD results with the manually gathered sources

### Step 3: Synthesize Answer

Based on question type:

| Type | Format |
|------|--------|
| Why/decision question | Quote relevant Decision Log entry (date, title, Why analysis) |
| Status question | Show State section + Compass progress |
| Timeline question | Chronological table of Decision Log entries |
| Cross-project pattern | Compare entries across projects with citations |
| How-to/lesson question | Collect Lesson fields from relevant entries |

**Citation format**: Always reference the source as `({project}, {date})`.
Example: "MSGLAON was abandoned due to redundancy with curriculum training (osprey, 2026-03-28)."

### Step 4: Answer

Present the synthesized answer to the user. Keep it concise and actionable.

If the answer reveals a potential cross-project connection not yet recorded in dashboard,
mention it: "This pattern can be recorded as a Cross-Project Observation via /log-review."
