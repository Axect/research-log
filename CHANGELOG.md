# Changelog

## [0.2.1] - 2026-04-08

### Fixed
- Replace real project names (OSPREY, SMEFTML, VaTD, etc.) with generic placeholders in all skill files and README examples to prevent user project name leakage

## [0.2.0] - 2026-04-08

### Breaking Changes
- **Complete redesign**: Replaced wiki-based system (thesis/project/concept/method/finding/paper types) with research-log system (Compass + State + Decision Log)
- **Renamed**: `research-wiki` → `research-log`
- **Removed skills**: `/wiki-init`, `/wiki-ingest`, `/wiki-query`, `/wiki-lint`
- **Removed**: 7-type taxonomy, mandatory frontmatter, thesis-centric cross-referencing, wikilinks, index.md, log.md

### Added
- `/log-init` — Initialize `~/.research-log/` and register projects interactively
- `/log-state` — Auto-update State section on session end (hook-compatible)
- `/log-record` — Record Decision Log entries with thorough Why analysis (AI drafts, user reviews)
- `/log-review` — Weekly Compass alignment check, cross-project pattern scan, dashboard regeneration, archive management
- `/log-query` — Search research logs and answer questions with citations
- **Compass**: Goal tree with completion percentages and `← current focus` focus marker
- **State**: 6-field session snapshot for 2-minute context reboot
- **Decision Log**: Structured entries with Why analysis (cause chain + domain reasoning + literature)
- **Dashboard**: Central view of all projects with timeline and cross-project observations
- **Concurrency**: Per-project `flock` for parallel session safety
- **Archive**: Decision Log auto-archiving when files grow (>30 entries or >500 lines)

### Design Philosophy
- No forced cross-project connections — links emerge from Decision Log "Lesson" fields only
- AI drafts, user decides — no automatic writes without approval (except State via hook)
- One file per project — Compass + State + Decision Log in a single markdown file
- Dashboard is derived data — regenerated from project files, never directly edited by hooks

## [0.1.0] - 2026-04-08

### Added
- Initial release: wiki-based system with 4 skills
- `/wiki-init`, `/wiki-ingest`, `/wiki-query`, `/wiki-lint`
- 7 page types: thesis, project, concept, method, finding, paper
- Thesis-centric cross-referencing with `[[type/slug]]` wikilinks
