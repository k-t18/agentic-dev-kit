# Mobile → Tablet

Makes the phone-first native app behave correctly on large and **resizable** screens.

This is the sibling of the migration workflow above: that workflow moves a component
across *platforms*, this reference moves it across *screen sizes*. Both leave
`packages/core` alone; both keep the props contract intact.

## The one rule

> **Tablet support is a reactivity problem before it is a design problem.**
> A layout that reads width through `useWindowDimensions()` already works on a
> tablet. A layout that reads it through `Dimensions.get()` is broken on *every*
> resize — rotation, Android multi-window, iPad Split View — regardless of how it
> looks.

Audit reactivity first. Only once every sizing read is reactive is it worth
discussing columns, sidebars, and master-detail.

**Corollary:** never open a "tablet redesign" PR before the reactivity defects are
fixed and merged. They are different sizes of change with different reviewers, and
bundling them hides one-line bug fixes inside a layout diff.

## Step 1 — Audit reactivity (always first)

```bash
grep -rn "Dimensions.get\|useWindowDimensions\|PixelRatio" packages/ui-native/src apps/native/src
```

Classify every hit:

| Pattern | Verdict |
|---|---|
| `useWindowDimensions()` | ✅ reactive — re-renders on rotation *and* live resize |
| `Dimensions.get('window')` inside a component | ❌ **defect** — never subscribes; value goes stale until an unrelated re-render |
| `Dimensions.get('window')` at module scope | ❌❌ **worse** — frozen at import, wrong even on first mount in split-view |
| `Dimensions.addEventListener` | ⚠️ works, but `useWindowDimensions` is the idiom — prefer it |

Every `Dimensions.get` hit is a one-line fix to `useWindowDimensions()`. There is no
case where `Dimensions.get` is correct.

Then sweep for layout that *can't* adapt — fixed widths, uncapped `flex: 1` bars, `%`
widths, one-time reads, font scaling.

## Step 2 — Confirm platform config allows large screens

Read, don't assume. Verify rather than edit by default.

**Android** — `apps/native/android/app/src/main/AndroidManifest.xml`:

- `android:configChanges` must include `orientation|screenLayout|screenSize|smallestScreenSize`,
  or rotation destroys and recreates the Activity (losing JS state).
- `android:resizeableActivity` absent ⇒ defaults `true` at `minSdkVersion ≥ 24` ⇒
  multi-window already enabled. Setting it `false` is almost always wrong.
- `targetSdkVersion ≥ 36`: **Android 16 ignores orientation and resizability
  restrictions on displays ≥600dp.** Large-screen adaptation is compulsory, not a
  choice. Do not plan around opting out.
- `values-sw600dp` resource folders are **not** how RN adapts layout — RN drives
  layout from JS. Use them only for native-side resources.

**iOS** — `ios/<App>/Info.plist` and `project.pbxproj`:

- `TARGETED_DEVICE_FAMILY = "1,2"` ⇒ iPad is already a target. Set `"1"` only if you
  are deliberately dropping iPad — that also removes the Split View obligation and
  the `UIActivityViewController` popover-anchor requirement.
- **Absent `UIRequiresFullScreen` ⇒ Split View and Slide Over are enabled.** This is
  the strictest constraint anywhere in the app: the window can be any width, down to
  a ~320pt Slide Over pane, and it changes *continuously* while the user drags the
  divider. A layout that only handles "phone or tablet" fails here.

## Step 3 — Fix the defects (Tier 1)

Small, mechanical, independently reviewable:

1. Swap every `Dimensions.get` → `useWindowDimensions()`.
2. Cap full-bleed chrome. A `flexDirection: 'row'` bar with `flex: 1` children looks
   correct on a phone and absurd at 1366pt. Add a `maxWidth` and center it.
3. Cap or center any container that is implicitly full-width and holds short content.

**Structural caps are not spacing tokens.** A hardcoded `maxWidth` that mirrors a web
breakpoint (e.g. `max-w-md`) is allowed and must carry a comment naming what it
mirrors — this is the documented exception to the no-hardcoded-values rule; do not
invent a token for it.

## Step 4 — Decide the orientation policy explicitly

Phones and tablets usually want different answers, and the two platforms default
differently — so this is always a decision, never a default.

Typical target: **phones portrait-locked, tablets free.** iOS expresses this natively
via `UISupportedInterfaceOrientations` + `UISupportedInterfaceOrientations~ipad`, so
`Info.plist` usually needs no change.

**Android cannot express it in the manifest.** `android:screenOrientation` is an enum
attribute and **cannot take a resource reference** — a `@bool/...` value there does
not work. The working approach is a bool resource read at runtime:

