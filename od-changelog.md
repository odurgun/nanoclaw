# od-changelog.md

Custom changelog for the `od-nanoclaw` fork. Tracks all edits, upstream syncs, and rollback points.

---

## 2026-03-30 — Full setup (fresh install)

### What happened

1. Git remits verified — `origin` → `odurgun/nanoclaw`, `upstream` → `qwibitai/nanoclaw`
2. Bootstrap: Node.js 24.14.0, deps, native modules — all OK
3. Environment: Linux, Docker running, no prior auth/groups
4. Timezone: `America/New_York` (auto-detected)
5. Container: Docker runtime, image built and tested
6. Credentials: OneCLI already running, existing Anthropic API key kept
7. Channel: Slack (main channel `C0ALUNEV0JX`, trigger `@Andy`)
8. Mounts: `/home/ozcan/Documents`, `/home/ozcan/projects` (read-write)
9. Service: systemd user service, running

### Setup results

| Step        | Status                      |
| ----------- | --------------------------- |
| Bootstrap   | ✅ Node 24.14.0, deps OK    |
| Environment | ✅ Linux, Docker running    |
| Timezone    | ✅ America/New_York         |
| Container   | ✅ Built, tested            |
| Credentials | ✅ OneCLI + Anthropic key   |
| Channel     | ✅ Slack main `C0ALUNEV0JX` |
| Mounts      | ✅ 2 directories (rw)       |
| Service     | ✅ systemd running          |
| Verify      | ✅ All checks passed        |

### Config snapshot

- **Platform:** Linux, x86_64
- **Runtime:** Docker
- **NanoClaw version:** 1.2.43
- **Node:** 24.14.0
- **Channel:** Slack (main)
- **Assistant name:** Andy
- **Credential system:** OneCLI Agent Vault

---

## 2026-03-30 — Upstream sync v1.2.43 + skill updates

### What happened

1. Merged upstream `main` into `od-nanoclaw` (v1.2.42 → v1.2.43)
2. Updated `skill/channel-formatting` (3 new commits)
3. Updated `skill/ollama-tool` (3 new commits)

### Upstream changes merged (v1.2.42 → v1.2.43)

**Fixes:**

- Corrected stale session regex and removed duplicate retry logic
- Fixed npm audit errors
- Auto-recovered from stale Claude Code session on exit code 1
- Setup skill skips `/use-native-credential-proxy` for apple container

**Features:**

- Added model management tools to `skill/ollama-tool`

### Skills updated

**channel-formatting:**

- Added `text-styles.ts` (Signal text style parser)
- Updated `formatting.test.ts` with text style tests
- Updated `router.ts` integration

**ollama-tool:**

- Added `OLLAMA_ADMIN_TOOLS` env flag
- Added `OLLAMA_HOST` to `.env.example`
- Updated `config.ts` to read `OLLAMA_ADMIN_TOOLS`
- Updated `container-runner.ts` to forward ollama flag to containers

### Conflicts resolved

| File                      | Resolution                                                     |
| ------------------------- | -------------------------------------------------------------- |
| `package-lock.json`       | Kept upstream version                                          |
| `.env.example`            | Combined `SLACK_*` + `OLLAMA_HOST` vars                        |
| `src/config.ts`           | Added `OLLAMA_ADMIN_TOOLS` to env reader, dropped `ONECLI_URL` |
| `src/container-runner.ts` | Kept native credential proxy, added ollama flag forwarding     |

### Test fix

- `src/container-runner.test.ts` — added `OLLAMA_ADMIN_TOOLS: false` to config mock

### Validation

- ✅ Build: `npm run build` — passed
- ✅ Tests: `npm test` — 330 tests passed
- ✅ Branch: `od-nanoclaw` is at `0717948`

### Rollback

```bash
git checkout od-nanoclaw && git reset --hard pre-update-2092d55-20260330-193449
```

### Commit history (this session)

```
0717948 Merge remote-tracking branch 'upstream/skill/ollama-tool' into od-nanoclaw
4b877af style: apply prettier formatting post-merge
d59c80b Merge remote-tracking branch 'upstream/main' into od-nanoclaw
b7e9c71 Merge remote-tracking branch 'upstream/skill/channel-formatting' into od-nanoclaw
2092d55 before updating to 1.2.42, with new oneCLI setup    ← fork base
```

---

## 2026-03-30 — containerConfig.additionalMounts MCP tool support

### What happened

1. Added `containerConfig` parameter to `register_group` MCP tool schema
2. Verified full pipeline (types, DB, IPC, mount validation, container startup) already existed
3. Tested end-to-end: DB storage, Docker mount verification

### Change

**Only file modified:** `container/agent-runner/src/ipc-mcp-stdio.ts`

- Added optional `containerConfig.additionalMounts` Zod schema to `register_group` tool (lines ~463–481)
- Passes `containerConfig` through in the IPC data object (lines ~505–506)

