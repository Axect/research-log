# CLAUDE.md

## What This Plugin Does

Research Log is a Claude Code plugin that maintains per-project research logs at `~/.research-log/`. Each project gets one file with three sections:

- **Compass**: Goal tree with completion percentages — prevents direction loss
- **State**: Session snapshot — enables 2-minute context reboot
- **Decision Log**: Experiment decisions with thorough Why analysis — prevents knowledge decay

A central `dashboard.md` shows all projects at a glance.

## Skills

- `/log-init`: Initialize ~/.research-log/ and register a new project
- `/log-record`: Write a Decision Log entry with Why analysis (AI drafts, user reviews)
- `/log-state`: Update State section (called automatically by session-end hook)
- `/log-review`: Weekly review — Compass alignment, cross-project patterns, dashboard regeneration
- `/log-query`: Search research logs and answer questions

## Key Principles

1. **No forced connections**: Cross-project links emerge only from Decision Log "교훈" fields, always with user approval
2. **AI drafts, user decides**: AI generates content; user reviews before any write
3. **Concurrency safe**: Per-project `flock` ensures parallel sessions don't corrupt files
4. **Minimal overhead**: State updates automatically; Decision Log is the only manual action

## File Location

All files live under `~/.research-log/`. See `shared/conventions.md` for format specification.

## Version Management

When bumping the version, update both files:

- `.claude-plugin/plugin.json`: `"version"` field
- `.claude-plugin/marketplace.json`: `"version"` field inside the `plugins` array
