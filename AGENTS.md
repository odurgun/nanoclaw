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

- **GitHub:** `odurgun/nanoclaw` (origin), `qwibitai/nanoclaw` (upstream)
- `main`: tracks upstream `qwibitai/nanoclaw`. Never commit directly.
- `od-nanoclaw`: development branch. All custom work happens here. Push to origin.

**Commit style:** imperative mood, concise. Prefix with scope when relevant (e.g., `fix:`, `feat:`, `docs:`, `style:`, `chore:`). Free-form messages OK for descriptive commits (e.g., `Documents and projects folders are shared`).

**Update from upstream:**

1. Switch to `main`: `git checkout main`
2. Run `/update-nanoclaw` (merge mode recommended)
3. Switch back: `git checkout od-nanoclaw`
4. Rebase onto updated main: `git rebase main`
5. Build/test: `npm run build && npm test`
6. Push: `git push origin od-nanoclaw`

**Always rebase `od-nanoclaw` onto `main` after an upstream update.**

**Push after commits:** `git push origin od-nanoclaw`

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
- Use the built-in `logger` (from `./logger.js`) for error reporting:
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

## Service Management

```bash
# Start / stop / restart
systemctl --user start nanoclaw
systemctl --user stop nanoclaw
systemctl --user restart nanoclaw

# Status
systemctl --user status nanoclaw

# Logs (stdout/stderr redirected to files)
tail -f logs/nanoclaw.log          # normal output
tail -f logs/nanoclaw.error.log    # errors only
```

**Service file:** `~/.config/systemd/user/nanoclaw.service`

## Key Paths

| Path                                      | Purpose                                        |
| ----------------------------------------- | ---------------------------------------------- |
| `store/messages.db`                       | SQLite database (groups, messages, tasks)      |
| `logs/nanoclaw.log`                       | Application stdout                             |
| `logs/nanoclaw.error.log`                 | Application stderr / errors                    |
| `~/.config/nanoclaw/mount-allowlist.json` | Mount security allowlist                       |
| `groups/`                                 | Per-group CLAUDE.md memory                     |
| `container/agent-runner/`                 | Separate sub-project built inside Docker image |

## Mount Allowlist

Location: `~/.config/nanoclaw/mount-allowlist.json`

```json
{
  "allowedRoots": [
    {
      "path": "/home/ozcan/Documents",
      "allowReadWrite": true
    },
    {
      "path": "/home/ozcan/projects",
      "allowReadWrite": true
    }
  ],
  "blockedPatterns": [],
  "nonMainReadOnly": false
}
```

**Important:** `allowedRoots` entries must be objects with a `path` property — plain strings will cause a crash (`Cannot read properties of undefined (reading 'startsWith')`).

## Container Image

- `container/agent-runner/` is a separate project with its own `package.json`
- Built inside the Docker image during `docker build` — not by `npm run build` at the project root
- After editing files under `container/agent-runner/src/`, the Docker image must be rebuilt for changes to take effect
- MCP tools are defined in `container/agent-runner/src/ipc-mcp-stdio.ts`
