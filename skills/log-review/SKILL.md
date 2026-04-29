---
name: log-review
description: Weekly review — Compass alignment check, cross-project pattern scan, dashboard regeneration, archive management
---

# Log Review

Weekly review of all research projects. Checks direction alignment,
discovers cross-project patterns, regenerates the dashboard, and manages archives.

Recommended frequency: weekly (~15 minutes).

## Usage

```
/log-review
```

## Arguments

None.

## Instructions

Read `shared/conventions.md` from this plugin directory for format specifications.

### Step 0: Load All Projects

1. Read `~/.research-log/dashboard.md`
2. Parse Project Registry to get all registered projects
3. For each project, read `~/.research-log/{slug}.md`

### Phase 1: Compass Alignment (per project)

For each registered project:

1. Read all Decision Log entries from the **past 2 weeks** (by date in entry headers)
2. For each entry, identify which Compass sub-goal it relates to (from the Context field)
3. Calculate the distribution of work across sub-goals:
   - Count entries per sub-goal
   - Identify any entries that don't map to any sub-goal

4. **Direction drift check**: If more than 40% of recent entries are outside the current focus
   (`← current focus` marker), warn the user:
   > "⚠ ProjectA: 4 of 6 entries in the past 2 weeks are unrelated to current focus (G2.2a).
   > Main work: 2 entries on G1, 2 unclassified.
   > Is this an intentional direction change, or are you lost in the details?"

5. **Progress update**: Based on recent entries, propose sub-goal percentage updates:
   > "Compass update proposal (ProjectA):
   > - G2.2a: 50% → 70% (β-annealing succeeded)
   > - G2: 40% → 45% (auto-computed)
   > Apply these changes?"

6. Apply only after user approval per project.

7. **Core Documents staleness check** (if section exists in `{slug}.md`):
   - Parse `## Core Documents` entries by tier (★★★ Core / ★★ Foundational)
   - For each ★★★ entry, extract the trailing `last YYYY-MM-DD` touch date
   - If touch date is **30+ days** older than today, propose demotion to ★★:
     > "Core Documents staleness (ProjectA):
     > - `outputs/<dir-a>/` (★★★, last 2026-02-15) — 73 days untouched. Demote to ★★ Foundational?
     > - `outputs/<dir-b>/` (★★★, last 2026-03-25) — 35 days untouched. Demote to ★★?
     > Apply these changes?"
   - Also flag: ★★★ entries with status `superseded` (should already be ★★ or removed) and entries exceeding the ≤15 cap.
   - Apply on user approval. Use `flock` for the write.

### Phase 2: Cross-Project Pattern Scan

1. Collect all Decision Log entries from the past 2 weeks across ALL projects
2. Analyze for patterns:
   - **Shared failure modes**: same type of error/failure in different projects
   - **Methodological insights**: a lesson from one project applicable to another
   - **Contradictory approaches**: one project doing the opposite of another in a similar situation
3. For each pattern found, present with evidence:
   > "Cross-project pattern found:
   > ProjectA (03-28): 'Don't add architectural complexity when training strategy already solves the problem'
   > ProjectB (04-07): Attempting complex architecture changes — same mistake?
   >
   > Record in Dashboard?"

4. User approves → add to Cross-Project Observations in `dashboard.md`
5. If no patterns found, skip silently (don't force connections)

### Phase 3: Dashboard Regeneration

Regenerate `dashboard.md` from project file data:

1. **Active Projects**: For each project, read State section and extract:
   - Current stage (from Location field)
   - Status (infer from Blocker: None → 🟢, has blocker → 🟡, no session > 14 days → 🔴)
   - Next milestone (from Compass next incomplete major goal)
   - Last session (from Session timestamp)

2. **Timeline**: Collect all Decision Log headers (date + title) from past 2 weeks across
   all projects, sort by date descending, take top 10.

3. **Preserve**: Keep existing Cross-Project Observations section unchanged
   (these are user-approved, not derived)

4. **Preserve**: Keep existing Project Registry unchanged

5. **Preserve**: Keep each project's `## Core Documents` section in `{slug}.md` intact
   — it is **user-curated** (not derived from State / Decision Log). Phase 1 step 7
   handles its updates separately.

6. Write the regenerated `dashboard.md` with updated date.

### Phase 4: Archive Management

For each project:

1. Count Decision Log entries in main file
2. Count total lines in main file
3. If entries > 30 OR lines > 500:
   > "{slug}.md has {N} Decision Log entries ({L} lines).
   > Move {M} entries older than 6 months to {slug}-decisions-{year}.md?"

4. If user approves:
   - Read entries older than 6 months from main file
   - Append them verbatim to `{slug}-decisions-{year}.md` (create if needed)
   - Remove them from main file
   - Use `flock` for both file operations

### Step Final: Summary

```
Weekly Review Complete
======================

Compass Alignment:
  - ProjectA: on track (G2.2a, 70%)
  - ProjectB: minor drift detected (see warning above)
  - ProjectC: on track (G3.2, paper writing)

Cross-Project Patterns: {N found, M approved}

Dashboard: regenerated

Archive: {N entries archived / no archiving needed}

Next review recommended: {date + 7 days}
```
