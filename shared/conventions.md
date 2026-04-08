# Research Log Conventions

## Location

`~/.research-wiki/`

## File Structure

```
~/.research-wiki/
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
- **현재 단계**: {current sub-goal ID and name}
- **상태**: {🟢 진행 중 | 🟡 blocked | 🔴 stalled} — {one-line detail}
- **다음 마일스톤**: {next milestone}
- **마지막 세션**: {YYYY-MM-DD}

## Timeline (최근 2주)
| 날짜 | 프로젝트 | 결정/이벤트 |
|------|---------|------------|
| MM-DD | {project} | {one-line summary} |

## Cross-Project Observations
> 자연 발생한 연결만 기록. /log-review에서 AI가 제안, 사용자 승인 후 추가.
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

- **G2. {title}** [{percentage}%] ← 현재 여기
  - G2.1 {task} [{percentage}%]

### Why This Approach
{Fundamental reasoning for current approach}

### Biggest Risk
{Key risk requiring direction change if realized}
```

**Percentage rules:**
- Leaf-level: set by user (AI proposes, user approves)
- Parent-level: weighted average of children (AI computes)
- `← 현재 여기` marker: only one at a time, indicates current focus

### State Section

```markdown
## State
- **Session**: {YYYY-MM-DDTHH:MM}
- **Last session**: {YYYY-MM-DDTHH:MM}
- **위치**: {G?.? reference to Compass}
- **하고 있던 것**: {what was being worked on}
- **현재 상태**: {current status}
- **블로커**: {blocker or "없음"}
- **다음 할 것**: {next concrete action}
- **Compass 연결**: {how this connects to sub-goal}
```

State is **overwritten** on each session end (not appended).

### Decision Log Section

```markdown
## Decision Log

### YYYY-MM-DD | {title}

**맥락**: {Which Compass goal this falls under}

**시도**: {What was attempted}
**기대**: {What was expected}
**결과**: {What actually happened}

**Why 분석**:
1. {Root cause chain}
2. {Domain-specific explanation}
3. {Literature/empirical evidence}

**결론**: {Next action}
**교훈**: {Generalizable principle — cross-project connection point}
```

Newest entries first. AI drafts Why 분석; user reviews before commit.

## Archive Rules

- Trigger: Decision Log > 30 entries OR main file > 500 lines
- `/log-review` proposes archiving; user approves
- Entries older than 6 months move to `{slug}-decisions-{year}.md`
- Entries preserved verbatim (no summarization)
- `/log-query` searches main + all archive files

## Concurrency: flock Protocol

- Before ANY write to `{slug}.md`: `flock ~/.research-wiki/.locks/{slug}.lock`
- Per-project locks: different projects are fully parallel
- Same project: serialized (one writer at a time)
- Crash safety: OS releases lock on process exit
- Dashboard: only written by `/log-review` (no contention)

## Project Identification

Skills identify the current project by matching CWD against the Project Registry
in `dashboard.md`. Each registry entry maps a slug to an absolute path. CWD is
checked as a prefix match (CWD starts with the registered path).
