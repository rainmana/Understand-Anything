# Workflows

## Analysis Pipeline (`/understand`)

The primary workflow that transforms a codebase into a knowledge graph:

```mermaid
flowchart TD
    START[User invokes /understand] --> DETECT[Detect changed files via fingerprints]
    DETECT -->|Full rebuild| SCAN[Scan all files]
    DETECT -->|Incremental| CHANGED[Scan changed files only]
    SCAN --> BATCH[Batch files for parallel extraction]
    CHANGED --> BATCH
    BATCH --> EXTRACT[extract-structure.mjs: tree-sitter AST extraction]
    EXTRACT --> FRAGMENTS[Batch JSON fragments in intermediate/]
    FRAGMENTS --> LLM_CHECK{LLM available?}
    LLM_CHECK -->|Yes| ENRICH[LLM enrichment: summaries, layers, tours]
    LLM_CHECK -->|No --no-llm| SKIP_LLM[Skip enrichment]
    ENRICH --> MERGE[merge-batch-graphs.py]
    SKIP_LLM --> MERGE
    MERGE --> NORMALIZE[Normalize: IDs, types, complexity, directions]
    NORMALIZE --> LINK[Link tests to production files]
    LINK --> DEDUP[Deduplicate edges]
    DEDUP --> VALIDATE[Schema validation via Zod]
    VALIDATE --> PERSIST[Save knowledge-graph.json + fingerprints]
    PERSIST --> DONE[Pipeline complete]
```

### Pipeline Stages in Detail

1. **Fingerprint check** — compare stored content hashes against current files to determine what changed
2. **File discovery** — walk project tree, apply `.understandignore` filters
3. **Batch extraction** — `extract-structure.mjs` runs tree-sitter on batches of 20-30 files (up to 5 concurrent)
4. **LLM enrichment** (optional) — generate summaries, detect layers, build tours
5. **Merge** — `merge-batch-graphs.py` combines all fragments into a single graph
6. **Normalization** — canonicalize node IDs, fix types, normalize complexity
7. **Test linking** — heuristically match test files to production files, add `tested_by` edges
8. **Edge deduplication** — remove duplicate/inverted edges, keep highest weight
9. **Validation** — Zod schema check, auto-fix minor issues
10. **Persistence** — write to `.understand-anything/`

## Domain Analysis (`/understand-domain`)

```mermaid
flowchart LR
    SCAN_D[extract-domain-context.py] --> DETECT_E[Detect entry points]
    DETECT_E --> EXTRACT_M[Extract file signatures]
    EXTRACT_M --> LLM_D[LLM: identify domains, flows, steps]
    LLM_D --> MERGE_D[merge-subdomain-graphs.py]
    MERGE_D --> SAVE_D[Save domain-graph.json]
```

## Knowledge Base Analysis (`/understand-knowledge`)

```mermaid
flowchart LR
    PARSE[parse-knowledge-base.py] --> WIKILINKS[Extract wikilinks, headings, categories]
    WIKILINKS --> LLM_K[LLM: discover implicit relationships]
    LLM_K --> MERGE_K[merge-knowledge-graph.py]
    MERGE_K --> SAVE_K[Save knowledge-graph.json kind:knowledge]
```

## Dashboard Workflow

```mermaid
flowchart TD
    LAUNCH[/understand-dashboard] --> VITE[Start Vite dev server]
    VITE --> FIND[Find knowledge-graph.json]
    FIND --> SERVE[Serve graph via /__graph endpoint]
    SERVE --> LOAD[Dashboard loads graph]
    LOAD --> INDEX[Build search indexes]
    INDEX --> LAYOUT[Compute ELK layout]
    LAYOUT --> RENDER[Render ReactFlow graph]
    RENDER --> INTERACT[User interaction: click, search, filter, tour]
```

## Build & Test Workflow

```mermaid
flowchart LR
    INSTALL[pnpm install] --> BUILD_CORE[pnpm --filter core build]
    BUILD_CORE --> BUILD_SKILL[pnpm --filter skill build]
    BUILD_SKILL --> TEST_CORE[pnpm --filter core test]
    TEST_CORE --> TEST_SKILL[pnpm --filter skill test]
    TEST_SKILL --> LINT[pnpm lint]
```

### CI Pipeline (GitHub Actions)

Triggered on pull requests:
1. Checkout → pnpm setup → Node 22
2. `pnpm install` (cached)
3. Build core → Build skill
4. Test core → Test skill

## Incremental Update Workflow

```mermaid
sequenceDiagram
    participant User
    participant Fingerprint as Fingerprint Store
    participant Pipeline
    participant Graph as Existing Graph

    User->>Pipeline: /understand (re-run)
    Pipeline->>Fingerprint: Load stored hashes
    Pipeline->>Pipeline: Hash current files
    Pipeline->>Fingerprint: Compare (isStale)
    Fingerprint-->>Pipeline: Changed file list
    Pipeline->>Pipeline: Extract only changed files
    Pipeline->>Graph: mergeGraphUpdate (patch)
    Pipeline->>Fingerprint: Save updated hashes
```

## Harness Execution Modes

The Kiro harness (`run-understand.sh`) supports multiple execution modes:

| Mode | Flag | LLM Backend |
|------|------|-------------|
| Full (with LLM) | (default) | LiteLLM proxy → any OpenAI-compatible API |
| No LLM | `--no-llm` | None — structure-only graph |
| Local (LM Studio) | `--local` | localhost:1234 |
| Ollama | `--ollama [model]` | localhost:11434 |
| Full rebuild | `--full` | Forces re-analysis of all files |

## Adding a New Language (Extension Workflow)

1. Create `packages/core/src/plugins/extractors/{lang}-extractor.ts` implementing `AnalyzerPlugin`
2. Add tree-sitter grammar dependency to `packages/core/package.json`
3. Register in `packages/core/src/plugins/extractors/index.ts`
4. Add language config in `packages/core/src/languages/configs/{lang}.ts`
5. Register config in `packages/core/src/languages/configs/index.ts`
6. Add tests in `packages/core/src/plugins/extractors/__tests__/{lang}-extractor.test.ts`
7. Add grammar to `pnpm.onlyBuiltDependencies` in root `package.json`
