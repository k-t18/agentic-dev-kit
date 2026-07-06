# Phases 5, 7, 8 — Reporting templates

## Phase 5 — Extraction summary (before file generation)

Produce this and confirm before writing files. Proceed automatically if confidence is
high; pause only for ambiguities that would materially break the system.

```md
## Design System Extraction Summary

### Environment
- Web app: Next.js + React + Tailwind CSS
- Native app: React Native CLI (bare, no Expo)
- Monorepo: Turborepo + pnpm
- Token source of truth: /packages/core/src/tokens/index.ts
- Tailwind relationship: extends from tokens (correct) / needs fixing
- Metro watchFolders: correct / needs fixing

### Foundation Tokens
#### Colors
| Token | Value | Confidence |
| ----- | ----- | ---------- |
#### Typography
| Role | Family | Size | Weight | Line Height | Letter Spacing | Confidence |
| ---- | ------ | ---- | ------ | ----------- | -------------- | ---------- |
#### Spacing
| Token | px Value | Confidence |
| ----- | -------- | ---------- |
#### Border Radius
| Token | Value | Confidence |
| ----- | ----- | ---------- |
#### Shadows
| Token | Web value | Native value | Confidence |
| ----- | --------- | ------------ | ---------- |

### Semantic Tokens
| Token | Maps to | Confidence |
| ----- | ------- | ---------- |

### Component Tokens (if present in Figma)
| Token | Value / Mapping | Confidence |
| ----- | --------------- | ---------- |

### Platform Split Required
- Shadows: yes/no
- Fonts: list font files needed in /apps/native/assets/fonts/
- Percentage values flagged: yes/no

### Missing / Ambiguous
- Tokens expected but not found in Figma
- Conflicts in Figma naming

### Drift Found
- Hardcoded values found during Phase 1 audit
```

---

## Phase 7 — Accessibility check

Before finalizing, verify (report failures explicitly — never silently pass):

- [ ] Primary text on canvas meets WCAG AA contrast (4.5:1 minimum)
- [ ] Secondary text on canvas meets WCAG AA large text (3:1 minimum)
- [ ] Error state is distinguishable without relying on color alone
- [ ] Focus/ring token is visible against all surface colors
- [ ] Disabled state is clearly distinct from default
- [ ] Success and error states differ in luminance

---

## Phase 8 — Output summary (after generation)

```md
## Design System Setup — Output Summary

### Files Generated
- [ ] /packages/core/src/tokens/index.ts
- [ ] /packages/core/src/tokens/rn-styles.ts
- [ ] /apps/web/tailwind.config.ts (imports from tokens)
- [ ] Web font loading (next/font or fonts.css)
- [ ] Native font setup instructions

### Token Counts
- Foundation color tokens: N
- Semantic tokens: N
- Component tokens (if any): N
- Confirmed / Inferred / Assumed / Missing: N / N / N / N

### Platform Split Applied
- Shadows: web (CSS box-shadow) + native (elevation + shadow* props)
- Fonts: web (next/font or @font-face) + native (react-native-asset linking required)
- % widths flagged and removed from rn-styles.ts: yes/no

### Drift Remediation
- Hardcoded values found in Phase 1: N — files affected: list
- Action: replace with token imports (manual or next PR)

### Tailwind Relationship
- Status: correctly imports from @repo/core/tokens / fixed / still incorrect

### Metro Config
- Status: watchFolders correct / fixed / still missing

### Missing Tokens (requires Figma input)
- List any marked Missing

### Accessibility Issues Found
- List any contrast/state visibility issues

### Next Steps
- [ ] Verify font PostScript names match fontFamily values in rn-styles.ts
- [ ] Run pnpm dev from root — confirm web resolves tokens
- [ ] Run npx react-native run-ios — confirm native resolves @repo/core/tokens
- [ ] QA: spot-check 3 web components (Tailwind token classes) + 3 native (rnTokens)
- [ ] Commit and open PR to develop branch
```
