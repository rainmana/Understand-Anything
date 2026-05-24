# Codebase Information

## Project

- **Name:** Understand Anything
- **Repository:** Lum1104/Understand-Anything
- **License:** MIT
- **Description:** Multi-agent pipeline that analyzes codebases, knowledge bases, and docs into interactive knowledge graphs. Works as a plugin for Claude Code, Kiro, Codex, Cursor, Copilot, Gemini CLI, and more.

## Technology Stack

| Category | Technology |
|----------|-----------|
| Language | TypeScript (strict, ES2022 target) |
| Runtime | Node.js ≥ 22 |
| Package Manager | pnpm 10.6.2 (workspace monorepo) |
| Build | `tsc` (core/skill), Vite 6 (dashboard) |
| Test | Vitest 3.1+ |
| Lint | ESLint |
| CI | GitHub Actions (Node 22, pnpm cache) |
| Frontend | React 19, Tailwind CSS v4, ReactFlow 12 |
| State | Zustand 5 |
| Layout | ELK.js, Dagre, d3-force |
| Parsing | web-tree-sitter (10 language grammars) |
| Search | Fuse.js (fuzzy), cosine similarity (semantic) |
| Validation | Zod 4 |
| i18n | Custom locale system (en, zh, zh-TW, ja, ko, ru) |
| Scripting | Python 3 (merge/normalize scripts), Bash (harnesses) |

## Workspace Structure

```
pnpm-workspace.yaml
├── understand-anything-plugin/          # Main plugin package (@understand-anything/skill)
│   ├── packages/core/                   # @understand-anything/core — analysis engine
│   ├── packages/dashboard/              # @understand-anything/dashboard — React SPA
│   ├── src/                             # Plugin-level skill implementations
│   ├── skills/                          # Agent skill definitions (SKILL.md + scripts)
│   └── agents/                          # AI agent prompt definitions
├── harnesses/                           # Standalone execution adapters
│   ├── kiro/                            # Kiro CLI harness (run-understand.sh)
│   ├── litellm/                         # LiteLLM proxy client (llm-client.mjs)
│   └── tests/                           # Harness integration tests
├── homepage/                            # Astro-based marketing site
├── scripts/                             # Dev utilities (graph generators)
└── install.sh                           # Multi-platform installer
```

## Packages

| Package | Name | Purpose |
|---------|------|---------|
| `understand-anything-plugin/packages/core` | `@understand-anything/core` | Graph building, tree-sitter extraction, schema validation, search, fingerprinting |
| `understand-anything-plugin/packages/dashboard` | `@understand-anything/dashboard` | Interactive web visualization (ReactFlow graph, domain view, knowledge view) |
| `understand-anything-plugin` | `@understand-anything/skill` | Plugin glue: chat context, diff analysis, explain, onboard builders |

## Supported Languages (Tree-sitter Extraction)

TypeScript/JavaScript, Python, Go, Rust, Java, C#, C/C++, Ruby, PHP

## Non-code Parsers

Dockerfile, Makefile, YAML, JSON, Terraform, Protobuf, GraphQL, SQL, Shell, Markdown, TOML, .env

## Skills (Agent Commands)

| Skill | Purpose |
|-------|---------|
| `/understand` | Full pipeline: scan → analyze → build graph |
| `/understand-dashboard` | Launch interactive visualization |
| `/understand-chat` | Ask questions about the codebase |
| `/understand-domain` | Extract business domain knowledge |
| `/understand-knowledge` | Analyze wiki/knowledge bases |
| `/understand-explain` | Deep-dive into specific files/functions |
| `/understand-diff` | Analyze impact of current changes |
| `/understand-onboard` | Generate onboarding guide |

## Key Entry Points

| File | Role |
|------|------|
| `harnesses/kiro/run-understand.sh` | Standalone pipeline orchestrator |
| `harnesses/litellm/llm-client.mjs` | OpenAI-compatible LLM proxy client |
| `install.sh` | Multi-platform installer |
| `packages/core/src/index.ts` | Core library exports |
| `packages/dashboard/src/main.tsx` | Dashboard UI entry |
| `skills/understand/merge-batch-graphs.py` | Central graph merge/normalization |
