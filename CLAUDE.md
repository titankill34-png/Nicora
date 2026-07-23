# CLAUDE.md

Two agents run this repo from GitHub Actions. They never talk directly — issues
are the only queue. Read this file fully before any work.

## 1. Project

| Key | Value |
|---|---|
| App | Nicora — a smoking interval timer |
| Bundle ID | `com.titankill34.nicora` |
| Xcode | `Nicora.xcodeproj` |
| Scheme | `Nicora` |
| iOS target | 17.0 |
| Deps | Swift Package Manager only |
| Signing | Never in CI. CI emits an unsigned `.ipa`; the user signs with KSign. |

### KSign constraints — hard limits

Never add: Push Notifications, iCloud, App Groups, HealthKit, Sign in with Apple,
Associated Domains, app extensions, widgets, watchOS targets, or non-SPM dynamic
frameworks. All of these break local signing. Use local notifications instead of
push. Entitlements live only in `Nicora/Nicora.entitlements`.

## 2. Product

Core loop: the user logs each cigarette. The app counts up since the last one and
sets a target interval for the next. The target lengthens as the user complies,
so the gaps grow gradually rather than by quitting cold.

Screens: Home (count-up hero + log button), Stats (per-day counts, average
interval, money saved), History (day-grouped timeline), Settings.

Visual direction: vertical gradient background deep navy to light blue, frosted
glass cards at 20pt radius with a hairline white border, large white numerals
with dimmed decimals, pill-shaped buttons, circular monogram avatars in list
rows, a labelled donut ring for goal vs actual, 5-item bottom tab bar with thin
line icons, SF Rounded throughout.

## 3. Lanes

| Agent | Label | Owns |
|---|---|---|
| Tony | `lane:tony` | `Sources/UI/**`, `Sources/Features/**/Views/**`, `Resources/**` |
| Jimmy | `lane:jimmy` | everything else, including `Sources/Contracts/**`, `Tests/**`, `.github/**`, and `Nicora.xcodeproj/project.pbxproj` |

An agent edits only its own lane. Need something outside it? Open an issue for
the other lane and stop.

**Jimmy owns `project.pbxproj` exclusively.** New UI files Tony merges are not in
the Xcode target until Jimmy wires them. Jimmy checks for unwired files on every
task.

## 4. Cross-lane contract

Everything crossing lanes goes through a protocol in `Sources/Contracts/`. Tony
never references a concrete type owned by Jimmy. Jimmy keeps working mocks in
`Sources/Contracts/Mocks/` so Tony is never blocked.

## 5. Verification

A PR is not ready until this passes:

```bash
xcodebuild -scheme Nicora -destination 'generic/platform=iOS' \
  -configuration Release \
  CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY="" build
```

On a Linux runner `xcodebuild` does not exist. In that case skip it and let the
macOS CI job be the gate — but never merge a PR with red CI.

## 6. Never

- Edit outside your lane
- Commit `.p12`, `.mobileprovision`, `.cer`, or any key
- Merge your own PR
- Change Bundle ID or deployment target without an approved issue
- Weaken or delete a test to make CI pass
