# xDrip UI Redesign Plan (fork)

Goal: modern, clean, dark-only UI. Reorder elements by usefulness. Mirror the Nightscout redesign
(`../cgm-remote-monitor/static/css/tokens.css`): semantic tokens first, then incremental `feat(ui)` commits.

Scope: fork only (no upstream constraints). Dark theme only. Stay on XML views + Java (no Compose migration).

## Current state (baseline)

- AppCompat 1.0.0, Material 1.1.0, `Theme.AppCompat` + Holo gradient (`res/drawable/holo_background.xml`), Holo themes still used.
- 99 layouts. `activity_home.xml` (665 lines): ~10 scattered status TextViews, 14 ImageButtons in one row,
  BG value overlaid on chart, notices overlaid on chart, time-range buttons at bottom.
- Overflow menu (`res/menu/menu_home.xml`): 23 items, mixes daily use with debug/destructive actions.
- Drawer (`NavDrawerBuilder.java`): one flat list (sensor actions, data views, alerts, settings).
- Settings: ~500 preferences, deeply nested (`res/xml/pref_*.xml`, `xdrip_plus_prefs.xml`).
- Graph: hellocharts, colors from user prefs via `ColorCache`.
- `colors.xml`: 10 colors, no token system.

## Baseline audit (screenshots in `Documentation/redesign/baseline/`, real device, 2026-10-28)

- **Home (01):** delta and age at top-left and BG value at top-right both overlap the chart. The "all alarms disabled" notice is drawn over the 350 axis label. The BG-entry and note buttons float over the chart's right edge. The low zone is a heavy dark-red block. The preview chart takes about 20% of the height. There's no clear visual hierarchy.
- **Overflow menu (09):** long flat list. Translation, update check, feedback and wear DB reset sit at the same level as reminders.
- **Settings (02, 03):** Holo-style list. Large grey legacy icons, inconsistent (some rows have none). Titles get cut off ("anze.."). Thick separators under section headers. Some summaries untranslated (fine, not our concern).
- **History (04):** grey boxy `<<` / `>>` / date buttons. Stats text (Avg, A1c, in/hi/lo) overlaps the chart and its axis labels. The toast hint covers the preview chart.
- **Statistics, percentile (05):** a different visual language: pill tabs TD/YTD/7D/30D/90D with a red selected state, and pure yellow/red range lines that don't match the Home chart colors.
- **Snooze (06):** three full-width flat buttons, the rest of the screen empty. No hierarchy between "disable low", "disable high" and "disable all".
- **System status (07, 08):** bright blue tab strip (the only saturated surface in the app). Grey flat buttons. A centered key/value table with a ":" column.
- **Common:** gradient background, no surfaces or cards, no consistent spacing, all-caps grey buttons, colors chosen one-off per screen.

## Phases

### 0. Foundation
- Bump appcompat to 1.7.x and material to 1.12.x. Satisfy the databinding version guard in `app/build.gradle`.
- Capture baseline screenshots on an emulator: Home, drawer, overflow menu, settings, treatment dialog, system status.

### 1. Tokens (same values as Nightscout)
- `colors.xml`: `bg #0b0d10`, `surface #14181d`, `surface2 #1c2127`, `border #2a313a`, `text #e8ecf0`,
  `textDim #9aa5b1`, `ok #3ddc84`, `warn #ffc233`, `urgent #ff5c5c`, `info #4aa8ff`.
- `dimens.xml`: spacing 4/8/16/24, radius 8/12.
- Type scale, including a display size for the BG value.

### 2. Theme
- `AppTheme` becomes `Theme.Material3.Dark.NoActionBar`.
- Remove the Holo gradient and Holo themes. Merge duplicate styles.
- Edge-to-edge with insets handling.

### 3. Home: new layout
```
┌ Toolbar: name · sensor status · ⋮ ┐
│ HERO CARD                        │
│  142  ↗  +4   · 2 min ago        │
│  low predicted in 40 min (warn)  │
├ Status chips (horizontal scroll) ┤
│ phone 80% · bridge 60% · sensor 9d · ❤72 · steps 4k │
├ 3h | 6h | 12h | 24h  (segmented) ┤
│ CHART (full width)               │
│ preview chart                    │
├ Notices card (not over chart)    ┤
└ Bottom bar: [+ Treatment] [Note] [BG] [Undo] ┘
```
- Pending treatment (approve/cancel, carb/insulin/time values): one confirm card that shows only while a treatment is pending.
- Status chips open System Status on tap.
- Source wizard: onboarding card, shown only when needed.
- Rule: keep every view ID and databinding variable. `Home.java` toggles views by ID, so move the layout, not the logic.

### 4. Drawer and overflow menu
- Drawer sections:
  - **Sensor:** start, stop, calibrate, override, calibration graph
  - **Data:** history, statistics, BG table, note search
  - **Alerts:** level alerts, snooze
  - **Devices:** Bluetooth scan, system status
  - **Tools:** export/import, backup, watch actions, sync, share config, backfill, events log (moved from overflow)
  - **Settings**
- Overflow menu keeps 4 items: speak readings, reminders, emergency messages, help.
- Debug items (test feature, delete all BG) move to Settings → Advanced → Developer. Delete requires confirmation.

### 5. Dialogs
- Treatment, calibration, snooze and alert dialogs.
- Keypad and treatment entry become a bottom sheet with large keys.
- Snooze screen and alert list use cards.

### 6. Settings
- Style first: dark Material theme, clearer categories, icons.
- Then regroup the top level: Data source, Alerts, Display, Sync/Nightscout, Watch, Advanced.
- **Preference keys never change** (saved settings and `findPreference` lookups depend on them).

### 7. Secondary screens, in batches
- Stats, system status, event log, history, BG/calibration tables, profile and basal editors.
- Reuse the cards and chips from Phase 3.
- History: stats move into a summary card above the chart (no overlap). Date navigation becomes one row: icon buttons plus a date chip.
- Statistics: the TD/YTD/7D/30D/90D pills become the same segmented control as Home's time range. Range lines use the chart tokens.
- Snooze: "No active alarm" status card at the top, then grouped actions. Disable low/high as tonal buttons, disable all as an outlined `urgent` button.
- System status: tab strip in the surface color with an indicator, no blue band. Key/value rows left-aligned in cards. Restart and refresh become one button row.

### 8. Graph, widgets and notifications
- New default chart colors in `ColorCache`, built from the tokens. **User color overrides are kept.**
- Home-screen widgets and the expanded notification.

### 9. Wear
- Separate effort, later.

## Guardrails
- Alarm and alert behavior never changes. Urgent notices never get hidden by the reorder.
- Text contrast ≥ 4.5:1. Touch targets ≥ 48dp.
- No changes to existing translated strings. New strings (section headers) English only.
- One branch per phase, small `feat(ui)` commits, before/after screenshots each.

## Skills
- impeccable: audit and critique each screen before changing it.
- ui-ux-pro-max: mobile UX rules.
- imagegen-frontend-mobile: Home mock-ups before Phase 3 code.
- dataviz: chart palette.
- Manually installed Android skills (user), e.g. android-xml-views, mobile-android-design.
