# iOS Build Readiness

Takes a native app that only ships on Android and gets iOS compiling, running, and
submittable — including from a Windows machine.

`native-setup` owns Metro and JS-side resolution. This reference owns the **native iOS
project**: CocoaPods, the Xcode project, `Info.plist`, assets, and signing.

## The one rule

> **Most iOS defects are visible from Windows. Only the build itself needs a Mac.**

An `ios/` folder can be extensively hand-edited and still have never compiled. Audit
and fix everything statically first, then buy Mac time once — with a short list of
verifications rather than a blind first attempt.

**The "has this ever built?" test** — run this before anything else:

```bash
ls apps/native/ios/Podfile.lock apps/native/ios/*.xcworkspace
```

Both absent ⇒ `pod install` has never run ⇒ **nothing in `ios/` has ever executed**,
no matter how polished it looks. Treat every iOS-only code path as unverified.

## Step 1 — Audit statically (all from Windows)

Read, don't assume. The high-signal files:

| File | What you're checking |
|---|---|
| `ios/Podfile` + `Podfile.lock` | lock exists? custom post-install? |
| `ios/<App>.xcodeproj/project.pbxproj` | bundle id, version, deployment target, `TARGETED_DEVICE_FAMILY`, signing, **Resources build phase** |
| `ios/<App>/Info.plist` | usage strings, ATS, `UIAppFonts`, orientations |
| `ios/<App>/PrivacyInfo.xcprivacy` | present **and actually in the Resources phase** |
| `ios/<App>/Images.xcassets/AppIcon.appiconset/` | any `.png` files, or just `Contents.json`? |
| `ios/<App>/LaunchScreen.storyboard` | still the RN template? |
| `react-native.config.js` | asset paths — see Step 3 |
| `.npmrc`, `package.json` | private registries, native modules needing pods |

A file being *referenced* in `project.pbxproj` does not mean it ships. It must appear
in `PBXResourcesBuildPhase`:

```bash
awk '/PBXResourcesBuildPhase/,/End PBXResourcesBuildPhase/' ios/*.xcodeproj/project.pbxproj
```

`PrivacyInfo.xcprivacy` missing from that list is an App Store rejection, and it is a
common oversight because the file *looks* correctly wired in the project navigator.

## Step 2 — Sweep Android→iOS parity

Android is the working reference. Every native capability it declares needs an iOS
counterpart, and the shapes differ:

| Android | iOS equivalent | Failure if missing |
|---|---|---|
| `<uses-permission>` + runtime `PermissionsAndroid` | `NS*UsageDescription` in `Info.plist` | **crash** on first API touch, or App Review rejection |
| `android:usesCleartextTraffic` | `NSAppTransportSecurity` | **silent** — every `http://` request fails on iOS only |
| `mipmap-*/ic_launcher*` | `AppIcon.appiconset` (opaque, no alpha, no rounded corners) | blank icon; upload rejected |
| theme-based splash | `LaunchScreen.storyboard` | ships the RN template text |
| `versionCode` / `versionName` | `CURRENT_PROJECT_VERSION` / `MARKETING_VERSION` | drift between stores |
| `applicationId` | `PRODUCT_BUNDLE_IDENTIFIER` | change **together** or they diverge |
| signing in `build.gradle` | `DEVELOPMENT_TEAM` + `ExportOptions.plist` | cannot archive |
| `<queries>` for `ACTION_VIEW` | usually nothing — the share sheet brokers it | — |
| `BackHandler` / `ToastAndroid` | **no equivalent** | inert stub — silent UX gap, not a crash |

> ⚠️ **ATS is the classic silent divergence.** Android often makes cleartext
> env-configurable while `Info.plist` hardcodes `NSAllowsArbitraryLoads = false`. A
> plain-HTTP backend then works on Android and fails every request on iOS with no
> obvious cause. Check the real `.env` before debugging anything else. If staging is
> genuinely HTTP, add a scoped `NSExceptionDomains` entry — never flip
> `NSAllowsArbitraryLoads`.

Also sweep for Android-only APIs with no iOS branch:

```bash
grep -rn "PermissionsAndroid\|BackHandler\|ToastAndroid\|Platform.OS === 'android'" apps/native/src packages/ui-native/src
```

An inert stub on iOS is not a crash — which makes it easy to miss. Classify each as
*correctly guarded*, *harmless no-op*, or *genuine UX gap needing an iOS affordance*.

## Step 3 — Fix assets with the tool, never by hand

**Do not hand-edit `project.pbxproj` to add or remove fonts and assets.** Use
`react-native-asset` — it is pure Node (`xcode` + `plist`), so **it runs on Windows**,
and its clean step removes stale entries via `link-assets-manifest.json` before
relinking.

```bash
npx react-native-asset
```

Watch for these in `react-native.config.js`:

- **Paths into `node_modules/.pnpm/<pkg>@<version>/…`** — the version is baked into
  every `PBXFileReference`. The first minor bump breaks the build with *"Build input
  file cannot be found"*, and the path assumes one pnpm store layout. Copy the fonts
  you actually use into the app's own `assets/fonts/` instead.
- **Globbing a whole vendor font directory** — ships every icon set in the IPA. Check
  which are actually imported (`grep -rhoE "react-native-vector-icons/[A-Za-z0-9_]+"`)
  and vendor only those.
- **Non-font files caught by the glob** — a stray `README.md` in the fonts folder gets
  linked as a bundle resource.

