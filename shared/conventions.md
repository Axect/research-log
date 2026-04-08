# Research Log Conventions

## Location

`~/.research-log/`

## File Structure

```
~/.research-log/
├── dashboard.md                       # All projects at a glance (derived data)
├── .locks/                            # flock files for concurrency safety
│   └── {slug}.lock                    # One lock per project
├── {slug}.md                          # Compass + State + Decision Log (one per project)
└── {slug}-decisions-{year}.md         # Archived Decision Log entries (when main file grows)
```

## dashboard.md Format

```markdown
# Research Dashboard
> Last updated: YYYY-MM-DD

## Project Registry
<!-- Format: "- {slug}: {absolute_path}" — parsed by skills to map CWD to project -->
- slug1: ~/path/to/project1
- slug2: ~/path/to/project2

## Active Projects

### {PROJECT NAME} — {one-line description}
- **Current stage**: {current sub-goal ID and name}
- **Status**: {🟢 active | 🟡 blocked | 🔴 stalled} — {one-line detail}
- **Next milestone**: {next milestone}
- **Last session**: {YYYY-MM-DD}

## Timeline (last 2 weeks)
| Date | Project | Decision/Event |
|------|---------|------------|
| MM-DD | {project} | {one-line summary} |

## Cross-Project Observations
> Only naturally emerging connections. AI proposes during /log-review; added after user approval.
```

Dashboard is **derived data**: regenerated from project files by `/log-review`.
Hooks never write to dashboard directly.

## Project File ({slug}.md) Format

Three sections: Compass, State, Decision Log.

### Compass Section

```markdown
## Compass

### Main Goal
{1-2 sentence research objective}

### Sub-goals

- **G1. {title}** [{percentage}%]
  - G1.1 {task} [{percentage}%] — {brief status}
  - G1.2 {task} [{percentage}%]

- **G2. {title}** [{percentage}%] ← current focus
  - G2.1 {task} [{percentage}%]

### Why This Approach
{Fundamental reasoning for current approach}

### Biggest Risk
{Key risk requiring direction change if realized}
```

**Percentage rules:**
- Leaf-level: set by user (AI proposes, user approves)
- Parent-level: weighted average of children (AI computes)
- `← current focus` marker: only one at a time, indicates current focus

### State Section

```markdown
## State
- **Session**: {YYYY-MM-DDTHH:MM}
- **Last session**: {YYYY-MM-DDTHH:MM}
- **Location**: {G?.? reference to Compass}
- **Working on**: {what was being worked on}
- **Current status**: {current status}
- **Blocker**: {blocker or "None"}
- **Next step**: {next concrete action}
- **Compass link**: {how this connects to sub-goal}
```

State is **overwritten** on each session end (not appended).

### Decision Log Section

```markdown
## Decision Log

### YYYY-MM-DD | {title}

**Context**: {Which Compass goal this falls under}

**Tried**: {What was attempted}
**Expected**: {What was expected}
**Got**: {What actually happened}

**Why analysis**:
1. {Root cause chain}
2. {Domain-specific explanation}
3. {Literature/empirical evidence}

**Conclusion**: {Next action}
**Lesson**: {Generalizable principle — cross-project connection point}
```

Newest entries first. AI drafts Why analysis; user reviews before commit.

## Archive Rules

- Trigger: Decision Log > 30 entries OR main file > 500 lines
- `/log-review` proposes archiving; user approves
- Entries older than 6 months move to `{slug}-decisions-{year}.md`
- Entries preserved verbatim (no summarization)
- `/log-query` searches main + all archive files

## Concurrency: flock Protocol

- Before ANY write to `{slug}.md`: `flock ~/.research-log/.locks/{slug}.lock`
- Per-project locks: different projects are fully parallel
- Same project: serialized (one writer at a time)
- Crash safety: OS releases lock on process exit
- Dashboard: only written by `/log-review` (no contention)

## Project Identification

Skills identify the current project by matching CWD against the Project Registry
in `dashboard.md`. Each registry entry maps a slug to an absolute path. CWD is
checked as a prefix match (CWD starts with the registered path).
