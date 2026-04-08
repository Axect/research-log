---
name: log-record
description: Record a Decision Log entry with thorough Why analysis for experiment decisions, failures, and insights
---

# Log Record

Record a decision or experiment result with thorough Why analysis.
AI drafts the entry; user reviews and approves before it's committed.

## Usage

```
/log-record [topic-hint]
```

## Arguments

- `topic-hint` (optional): Brief description of what to record (e.g., "CVAE collapse 분석").
  If omitted, AI infers the topic from recent session activity.

## Instructions

Read `shared/conventions.md` from this plugin directory for format specifications.

### Step 1: Identify Project

1. Read `~/.research-wiki/dashboard.md`
2. Parse Project Registry to match CWD to a project slug
3. If no match: ask user which project this entry belongs to
4. Read `~/.research-wiki/{slug}.md` — specifically the Compass section (to identify goal context)

### Step 2: Gather Context

Collect information from available sources:

1. **Recent git history**: `git log --oneline -10` and `git diff HEAD~3` in the project repo
2. **Config changes**: look for recently modified YAML/TOML config files
3. **Experiment outputs**: check for recent log files, metrics, or result summaries
4. **Conversation context**: what was discussed/worked on in the current Claude Code session
5. **Topic hint**: if provided, use it to focus the context gathering

### Step 3: Draft Decision Log Entry

Write a draft entry following this exact format:

```markdown
### {today's date} | {descriptive title}

**맥락**: {Which Compass sub-goal (G?.?) this falls under and why}

**시도**: {What was attempted — specific parameters, configs, approaches}
**기대**: {What result was expected and why}
**결과**: {What actually happened — specific metrics, observations}

**Why 분석**:
1. {Root cause chain: A caused B which caused C}
2. {Why this is particularly the case in this problem/domain — what's specific about this physics/ML context}
3. {Literature or empirical evidence supporting this interpretation — cite papers, known phenomena, or prior experiments if applicable}

**결론**: {Next concrete action based on this analysis}
**교훈**: {Generalizable principle that applies beyond this specific experiment — this is the cross-project connection point}
```

**Why 분석 guidelines:**
- Point 1 is mandatory: trace the causal chain from action to observed result
- Point 2 is mandatory: explain why this specific domain/problem exhibits this behavior
- Point 3 is encouraged but optional: cite literature, known phenomena, or analogous results from other experiments
- Each point should be 2-4 sentences, not a single phrase
- If the cause is genuinely unknown, say so explicitly: "원인 미확정 — 추가 실험 필요"

**교훈 guidelines:**
- State as a general principle, not a project-specific fact
- If the lesson applies to another registered project, mention it by name
- Example: "VAE 계열에서 고분산 물리 데이터를 다룰 때, β=1.0에서 시작하지 말 것. SMEFTML의 cINN에서도 유사한 문제가 예상됨."

### Step 4: Present Draft for Review

Show the complete draft to the user. Ask:
> "이 Decision Log 항목을 검토해주세요. 수정할 부분이 있으면 말씀해주세요."

Wait for user to:
- Approve as-is
- Request edits (apply and re-present)
- Cancel (discard the entry)

### Step 5: Write Entry

After user approval:

1. Acquire lock: `flock ~/.research-wiki/.locks/{slug}.lock`
2. Read `~/.research-wiki/{slug}.md`
3. Find the `## Decision Log` section
4. Insert the new entry immediately after the `## Decision Log` heading (newest first)
   - If the first line after `## Decision Log` is "(아직 항목 없음)", remove it
   - Add a `---` separator after the new entry
5. Release lock

### Step 6: Check Compass Update

After writing the entry, assess whether the Compass needs updating:

1. Read the Compass sub-goals
2. Based on the Decision Log entry, check:
   - Did a sub-goal reach a new completion milestone? → propose % increase
   - Was a sub-goal completed? → propose marking as 100%
   - Did the entry reveal a new sub-goal? → propose adding it
   - Did the entry invalidate or change a sub-goal? → propose modification
   - Should the `← 현재 여기` marker move? → propose moving it

3. If any changes are needed, present them:
   > "Compass 업데이트 제안:
   > - G2.2a CVAE + PIC Loss: 20% → 50% (loss 진동 문제 진단 완료)
   > - `← 현재 여기` 유지: G2.2a
   >
   > 적용할까요?"

4. Apply only after user approval. Use `flock` for the write.
