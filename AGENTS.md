# AGENTS.md

> Navigational guide for AI coding assistants working in the Understand-Anything codebase.

## Project Summary

Multi-agent pipeline that analyzes codebases into interactive knowledge graphs. Plugin architecture with tree-sitter extraction, optional LLM enrichment, and a React dashboard for visualization.

## Directory Map

```
├── understand-anything-plugin/     # Main plugin monorepo
│   ├── packages/core/src/          # Analysis engine (tree-sitter, graph, schema, search)
│   │   ├── plugins/extractors/     # Per-language AST extractors (TS, Py, Go, Rust, Java, C#, C++, Ruby, PHP)
│   │   ├── plugins/parsers/        # Non-code file parsers (Dockerfile, Terraform, YAML, etc.)
│   │   ├── analyzer/               # Graph builder, layer detector, tour generator, LLM prompts
│   │   ├── persistence/            # File I/O for .understand-anything/ directory
│   │   └── languages/              # Language/framework registries and configs
│   ├── packages/dashboard/src/     # React SPA (ReactFlow, ELK layout, Zustand state)
│   │   ├── components/             # GraphView, NodeInfo, SearchBar, FilterPanel, etc.
│   │   ├── utils/                  # Layout engines, edge aggregation, community detection
│   │   ├── themes/                 # CSS custom property theming
│   │   └── locales/                # i18n (en, zh, zh-TW, ja, ko, ru)
│   ├── src/                        # Plugin skill implementations (chat, diff, explain, onboard)
│   └── skills/                     # Agent skill definitions (SKILL.md + scripts)
│       ├── understand/             # Main pipeline: merge-batch-graphs.py, extract-structure.mjs
│       ├── understand-domain/      # Business domain extraction
│       ├── understand-knowledge/   # Wiki/knowledge base parsing
│       └── understand-{chat,diff,explain,onboard,dashboard}/
├── harnesses/                      # Standalone execution adapters
│   ├── kiro/run-understand.sh      # Bash orchestrator (supports --no-llm, --local, --ollama)
│   ├── litellm/llm-client.mjs     # OpenAI-compatible LLM proxy client
│   └── tests/test-harness.sh      # Integration tests
├── scripts/                        # Dev utilities (large graph generator)
├── homepage/                       # Astro marketing site
└── install.sh                      # Multi-platform installer (15+ platforms)
```

## Key Entry Points

| What you want to do | Start here |
|---------------------|-----------|
| Understand the pipeline | `harnesses/kiro/run-understand.sh` |
| Add a language extractor | `packages/core/src/plugins/extractors/` + `languages/configs/` |
| Modify graph schema | `packages/core/src/types.ts` + `schema.ts` |
| Change dashboard behavior | `packages/dashboard/src/App.tsx` + `store.ts` |
| Fix graph merge logic | `skills/understand/merge-batch-graphs.py` |
| Add a non-code parser | `packages/core/src/plugins/parsers/` |
| Modify LLM prompts | `packages/core/src/analyzer/llm-analyzer.ts` |

## Repo-Specific Patterns

- **Graph-as-data:** The knowledge graph is a single JSON file (`.understand-anything/knowledge-graph.json`). All downstream features (dashboard, chat, diff) consume this file — they never re-analyze source code.
- **Python in a TypeScript repo:** `merge-batch-graphs.py` is the most complex single file. It handles edge deduplication, test-to-production linking, and direction canonicalization. Uses only stdlib (no pip deps).
- **Workspace protocol:** Packages reference each other via `workspace:*` in package.json. Always build core before skill/dashboard.
- **Tree-sitter WASM:** Grammars compile to native during `pnpm install`. Listed in `pnpm.onlyBuiltDependencies` to control which packages trigger native builds.
- **Incremental by default:** The pipeline fingerprints files (content hash). Re-runs only process changed files unless `--full` is passed.
- **21 node types, 35 edge types:** The type system is intentionally rich. See `packages/core/src/types.ts` for the canonical list.

## CI & Config Discovery

- **CI:** `.github/workflows/ci.yml` — runs on PRs, builds core → skill, tests both
- **No formatter config:** No Prettier/ESLint config file at root (ESLint configured via `pnpm lint` script)
- **TypeScript:** `tsconfig.json` at root — strict mode, ES2022, bundler module resolution
- **pnpm workspace:** `pnpm-workspace.yaml` defines three package groups
- **Dashboard dev:** `pnpm dev:dashboard` starts Vite with custom `/__graph` and `/__source` endpoints

## Testing Conventions

- Test files: `*.test.ts` or `__tests__/*.test.ts`
- Framework: Vitest (not Jest)
- Run: `pnpm --filter @understand-anything/core test` or `pnpm --filter @understand-anything/skill test`
- Python tests: `python -m pytest skills/understand/test_merge_batch_graphs.py`
- Harness tests: `bash harnesses/tests/test-harness.sh`

## Detailed Documentation

For deeper context, see `.agents/summary/index.md` which indexes:
- `architecture.md` — system design, data flow, design patterns
- `components.md` — all modules with responsibilities
- `interfaces.md` — TypeScript API contracts
- `data_models.md` — full graph schema (21 node types, 35 edge types)
- `workflows.md` — pipeline stages, extension guides
- `dependencies.md` — every dependency with rationale

## Custom Instructions
<!-- This section is for human and agent-maintained operational knowledge.
     Add repo-specific conventions, gotchas, and workflow rules here.
     This section is preserved exactly as-is when re-running codebase-summary. -->
