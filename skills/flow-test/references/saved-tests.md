# Saved tests

After the report, the checks are written down as end-to-end tests so they can be rerun
without the agent. Test files are the only files this skill creates.

## 1. Find the project's end-to-end setup

Look, in this order, in `apps/web/` and at the repo root:

| Evidence | Where |
| --- | --- |
| A runner config file | `playwright.config.*`, `cypress.config.*`, `wdio.conf.*`, or another runner's config |
| A runner in `devDependencies` | `apps/web/package.json`, root `package.json`, a dedicated end-to-end workspace package |
| An end-to-end script | `e2e`, `test:e2e`, or similar in `package.json`; a matching task in `turbo.json` |
| An existing test folder | `e2e/`, `tests/e2e/`, `cypress/`, or wherever the config's test directory points |
| A CI job that runs them | `.github/workflows/` |

A unit-test runner alone (component or hook tests) is **not** an end-to-end setup.

## 2a. A setup exists — follow it

Read two or three existing tests before writing. Match:

- the folder and the file-name pattern;
- the language and the import style;
- how tests get a signed-in session (a setup step, stored session state, a fixture, a
  custom command) — reuse it, do not write a second way;
- how the base URL is supplied — never hard-code the URL from this run;
- helpers, page objects, and fixtures already there;
- how elements are selected.

Then write **one file for the flow**, named after the flow in the project's pattern,
with **one test per criterion**.

## 2b. No setup exists — ask first

Do not install anything. Say:

```
This project has no end-to-end test runner, so the checks cannot be saved yet.
Which runner do you want? (for example Playwright or Cypress — or name another)
I will not add anything until you choose. You can also skip saved tests for now.
```

- The developer names a runner → say exactly what will be added (the dependency, the
  config file, the script, the test folder, and which `package.json`), wait for a yes,
  then add the minimum and nothing else. Use pnpm, in the workspace the developer
  chooses. No CI changes unless asked.
- The developer skips → write no tests, and say so in the report's "Saved tests" line.

Do not pick a runner on the developer's behalf, and do not argue for one.

## 3. What each test looks like

- **Title starts with the criterion's ID**, then the criterion: `AC-04: given <…>, when
  <…>, then <…>`. One test per `AC-` ID; one `AC-` ID per test. The ID in the title is
  how a failing test is traced back to the spec.
- **Group** the tests under the flow name, with the spec path in a comment at the top of
  the file.
- **Body follows the criterion:** the steps that reach the Given, the single When, and
  assertions for the Then — the same steps and the same observations that were used in
  the browser run.
- **Assert what the spec states:** the message text from section 10, the screen from
  section 4, the route. Where the run also checked a request, assert the request and its
  status the way the runner supports.
- **Select elements the way a user finds them** — by role and accessible name, label, or
  visible text — unless the project's tests use another convention. If a control has no
  stable way to be selected, do **not** add a test id to the application code; write the
  best selector available and list the control in the summary as needing one.
- **Each test stands on its own.** It sets up its own Given and does not depend on
  another test having run first.

## 4. Which criteria get a test

| Result in the run | Saved test |
| --- | --- |
| `Pass` | Written. |
| `Fail` | Written to assert what the **spec** says. It will fail until the app is fixed — that is the point. Ask the developer whether to leave it failing or mark it with the runner's expected-failure marker, with the `AC-` ID and the date in the reason. Never rewrite the assertion to match the wrong behaviour. |
| `Could not test` / `Skipped` | Written as a skipped placeholder using the runner's skip marker, with the reason — so the gap shows up in the test output. Leave it out if the developer prefers. |

A criterion whose When is a hard-to-undo action against a real backend (see "Stop and
ask" in `executing-criteria.md`) gets a test only if the project already has a safe way
to run it — a mocked route or a dedicated test environment. Otherwise it is a skipped
placeholder with that reason.

## 5. No credentials, no personal data

- No password, one-time code, token, API key, or session cookie in a test file, a
  fixture, or a committed session-state file.
- Sign-in comes from the project's existing mechanism. If there is none, read the values
  from environment variables **by name** and tell the developer which names to set; add
  the names (not values) to `.env.example` only if the developer agrees, since that is
  an existing file.
- If the runner stores a signed-in session in a file, that file must be git-ignored.
  Check; if it is not, tell the developer rather than committing it.
- Test data is invented placeholder data, never values copied from the page during the
  run.

## 6. After writing

- Do not run the new tests unless the developer asks. If they ask, use the project's own
  end-to-end script, and report the result separately from the browser run.
- If the tests were not run, say so: they were written from the browser run and have not
  been executed.
- Do not commit. List the files written and leave the commit to the developer.

## 7. Rerunning later

When a test file for the flow already exists:

- Read it first. Update the test whose title carries the same `AC-` ID; add tests for
  criteria that have none; do not duplicate.
- A criterion marked `removed` in the spec: say its test is now stale and ask before
  deleting it.
- Do not touch tests that do not carry an `AC-` ID from this spec.

## Summary to print

```
Saved tests
  <path to test file> — 8 tests
    written:  AC-01, AC-02, AC-03, AC-05, AC-07
    failing by design (app does not match spec): AC-04
    skipped placeholders: AC-08 (API-03 pending), AC-10 (native)
  Runner: <name>, existing setup · run with: <the project's script>
  Not executed — written from the browser run.
  Needs a stable selector: <control> on S-03
```
