<h1 align="center">Research Log</h1>

<p align="center">
  <strong>Compass + State + Decision Log for multi-project researchers</strong><br/>
  <em>Know where you're headed, where you left off, and why you made each decision.</em>
</p>

<p align="center">
  <a href="https://github.com/Axect/research-log/stargazers"><img src="https://img.shields.io/github/stars/Axect/research-log?style=social" alt="GitHub Stars" /></a>&nbsp;
  <img src="https://img.shields.io/badge/claude--code-plugin-blueviolet" alt="Claude Code Plugin" />&nbsp;
  <img src="https://img.shields.io/badge/license-MIT-green" alt="License: MIT" />
</p>

<p align="center">
  <a href="#the-problem">The Problem</a> &bull;
  <a href="#the-solution">The Solution</a> &bull;
  <a href="#get-started">Get Started</a> &bull;
  <a href="#skills">Skills</a> &bull;
  <a href="#file-structure">Structure</a>
</p>

---

## The Problem

Multi-project researchers face three recurring pain points:

1. **Experiment knowledge decay** — After dozens of experiments, *why* a scenario succeeded or failed becomes vague
2. **Direction loss** — Deep in implementation details, you drift from the main research question
3. **Context reboot** — Switching between projects, you can't quickly recover *what you were doing, why, and what's next*

## The Solution

Each project gets **one file** with **three sections**, each solving one problem:

| Section | Solves | Read When |
|---------|--------|-----------|
| **Compass** | Direction loss | Feeling lost in details |
| **State** | Context reboot | Returning to a project |
| **Decision Log** | Knowledge decay | Wondering why a past decision was made |

**Compass** is a goal tree with completion percentages. When you're deep in CVAE debugging, you can see: "I'm at G2.2a → G2.2 → G2 → Main Goal." It's a ladder out of the details.

**State** is a 6-line session snapshot. Read it in 2 minutes when you come back to a project: what was I doing, what's the blocker, what's next.

**Decision Log** records each significant decision with a thorough **Why analysis** — not just what happened, but a cause chain, domain-specific reasoning, and generalizable lessons. AI drafts the analysis; you review and approve.

A central **dashboard** shows all projects at a glance.

## Get Started

### Prerequisites

| Requirement | Purpose | Required? |
|-------------|---------|-----------|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | Plugin host | **Yes** |

### Installation

**1. Install the plugin** (inside Claude Code):
```
/install Axect/research-log
```

**2. Register your first project:**
```
/log-init
```

This creates `~/.research-wiki/`, asks for project info interactively, and generates the Compass + State + Decision Log file.

## Skills

| Skill | Description | Frequency |
|-------|-------------|-----------|
| `/log-init` | Register a new project | Once per project |
| `/log-record` | Record a Decision Log entry with Why analysis | After experiments/decisions |
| `/log-state` | Update State snapshot | Auto (session-end hook) |
| `/log-review` | Compass alignment + cross-project patterns + dashboard | Weekly (~15 min) |
| `/log-query` | Search logs, answer questions | On demand |

### `/log-init`

```
/log-init
```

Interactive — asks for project name, slug, repo path, main goal, and sub-goals. Generates the project file and updates the dashboard. Can pre-populate from existing `CLAUDE.md` and memory files.

### `/log-record`

```
/log-record                         # AI infers topic from recent work
/log-record "CVAE collapse analysis"  # Provide topic hint
```

AI gathers context (git diffs, configs, experiment outputs), drafts a Decision Log entry with full Why analysis, and presents it for your review. Also proposes Compass percentage updates if progress was made.

### `/log-state`

```
/log-state                          # Manual invocation
```

Primarily called by the **session-end hook** — updates the State section automatically when you finish working. Can also be called manually. Identifies the project from your current working directory.

### `/log-review`

```
/log-review
```

Weekly review (~15 minutes):
1. **Compass alignment**: flags if recent work drifted from the current focus
2. **Cross-project patterns**: scans Decision Logs for shared failure modes or transferable lessons
3. **Dashboard regeneration**: updates project cards and timeline from current data
4. **Archive management**: proposes moving old Decision Log entries to archive files

### `/log-query`

```
/log-query "Why did we abandon MSGLAON in OSPREY?"
/log-query "What did I work on last month?"
/log-query "Lessons learned about inverse problems"
```

Searches project files and archives, synthesizes answers with citations to specific Decision Log entries.

## File Structure

```
~/.research-wiki/
├── dashboard.md              # All projects at a glance
├── .locks/                   # Per-project flock files
│   ├── osprey.lock
│   └── smeftml.lock
├── osprey.md                 # Compass + State + Decision Log
├── osprey-decisions-2025.md  # Archived entries (when file grows)
└── smeftml.md
```

### Example Project File

```markdown
# OSPREY — PBH Hawking Radiation Neural Operator

## Compass

### Main Goal
Build a neural operator emulator for PBH Hawking radiation inverse problems.

### Sub-goals
- **G1. Forward Emulator** [100%]
- **G2. Inverse Emulator** [40%] ← current focus
  - G2.1 Deterministic inverse [100%]
  - G2.2 Probabilistic inverse [20%] ← current focus
- **G3. Paper → JCAP** [10%]

---

## State
- **Session**: 2026-04-05T14:30
- **Location**: G2.2a
- **Working on**: CVAE loss convergence test
- **Blocker**: KL term posterior collapse
- **Next step**: Try β-annealing

---

## Decision Log

### 2026-04-05 | CVAE posterior collapse analysis

**Context**: G2.2a — CVAE + PIC Loss

**Tried**: Standard CVAE (β=1.0)
**Expected**: Smooth convergence
**Got**: Loss oscillation after epoch 50

**Why Analysis**:
1. KL penalty too strong from the start at β=1.0 → posterior collapse
2. PBH spectrum has high variance → large KL divergence between posterior and prior
3. Bowman et al. (2016): β-annealing is the standard fix

**Conclusion**: Try β-annealing (0.001 → 1.0 over 100 epochs)
**Lesson**: For high-variance physics data, always start VAE training with β-annealing
```

## Key Principles

- **No forced connections** — Cross-project links emerge only from Decision Log lessons, with user approval
- **AI drafts, user decides** — No automatic writes without approval (except State via hook)
- **Concurrency safe** — Per-project `flock` ensures parallel sessions don't corrupt files
- **Minimal overhead** — State updates automatically; Decision Log is the only manual action

## License

MIT
