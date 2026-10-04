# Stages

Six stages, fixed order. Each ends with one result — `passed`, `failed`, or
`not run — <reason>` — and its findings. Print one line when each finishes.

Before any stage, check that it can run at all. A stage's command or agent that is not
available in this session is `not run — not installed`; the chain goes on. A stage
skipped with `--skip` is `not run — skipped by flag`.

## a — Lint · type-check · tests

Three results, one each. Find the scripts first; run nothing until they are known.

1. **Package manager** — the `packageManager` field in the root `package.json`, else the
   lockfile (`pnpm-lock.yaml` → pnpm).
2. **Scripts** — the `scripts` in the root `package.json`, and the tasks in `turbo.json`
   (`tasks`, or `pipeline` in older configs). If the root has no script for a check,
   look in the `package.json` of the workspaces the branch changed.
3. **Match by what the script does, not by a name you expect.** A type-check script may
   be called `typecheck`, `type-check`, `check-types`, `tsc`, or something else — read
   its command. If two scripts could be the one, ask.

| Check | Run | When no script exists |
| --- | --- | --- |
| Lint | The lint script, as defined | `not run — no script defined` |
| Type-check | The type-check script, as defined | `not run — no script defined` |
| Tests | The test script, as defined, in its run-once form | `not run — no script defined` |

- Run each script through the package manager exactly as the project defines it. Do not
  add `--fix`, do not run `tsc` or a linter directly as a substitute, do not narrow the
  run with filters the project does not use.
- A test script that starts a watcher must be run in the form the project uses in CI.
  If that cannot be told from the config, ask.
- Exit code non-zero → `failed`. Keep the first errors of the output (file, line,
  message) for the report; do not paste the whole log.
- All three run even if the first fails, so one run shows everything that is broken.
  Then the chain stops.

A missing script is listed under "still to verify" — it is not a pass.

## b — Design QA

Find the components the branch changed:

```bash
git diff --name-only origin/<base>...HEAD -- packages/ui-web/src/components/
```

Take the distinct `<Name>` folders that still exist. None → `not run — nothing to
check`.

For each one, spawn the agent (Agent tool, `subagent_type: agentic-dev-kit:design-qa`)
with the component name, one after another. Keep each report.

| Agent result | Stage record for that component |
| --- | --- |
| Verdict PASS | passed, with any 🟡 issues |
| Verdict FAIL (any 🔴) | failed — stops the chain after every component has been checked |
| "no Figma source recorded" | `not run — no Figma source recorded` |
| No Figma tool available, or Figma not authorised | `not run — no Figma access` |

The stage is `failed` if any component failed, `passed` if at least one passed and none
failed, `not run` if none could be checked. Design QA findings have no IDs of their own;
refer to them as `<ComponentName> · check <1–5>`.

## c — PR check

Run `/agentic-dev-kit:pr-check`. It spawns the read-only `agentic-dev-kit:pr-checker`
agent, which reviews the branch's changes against the spec and the `CLAUDE.md` rules and
returns findings `P-nn` (🔴 Blocker / 🟡 Risk / ⚪ Note) and a verdict.

- Show the report in full.
- Let the command make its fix-offer. The developer chooses which findings to fix.
- Fixes made → see "After a fix" below.
- 🔴 still open after the developer has chosen → `failed`; the chain stops.
- No 🔴 open → `passed`. Unfixed 🟡 and ⚪ stay in the report.

## d — Security check

Run `/agentic-dev-kit:security-check`. It spawns the read-only
`agentic-dev-kit:security-checker` agent, which reviews the branch's changes using the
spec's sensitive-data section and returns findings `X-nn` with the same severities and a
verdict. Handle the report, the fix-offer, and the result exactly as in stage c.

## e — Flow test

This stage needs the web app running and the developer's Chrome. **Ask before running
it**, naming the acceptance criteria it would check:

> Flow test would check AC-nn, AC-nn on the running web app in your browser. Run it now?

| Situation | Result |
| --- | --- |
| Developer says no | `not run — declined` |
| No browser tools in the session | `not run — no browser tools in this session` |
| No acceptance criterion covers this piece of work | `not run — nothing to check` |
| No spec, but an issue | Offer `/agentic-dev-kit:flow-test #<issue>`, which takes its checks from the issue's stated expected behaviour (`T-nn`). Treat a failed `T-nn` like a failed `AC-nn`. |
| No spec and no issue | `not run — no spec or issue` |
| The piece is a `native port` | `not run — nothing to check` (the flow test drives the web app) |

Otherwise run `/agentic-dev-kit:flow-test` for those criteria. It asks for the app URL
and the sign-in itself, every run — do not answer for the developer and do not reuse
values from an earlier run. Do not start the app on the developer's behalf unless they
ask.

It returns Pass / Fail / Could not test per `AC-nn`.

- Any Fail → `failed`; the chain stops.
- No Fail → `passed`. Every Could not test goes on the "still to verify" list with its
  reason.
- Every criterion Could not test → `not run — <the reason the flow test gave>`.

If it offers to save test files, that is the developer's call. Saved files are
uncommitted changes — ask before committing them.

## f — Docs sync

Run `/agentic-dev-kit:docs-sync`. It proposes edits to the spec's build status, the
feature doc, the changelog, and README / `CLAUDE.md`, shows them, and asks before
writing.

| Outcome | Result |
| --- | --- |
| Edits written, or nothing needed updating | `passed` |
| Developer declined all edits | `not run — declined` |
| It could not finish | `not run — <reason>` |

This stage never stops the chain. When it wrote files, show them, propose a commit
message in the repo's style (`docs(<scope>): …`), and commit on a yes. If the developer
says no, the PR goes up without them and the report says so.

## After a fix

A fix changes the code every earlier stage looked at, so earlier results are stale.

1. Show the changed files and ask before committing them.
2. Rerun, in order, from stage a up to and including the stage that offered the fix:
   - lint · type-check · tests — always;
   - design QA — only for components whose files the fix touched;
   - PR check and security check — whichever have already run.
3. Use the rerun's results in the report; say `rerun after fix` beside the stage.
4. A rerun that still has a 🔴, and the developer does not want another fix → `failed`;
   the chain stops.

Do not loop on your own. Each further round of fixes is the developer's decision.

## When the chain stops

1. Mark every stage after the stop `not run — chain stopped at <stage>`.
2. Print the combined report as far as the run got (`combined-report.md`).
3. Say what stopped it — the failing script and its first errors, or the 🔴 finding IDs,
   or the failed `AC-nn`.
4. Say the next step: fix it, then run `/agentic-dev-kit:ship` again. Every stage runs
   again from the start.
5. Do not push and do not open a PR.