**Ordering hazard:** `react-native-asset`'s `removeResourceFile` "has the side effect of
unlinking all other targets". Always relink assets **before** any hand-added Resources
entry (e.g. `PrivacyInfo.xcprivacy`), or the manual edit is lost.

**Font names differ by platform.** iOS matches `fontFamily` by **PostScript name**;
Android matches by asset filename. A CSS-style family (`'DM Sans'`) matches neither and
silently falls back to the system font. Native font names belong in
`packages/core/src/tokens/rn-styles.ts` — never in `tokens/index.ts`, which is
Figma-generated and is the web/Tailwind source of truth.

## Step 4 — Buy the right kind of Mac time

Do this **after** Steps 1–3, so the first build tests fixed code.

Choose deliberately:

| Route | Good for | Cost reality |
|---|---|---|
| **Interactive Mac** (rented cloud Mac, teammate's machine) | **first bring-up** — real terminal, Xcode, a debugger | ~€0.10/hr; a day or two covers it |
| **macOS CI** (`macos-latest`) | regression guard **after** the build is green | **10× minute billing on private repos**; ~150–250 billed min per run |

CI is the wrong tool for exploratory bring-up: a ~20-minute blind feedback loop per
attempt, no debugger, no Xcode. Get it green interactively, *then* add CI.

**Check private registries before provisioning anything.** An `.npmrc` scope line
pointing at GitHub Packages with no token means every fresh machine and every CI job
dies at `pnpm install`. Needs a PAT with `read:packages` — in `~/.npmrc` on a Mac, or
`NODE_AUTH_TOKEN` from a secret in CI.

### Bring-up order on macOS

1. `pnpm install` — `node-linker=hoisted` matters: CocoaPods' `use_native_modules!`
   needs real `node_modules/<pkg>` paths, not pnpm symlink farms.
2. **Ensure `apps/native/.env` exists.** `react-native-config`'s pod script phase reads
   `HOST_PATH="$SRCROOT/../.."`. For a *pod* target `$SRCROOT` is `<app>/ios/Pods`, so
   this resolves to the app root — correct in this monorepo, but the build fails
   outright if the file is absent. Values are baked at **build** time; changing `.env`
   needs a rebuild, not a Metro reload.
3. Confirm Xcode meets the RN version's floor:
   `node -p "require('react-native/scripts/cocoapods/helpers.rb')"` won't work — read
   `min_xcode_version_supported` and `min_ios_version_supported` directly from
   `react-native/scripts/cocoapods/helpers.rb`. **Read them; don't guess** — the
   template's `IPHONEOS_DEPLOYMENT_TARGET` is usually already correct.
4. `cd ios && bundle exec pod install`, then **commit `Podfile.lock`**.
5. `xcodebuild -workspace … -sdk iphonesimulator CODE_SIGNING_ALLOWED=NO` — a simulator
   build needs no Apple account, so it works before any signing setup.
6. `.xcode.env` uses `export NODE_BINARY=$(command -v node)`, which often resolves to
   nothing in Xcode's non-login shell under nvm/fnm. `.xcode.env.local` is gitignored —
   expect to create one per machine.

## Step 5 — Verify in dependency order

Each check proves a specific thing. Run them in this order so a failure localizes:

| Check | Proves |
|---|---|
| Launches past splash, no redbox | bootstrap ran; native modules registered |
| Text renders in the brand font | asset linking + font-name resolution (Step 3) |
| Any API call succeeds | env reached the app **and** ATS isn't blocking (Step 2) |
| Data survives a cold restart | the storage driver works on iOS |
| A permission-gated flow prompts | the `NS*UsageDescription` string is present |
| Share/print flow opens | iOS-only branches that never executed on Android |
| `.app` bundle contains `PrivacyInfo.xcprivacy` | Resources build phase (Step 1) |
| Home screen shows a real icon | asset catalog populated |

`tsc` and lint prove nothing about any of this.

## Step 6 — Legacy native modules on the new architecture

An old, unmaintained pod (`RCT_EXPORT_MODULE`, `s.dependency "React-Core"`, no codegen
spec) is a *smoke test*, not an automatic rewrite. Modern RN ships first-class interop —
check `RCTTurboModuleManager.mm` for `LegacyModuleNativeMethodCallInvoker` and
`_legacyModuleCache`, which also **preserve a module's custom `methodQueue`**, so
serial-queue guarantees a JS driver depends on still hold.

Verify by exercising the module, not by reading its publish date. If it genuinely fails
to register, check whether the consuming layer is a DI seam — a well-factored storage
or platform driver means swapping the library touches one file, with `packages/core`
untouched.

## Never

- ❌ Assume `ios/` works because it looks complete — check for `Podfile.lock`
- ❌ Hand-edit `project.pbxproj` for fonts/assets — use `react-native-asset`
- ❌ Add a Resources entry *before* running `react-native-asset` (it unlinks)
- ❌ Reference `node_modules/.pnpm/<pkg>@<version>/…` from `react-native.config.js` or the pbxproj
- ❌ Put native font names in `tokens/index.ts` — that file is Figma-generated
- ❌ Flip `NSAllowsArbitraryLoads` to fix an HTTP backend — scope `NSExceptionDomains`
- ❌ Ship an app icon with an alpha channel or pre-rounded corners
- ❌ Link a permission-bearing pod that nothing imports — remove the dependency instead
- ❌ Reach for macOS CI to do a first bring-up
- ❌ Commit `Podfile.lock` from CI — generate it once interactively
- ❌ Change `applicationId` without changing `PRODUCT_BUNDLE_IDENTIFIER` in the same PR
