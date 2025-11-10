# GitHub Copilot Instructions for vscode-github-issue-notebooks

This repository contains a VS Code extension that enables GitHub Issue Notebooks - a notebook interface for querying and displaying GitHub issues and pull requests within VS Code.

## Project Overview

- **Extension Type**: VS Code extension with both Node.js and Web support
- **Language**: TypeScript with ESM modules
- **Build Tool**: esbuild (see `esbuild.mjs`)
- **Testing**: Mocha for unit and integration tests
- **Linting**: ESLint with TypeScript parser
- **Package Manager**: npm

## Architecture

### Key Components

1. **Extension Entry Point** (`src/extension/extension.ts`)
   - Activates on `github-issues` notebooks
   - Registers notebook kernel, serializer, language provider, and commands

2. **Notebook Provider** (`src/extension/notebookProvider.ts`)
   - `IssuesNotebookKernel`: Executes GitHub issue queries
   - `IssuesNotebookSerializer`: Handles notebook serialization
   - `IssuesStatusBarProvider`: Provides status bar items for cells

3. **Language Provider** (`src/extension/languageProvider.ts`)
   - Syntax highlighting, validation, completions
   - Code navigation (find references, go to definition)
   - Rename and formatting support

4. **Parser** (`src/extension/parser/`)
   - `scanner.ts`: Tokenizes GitHub issue query syntax
   - `parser.ts`: Parses tokens into AST
   - `nodes.ts`: AST node definitions
   - `symbols.ts`: Symbol table for variables
   - `validation.ts`: Query validation logic

5. **GitHub Integration** (`src/extension/octokitProvider.ts`)
   - Manages GitHub authentication via Octokit
   - Provides API access for issue queries

6. **Renderer** (`src/renderer/`)
   - Notebook cell output rendering
   - Uses Preact for UI components

## Coding Standards

### TypeScript

- **Module System**: Use ESM (ES Modules) with `.js` extensions in imports
- **Strict Mode**: Follow TypeScript strict type checking
- **Semicolons**: Always end statements with semicolons (enforced by ESLint rule `@typescript-eslint/semi`)
- **No `var`**: Use `const` or `let` (enforced by ESLint rule `no-var`)
- **Equality**: Use strict equality `===` and `!==` (enforced by ESLint rule `eqeqeq`)
- **Curly Braces**: Always use curly braces for control statements (enforced by ESLint rule `curly`)

### File Organization

- Extension code: `src/extension/`
- Renderer code: `src/renderer/`
- Common utilities: `src/common/`
- Unit tests: `test/test-unit/`
- Integration tests: `test/test-integration/`

### Import Conventions

- Always use `.js` extensions in TypeScript imports (ESM requirement)
- Example: `import { foo } from './bar.js';`
- Import from VS Code: `import * as vscode from 'vscode';`
- External imports should be clearly separated from local imports

### Naming Conventions

- **Classes**: PascalCase (e.g., `IssuesNotebookKernel`, `ProjectContainer`)
- **Interfaces**: PascalCase with "I" prefix where appropriate
- **Functions/Methods**: camelCase (e.g., `registerCommands`, `executeCell`)
- **Constants**: UPPER_SNAKE_CASE for true constants, camelCase for config objects
- **Files**: camelCase for TypeScript files (e.g., `notebookProvider.ts`)

## Building and Testing

### Build Commands

```bash
# Compile TypeScript
npm run ts-compile

# Bundle with esbuild (development)
npm run esbuild

# Bundle with esbuild (watch mode)
npm run esbuild:watch

# Bundle with esbuild (minified for production)
npm run esbuild:minify

# Prepare for publishing (runs esbuild)
npm run vscode:prepublish
```

### Testing Commands

```bash
# Run unit tests
npm run unit-test

# Run integration tests
npm run integration-test

# Lint the codebase
npm run lint

# Full validation (compile + lint + test)
npm run compile-lint-test
```

### Build Output

- Compiled TypeScript output: `out/`
- Bundled extension output: `dist/`
  - `dist/extension-node.js`: Node.js entry point
  - `dist/extension-web.cjs`: Web/browser entry point
  - `dist/renderer.js`: Notebook renderer

