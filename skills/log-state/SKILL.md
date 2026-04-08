---
name: log-state
description: Update the State section of a project's research log — called automatically by session-end hook or manually
---

# Log State

Snapshot the current working state for context recovery.
Primarily called by the session-end hook; can also be invoked manually.

## Usage

```
/log-state
```

## Arguments

None. Project is identified from CWD.

## Instructions

Read `shared/conventions.md` from this plugin directory for format specifications.

### Step 1: Identify Project

1. Read `~/.research-log/dashboard.md`
2. Parse the Project Registry section: each line is `- {slug}: {path}`
3. Check if the current working directory (CWD) starts with any registered path
   - Expand `~` to the home directory before comparison
4. If no match: skip silently. This is not a registered research project directory.
5. If match: use the matched slug as the target project.

### Step 2: Read Current State

1. Acquire lock: `flock ~/.research-log/.locks/{slug}.lock`
2. Read `~/.research-log/{slug}.md`
3. Parse the current `## State` section
4. Note the current `Session` timestamp as the new `Last session` value

### Step 3: Generate New State

From the current session context, generate:

- **Session**: current timestamp (YYYY-MM-DDTHH:MM)
- **Last session**: previous Session timestamp (from Step 2)
- **Location**: identify which Compass sub-goal the recent work relates to (read Compass section)
- **Working on**: summarize the main task from this session
- **Current status**: current status of that task (completed, in progress, blocked, etc.)
- **Blocker**: any blockers, or "None"
- **Next step**: the most logical next step
- **Compass link**: explain how the current work connects to the identified sub-goal

Keep each field to 1-2 sentences max. This is meant to be read in 2 minutes.

### Step 4: Write State

1. Replace the `## State` section in `{slug}.md` with the new State
   - Preserve everything before `## State` (Compass) and after the State section (Decision Log)
   - The State section starts at `## State` and ends at the next `---` separator before `## Decision Log`
2. Release lock

### Step 5: Silent Completion

Do NOT produce any output to the user when called from a hook.
When called manually, briefly confirm: "State updated for {slug}."
