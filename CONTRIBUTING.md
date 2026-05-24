# Contributing to Understand Anything

Thank you for your interest in contributing to Understand Anything! This document provides guidelines and instructions for contributing to the project.

## 🌟 Ways to Contribute

- **Bug Reports**: Found a bug? Open an issue with detailed reproduction steps
- **Feature Requests**: Have an idea? Share it in the issues section
- **Documentation**: Improve or translate documentation
- **Code**: Fix bugs, add features, or improve performance
- **Testing**: Write tests to improve code coverage

## 🚀 Getting Started

### Prerequisites

- Node.js >= 22 (developed on v24)
- pnpm >= 10 (pinned via `packageManager` field in root `package.json`)
- Python 3 (for merge/normalize scripts — stdlib only, no pip deps)
- Git for version control

### Setup

1. **Fork and Clone**
   ```bash
   git clone https://github.com/YOUR_USERNAME/Understand-Anything.git
   cd Understand-Anything
   ```

2. **Install Dependencies**
   ```bash
   pnpm install
   ```

3. **Build Core Package**
   ```bash
   pnpm --filter @understand-anything/core build
   ```

4. **Run Tests**
   ```bash
   pnpm --filter @understand-anything/core test
   pnpm --filter @understand-anything/skill test
   ```

5. **Start Dashboard (Optional)**
   ```bash
   pnpm dev:dashboard
   ```

## 📝 Development Workflow

### 1. Create a Branch

Create a descriptive branch name:
```bash
git checkout -b feat/my-feature        # For new features
git checkout -b fix/bug-description    # For bug fixes
git checkout -b docs/update-readme     # For documentation
```

### 2. Make Changes

- Write clean, readable code
- Follow existing code style and conventions
- Add tests for new functionality
- Update documentation as needed

### 3. Test Your Changes

```bash
# Run all tests
pnpm --filter @understand-anything/core test
pnpm --filter @understand-anything/skill test

# Run Python tests (merge logic)
python -m pytest understand-anything-plugin/skills/understand/test_merge_batch_graphs.py

# Run harness integration tests
bash harnesses/tests/test-harness.sh

# Run linter
pnpm lint

# Build packages
pnpm build
```

### 4. Commit Your Changes

Write clear, descriptive commit messages:
```bash
git add .
git commit -m "feat: add keyboard shortcuts to dashboard"
```

**Commit Message Convention:**
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation changes
- `style:` - Code style changes (formatting, etc.)
- `refactor:` - Code refactoring
- `test:` - Adding or updating tests
- `chore:` - Maintenance tasks

### 5. Push and Create Pull Request

```bash
git push origin your-branch-name
```

Then open a Pull Request on GitHub with:
- Clear title describing the change
- Detailed description of what changed and why
- Link to related issues (if any)
- Screenshots (for UI changes)

## 🏗️ Project Structure

```
understand-anything-plugin/
├── packages/
│   ├── core/              # Core analysis engine (@understand-anything/core)
│   │   └── src/
│   │       ├── plugins/extractors/  # Per-language tree-sitter extractors
│   │       ├── plugins/parsers/     # Non-code file parsers
│   │       ├── analyzer/            # Graph builder, layer detector, tours
│   │       ├── persistence/         # File I/O
│   │       └── languages/           # Language/framework registries
│   └── dashboard/         # React dashboard (@understand-anything/dashboard)
│       └── src/
│           ├── components/  # React components
│           ├── utils/       # Layout, filtering, aggregation
│           ├── themes/      # CSS theming
│           └── locales/     # i18n translations
├── src/                   # Plugin skill implementations
├── skills/                # Skill definitions (SKILL.md + scripts)
└── package.json           # @understand-anything/skill
harnesses/
├── kiro/                  # Kiro CLI harness
├── litellm/               # LiteLLM proxy client
└── tests/                 # Integration tests
```

### Build Order

Packages must be built in dependency order:
1. `@understand-anything/core` (no internal deps)
2. `@understand-anything/skill` (depends on core)
3. `@understand-anything/dashboard` (depends on core)

## 🧪 Testing Guidelines

### Writing Tests

- Use Vitest for TypeScript tests
- Place tests in `__tests__` directories or `*.test.ts` files
- Use `python -m pytest` for Python test files
- Aim for high test coverage for new features
- Test edge cases and error conditions