## Dependencies

### Runtime Dependencies

- `@octokit/rest`: GitHub API client (v22.0.0+)

### Development Dependencies

- TypeScript 5.8.3+
- ESLint with TypeScript support
- esbuild for bundling
- Mocha for testing
- Preact for UI rendering

## Security Considerations

- **GitHub Tokens**: Always use VS Code's authentication provider (`vscode.authentication.getSession`)
- **User Input**: Validate all user input in queries before execution
- **External Dependencies**: Keep dependencies up to date, especially `@octokit/rest`
- **Secrets**: Never commit GitHub tokens or personal access tokens

## Extension-Specific Guidelines

### Notebook Cell Execution

- Cells contain GitHub issue query syntax
- Variables start with `$` (e.g., `$vscode=repo:microsoft/vscode`)
- Queries support GitHub search syntax: https://docs.github.com/en/search-github/searching-on-github/searching-issues-and-pull-requests
- OR queries: separate queries with `OR`
- Comments start with `//`

### Query Parser

- When modifying the parser, update tests in `test/test-unit/parser.test.ts`
- Scanner produces tokens, parser builds AST, validation checks semantics
- Symbol table tracks variable definitions across cells

### Language Provider Features

- Completions should include GitHub search qualifiers (e.g., `repo:`, `author:`, `label:`)
- Validation should provide clear error messages for invalid syntax
- Hover information should explain query syntax

### Renderer Updates

- The renderer runs in a separate webview context
- Use Preact for UI components
- Keep renderer bundle size small for performance
- Style with CSS modules when possible

## Testing Guidelines

### Unit Tests

- Test parser components in isolation
- Mock VS Code API using `vscode.d.ts` or test doubles
- Focus on logic, not VS Code integration
- Run with: `npm run unit-test`

### Integration Tests

- Test full extension activation and features
- Use VS Code test runner environment
- Test notebook creation, cell execution, and output
- Run with: `npm run integration-test`

### Test File Naming

- Unit tests: `*.test.ts` in `test/test-unit/`
- Integration tests: `*.test.ts` in `test/test-integration/`

## Documentation

- Update README.md when adding user-facing features
- Document new query syntax in README.md examples
- Add JSDoc comments for public APIs
- Keep CONTRIBUTING.md updated with development workflow changes

## Pre-commit Checks

- Run `npm run precommit` (lints the code)
- Ensure `npm run compile-lint-test` passes before submitting PRs
- Fix all ESLint warnings before committing

## VS Code Extension Guidelines

- Follow VS Code extension best practices: https://code.visualstudio.com/api/references/extension-guidelines
- Support both VS Code Desktop and vscode.dev (web)
- Extension must work in untrusted workspaces
- Support virtual workspaces (no file system access required)

## Localization

- Localization files in `l10n/` directory
- Use `package.nls.json` for extension contribution localization keys
- Format: `%key%` in `package.json`, with key defined in `package.nls.json`

## Publishing

- Use `npm run deploy` to publish (requires vsce and proper permissions)
- Ensure version is bumped in `package.json`
- Extension is published under `ms-vscode` publisher
- Currently marked as `"preview": true`

## Common Patterns

### Disposables

Always register disposables with extension context:
```typescript
context.subscriptions.push(disposable);
```

### VS Code API Usage

- Use `vscode.*` namespaces appropriately
- Register providers with proper disposal
- Handle errors gracefully with user-friendly messages

### Async Operations

- Use `async/await` for asynchronous code
- Handle GitHub API rate limits gracefully
- Show progress indicators for long-running operations

## Troubleshooting

- **Build Failures**: Check that `npm install` completed successfully
- **Extension Not Activating**: Verify `activationEvents` in `package.json`
- **Notebook Not Opening**: Check file pattern `*.github-issues` matches
- **Query Execution Fails**: Verify GitHub authentication is working

## Additional Resources

- VS Code Extension API: https://code.visualstudio.com/api
- GitHub Search Syntax: https://docs.github.com/en/search-github/searching-on-github/searching-issues-and-pull-requests
- Octokit REST API: https://octokit.github.io/rest.js/
- Notebook API: https://code.visualstudio.com/api/extension-guides/notebook
