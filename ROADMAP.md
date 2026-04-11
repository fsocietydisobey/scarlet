# Scarlet Roadmap

Living document tracking planned features and improvements. Items move from "Backlog" → "In progress" → "Shipped" as work happens.

## Shipped

### v0.1 — Initial scaffold
- Project detection (`analyze_project`)
- Feature scanning (`scan_features`)
- Tree-sitter export extraction (`extract_feature_metadata`)
- Import graph (`build_feature_graph`)
- Barrel export generator
- CLAUDE.md generator with manual-section preservation
- Mermaid dep graph
- CLAUDE.md linter
- MCP server (FastMCP) + CLI (Click)

### v0.2 — Detection accuracy and invariants
- Multi-package.json merging (handles workspace setups)
- Component/hook/slice counts walk whole feature folder
- File-level granularity in consumers section
- `extract_invariants` tool (warning comments, magic numbers, TODOs)
- Linter handles legacy CLAUDE.md files (no markers → schema checks skipped)
- Linter file reference check tightened (source extensions, project-root prefixes)
- Linter public API freshness check now scoped to AUTO block only
- Example `.scarlet.yml` configs (Next.js monorepo, FastAPI, Vite+React)

---

## v0.3 — Doc richness and architectural intelligence

The five items below were carried over from the v0.2 wrap-up. They're the "next obvious things" the v0.1/v0.2 work surfaced.

### 1. Sub-feature granularity
Large features like jeevy_portal's `dashboard` (27 components) and `deliverable-workspace` (45 components) need their CLAUDE.md auto sections sub-grouped. Walk subfolders inside `components/` and group by subdirectory in the Public API + Key files sections so the doc doesn't become a flat 30-line export list.

### 2. JSDoc extraction
Parse `@description`, `@param`, `@example` from JSDoc blocks above exported functions/components and inject them into the auto sections. Lets prose come from the code itself, not from manual writing. Critical for the "documentation as you go" rule the user added to jeevy_portal.

### 3. Component prop extraction
For each React component, parse the props type/interface (from TypeScript or PropTypes or destructured params) and list them in the Public API section. Critical for AI to know what a component accepts without reading the source. Should handle:
- TypeScript `interface FooProps { ... }`
- TypeScript inline types `({ a, b }: { a: string; b: number })`
- Destructured JS params `({ projectId, onSelect })`
- PropTypes (rare in modern code, but cover for completeness)

### 4. Cyclic dependency detection
The dep graph builder already finds cycles (we saw `drawings ↔ file_manager` in jeevy_portal) but doesn't call them out. Add a `detect_cycles` step that returns an explicit list, render them as red edges in the Mermaid output, and add a CLI flag `--fail-on-cycle` for CI use.

### 5. Markdown chunking integration with Séance
Séance currently skips `.md` files. Extending Séance's chunker to handle markdown (split by H2 headers) means scarlet's generated CLAUDE.md files become semantically searchable. This is cross-tool composition: scarlet writes the docs, séance indexes them, claude searches them.

---

## v0.3 — Additional candidates (longer list)

Brainstormed during the v0.2 wrap-up. These are organized by category and ranked by leverage at the bottom.

### Documentation richness — extract more from code

| Feature | What it does | Why it matters |
|---|---|---|
| **Test name extraction** | Pull `it()` / `describe()` strings into a "Tested behaviors" auto section | Test names are free behavior specs — they belong in the docs |
| **Storybook story extraction** | If `*.stories.tsx` exists, extract story names into a "Variants" or "Use cases" section | Stories are executable documentation; making them visible to AI is huge |
| **API endpoint inventory** | For features with `api/` subdirs, list every RTK Query endpoint with HTTP method + URL pattern + cache tags | Backend contract is invisible without this; AI guesses URLs otherwise |
| **Reducer extraction** | For features with slices, list reducer names + action shapes | State mutations are critical; extracting them means AI knows what state changes are possible |
| **CSS module class inventory** | For components using CSS modules, list all class names | AI knows what classes are available without grepping the .module.css |
| **Component hierarchy diagram** | Generate a Mermaid tree showing how components inside a feature compose each other | Visual is easier to parse than a flat list |

### Cross-feature analysis — architecture intelligence

| Feature | What it does |
|---|---|
| **Coupling metrics** | Fan-in / fan-out per feature; flag features with too many consumers (god features) or too many dependencies (tightly coupled features) |
| **Orphan detection** | Find features with zero consumers — likely dead code |
| **Forbidden import enforcement** | Check `.scarlet.yml` layer rules and validate imports don't cross them (e.g., "components can't import from store directly") |
| **Public API surface area tracking** | Track how big each feature's barrel export is over time; alert on growth |
| **Cross-project pattern detection** | Compare two projects' shapes and identify common patterns |

### Freshness and maintenance

