---
name: flow-test
description: Test the web app in the developer's own Chrome against the acceptance criteria in a functional spec — or, when there is no spec, against the expected behaviour stated in a GitHub issue. Runs the flow-test skill. Prints a pass/fail report per criterion and saves rerunnable end-to-end tests. Never edits application code.
argument-hint: "[flow name | path to spec.md] [optional subset: AC-01 AC-04 | AC-01-AC-06 | W-03]  |  [#issue | issue URL | --checks]"
---

# /flow-test

**Arguments:** $ARGUMENTS

Runs the **`flow-test`** skill in this conversation — not as a subagent, because it has
to ask the developer questions and drive the browser while they watch. It tests the
**web** app only, through the Claude in Chrome extension. It reports failures; it never
fixes them, and never edits application code.

## What to do

1. **Check the browser tools first.** The skill needs the Claude in Chrome tools
   (`mcp__claude-in-chrome__*`). If the session does not expose them, say so, say what
   the developer has to do (install or connect the extension, then run the command
   again), and **stop**. Do not test by reading the code instead.

2. **Parse `$ARGUMENTS`:**
   - A path to a `spec.md` → test that spec.
   - A flow name → test `docs/specs/<flow-name>/spec.md`.
   - Anything after the flow name or path is the **subset**: one or more `AC-` IDs, an
     `AC-` range, or one `W-` ID (every criterion whose `Covers` overlaps that work
     item's `Covers`). No subset → every criterion in section 13.
   - `#<n>`, `issue <n>`, or an issue URL → **no-spec mode**: the checks come from that
     GitHub issue. `--checks` → no-spec mode with checks the developer states. Follow
     `references/without-a-spec.md`, then continue from step 3.
   - Empty → list the specs under `docs/specs/` with their status and ask which one —
     or whether to test against an issue instead.
   - If a spec file does not exist, or it has no section 13 rows, say so and offer
     no-spec mode. Do not test from a guess — stated criteria or stated checks are the
     test.

3. **Ask the setup questions** per `flow-test` → `references/run-setup.md`: the URL the
   web app is running at, what it is pointed at (a real test backend or mocked APIs),
   and how sign-in works. Ask every run; do not reuse answers from an earlier run. The
   developer signs in themselves in Chrome.

4. **Build the test plan and show it** per `references/test-plan.md` — which criteria
   will run, which are skipped and why. Wait for the developer to confirm or change it
   before touching the app.

5. **Run each criterion** per `references/executing-criteria.md`, collecting evidence
   per `references/evidence.md`. Stop and ask before any action that is hard to undo or
   leaves the app.

6. **Print the report** in the terminal, in the format under `flow-test` → Output
   Templates.

7. **Save the tests** per `references/saved-tests.md`. If the project has no end-to-end
   runner, tell the developer and ask which one they want before adding anything.

8. **Print the closing summary** — the counts, the test files written, and what a human
   still has to test.

## Notes

- The report goes to the terminal. Post it as a GitHub PR comment **only** when the
  developer asks, and only after showing them what will be posted.
- A failure is reported with reproduction steps and evidence. It is not fixed here.
- Behaviour the spec does not describe is listed as a question for the spec — the
  developer raises it with the PM through `/agentic-dev-kit:spec-builder`. It is never
  judged pass or fail.
- Native is not tested. The report says so and lists what is left for a person.
- Without a spec, a pass covers only the checks taken from the issue. The report says the
  rest of the flow was not tested.