1. `res/values/bools.xml` → `<bool name="portrait_only">true</bool>`
2. `res/values-sw600dp/bools.xml` → `<bool name="portrait_only">false</bool>`
3. `MainActivity.kt` → add `onCreate` setting `requestedOrientation` to
   `SCREEN_ORIENTATION_PORTRAIT` or `SCREEN_ORIENTATION_UNSPECIFIED` from that bool.

Under the API 36 rule this lock is ignored on large screens anyway — which is the
correct outcome, and is why the `sw600dp` split is belt-and-braces rather than
load-bearing.

> ⚠️ Check for a **pre-existing** divergence before changing anything: an app can
> easily be portrait-locked on iOS and freely rotating on Android, meaning Android
> phones already ship a landscape layout nobody designed. Surface that; don't silently
> "fix" it in a tablet PR.

## Step 5 — Breakpoints: only centralize when it earns it

RN has no media queries. Breakpoints are plain numbers compared against
`useWindowDimensions().width`, mirroring web's Tailwind stops:

```ts
// Web's grid-cols breakpoints (2 → 3 → 4 at Tailwind's md/xl) re-expressed via
// useWindowDimensions since RN has no media queries.
function columnsForWidth(width: number): number {
  if (width >= 1280) return 4;
  if (width >= 768) return 3;
  return 2;
}
```

Keep the literal **local to the component** while there are ≤3 call sites. Extracting a
shared token earlier is premature and drags `packages/core` into a UI-only concern.

Past ~3 sites, add a `breakpoints` export to `packages/core/src/tokens/rn-styles.ts`
(the **native-derived** file) and a `useBreakpoint()` hook. Never add breakpoints to
`packages/core/src/tokens/index.ts` — that file is generated from Figma and is the
web/Tailwind source of truth.

`FlatList` cannot change `numColumns` on a live instance — pass `key={columns}` to
force the remount.

## Step 6 — Decide the redesign separately (Tier 2)

Only after Tier 1 ships. Three honest bars:

| Bar | Meaning | Cost |
|---|---|---|
| **A. Runs on a tablet** | Phone layout scaled up. Nothing clipped, rotation and resize correct. | Tier 1 only |
| **B. Designed for tablet** | Persistent side panel instead of full-screen drawers, capped content width, real column counts. | Moderate |
| **C. Tablet-first** | Different navigator on large screens — master-detail, persistent sidebar. | Second app shell |

**Before proposing B or C, search for a shell that already exists.** Components ported
from web (a side navigation, a docked drawer variant) often survive in the
`ui-native` barrel while no screen imports them — a wide layout may be a rewiring job,
not a build. Equally, check git history and issue references in screen-header comments:
a responsive shell that was **deliberately removed** is a product decision to re-open
with the team, not a regression to quietly restore.

## Verification matrix

Layout bugs hide in the *transition*, not the static size. Never verify by launching
straight into one geometry.

| Case | How | Watching for |
|---|---|---|
| Tablet portrait + landscape | Pixel Tablet AVD, or `adb shell wm size 2560x1600` | column counts, wide-split sections |
| **Rotation** | rotate with content on screen | tables/grids re-laying out **immediately** — this is the `Dimensions.get` regression test |
| **Android split-screen** | multi-window, then **drag the divider** | continuous tracking, not just on release |
| **iPad Split View / Slide Over** | drag from ~320pt to full width | continuous re-layout; the case with no Android equivalent |
| Phone regression | phone AVD | unchanged behavior |
| Overlays at each size | drawer, bottom sheet, modal | drawer capped at its size, modal at its `maxWidth`, sheet content-sized |
| iPad share sheet | print/share flow | `UIActivityViewController` needs a popover anchor or it **crashes** on iPad |

`tsc` and lint prove nothing here. Every item above is a runtime check.

## Never

- ❌ Use `Dimensions.get()` for layout — it does not subscribe
- ❌ Set `android:resizeableActivity="false"` or plan around opting out of large screens at `targetSdk ≥ 36`
- ❌ Put `@bool/...` in `android:screenOrientation` — the attribute cannot take a resource
- ❌ Add breakpoints to `packages/core/src/tokens/index.ts` (Figma-generated)
- ❌ Assume "tablet" means one width — Split View and multi-window make it continuous
- ❌ Bundle Tier 1 reactivity fixes into a Tier 2 redesign PR
- ❌ Change `numColumns` without a `key` remount on `FlatList`
- ❌ Restore a deliberately-removed responsive shell without re-opening the decision
