# Interfaces

## Overview

The system defines its contracts in `packages/core/src/types.ts`. All interfaces are TypeScript-first with strict typing. The core exports sub-paths (`/types`, `/schema`, `/search`, `/languages`) for selective imports.

## Plugin Interface

The primary extension point for adding language support:

```typescript
interface AnalyzerPlugin {
  name: string;
  languages: string[];
  analyzeFile(filePath: string, content: string): StructuralAnalysis;
  resolveImports?(filePath: string, content: string): ImportResolution[];
  extractCallGraph?(filePath: string, content: string): CallGraphEntry[];
  extractReferences?(filePath: string, content: string): ReferenceResolution[];
}
```

- `analyzeFile` is required — returns functions, classes, imports, exports, and optional non-code structures
- `resolveImports`, `extractCallGraph`, `extractReferences` are optional capabilities
- Plugins register via `PluginRegistry.register(plugin)`

## Plugin Registry API

```typescript
class PluginRegistry {
  register(plugin: AnalyzerPlugin): void;
  unregister(name: string): void;
  getPluginForFile(filePath: string): AnalyzerPlugin | undefined;
  getPluginForLanguage(language: string): AnalyzerPlugin | undefined;
  getLanguageForFile(filePath: string): string | undefined;
  analyzeFile(filePath: string, content: string): StructuralAnalysis;
  resolveImports(filePath: string, content: string): ImportResolution[];
  extractCallGraph(filePath: string, content: string): CallGraphEntry[];
  getSupportedLanguages(): string[];
  getPlugins(): AnalyzerPlugin[];
}
```

## Structural Analysis Result

Returned by every `analyzeFile` call:

```typescript
interface StructuralAnalysis {
  functions: Array<{ name: string; lineRange: [number, number]; params: string[]; returnType?: string }>;
  classes: Array<{ name: string; lineRange: [number, number]; methods: string[]; properties: string[] }>;
  imports: Array<{ source: string; specifiers: string[]; lineNumber: number }>;
  exports: Array<{ name: string; lineNumber: number; isDefault?: boolean }>;
  // Non-code structural data (optional)
  sections?: SectionInfo[];
  definitions?: DefinitionInfo[];
  services?: ServiceInfo[];
  endpoints?: EndpointInfo[];
  steps?: StepInfo[];
  resources?: ResourceInfo[];
}
```

## Graph Builder API

```typescript
class GraphBuilder {
  constructor();
  addFile(filePath: string, language: string): void;
  addFileWithAnalysis(filePath: string, analysis: StructuralAnalysis, language: string): void;
  addNonCodeFile(filePath: string): void;
  addNonCodeFileWithAnalysis(filePath: string, analysis: StructuralAnalysis): void;
  addImportEdge(sourceId: string, targetId: string): void;
  addCallEdge(callerId: string, calleeId: string): void;
  addChildNode(parentId: string, child: { name: string; type: NodeType; lineRange: [number, number] }): void;
  build(): KnowledgeGraph;
}
```

## Search Engine API

```typescript
class SearchEngine {
  constructor(nodes: GraphNode[]);
  search(query: string): GraphNode[];
  updateNodes(nodes: GraphNode[]): void;
}

class SemanticSearchEngine {
  constructor();
  addEmbedding(nodeId: string, embedding: number[]): void;
  search(query: number[], limit?: number): Array<{ nodeId: string; score: number }>;
  hasEmbeddings(): boolean;
  updateNodes(nodes: GraphNode[]): void;
}
```

## Persistence API

```typescript
// All functions take a projectRoot path
function saveGraph(projectRoot: string, graph: KnowledgeGraph): void;
function loadGraph(projectRoot: string): KnowledgeGraph | null;
function saveFingerprints(projectRoot: string, fingerprints: Map<string, string>): void;
function loadFingerprints(projectRoot: string): Map<string, string>;
function saveMeta(projectRoot: string, meta: AnalysisMeta): void;
function loadMeta(projectRoot: string): AnalysisMeta | null;
function saveConfig(projectRoot: string, config: ProjectConfig): void;
function loadConfig(projectRoot: string): ProjectConfig | null;
function saveDomainGraph(projectRoot: string, graph: KnowledgeGraph): void;
function loadDomainGraph(projectRoot: string): KnowledgeGraph | null;
```

## Schema Validation API

```typescript
function validateGraph(graph: unknown): { valid: boolean; errors: string[] };
function normalizeGraph(graph: KnowledgeGraph): KnowledgeGraph;
function autoFixGraph(graph: KnowledgeGraph): KnowledgeGraph;
function sanitizeGraph(graph: KnowledgeGraph): KnowledgeGraph;
```

## LLM Client Interface (Harness)

The LiteLLM client (`harnesses/litellm/llm-client.mjs`) exposes a stdin/stdout interface:

```
Input:  JSON on stdin { "prompt": string, "system"?: string, "temperature"?: number }
Output: JSON on stdout { "content": string }
```

Environment variables: `LITELLM_BASE_URL`, `LITELLM_API_KEY`, `LITELLM_MODEL`

## Dashboard Vite Dev Server API

The dashboard's Vite config exposes custom endpoints:

| Endpoint | Purpose |
|----------|---------|
| `/__graph` | Serves the knowledge graph JSON |
| `/__source?file=<path>` | Serves source file content for CodeViewer |

## Language Registry API

```typescript
class LanguageRegistry {
  register(config: LanguageConfig): void;
  getById(id: string): LanguageConfig | undefined;
  getByExtension(ext: string): LanguageConfig | undefined;
  getForFile(filePath: string): LanguageConfig | undefined;
  getAllLanguages(): LanguageConfig[];
  static createDefault(): LanguageRegistry;
}
```

## Framework Registry API

```typescript
class FrameworkRegistry {
  register(config: FrameworkConfig): void;
  getById(id: string): FrameworkConfig | undefined;
  getForLanguage(languageId: string): FrameworkConfig[];
  getAllFrameworks(): FrameworkConfig[];
  detectFrameworks(files: string[]): FrameworkConfig[];
  static createDefault(): FrameworkRegistry;
}
```
