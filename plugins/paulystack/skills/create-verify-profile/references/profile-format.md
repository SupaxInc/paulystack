# Verify profile format

## Contents
- [Location](#location)
- [Frontmatter](#frontmatter)
- [Sections](#sections)
- [Never stored](#never-stored)
- [Monorepos](#monorepos)
- [Example](#example)

## Location

`~/.claude/verify-profiles/<repo>/profile.md`. `<repo>` is the basename of the parent of `git rev-parse --path-format=absolute --git-common-dir`, so every worktree of a repo shares one profile. It stays machine-local: it can name internal systems, so never copy it into the repo or any synced config.

## Frontmatter

```yaml
---
repo: <repo key>
origin: <git remote get-url origin, or none>
generated_from: <40-char HEAD sha when last written>
generated_at: <YYYY-MM-DD>
watch: [<files and dirs whose change can make this profile wrong>]
---
```

- `origin`: pauly-mode stops when it differs from the current origin (two repos sharing a basename).
- `watch`: launch, build, dependency, CI, and deploy files, plus any `.claude/skills/run-*/` or `.claude/skills/verify/` recipe the profile points at. pauly-mode diffs them since `generated_from` to flag drift.

## Sections

Each stage heading ends in `proved: yes` or `proved: no (<reason>)`, from the generator's own trial. Each command cites the file it came from, e.g. `(Makefile:12)`.

**Surfaces**: one line per surface a user touches (HTTP API, web UI, CLI, worker, mobile app): its entry point and source file.

**Local**
- `Run`: exact command(s), `ready when` (log line, port, prompt), `stop` (how to end what you started).
- `Env`: variable names and where values come from (`.env.example`, `1Password: "ledger dev"`). Never values.
- `Doctor`: one read-only check that the running instance is the right one and worth driving.
- `Drive`: per surface, the tool and real handles from this repo (routes, selectors, commands, REPL namespaces).
- `Tests (backup)`: whole suite, one file, one test.
- `Existing recipes`: paths to `.claude/skills/run-*/` or `.claude/skills/verify/SKILL.md`, pointed at rather than copied.
- `Can't run locally`: only when true, with the reason; proof then falls back to tests.

**Staging**
- Base URL, environment name, and observability tags (`service:ledger-api env:staging`).
- `Is my commit deployed`: the read-only command, how to read its output, and known false negatives.
- `Observe`: tool suffix → query template, per signal (errors, logs, latency, deploy events).
- `Safe test actions`: user-supplied only, each with its cleanup. `none` means staging is read-only too.

**Prod (read-only)**: `Is my commit deployed`, `Observe`, and a `Forbidden` list (writes, restarts, flag flips, anything not a read).

**Services**: name · where its code is (path or `../sibling`) · how this repo reaches it · its log/APM tag.

**Gotchas**: traps that silently invalidate a run (seed data needed, port shared with other worktrees, a flag that must be on).

## Never stored

Secret values, tokens, cookies, passwords, customer data, raw prod log lines, and any staging write the user didn't approve.

## Monorepos

When services build or deploy separately, `profile.md` keeps the shared parts and links one `<service>.md` per service with that service's stage sections.

## Example

A fictional service, for shape only.

```markdown
---
repo: ledger-api
origin: git@github.com:acme/ledger-api.git
generated_from: 3f9c2a1e7b4d8c0f5a6e2b1d9c8f7a6b5e4d3c2b
generated_at: 2026-09-27
watch: [Makefile, package.json, docker-compose.yml, .github/workflows/, deploy/]
---
# Verify profile: ledger-api

## Surfaces
- HTTP API: `src/routes/*.ts`, mounted in `src/server.ts:14`

## Local · proved: yes
Run: `docker compose up -d db` then `npm run dev` (package.json:8) · ready when: `listening on :4000` · stop: Ctrl-C, `docker compose stop db`
Env: DATABASE_URL (.env.example), LEDGER_SIGNING_KEY (1Password: "ledger dev")
Doctor: `curl -s localhost:4000/health` → `{"status":"ok"}`
Drive: HTTP · `curl -s -X POST localhost:4000/transfers -H 'Idempotency-Key: <uuid>' -d @fixtures/transfer.json`; rows via `docker compose exec db psql -c 'select count(*) from transfers'`
Tests (backup): `npm test` · one file `npx vitest run src/transfers.test.ts` · one test `-t "<name>"`
Existing recipes: none

## Staging · proved: yes
Base: https://ledger.staging.acme.dev · tags `service:ledger-api env:staging`
Is my commit deployed: `argocd app get ledger-api-staging -o json` → `.status.sync.revision`, then `gh api -X GET repos/acme/ledger-api/compare/<revision>...<sha> --jq .status` (`behind`/`identical` = included). False negative: image-tag deploys without a git revision.
Observe: errors `search_datadog_error_tracking_issues` (service:ledger-api env:staging) · logs `search_datadog_logs` (`service:ledger-api env:staging status:error`) · latency `get_datadog_metric` (`trace.http.request.duration{service:ledger-api,env:staging}`)
Safe test actions: POST /transfers with test account `acct_test_01` (1Password: "ledger staging test") · cleanup: `DELETE /test-support/transfers?account=acct_test_01`

## Prod (read-only) · proved: no (no prod read access yet)
Is my commit deployed: same as staging with app `ledger-api-prod`
Observe: same tools with `env:prod`
Forbidden: any request other than GET; flag changes; restarts

## Services
- notifications · ../notifications-svc · publishes `transfer.created` (src/events.ts:22) · `service:notifications`

## Gotchas
- Transfers need a seeded account: `npm run seed` (package.json:11), or POST /transfers returns 404.
```
