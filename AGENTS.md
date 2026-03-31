# AGENTS.md

Instructions for agentic coding agents operating in this repository.

## Build / Lint / Test

```bash
# Build
npm run build        # tsc — compile TypeScript to dist/
npm run typecheck    # tsc --noEmit — type-check only

# Dev
npm run dev          # tsx src/index.ts — run with hot reload

# Format
npm run format:fix   # Prettier auto-fix
npm run format:check # Prettier check (CI gate)

# Lint
npm run lint:fix     # ESLint auto-fix

# Test
npm test             # vitest run — all tests
npm run test:watch   # vitest — watch mode

# Single test file
npx vitest run src/db.test.ts

# Single test by name
npx vitest run -t "test name pattern"
```

**CI pipeline order:** `format:check` → `tsc --noEmit` → `vitest run`

**Always run `npm run build` and `npm test` after making changes.**

## Fork Workflow (od-nanoclaw)

- `main`: tracks upstream `qwibitai/nanoclaw`. Never commit directly.
- `od-nanoclaw`: development branch. All custom work happens here.

**Update from upstream:**

1. Switch to `main`: `git checkout main`
2. Run `/update-nanoclaw` (merge mode recommended)
3. Switch back: `git checkout od-nanoclaw`
4. Rebase onto updated main: `git rebase main`
5. Build/test: `npm run build && npm test`

**Always rebase `od-nanoclaw` onto `main` after an upstream update.**

## Code Style

### Imports

- ESM with `.js` extensions (NodeNext module resolution required):
  ```ts
  import { ASSISTANT_NAME } from './config.js';
  import { logger } from './logger.js';
  ```
- Node builtins first, blank line, then local imports
- Named imports preferred; no barrel re-exports

### Naming

- **Files:** kebab-case (`container-runner.ts`, `group-folder.ts`)
- **Tests:** co-located as `<module>.test.ts` (e.g., `src/db.test.ts`)
- **Constants:** UPPER_SNAKE_CASE (`POLL_INTERVAL`, `CONTAINER_IMAGE`)
- **Functions:** camelCase (`formatMessages`, `resolveGroupFolderPath`)
- **Types/Interfaces:** PascalCase (`RegisteredGroup`, `NewMessage`, `ContainerOutput`)

### Formatting

- Single quotes (Prettier)
- 2-space indentation
- Semicolons always
- Trailing commas in multi-line structures

### Typing

- `strict: true` in tsconfig
- Explicit return types on most functions
- Interfaces over type aliases for objects
- Union literal types for status fields: `'success' | 'error'`

## Error Handling

- Named catch parameters required — no bare `catch {}`:
  ```ts
  catch (err) { ... }     // used
  catch (_err) { ... }    // unused, prefix with _
  ```
- `no-catch-all` ESLint plugin warns on overly broad catches
- Use `pino` logger for error reporting:
  ```ts
  logger.error({ err, groupId }, 'Failed to process message');
  ```

## Testing

- Framework: **Vitest** (not Jest)
- API: `describe`, `it`, `expect`, `beforeEach`, `afterEach`, `vi`
- Mocks: `vi.mock()` at module level, `vi.fn()` for spies
- Use `vi.hoisted()` for mock state needed before imports
- One behavior per `it` block
- Tests are self-contained — no shared fixtures

## Project Structure

| Path              | Purpose                                                 |
| ----------------- | ------------------------------------------------------- |
| `src/`            | Main source code                                        |
| `dist/`           | Compiled output (gitignored)                            |
| `container/`      | Agent container (Dockerfile, agent-runner, skills)      |
| `.claude/skills/` | Skills loaded by Claude (4 types — see CONTRIBUTING.md) |
| `groups/`         | Per-group memory (CLAUDE.md per group)                  |
| `setup/`          | First-time installation scripts                         |
| `docs/`           | Architecture docs, security, requirements               |

## Module System

- ESM (`"type": "module"` in package.json)
- TypeScript target: ES2022, moduleResolution: NodeNext
- Node.js >= 20 required
- Runtime: `tsx` for dev, compiled `tsc` for production
