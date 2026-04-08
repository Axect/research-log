# CLAUDE.md

## What This Plugin Does

Research Wiki is a Claude Code plugin that maintains a personal research knowledge base at `~/.research-wiki/`. It provides four skills for initializing, ingesting, querying, and linting wiki pages.

## Skills

- `/wiki-init`: Initialize the wiki directory structure (safe to re-run)
- `/wiki-ingest`: Ingest sources (MAGI reports, papers, findings, project know-how) into the wiki
- `/wiki-query`: Search the wiki and synthesize answers from relevant pages
- `/wiki-lint`: Health-check the wiki for broken links, orphans, stale pages, and more

## Wiki Location

All wiki pages live under `~/.research-wiki/`. Never modify files outside this directory.

## Page Conventions

See `shared/conventions.md` for the full specification: directory layout, frontmatter schema, required body sections, cross-referencing rules, and index/log formats.

## Version Management

When bumping the version, update both files:

- `.claude-plugin/plugin.json`: `"version"` field
- `.claude-plugin/marketplace.json`: `"version"` field inside the `plugins` array
