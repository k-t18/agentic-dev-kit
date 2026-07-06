---
name: design-system-setup
description: Use when setting up or maintaining the design-token system for a Turborepo monorepo with a Next.js web app and a bare React Native CLI app. Extracts tokens from Figma via MCP, establishes the tokens/index.ts (master) + tokens/rn-styles.ts (native-derived) dual-output pipeline in packages/core, wires Tailwind on web (as a consumer), configures StyleSheet constants for native, and audits token drift. Invoke for initial design-system setup, Figma token extraction, adding a token category, syncing after Figma updates, or debugging Tailwind/Metro token resolution. tokens/index.ts is always the single source of truth — never tailwind.config.
license: MIT
metadata:
  author: Frontend
  version: "1.0.0"
  domain: frontend
  triggers: design system, design tokens, Figma tokens, tokens.ts, rn-styles.ts, token extraction, token drift, Tailwind tokens, StyleSheet tokens, design system setup
  role: specialist
  scope: implementation
  output-format: code
  related-skills: web-component, rn-component, web-to-native
---

# Design System Setup

Establishes a monorepo-aware design system: a **single source of truth in
`@repo/core`** that feeds two different styling systems — Tailwind on web (Next.js),
`StyleSheet` on native (bare RN CLI) — with no duplication, drift, or platform bleed.

Read root `CLAUDE.md` first; this skill never overrides it.

## Architecture (read first)

```
Figma (source of truth)
      │  [Figma MCP extraction]
      ▼
/packages/core/src/tokens/
   ├── index.ts       ← Master token file. Web + native both import from here.
   └── rn-styles.ts   ← StyleSheet-ready constants DERIVED from index.ts.
                        Numeric values only. No CSS shorthand. No % widths.
      │
      ├─▶ /apps/web/tailwind.config.ts   (Tailwind extends tokens — consumer, never source)
      └─▶ /apps/native components         (StyleSheet.create via rn-styles.ts)
```

**Non-negotiable rules this skill enforces:**

- `tokens/index.ts` is always the single source of truth.
- `tailwind.config.ts` imports tokens — it never defines them.
- `rn-styles.ts` is always derived from `index.ts` — never hand-edited.
- No platform-specific code ever enters `@repo/core`.
- Every extraction produces **both** `index.ts` and `rn-styles.ts` — paired output.

## Core principles

1. **`index.ts` is master; everything else is a consumer.** Tailwind, StyleSheet
   constants, and docs are all derived — never let a consumer become the source.
2. **Always dual output.** Every extraction emits both `index.ts` and `rn-styles.ts`.
   Producing only one is incomplete.
3. **Split platform-specific categories at extraction time** (shadows, fonts) — not
   during component work.
4. **Normalize messy Figma naming** into stable engineering names. Never promote
   accidental mockup values into global tokens.
5. **Separate certainty from inference.** Tag every token Confirmed / Inferred /
   Assumed / Missing. Never silently invent values.
6. **One system, two render targets** — feels native to both web and RN.

## When to use

- Setting up the design system from scratch, or extracting tokens from Figma via MCP
- Auditing token drift (hardcoded values in components)
- Adding a token category, or syncing after a Figma update
- Debugging Tailwind or Metro failing to resolve token values

**Not for:** individual components (`web-component` / `rn-component`), API hooks
(`feature-slice`), or web→native migration (`web-to-native`).

## Workflow (8 phases)

Each phase links to its reference — load only the one you're in.

1. **Audit** the repo before generating anything: monorepo structure, Tailwind
   relationship (consumer vs source), Metro `watchFolders`, existing token drift.
   → `references/audit.md`
2. **Extract from Figma** via MCP (prefer `get_variable_defs`): foundation, semantic,
   and (only if explicitly defined) component tokens; interaction states; theme modes.
   → `references/figma-extraction.md`
3. **Normalize** Figma naming into the 3-layer architecture (foundation → semantic →
   component) and tag confidence levels. → `references/normalization.md`
4. **Platform-split** categories that differ across web/native — shadows (`{ web,
native }`) and fonts; forbid `%` widths and CSS shorthand in native.
   → `references/platform-splitting.md`
5. **Summarize** the extraction (tokens + confidence + gaps) and confirm before writing.
   → `references/reporting.md`
6. **Generate files** in order: `index.ts` → `rn-styles.ts` → `tailwind.config.ts` →
   web fonts → native font docs → barrel. → `references/file-templates.md`
7. **Accessibility check** — WCAG AA contrast, state distinguishability.
   → `references/reporting.md`
8. **Output summary** — files generated, token counts, drift, next steps.
   → `references/reporting.md`

If extraction confidence is high, proceed through generation automatically; pause only
for ambiguities that would materially break the system.

## Reference guide

| Topic                                     | Reference                          | Load when      |
| ----------------------------------------- | ---------------------------------- | -------------- |
| Repo/Tailwind/Metro audit + drift scan    | `references/audit.md`              | Phase 1        |
| What to pull from Figma                   | `references/figma-extraction.md`   | Phase 2        |
| Naming, 3-layer architecture, confidence  | `references/normalization.md`      | Phase 3        |
| Shadow/font split, native forbidden forms | `references/platform-splitting.md` | Phase 4        |
| Generated file scaffolds                  | `references/file-templates.md`     | Phase 6        |
| Summary + a11y + output templates         | `references/reporting.md`          | Phases 5, 7, 8 |

## Pitfalls

- ❌ Defining token values in `tailwind.config.ts` — it consumes, never defines
- ❌ Producing only web tokens — always generate `rn-styles.ts` in the same pass
- ❌ CSS shorthand or `%` widths in `rn-styles.ts` — RN `StyleSheet` rejects them
- ❌ Using `.web` shadow values in native — use the `.native` object
- ❌ Promoting accidental Figma mockup values into global tokens
- ❌ Fabricating dark-mode tokens absent from Figma — mark as pending
- ❌ Hardcoding font names — pull from `typography.family`
- ❌ Removing `watchFolders` from `metro.config.js` — Metro silently breaks
- ❌ `any` in token files — use `as const` for full inference

## Outcome

A single token source in `@repo/core/src/tokens/index.ts`, a StyleSheet-safe derived
`rn-styles.ts`, a Tailwind config that imports tokens, web font loading, native font
instructions, zero hardcoded values in component packages, and a documented list of
missing/ambiguous tokens needing designer input.
