# Dependencies

## Overview

The project uses pnpm workspaces with three packages. Dependencies are split between the core analysis engine, the dashboard visualization, and the plugin skill layer.

## Core Package (`@understand-anything/core`)

### Runtime Dependencies

| Package | Purpose | Why This Choice |
|---------|---------|-----------------|
| `web-tree-sitter` | WASM-based tree-sitter runtime | Enables AST parsing in Node.js without native compilation per-platform |
| `tree-sitter-typescript` | TypeScript/JavaScript grammar | AST extraction for TS/JS files |
| `tree-sitter-python` | Python grammar | AST extraction for Python files |
| `tree-sitter-go` | Go grammar | AST extraction for Go files |
| `tree-sitter-rust` | Rust grammar | AST extraction for Rust files |
| `tree-sitter-java` | Java grammar | AST extraction for Java files |
| `tree-sitter-c-sharp` | C# grammar | AST extraction for C# files |
| `tree-sitter-cpp` | C++ grammar | AST extraction for C/C++ files |
| `tree-sitter-ruby` | Ruby grammar | AST extraction for Ruby files |
| `tree-sitter-php` | PHP grammar | AST extraction for PHP files |
| `tree-sitter-javascript` | JavaScript grammar | Shared with TypeScript extractor |
| `fuse.js` | Fuzzy text search | Fast client-side fuzzy matching for node search |
| `ignore` | Gitignore-style pattern matching | Respects `.understandignore` and `.gitignore` patterns |
| `yaml` | YAML parser | Parse YAML config files (docker-compose, k8s, etc.) |
| `zod` | Schema validation | Runtime validation of knowledge graph structure |

### Dev Dependencies

| Package | Purpose |
|---------|---------|
| `typescript` | Type checking and compilation |
| `vitest` | Test runner |
| `@vitest/coverage-v8` | Code coverage |
| `@types/node` | Node.js type definitions |

## Dashboard Package (`@understand-anything/dashboard`)

### Runtime Dependencies

| Package | Purpose | Why This Choice |
|---------|---------|-----------------|
| `react` / `react-dom` | UI framework | Component-based rendering for interactive graph |
| `@xyflow/react` | Graph rendering (ReactFlow v12) | Purpose-built for node/edge graph visualization with pan/zoom |
| `elkjs` | Hierarchical graph layout | Handles large graphs (1000+ nodes) with layered layout |
| `@dagrejs/dagre` | Directed graph layout | Simpler fallback layout for smaller graphs |
| `d3-force` | Force-directed layout | Used for knowledge graph community visualization |
| `zustand` | State management | Minimal boilerplate, single store, computed selectors |
| `graphology` | Graph data structure | In-memory graph operations for community detection |
| `graphology-communities-louvain` | Community detection | Louvain algorithm for knowledge graph clustering |
| `prism-react-renderer` | Syntax highlighting | Code viewer with language-aware highlighting |
| `react-markdown` | Markdown rendering | Render node summaries and tour descriptions |
| `hast-util-to-jsx-runtime` | HAST to React | Markdown rendering pipeline support |
| `devlop` | Development utilities | Assertion helpers used by markdown pipeline |
| `@understand-anything/core` | Core types and utilities | Shared types, schema validation, search |

### Dev Dependencies

| Package | Purpose |
|---------|---------|
| `vite` | Build tool and dev server |
| `@vitejs/plugin-react` | React Fast Refresh for Vite |
| `tailwindcss` | Utility-first CSS framework (v4) |
| `@tailwindcss/vite` | Tailwind Vite integration |
| `typescript` | Type checking |
| `vitest` | Test runner |
| `@vitest/coverage-v8` | Code coverage |
| `@types/react` / `@types/react-dom` | React type definitions |
| `@types/d3-force` | d3-force type definitions |

## Skill Package (`@understand-anything/skill`)

### Runtime Dependencies

| Package | Purpose |
|---------|---------|
| `@understand-anything/core` | Core analysis engine (workspace link) |

### Dev Dependencies

| Package | Purpose |
|---------|---------|
| `typescript` | Type checking |
| `vitest` | Test runner |
| `@types/node` | Node.js type definitions |

## Root Workspace

### Dev Dependencies

| Package | Purpose |
|---------|---------|
| `typescript` | Shared TypeScript version |
| `vitest` | Root-level test configuration |

## Python Dependencies (Scripts)

The Python scripts (`merge-batch-graphs.py`, `extract-domain-context.py`, `parse-knowledge-base.py`, `merge-knowledge-graph.py`) use only the Python standard library — no pip dependencies required.

## External Tools (Not in package.json)

| Tool | Used By | Purpose |
|------|---------|---------|
| `pnpm` | Build system | Package management and workspace orchestration |
| `node` (≥22) | Runtime | JavaScript/TypeScript execution |
| `python3` | Pipeline scripts | Graph merge and normalization |
| `git` | Fingerprinting | Commit hash detection for metadata |
| `litellm` (optional) | LLM proxy | Routes to any model provider (pip install) |

## Dependency Graph

```mermaid
graph TD
    ROOT[Root workspace] --> CORE[@understand-anything/core]
    ROOT --> DASH[@understand-anything/dashboard]
    ROOT --> SKILL[@understand-anything/skill]
    SKILL --> CORE
    DASH --> CORE
    CORE --> TS_GRAMMARS[tree-sitter grammars x10]
    CORE --> FUSE[fuse.js]
    CORE --> ZOD[zod]
    CORE --> YAML_P[yaml]
    CORE --> IGNORE[ignore]
    DASH --> REACTFLOW[@xyflow/react]
    DASH --> ELK[elkjs]
    DASH --> DAGRE[dagre]
    DASH --> ZUSTAND[zustand]
    DASH --> REACT[react]
```

## Version Pinning Strategy

- **Workspace packages** use `workspace:*` (always latest local version)
- **Major dependencies** use caret ranges (`^`) — allows minor/patch updates
- **pnpm** is pinned exactly via `packageManager` field in root `package.json`
- **Node.js** requires ≥22 (CI uses exactly 22)
- **Tree-sitter grammars** are listed in `pnpm.onlyBuiltDependencies` to control native compilation
