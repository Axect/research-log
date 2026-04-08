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

- `topic-hint` (optional): Brief description of what to record (e.g., "CVAE collapse analysis").
  If omitted, AI infers the topic from recent session activity.

## Instructions

Read `shared/conventions.md` from this plugin directory for format specifications.

### Step 1: Identify Project

1. Read `~/.research-log/dashboard.md`
2. Parse Project Registry to match CWD to a project slug
3. If no match: ask user which project this entry belongs to
4. Read `~/.research-log/{slug}.md` — specifically the Compass section (to identify goal context)

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

**Context**: {Which Compass sub-goal (G?.?) this falls under and why}

**Tried**: {What was attempted — specific parameters, configs, approaches}
**Expected**: {What result was expected and why}
**Got**: {What actually happened — specific metrics, observations}

**Why analysis**:
1. {Root cause chain: A caused B which caused C}
2. {Why this is particularly the case in this problem/domain — what's specific about this physics/ML context}
3. {Literature or empirical evidence supporting this interpretation — cite papers, known phenomena, or prior experiments if applicable}

**Conclusion**: {Next concrete action based on this analysis}
**Lesson**: {Generalizable principle that applies beyond this specific experiment — this is the cross-project connection point}
```

**Why analysis guidelines:**
- Point 1 is mandatory: trace the causal chain from action to observed result
- Point 2 is mandatory: explain why this specific domain/problem exhibits this behavior
- Point 3 is encouraged but optional: cite literature, known phenomena, or analogous results from other experiments
- Each point should be 2-4 sentences, not a single phrase
- If the cause is genuinely unknown, say so explicitly: "Cause undetermined — further experiments needed"

**Lesson guidelines:**
- State as a general principle, not a project-specific fact
- If the lesson applies to another registered project, mention it by name
- Example: "For high-variance data in generative models, never start with default hyperparameters. Same issue likely in ProjectB."

### Step 4: Present Draft for Review

Show the complete draft to the user. Ask:
> "Please review this Decision Log entry. Let me know if anything needs to be changed."

Wait for user to:
- Approve as-is
- Request edits (apply and re-present)
- Cancel (discard the entry)

### Step 5: Write Entry

After user approval:

1. Acquire lock: `flock ~/.research-log/.locks/{slug}.lock`
2. Read `~/.research-log/{slug}.md`
3. Find the `## Decision Log` section
4. Insert the new entry immediately after the `## Decision Log` heading (newest first)
   - If the first line after `## Decision Log` is "(No entries yet)", remove it
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
   - Should the `← current focus` marker move? → propose moving it

3. If any changes are needed, present them:
   > "Compass update proposal:
   > - G2.2a CVAE + PIC Loss: 20% → 50% (loss oscillation diagnosed)
   > - `← current focus` stays at G2.2a
   >
   > Apply these changes?"

4. Apply only after user approval. Use `flock` for the write.