Example test structure:
```typescript
import { describe, it, expect } from 'vitest';

describe('MyFeature', () => {
  it('should do something', () => {
    const input = 'test';
    const result = myFunction(input);
    expect(result).toBe('expected');
  });
});
```

### Running Tests

```bash
# Run all TypeScript tests
pnpm test

# Run tests for specific package
pnpm --filter @understand-anything/core test

# Run tests in watch mode
pnpm --filter @understand-anything/core test -- --watch

# Run Python merge tests
python -m pytest understand-anything-plugin/skills/understand/test_merge_batch_graphs.py -v
```

## 📚 Code Style Guidelines

### TypeScript

- Use TypeScript strict mode (enforced by `tsconfig.json`)
- Define explicit types for function parameters and return values
- Avoid `any` type — use `unknown` if type is truly unknown
- Use interfaces for object shapes
- Use type aliases for unions and complex types
- Target ES2022 with bundler module resolution

### Formatting

- The project uses ESLint for code quality
- Consistent indentation (2 spaces)
- Use meaningful variable and function names
- Keep functions small and focused

### React/Dashboard

- Use functional components with hooks
- Keep components focused and single-purpose
- Use Zustand for state management (single store pattern)
- Follow the existing component structure
- Use Tailwind CSS v4 for styling

### Python Scripts

- Use only Python standard library (no pip dependencies)
- Follow existing patterns in `merge-batch-graphs.py`
- Include type hints where practical
- Test with `pytest`

### Tech Stack

TypeScript, pnpm workspaces, React 19, Vite 6, Tailwind CSS v4, ReactFlow 12, Zustand 5, web-tree-sitter, Fuse.js, Zod 4, ELK.js, Dagre

## 🔌 Adding a New Language Extractor

1. Create `packages/core/src/plugins/extractors/{lang}-extractor.ts` implementing `AnalyzerPlugin`
2. Add tree-sitter grammar to `packages/core/package.json` dependencies
3. Register in `packages/core/src/plugins/extractors/index.ts`
4. Add language config in `packages/core/src/languages/configs/{lang}.ts`
5. Register config in `packages/core/src/languages/configs/index.ts`
6. Add grammar to `pnpm.onlyBuiltDependencies` in root `package.json`
7. Write tests in `packages/core/src/plugins/extractors/__tests__/{lang}-extractor.test.ts`

## 🌍 Translation Guidelines

### Adding a New Language

1. Create locale file in `packages/dashboard/src/locales/{code}.ts`
2. Export all keys matching the `en.ts` structure
3. Register in `packages/dashboard/src/locales/index.ts`
4. Create `READMEs/README.{language-code}.md`
5. Update main `README.md` to include language link

## 🐛 Bug Reports

When reporting bugs, include:

- **Description**: Clear description of the issue
- **Steps to Reproduce**: Detailed steps to reproduce the bug
- **Expected Behavior**: What you expected to happen
- **Actual Behavior**: What actually happened
- **Environment**: OS, Node version, pnpm version
- **Screenshots**: If applicable
- **Error Messages**: Full error output

## 💡 Feature Requests

When requesting features:

- **Use Case**: Describe the problem you're trying to solve
- **Proposed Solution**: How you envision the feature working
- **Alternatives**: Other solutions you've considered
- **Additional Context**: Any other relevant information

## 📋 Pull Request Checklist

Before submitting a PR, ensure:

- [ ] Code follows the project's style guidelines
- [ ] All tests pass (`pnpm test`)
- [ ] New code has test coverage
- [ ] Documentation is updated (if needed)
- [ ] Commit messages follow convention
- [ ] PR description clearly explains changes
- [ ] No console.log or debug code left behind
- [ ] Branch is up to date with main

## 🤝 Code Review Process

1. **Automated Checks**: CI runs tests and linting
2. **Maintainer Review**: Project maintainers review the code
3. **Feedback**: Address any requested changes
4. **Approval**: Once approved, PR will be merged
5. **Cleanup**: Delete your branch after merge

## 📞 Getting Help

- **Issues**: For bugs and feature requests
- **Discussions**: For questions and general discussion
- **Discord**: [Join the community](https://discord.gg/pydat66RY)
- **Documentation**: Check existing docs first

## 📄 License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for contributing to Understand Anything! Your contributions help make code understanding accessible to everyone.** 🚀