| Feature | What it does |
|---|---|
| **Watch mode** | `scarlet watch` runs sync on file changes; the index stays fresh while you code |
| **Git hook integration** | `scarlet install-hooks` adds a pre-commit hook that re-runs sync on changed features |
| **Diff-based sync** | `scarlet sync --since=HEAD~1` only re-processes features touched by recent commits |
| **Drift detection** | `scarlet drift` compares current code state to last sync snapshot, reports what changed but isn't documented |
| **Snapshot/checkpoint** | `scarlet snapshot` saves the current metadata to a JSON file; `scarlet diff-snapshot` shows what changed since |

### Backend support — extend beyond React

| Feature | What it does |
|---|---|
| **Python AST chunker** | Extend the analyzer to parse Python files: FastAPI routers, Pydantic models, services, repositories |
| **FastAPI route inventory** | Auto-generate API surface map: every endpoint, its method, path, request body schema, response schema |
| **Database schema extraction** | Pull table/column metadata from Alembic migrations or SQLAlchemy models |
| **gRPC / GraphQL schema parsing** | Same idea, different IDLs |

### AI/LLM integration — Scarlet ↔ Claude composability

| Feature | What it does |
|---|---|
| **Skill template generator** | Generate `.claude/commands/*.md` skill files from project conventions (e.g., `/add-feature` skill that follows the project's feature template) |
| **Prose synthesis tool** | An MCP tool Claude calls with `(structured_metadata, section_name)` → returns a draft prose section. Lets Claude generate vocabulary/gotchas drafts that the human edits |
| **Test case suggestion** | Given extracted invariants, suggest test cases that should exist (e.g. `if "don't show error if aborted" → test "should not show error when request is aborted"`) |
| **Structured-data extraction tool** | A general MCP tool that extracts any pattern from a project (`extract_pattern("RTK Query endpoint")`) and returns structured results |

### Quality and validation

| Feature | What it does |
|---|---|
| **Schema-validated `.scarlet.yml`** | JSON Schema for the config so editors autocomplete and catch typos |
| **Custom lint rules** | Projects can define their own lint rules in `.scarlet.yml` (e.g., "every CLAUDE.md must have a 'Last reviewed' timestamp") |
| **CI integration** | `--exit-on-issues` flag, machine-readable output, GitHub Actions example workflow |
| **Snapshot tests** | When sync runs, write a snapshot of the generated content. Future sync compares against the snapshot to detect unexpected changes |

### Visualizations — Mermaid generators

| Feature | What it does |
|---|---|
| **Component hierarchy per feature** | Tree showing how components compose within one feature |
| **Cross-feature data flow** | API → store → components → UI flow, drawn as a sequence or flow diagram |
| **Permission boundary map** | If features use `PermissionGate` or similar, visualize which gates wrap which components |
| **State machine visualization** | If features use XState or similar, render the state graph |
| **Bundle size impact** | Integrate with bundle analyzer output to show feature footprint |

### Developer experience

| Feature | What it does |
|---|---|
| **Web UI dashboard** | Browser dashboard showing features, state, recent changes, lint issues |
| **`scarlet find <symbol>`** | Locate any symbol across the project — alternative to grep when you only know a name |
| **`scarlet inverse <component>`** | Reverse query: "show me everything that uses `<UserCard>`" |
| **Bulk operations** | `scarlet bulk-edit --add-section "Tested behaviors"` adds a section to every CLAUDE.md at once |
| **REPL / interactive mode** | `scarlet shell` opens a REPL where you can poke at the indexed data without re-parsing |

---

## Master ranking — value × ease

| Rank | Feature | Category | Value | Ease |
|---|---|---|---|---|
| 1 | **Diff-based sync** (`--since`) | Maintenance | High — unlocks watch mode, git hooks, CI | Medium |
| 2 | **API endpoint inventory** | Doc richness | High — backend contracts are invisible without it | Medium |
| 3 | **Cyclic dependency detection** | Cross-feature | High — architectural insight | Easy |
| 4 | **Component prop extraction** | Doc richness | High — props are the contract | Medium |
| 5 | **JSDoc extraction** | Doc richness | High — prose from code | Easy |
| 6 | **Watch mode** | DX | Medium-high — dev productivity | Easy |
| 7 | **Python AST chunker** | Backend support | High — doubles project coverage | Hard |
| 8 | **Coupling metrics + forbidden imports** | Cross-feature | High for mature teams | Medium |
| 9 | **Test name extraction** | Doc richness | Medium — free behavior specs | Easy |
| 10 | **CI integration** | Quality | Medium — team adoption | Easy |
| 11 | **Prose synthesis tool** | AI integration | High but ambitious | Hard |
| 12 | **`scarlet inverse`** | DX | Medium | Easy |

## Recommended v0.3 cut

Three additions beyond the original five v0.3 items:

1. **Diff-based sync** — the keystone that makes everything else viable (watch mode, git hooks, CI all need fast incremental updates)
2. **API endpoint inventory** — biggest value-per-line addition for full-stack projects
3. **Watch mode** — small, but transforms the daily UX

## Save for v0.4+

- Python AST chunker (big effort, big payoff — extend Scarlet beyond frontend)
- Web UI dashboard (cool but not load-bearing)
- Prose synthesis tool (depends on a separate prompt strategy worth designing carefully)
- Cross-project pattern detection
- gRPC / GraphQL schema parsing
