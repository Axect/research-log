# Changelog

## [0.3.0] - 2026-04-29

### Added
- **Core Documents section** (optional, between Compass and State in `{slug}.md`): user-curated pointers to current research-frontier artifacts (e.g., `outputs/<dir>/` paths produced by other workflows). Two-tier system — **★★★ Core** (active canonical) and **★★ Foundational** (architecture / training-data / paper-substrate references) — with a small status enum (`active canonical`, `component → <other>`, `preserved + header`, `foundational`, `reference §X`, `superseded`), `last YYYY-MM-DD` touch date, ` · ` field separator, and a ≤15-entry cap.
- `shared/conventions.md`: full spec for the new section (tier semantics, status enum, format, separator, cap, maintenance rules).
- `/log-init`: emits an empty Core Documents section in the new project file template.
- `/log-query` Step 2: reads `## Core Documents` when present and treats it as authoritative for "what matters now". Step 3 forward-looking guidance: prefer pointing at concrete `outputs/` paths from Core Documents over searching the broader output directory; flag artifacts not yet pinned for promotion via `/log-record`.
- `/log-record` Step 7 (new): after the Compass-update prompt, assess whether the recorded decision implies a Core Documents diff (new canonical artifact, supersession, status change, or new foundational reference) and propose the diff for user approval.
- `/log-review` Phase 1 step 7 (new): staleness check — flag ★★★ entries with `last YYYY-MM-DD` 30+ days old and propose demotion to ★★. Phase 3 step 5 (new): preserve Core Documents during dashboard regeneration (user-curated, not derived).

### Design Philosophy
- **User-curated, not derived**: unlike the dashboard (which is regenerated from project files), the Core Documents list is maintained by the user with skill assistance. Skills propose updates; never auto-edit without approval.
- **Forward-looking pointer layer**: complements Decision Log (history) and State (current snapshot) by pinning the *artifacts* the project's frontier currently rests on. Useful when a project produces many `outputs/*` directories and only a handful are actively cited by current paper substrate / canonical artifacts.

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