### Why it was small

The host-side infrastructure was already complete:

| Layer             | File                              | Status       |
| ----------------- | --------------------------------- | ------------ |
| Types             | `src/types.ts:1-43`               | Pre-existing |
| DB schema         | `src/db.ts:82`                    | Pre-existing |
| DB read/write     | `src/db.ts:606-674`               | Pre-existing |
| IPC handler       | `src/ipc.ts:453`                  | Pre-existing |
| Mount validation  | `src/mount-security.ts:336-385`   | Pre-existing |
| Container startup | `src/container-runner.ts:213-221` | Pre-existing |

### Test data applied

Group `slack:C0ALUNEV0JX` updated with:

```json
{
  "additionalMounts": [
    {
      "hostPath": "/home/ozcan/Documents",
      "containerPath": "documents",
      "readonly": false
    },
    {
      "hostPath": "/home/ozcan/projects",
      "containerPath": "projects",
      "readonly": false
    }
  ]
}
```

### Validation

- ✅ Build: `npm run build` — passed
- ✅ Tests: `npm test` — 285 tests passed
- ✅ Docker: `/workspace/extra/documents` and `/workspace/extra/projects` verified inside container

---

## 2026-03-27 — Upstream sync v1.2.41 + AGENTS.md

### What happened

1. Updated `main` from upstream (`c3e9a89`, v1.2.41)
2. Rebased `od-nanoclaw` onto updated `main`
3. Created `AGENTS.md` with build/test/style/fork-workflow guidelines

### Upstream changes merged (1.2.14 → 1.2.41)

**Breaking / notable:**

- Replaced `pino`/`pino-pretty` with built-in logger
- Removed `credential-proxy.ts` — replaced with OneCLI Agent Vault (`@onecli-sh/sdk`)
- Removed `yaml`, `zod`, `@vitest/coverage-v8` dependencies
- New dependency: `@onecli-sh/sdk ^0.2.0` (OneCLI Agent Vault)

**Features added:**

- `/init-onecli` skill for OneCLI Agent Vault setup and credential migration
- `/use-native-credential-proxy` skill
- `/add-macos-statusbar` skill
- `/add-emacs` channel skill
- `/channel-formatting` skill
- Scheduled task scripts (pass scripts to container agents)
- Remote Control sessions (Claude Code bridge)
- Telegram DM backfill fix
- Message history overflow prevention (don't send full history to container agents)

**Security fixes:**

- Command injection prevention in `stopContainer`
- Mount path injection prevention
- IPC auth hardening

**Bug fixes:**

- Single-character `.env` value parser crash
- `isMain` template selection in runtime registration
- CLAUDE.md template copy on IPC group registration
- Timezone POSIX-style TZ crash
- WhatsApp phone number prompt clarification
- `enable-linger` for user service persistence
- Per-group trigger patterns
- Agent-runner source cache refresh on code changes

**Docs:**

- Added OneCLI secrets management to CLAUDE.md
- Windows (WSL2) support documented
- Updated security docs, debug checklist, requirements

### Conflicts resolved

| File                    | Resolution                                                                  |
| ----------------------- | --------------------------------------------------------------------------- |
| `.env.example`          | Kept `SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN` from Slack integration            |
| `package.json`          | Merged deps: kept upstream versions + added `@slack/bolt`, `@onecli-sh/sdk` |
| `repo-tokens/badge.svg` | Used upstream version (42.4k tokens)                                        |

### Backup tags (rollback points)

```bash
# Main branch backup (before upstream merge)
git checkout main && git reset --hard pre-update-deee4b2-20260327-190543

# od-nanoclaw backup (before rebase)
git checkout od-nanoclaw && git reset --hard pre-rebase-348ffa6-20260327-190631
```

### Validation

- ✅ Build: `npm run build` — passed
- ✅ Tests: `npm test` — 286 tests passed
- ✅ Branch: `od-nanoclaw` is at `a104cc7`

### Commit history (this session)

```
a104cc7 docs: add AGENTS.md with build, test, code style, and fork workflow guidelines
2927cc5 docs: update token count to 41.0k tokens · 21% of context window
69221a7 skill/slack: Slack channel integration
c3e9a89 chore: bump version to 1.2.41              ← upstream HEAD
acb0aba fix: broken tests and stale .env.example
...
32dda34 status-icon-01                              ← fork base
```

---

## How to use this file

**Before any update:** always create a backup tag:

```bash
HASH=$(git rev-parse --short HEAD)
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
git tag pre-update-$HASH-$TIMESTAMP
```

**Rollback:**

```bash
git reset --hard <tag-name>
```

**Update workflow:**

1. `git checkout main`
2. `/update-nanoclaw` (merge mode)
3. `git checkout od-nanoclaw`
4. `git rebase main`
5. `npm run build && npm test`
6. Add entry to this file
