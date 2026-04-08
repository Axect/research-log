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
2. For each entry, identify which Compass sub-goal it relates to (from the 맥락 field)
3. Calculate the distribution of work across sub-goals:
   - Count entries per sub-goal
   - Identify any entries that don't map to any sub-goal

4. **Direction drift check**: If more than 40% of recent entries are outside the current focus
   (`← 현재 여기` marker), warn the user:
   > "⚠ OSPREY: 최근 2주간 6개 중 4개 항목이 현재 포커스(G2.2a)와 무관합니다.
   > 주요 작업: G1 관련 2건, 미분류 2건.
   > 의도된 방향 전환인가요, 아니면 디테일에 빠진 건가요?"

5. **Progress update**: Based on recent entries, propose sub-goal percentage updates:
   > "Compass 업데이트 제안 (OSPREY):
   > - G2.2a: 50% → 70% (β-annealing 성공)
   > - G2: 40% → 45% (자동 계산)
   > 적용할까요?"

6. Apply only after user approval per project.

### Phase 2: Cross-Project Pattern Scan

1. Collect all Decision Log entries from the past 2 weeks across ALL projects
2. Analyze for patterns:
   - **Shared failure modes**: same type of error/failure in different projects
   - **Methodological insights**: a lesson from one project applicable to another
   - **Contradictory approaches**: one project doing the opposite of another in a similar situation
3. For each pattern found, present with evidence:
   > "Cross-project 패턴 발견:
   > OSPREY (03-28): '학습 전략이 해결한 문제에 아키텍처 복잡성 추가하지 말 것'
   > SMEFTML (04-07): cINN 아키텍처를 복잡하게 변경하려는 시도 — 같은 실수 가능성?
   >
   > Dashboard에 기록할까요?"

4. User approves → add to Cross-Project Observations in `dashboard.md`
5. If no patterns found, skip silently (don't force connections)

### Phase 3: Dashboard Regeneration

Regenerate `dashboard.md` from project file data:

1. **Active Projects**: For each project, read State section and extract:
   - 현재 단계 (from 위치 field)
   - 상태 (infer from 블로커: 없음 → 🟢, has blocker → 🟡, no session > 14 days → 🔴)
   - 다음 마일스톤 (from Compass next incomplete major goal)
   - 마지막 세션 (from Session timestamp)

2. **Timeline**: Collect all Decision Log headers (date + title) from past 2 weeks across
   all projects, sort by date descending, take top 10.

3. **Preserve**: Keep existing Cross-Project Observations section unchanged
   (these are user-approved, not derived)

4. **Preserve**: Keep existing Project Registry unchanged

5. Write the regenerated `dashboard.md` with updated date.

### Phase 4: Archive Management

For each project:

1. Count Decision Log entries in main file
2. Count total lines in main file
3. If entries > 30 OR lines > 500:
   > "{slug}.md has {N} Decision Log entries ({L} lines).
   > 6개월 이상 된 항목 {M}개를 {slug}-decisions-{year}.md로 이동할까요?"

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
  - OSPREY: on track (G2.2a, 70%)
  - SMEFTML: minor drift detected (see warning above)
  - VaTD: on track (G3.2, paper writing)

Cross-Project Patterns: {N found, M approved}

Dashboard: regenerated

Archive: {N entries archived / no archiving needed}

Next review recommended: {date + 7 days}
```
