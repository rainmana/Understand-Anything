# Review Notes

## Consistency Check

### Cross-document Terminology ✅
- "Knowledge graph" used consistently across all docs
- Node/edge type names match `types.ts` definitions exactly
- Package names (`@understand-anything/core`, `@understand-anything/dashboard`, `@understand-anything/skill`) consistent
- Harness naming (Kiro, LiteLLM) consistent

### Cross-references ✅
- `index.md` references all other docs correctly
- `architecture.md` references patterns described in `components.md`
- `data_models.md` types align with `interfaces.md` API signatures
- `workflows.md` pipeline stages match `components.md` module descriptions

### Version/Tech Consistency ✅
- React 19 referenced in `dependencies.md` and `codebase_info.md`
- Node ≥22 consistent across `codebase_info.md`, `dependencies.md`, and CI description
- pnpm 10.6.2 consistent
- Tailwind v4 consistent

### Minor Inconsistency Found
- `CONTRIBUTING.md` (existing) mentions "React 18" in the Tech Stack section, but `package.json` specifies React 19 (`^19.0.0`). This should be corrected in the consolidated CONTRIBUTING.md.

## Completeness Check

### Well-Covered Areas ✅
- Core analysis engine (extractors, parsers, graph builder)
- Dashboard components and layout system
- Data model (full type coverage from types.ts)
- Pipeline workflow (all stages documented)
- Dependency rationale
- Harness system (Kiro, LiteLLM, test)

### Gaps Identified

| Area | Gap | Severity | Recommendation |
|------|-----|----------|----------------|
| Agent prompts | The `agents/` directory and SKILL.md files are referenced but their prompt engineering patterns are not documented | Low | Document prompt structure conventions in a future iteration |
| Error handling | No documentation of error recovery patterns in the pipeline | Low | Add error handling section to workflows.md when patterns are formalized |
| Dashboard routing | Token-gated access and URL parameter handling not fully documented | Low | Document in interfaces.md if public API |
| Locale contribution | Process for adding new i18n locales not documented | Low | Add to workflows.md extension section |
| Homepage | Astro-based homepage package barely mentioned | Low | Minimal relevance to core functionality |
| `install.sh` internals | Multi-platform installer logic not deeply documented | Medium | Document platform detection and symlink strategy |
| Test patterns | Testing conventions (mocking tree-sitter, fixture patterns) not documented | Medium | Would help contributors write tests faster |

### Language Support Gaps
- **Lua, Swift, Kotlin** — language configs exist but no tree-sitter extractors (handled by non-code parsers or LLM-only analysis)
- This is a known design choice, not a documentation gap

## Recommendations

1. **Fix React version** in existing CONTRIBUTING.md (18 → 19)
2. **Consider documenting** the SKILL.md prompt format for contributors who want to add new skills
3. **Consider documenting** test fixture patterns (mock tree-sitter setup) for contributor onboarding
4. **The `install.sh` platform detection** logic is complex enough to warrant a brief section in workflows.md

## Overall Assessment

Documentation is **comprehensive and internally consistent**. The generated docs accurately reflect the codebase structure as observed in package.json files, type definitions, and directory layout. The one factual inconsistency (React version in existing CONTRIBUTING.md) will be corrected during consolidation.
