---
name: log-init
description: Initialize ~/.research-log/ and register a new research project with Compass, State, and Decision Log
---

# Log Init

Initialize the research log system and register a project.
Safe to run multiple times; existing projects prompt for update, not overwrite.

## Usage

```
/log-init
```

## Arguments

None. Interactive — collects project info through conversation.

## Instructions

Read `shared/conventions.md` from this plugin directory for format specifications.

### Step 1: Ensure Directory Structure

1. Create `~/.research-log/` if it doesn't exist
2. Create `~/.research-log/.locks/` if it doesn't exist
3. If `~/.research-log/dashboard.md` doesn't exist, create it:

```markdown
# Research Dashboard
> Last updated: {today's date}

## Project Registry
<!-- Format: "- {slug}: {absolute_path}" — parsed by skills to map CWD to project -->

## Active Projects

## Timeline (last 2 weeks)
| Date | Project | Decision/Event |
|------|---------|------------|

## Cross-Project Observations
> Only naturally emerging connections. AI proposes during /log-review; added after user approval.
```

### Step 2: Collect Project Information

Ask the user for (one question at a time):

1. **Project name**: Full display name (e.g., "MyProject — Short Description of Research")
2. **Slug**: Short identifier for filenames (e.g., "myproject"). Suggest one based on name.
3. **Repo path**: Absolute path to the project directory (e.g., `~/Documents/Project/Research/MyProject`)
4. **Main Goal**: 1-2 sentence research objective
5. **Initial sub-goals**: Ask user to list the major milestones/goals. For each, ask for a percentage estimate.

If the slug already exists in `dashboard.md` Project Registry, ask: "This project is already registered. Update it?"
- If yes: read existing file, update Compass with new info, preserve State and Decision Log
- If no: stop

### Step 3: Generate Project File

Write `~/.research-log/{slug}.md`:

```markdown
# {Project Name}

## Compass

### Main Goal
{user's main goal text}

### Sub-goals

{generated goal tree from user's input, with percentages}

### Why This Approach
{ask user, or leave as "To be filled after initial experiments."}

### Biggest Risk
{ask user, or leave as "To be identified."}

---

## Core Documents
> Pointers to current research-frontier artifacts. Read by `/log-query`,
> `/log-record`, `/log-review` for forward-looking recommendations. Updated
> via skill prompts on user approval. Cap: ≤15 entries; demote stale items
> to ★★ before removal.

### ★★★ Core (current frontier)

(No entries yet — added as the project produces canonical artifacts.)

### ★★ Foundational (architecture / training-data / paper-substrate references)

(No entries yet.)

---

## State
- **Session**: —
- **Last session**: —
- **Location**: —
- **Working on**: —
- **Current status**: Project initialized
- **Blocker**: None
- **Next step**: {first sub-goal from Compass}
- **Compass link**: {first sub-goal ID}

---

## Decision Log

(No entries yet)
```

### Step 4: Create Lock File

```bash
touch ~/.research-log/.locks/{slug}.lock
```

### Step 5: Update Dashboard

1. Add to Project Registry section:
   ```
   - {slug}: {repo_path}
   ```

2. Add to Active Projects section:
   ```markdown
   ### {Project Name}
   - **Current stage**: {first sub-goal}
   - **Status**: 🟢 active — project initialized
   - **Next milestone**: {first major milestone}
   - **Last session**: {today}
   ```

3. Update the `Last updated` date.

### Step 6: Pre-populate from Existing Sources (Optional)

If the project repo has existing knowledge sources, offer to pre-populate:

1. Check if `{repo_path}/CLAUDE.md` exists → extract know-how for Compass "Why This Approach" and Decision Log seeds
2. Check Claude Code memory files at `~/.claude/projects/` for this project path → extract feedback memories as Decision Log entries

Present extracted items to user for approval before adding.

### Step 7: Report

```
Project registered: {Project Name}

Files:
  ~/.research-log/{slug}.md (Compass + State + Decision Log)
  ~/.research-log/.locks/{slug}.lock

Dashboard updated with project card.

Next steps:
  Work on the project normally. State updates automatically on session end.
  After experiments: /log-record to capture decisions with Why analysis.
  Weekly: /log-review for Compass alignment and cross-project patterns.
```
