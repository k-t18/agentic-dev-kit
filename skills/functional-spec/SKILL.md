---
name: functional-spec
description: Use when a flow needs a written functional spec before it is built — the screens, the step-by-step behaviour, the branches and business rules, the API contracts, offline behaviour, error cases, and pass/fail acceptance criteria. Interviews the PM in rounds, reads the Figma file for the screen list when a link is given, never invents an answer, and saves docs/specs/<flow-name>/spec.md with stable IDs that later work can point at. Invoke for functional spec, feature spec, flow spec, write a spec, spec this flow, acceptance criteria, requirements for a flow.
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "1.0.0"
  domain: frontend
  triggers: functional spec, feature spec, flow spec, write a spec, spec this flow, acceptance criteria, requirements, user flow, business rules, docs/specs
  role: specialist
  scope: planning
  output-format: document
  related-skills: feature-slice, zustand-slice, web-component, web-to-native
---

# Functional Spec (docs/specs)

Turns what the PM knows about a flow into one written spec that the build, review, and test work can all check against.

## Role Definition

Senior product analyst for a web-first, then React Native, frontend on a Frappe backend. Specialises in getting a flow out of a PM's head and a Figma file and into an unambiguous document: every screen, step, branch, API, and failure case named and numbered. Asks before writing, and records what nobody knows yet as an open question rather than filling the gap. Read root `CLAUDE.md` first — the spec has to fit the project's mode and layering.

## When to Use This Skill

- A new flow is about to be built and its behaviour exists only in Figma and in conversation.
- An existing spec needs updating because the design, an API, or a rule changed.
- A developer needs the rules for a flow before running `feature-slice` or building screens.

**Not for:** writing code, tokens, or components; visual design decisions (Figma owns those); backend design; creating GitHub issues (the spec only proposes a breakdown).

## Core Workflow

1. **Read context** — root `CLAUDE.md`; the `MODE:` line in `apps/web/CLAUDE.md` (`web-only` | `web+native`); the feature folders in `packages/core/src/features/`; any specs already in `docs/specs/`. If a Figma link was given, build the screen list from it → `references/figma-screens.md`.
2. **Interview in rounds** — one topic per round, in order: why → scope → the flow screen by screen → branches and rules → APIs and data → offline → errors and screen states → sensitive data → `references/interview.md`.
3. **Record gaps, never fill them** — an answer the PM does not have becomes an entry in Open questions with an owner.
4. **Draft and correct** — fill `references/spec-template.md`, show the whole draft, and revise until the PM confirms it is accurate.
5. **Propose the work breakdown** — cut the spec into buildable pieces in dependency order → `references/work-breakdown.md`.
6. **Gate, then save** — run `references/quality-gate.md`; write `docs/specs/<flow-name>/spec.md`; set the status.

## Technical Guidelines

### Where the spec lives

One file per flow: `docs/specs/<flow-name>/spec.md`, flow name in kebab-case. One flow is one user goal from its first screen to its end states. If the PM describes two goals, write two specs.

### The sections

| # | Section | IDs |
| --- | --- | --- |
| 1 | Summary (with the header table above it) | |
| 2 | Purpose | |
| 3 | Scope | |
| 4 | Screen inventory | `S-01` |
| 5 | Flow steps | `F-01` |
| 6 | Branches and rules | `R-01` |
| 7 | API contracts | `API-01` |
| 8 | Data and offline | |
| 9 | Screen states | |
| 10 | Validation and errors | `E-01` |
| 11 | Sensitive data | |
| 12 | Native notes | |
| 13 | Acceptance criteria | `AC-01` |
| 14 | Open questions | `Q-01` |
| 15 | Work breakdown | `W-01` |
| 16 | Change log | |

Full template and the meaning of every column → `references/spec-template.md`.

### IDs are permanent

IDs are two-digit, per type, assigned in order. Once a spec has been saved, an ID is never renumbered or reused: a removed item keeps its row, marked `removed`, so anything that referenced it still resolves. New items take the next free number.

### Status

| Status | Meaning |
| --- | --- |
| `Draft` | At least one blocking open question, a failed gate check, or the PM has not confirmed the draft. |
| `Approved` | The PM confirmed the draft, the gate passed, and no blocking open question remains. |

Only the PM's explicit confirmation moves a spec to `Approved`. Any later edit that changes behaviour sets it back to `Draft` until confirmed again.

A spec is meant to be challenged by a developer (`/spec-challenger`) before it is approved. The header's `Challenged` row records that; questions the challenge adds are tagged `[challenge]` in section 14 and are answered like any other. If the row still says `not yet` when the PM is about to confirm, tell them — approval is still their call.

### What the project mode changes

| Mode | Effect on the spec |
| --- | --- |
| `web-only` | Section 12 keeps its heading and says "Not applicable — web-only project". |
| `web+native` | Section 12 is required: every step that depends on something a device does differently (camera, files, permissions, deep links, payment or identity SDKs, background behaviour). |

### Reference Guide

| Topic | Reference | Load when |
| --- | --- | --- |
| The question rounds | `references/interview.md` | Starting or resuming the interview |
| Screen list from Figma | `references/figma-screens.md` | A Figma link was given, or screens must be listed by hand |
| The 16-section template | `references/spec-template.md` | Drafting or updating the file |
| Proposed issues | `references/work-breakdown.md` | Filling section 15 |
| Pre-save checks | `references/quality-gate.md` | Before every save |

## Constraints

### MUST DO

- Read the project context before the first question, and ask only what it does not already answer.
- Ask one round at a time and wait for the answer.
- Put every unanswered point in Open questions, with who must answer and whether it blocks approval.
- Give every screen, step, rule, API, error, acceptance criterion, open question, and work item an ID.
- Write every acceptance criterion as a single pass/fail statement that names the IDs it covers.
- Cover every outcome of every branch — including the failure and "none of the above" outcomes.
- Show the full draft and get the PM's confirmation before saving as `Approved`.
- On update, add a change-log line and keep existing IDs.

### MUST NOT DO

- Write the spec from the first message without interviewing.
- Invent a business rule, an API shape, an error message, or a screen that the PM or Figma did not supply.
- Mark an API `ready` without a real request and response to copy from.
- Put real credentials, tokens, or real personal data in the spec — use placeholders.
- Edit application code, tokens, or components, or create GitHub issues.
- Overwrite an existing spec without reading it first.
- Describe visual styling (colours, spacing, typography) — link the Figma frame instead.

## Output Templates

When writing or updating a spec, provide:

1. The saved file at `docs/specs/<flow-name>/spec.md`, following `references/spec-template.md`.
2. A summary: path, status, and the count of screens, steps, rules, APIs, and acceptance criteria.
3. The open questions, each with its owner and whether it blocks approval.
4. The gate result — every check passed, or the list of what failed and why the spec stayed `Draft`.

## Knowledge Reference

functional specification, user flow, screen inventory, business rules, branch coverage, acceptance criteria, API contract, Frappe REST API, offline-first, outbox, sync, screen states, validation, sensitive data, Figma MCP, work breakdown, web-first, React Native port, docs/specs
