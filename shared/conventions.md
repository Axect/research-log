# Wiki Page Conventions

## Wiki Root

`~/.research-wiki/`

## Directory Layout

| Directory | Type | Purpose |
|-----------|------|---------|
| `thesis/` | thesis | Research narrative, central claims, evidence tracking |
| `projects/` | project | Per-project status, know-how, findings |
| `concepts/` | concept | Physics & ML concepts |
| `methods/` | method | Techniques, architectures, what works/fails |
| `findings/` | finding | Experimental results, failures, lessons learned |
| `papers/` | paper | Literature interpretation (not metadata) |
| `raw/` | — | Immutable source symlinks (never modified) |

## Common Frontmatter

Every page has:

    ---
    title: "Page Title"
    type: concept | method | project | finding | paper | thesis
    created: YYYY-MM-DD
    updated: YYYY-MM-DD
    tags: [tag1, tag2]
    projects: [osprey, smeftml]
    sources: []
    ---

## Type-Specific Frontmatter

**project:** `status: active|paused|completed|archived`, `repo: <path>`, `target_venue: <journal>`
**finding:** `result: positive|negative|mixed`, `confidence: high|medium|low`, `experiment: "<desc>"`
**paper:** `arxiv: "<id>"`, `authors: [...]`, `year: <int>`, `relevance: high|medium|low`

## Required Body Sections

| Type | Sections |
|------|----------|
| project | Overview, Current Status, Key Findings, Know-how, Open Questions |
| finding | Context, What Happened, Why, Lesson, Related |
| thesis | Claim, Evidence For, Evidence Against, Implications, Next Steps |
| concept | Definition, Role in Research, Connections, References |
| method | Description, When to Use, Known Pitfalls, Project Usage, References |
| paper | Key Contribution, Relevance to My Research, Connections, Critical Notes |

## Cross-Referencing

- Use `[[type/slug]]` format: `[[projects/osprey]]`, `[[methods/deeponet]]`
- Within same directory, short form allowed: `[[msglaon-failure]]`
- Thesis pages always use full paths
- Only link to existing pages (no red links)

## index.md Format

- One entry per page, under 150 characters
- Grouped by type with count: `## Projects (3)`
- Entry format: `- [[slug]]: one-line description [status]`

## log.md Format

- Reverse-chronological (newest first)
- Entry: `## [YYYY-MM-DD] <op> | <subject>` where op = ingest|query|lint|update
- List pages created/updated and total count
