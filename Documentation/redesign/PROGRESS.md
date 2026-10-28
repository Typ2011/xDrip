# xDrip Redesign: Progress / Handoff

Read this first when continuing on another machine. Full plan: [`../redesign-plan.md`](../redesign-plan.md).
Baseline screenshots (real device, before redesign): [`baseline/`](baseline/).

Last updated: 2026-10-28

## Decisions (fixed)

- Fork only. `origin` = https://github.com/Typ2011/xDrip.git. No `upstream` remote (sync with NightscoutFoundation via GitHub when needed).
- Dark theme only. Material 3 (`Theme.Material3.Dark.*`). Stay on XML views + Java (no Compose migration).
- Same color tokens as the Nightscout redesign (`cgm-remote-monitor/static/css/tokens.css`).
- Clean up and reorder Home, drawer, overflow menu and settings (see plan, Phases 3, 4 and 6).
- Keep all view IDs, databinding variables and preference keys. Alarm and alert behavior never changes.
- Work style: one branch per phase, small `feat(ui)` commits. Heavy work is delegated to subagents on a lower-tier model (sonnet).

## Status

| Phase | State | Branch |
|---|---|---|
| 0. Foundation (deps + baseline) | **Done** (build + unit tests pass; emulator smoke test OK: Snackbar, Reminders FAB) | `redesign/phase-0-foundation` |
| 1. Tokens | **Done** (`xd_*` colors + dimens added, resources compile; nothing uses them yet) | `redesign/phase-1-tokens` |
| 2. Theme | Todo | - |
| 3. Home layout | Todo (make mock-ups first with `imagegen-frontend-mobile`) | - |
| 4. Drawer + overflow menu | Todo | - |
| 5. Dialogs | Todo | - |
| 6. Settings | Todo | - |
| 7. Secondary screens | Todo | - |
| 8. Graph, widgets, notifications | Todo | - |
| 9. Wear | Later | - |

### Phase 0 result
`app/build.gradle` bumps (wear module untouched):

| Library | Before | After |
|---|---|---|
| appcompat | 1.0.0 (resolved 1.1.0) | 1.7.0 |
| material | 1.1.0 | 1.12.0 |
| constraintlayout | 1.1.3 | 2.1.4 |
| recyclerview | 1.0.0 (resolved 1.1.0) | 1.3.2 |
| preference | 1.0.0 | 1.2.1 |

Transitively resolved: core 1.13.0, fragment 1.5.4.

- `./gradlew :app:assembleFastDebug` passes. APK: `app/build/outputs/apk/fast/debug/app-fast-debug.apk`.
- `./gradlew :app:testFastDebugUnitTest` passes.
- JDK 24 (temurin) works. AGP 8.13.2, Gradle 8.14.3.
- Variants are flavors `fast`/`prod` × build types `debug`/`release`/`dev`. There is no "fastDev" debug task. Use `assembleFastDebug`.
- The databinding version guard (`app/build.gradle` ~L48-58) accepts 8.13.2, so it didn't need changes.

## Open items / next steps

1. **Done (2026-10-28, Windows emulator):** `fast` flavor now has `applicationIdSuffix ".redesign"` (plus `app/src/fast/google-services.json` with the renamed package). Gradle needs JDK 17-24 (JDK 25 fails; use `~/.jdks/jbr-21.0.11` and set Gradle JDK in Android Studio). Original plan: Don't install it over the live xDrip: different signing key, and uninstalling wipes data. Plan: add `applicationIdSuffix ".redesign"` to the `fast` flavor for a side-by-side install, with alerts off or a different data source in the test app (otherwise both apps poll Share and alert twice). Check:
   - The Snackbar on Home (`Home.java` ~L3676, uses `com.google.android.material.R.id.snackbar_text/snackbar_action`).
   - The `FloatingActionButton` in `res/layout/activity_reminders.xml:50`.
   - Both currently run under the `Theme.AppCompat` parent.
2. **Phase 1:** tokens in `res/values/colors.xml` plus `dimens.xml`.
3. **Phase 2:** `AppTheme` becomes `Theme.Material3.Dark.NoActionBar` (or the Bridge variant first). Plain `Button`s will inflate as `MaterialButton`, so check every screen. Call `AppCompatDelegate.setDefaultNightMode(MODE_NIGHT_YES)`.
4. **Before Phase 3:** Home mock-ups for user sign-off.

## Key files

- `app/src/main/res/layout/activity_home.xml`: Home (665 lines, hellocharts, databinding).
- `app/src/main/java/com/eveningoutpost/dexdrip/Home.java`: toggles Home views by ID.
- `app/src/main/res/menu/menu_home.xml`: overflow menu (23 items).
- `app/src/main/java/com/eveningoutpost/dexdrip/NavDrawerBuilder.java`: drawer.
- `app/src/main/res/values/styles.xml`, `colors.xml`: themes and colors.
- `app/src/main/java/com/eveningoutpost/dexdrip/utilitymodels/ColorCache.java`: chart colors (user-overridable).
- `app/src/main/res/xml/pref_*.xml`, `xdrip_plus_prefs.xml`: settings (~500 prefs).

## Skills to install

### Used in this project (installed on the original machine)

| Skill | Purpose | Install |
|---|---|---|
| impeccable | Audit, critique, tokens, hierarchy (main design driver) | `npx skills add pbakaus/impeccable --skill impeccable` |
| ui-ux-pro-max | Mobile UX rules, palettes, touch targets | `/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill` then `/plugin install ui-ux-pro-max@ui-ux-pro-max-skill` |
| imagegen-frontend-mobile | Home mock-ups (Phase 3) | `npx skills add Leonxlnx/taste-skill --skill imagegen-frontend-mobile` |
| redesign-existing-projects | Audit-first redesign checklist | `npx skills add Leonxlnx/taste-skill --skill redesign-existing-projects` |
| frontend-design | Aesthetic direction | `/plugin install frontend-design@claude-plugins-official` |
| superpowers (brainstorming, writing-plans, executing-plans, verification) | Process | `/plugin install superpowers@claude-plugins-official` |
| dataviz | Chart palette (Phase 8) | Built into Claude Code (no install) |

### Android-specific (recommended, to install manually)

| Skill | Purpose | Install |
|---|---|---|
| android-xml-views (hoangnguyen0403) | XML views, ViewBinding, RecyclerView standards (matches xDrip's stack) | `npx skills add hoangnguyen0403/agent-skills-standard --skill android-xml-views` |
| mobile-android-design (wshobson) | Material 3 / Material You guidance (examples are Compose) | `npx skills add https://github.com/wshobson/agents --skill mobile-android-design` |
| material-3 (hamen), optional | Material 3 compliance audit | `/plugin marketplace add hamen/material-3-skill` then `/plugin install material-3@material-3-skill` |
| android/skills (Google), optional | Official, mostly Compose (e.g. edge-to-edge) | `npx skills add android/skills --skill edge-to-edge --agent claude-code` |

### Optional workflow tools (used on the original machine)

- caveman (terse output): `/plugin marketplace add JuliusBrussee/caveman`
- ponytail (minimal-diff discipline): `/plugin marketplace add DietrichGebert/ponytail`
- MCP servers: codeseeker (semantic code index; project indexed as `xDrip`), serena. Their local folders `.codeseeker/` and `.serena/` are gitignored.
