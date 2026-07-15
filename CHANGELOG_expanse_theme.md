# Expanse Theme — Change Log

This file tracks every real change made to the live Home Assistant instance for the
"Expanse" sci-fi theme project, plus exactly how to undo each one. Written in English
per Jan's request. Mockups (not live) live in `mockups/expanse_*.html` in this same
folder — those are just static previews, nothing on HA.

---

## 2026-07-06 — Step 0: Full backup

**What:** Created a full HA snapshot before touching anything.
- Backup name: `pre-expanse-theme-2026-07-06`
- Backup ID: `cb922ff9`
- Size: ~47 MB, took 18s
- Config only (no database included)

**How to revert / use it:** Settings → System → Backups → find
`pre-expanse-theme-2026-07-06` → Restore. This restores the *entire* config to this
exact point in time (restarts HA). Only use this as a last resort if something in this
project goes badly wrong — for undoing a single step, see the specific entries below,
which are much less drastic.

---

## 2026-07-06 — Step 1: Added Google Fonts resource (Tektur + Share Tech Mono)

**What:** Added a new CSS dashboard resource pointing at Google Fonts, same mechanism
already used for the existing Oxygen/Ubuntu font resource on this instance.

- Resource ID: `de50ce9b6c9d47519e9746fc01e83abe`
- Type: `css`
- URL: `https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Tektur:wght@400;500;600;700;800;900&display=swap`

**Effect right now:** None visible yet. Loading a font doesn't change anything until
something (a theme or card-mod style) actually references `Tektur` or `Share Tech Mono`
as a font-family. Completely safe, purely additive.

**How to revert:** Settings → Dashboards → Resources → find the resource with this URL
→ delete it (trash icon). Or ask me to remove it via `ha_config_set_dashboard_resource`
/ the resources list.

---

## 2026-07-06 — Step 2: "Expanse" theme file — NOT YET APPLIED (manual step needed)

**Important limitation:** creating a brand-new HA theme (so it shows up as its own
entry in Settings → General → User preferences → Theme, next to `waves`) requires
writing a file under `/config/themes/`. The tool I'd normally use for that
(`ha_config_set_yaml`) is not available in this session — I can list/select existing
themes (`ha_manage_theme`) but not author a new theme file directly.

**What this means for you:** the theme content below needs to be added manually, once,
via the **Studio Code Server** add-on you already have in your sidebar. This is a
one-time, ~2 minute task:

1. Open **Studio Code Server** from the HA sidebar.
2. Navigate to `/config/themes/` (create the folder if it doesn't exist yet).
3. Create a new file: `expanse.yaml`
4. Paste the YAML block below into it, save.
5. In HA: **Developer Tools → YAML → Themes** → click reload (or restart HA).
6. It will now appear in Settings → General → Theme as **"Expanse"** — pick it there
   whenever you want to try it. Switching back to `waves` or `Home Assistant` is just
   picking a different item in that same dropdown — instant, no side effects.

```yaml
Expanse:
  modes:
    dark:
      primary-color: "#5eead4"
      accent-color: "#eab13a"
      primary-background-color: "#070d0c"
      secondary-background-color: "#0a1615"
      card-background-color: "#0e1a19"
      primary-text-color: "#eaf7f4"
      secondary-text-color: "#8fada8"
      disabled-text-color: "#3d524e"
      divider-color: "rgba(205, 238, 231, 0.13)"
      state-icon-active-color: "#59d98c"
      error-color: "#e05a56"
      warning-color: "#eab13a"
      success-color: "#59d98c"
      primary-font-family: "Tektur, sans-serif"
      ha-card-border-radius: "4px"
      ha-card-border-width: "1px"
      ha-card-box-shadow: "none"
```

**How to revert:** delete `expanse.yaml` from `/config/themes/` (via Studio Code
Server), then reload themes the same way (Developer Tools → YAML → Themes). The
"Expanse" entry disappears from the dropdown; anyone using it falls back to the
default theme automatically.

**Note on scope:** this theme file only covers global colors/fonts. It does **not**
include the cut-corner panels, hero SECURED panel, sidebar-tab layout, or tick/bar
gauges from the mockups — those require rebuilding the actual dashboards (separate,
future step), not just this theme.

---

## 2026-07-06 — Applied theme, colors work, font didn't (troubleshooting)

Jan added `themes/expanse/expanse.yaml` (matching the existing subfolder convention
used by `waves/`, `dark_themes/`, `green_and_dark/`), then did a full HA **Restart**
(instead of just the targeted "Themes" YAML reload — a full restart covers themes too,
so that's fine). Result: theme colors applied (tab accent color changed), but the
**Tektur font did not visibly apply**.

**Two likely causes, in order of likelihood:**

1. **Browser cache.** The dashboard resource note explicitly warns: "Clear browser
   cache or hard refresh (Ctrl+Shift+R) to load changes." A server restart reloads the
   backend/theme, but the *browser* can still be serving a cached frontend bundle that
   never re-fetched the new Google Fonts CSS resource. **Try a hard refresh
   (Ctrl+Shift+R, or Ctrl+F5) first** — this fixes it most of the time.

2. **`primary-font-family` alone isn't always enough.** Some parts of the HA frontend
   (older "Paper" element based components) read different, more specific CSS
   variables rather than `primary-font-family` directly. If the hard refresh doesn't
   fix it, add these extra lines to `expanse.yaml` under the same `dark:` block:

   ```yaml
       paper-font-common-base_-_font-family: "Tektur, sans-serif"
       paper-font-common-code_-_font-family: "Share Tech Mono, monospace"
       paper-font-body1_-_font-family: "Tektur, sans-serif"
       mdc-typography-font-family: "Tektur, sans-serif"
       ha-card-header-font-family: "Tektur, sans-serif"
   ```

   Save, then reload Themes again (Developer Tools → YAML → Themes, or another
   restart).

**How to revert this troubleshooting step:** it only adds a few more lines to the same
file — deleting them (or the whole file) undoes it exactly like Step 2 above.

---

## 2026-07-06 — Step 3: card-mod fix for fonts on dashboard cards

**The actual problem:** hard refresh + the extra Paper/MDC font variables fixed *some*
Settings-page headings, but the real Overview dashboard (section titles like "Home" /
"Daily" / "Caitlyn", tile names, values) showed **zero** font change. This is a known,
documented Home Assistant limitation: `primary-font-family` and its Paper/MDC cousins
are read by older components, but newer "tile" cards and section headings in the
Sections-view dashboards don't reliably inherit them. Theme variables alone can't reach
these — confirmed against the official card-mod wiki, not a guess.

**The fix:** `card-mod` (you already have this installed via HACS) can apply CSS
globally through your theme file, instead of setting a font-family variable that some
cards ignore. Add these lines to `expanse.yaml`, inside the `dark:` block, at the same
indent level as `primary-color` etc. (6 spaces):

```yaml
      card-mod-theme: "Expanse"
      card-mod-card: |
        ha-card, ha-card * {
          font-family: "Tektur", sans-serif !important;
        }
      card-mod-view: |
        hui-view, hui-view * {
          font-family: "Tektur", sans-serif !important;
        }
      card-mod-root: |
        * {
          font-family: "Tektur", sans-serif !important;
        }
```

What each line does:
- `card-mod-theme` — switches card-mod's theme-wide styling on for this theme (required
  for the three lines below to do anything).
- `card-mod-card` — forces the font on every card (`ha-card` and everything inside it) —
  this is the one that fixes tiles, values, tile names.
- `card-mod-view` — forces the font on section headings ("Home" / "Daily" / "Caitlyn")
  which live in the view, not inside a card.
- `card-mod-root` — forces the font on the top header/tab bar, which was also still
  showing the wrong font.

**Steps for you:**
1. Open Studio Code Server → `/config/themes/expanse/expanse.yaml` (or wherever the
   file ended up — same one you already edited for Step 2/troubleshooting).
2. Paste the block above, indented to match the existing keys, save.
3. Developer Tools → YAML → reload Themes (or a full restart, like last time).
4. Hard refresh the dashboard (Ctrl+Shift+R) and check.

**How to revert:** delete these 4 new lines (or the whole file) — same as before,
nothing else is affected.

**Result (2026-07-06, later same evening):** regression, not a fix — after adding this
block and reloading, even the Settings headings that were previously showing Tektur
correctly went back to the default font. The theme file most likely failed to parse
(a YAML indentation slip inside the new block is the likely cause) and HA silently fell
back to no custom theme at all. Jan called this correctly and asked to step back rather
than keep patching on top of a broken state — good instinct, followed here.

**Rollback done:** remove the 4 `card-mod-*` lines added in this step, keeping
everything from Step 2 + the troubleshooting paper-font lines. That restores the
last known-good state (colors work, Settings headings in Tektur, dashboard cards
still not themed — the original open problem).

---

## 2026-07-06 — Step 4: switched approach to a dedicated font integration

**Why card-mod was dropped:** card-mod requires hand-written CSS in the theme YAML,
which is fragile (one indentation mistake silently breaks the whole theme) and still
only reaches the shadow-DOM elements you explicitly target — HA's frontend is full of
nested Shadow DOM, and that's exactly why fonts weren't reaching tile cards to begin
with.

**New approach:** [`itsyuni/homeassistant-customfonts`](https://github.com/itsyuni/homeassistant-customfonts)
— a small HACS custom integration built specifically to solve this problem. It patches
`Element.prototype.attachShadow` and runs a `MutationObserver` so it can inject the
font `<style>` into *every* shadow root as it's created, including ones inside tile
cards and sections — not just the top-level document. No YAML at all; it has its own
settings panel with a live preview and an "Apply to Shadow DOM" toggle.

**Install steps (for Jan, via HACS):**
1. HACS → ⋮ (top right) → Custom repositories
2. Add `https://github.com/itsyuni/homeassistant-customfonts`, category: Integration
3. Find "Home Assistant Custom Fonts" in the HACS store → Download
4. Restart Home Assistant
5. Settings → Devices & Services → Add Integration → "Home Assistant Custom Fonts"
6. In the panel: Font Source = Google Fonts, paste the Tektur Google Fonts URL (same
   one already used for the dashboard resource), enable "Apply to Shadow DOM"

**How to revert:** disable the integration's toggle (fonts turn off instantly), or
remove it entirely via HACS — doesn't touch the theme file or anything else.

**Result (2026-07-06, later same evening):** Installed, configured with Tektur +
"Apply to Shadow DOM" on. Font now applies correctly across the dashboard — tile
cards, section headings, values all show Tektur. Only remaining gap: the HA sidebar
itself doesn't pick up the font. Jan's call: not worth chasing, just collapse the
sidebar. **Font problem considered solved.**

---

## 2026-07-06 — Step 5: heading-card styling — theme-level card-mod doesn't reach Sections view

**Goal:** style the 6 `heading` cards on the Home dashboard (Home, Daily, Caitlyn, Camina,
Fit, Sailing — Galgenweel) to match `mockups/expanse_home_mockup.html`'s panel look
(cyan border, dark panel background, small corner brackets, uppercase/letter-spaced
title) — via **theme-level** `card-mod-theme`/`card-mod-card` in `expanse.yaml`, so the
style lives in one place instead of being repeated on every card.

**What was tried (all failed silently — no visual change, no console errors):**
1. `:host(.expanse-heading)` selector + `card_mod: class: expanse-heading` on the card.
2. `ha-card.expanse-heading` selector + same `card_mod: class:` mechanism.
3. `ha-card.type-heading` selector with no `class:` needed (heading cards get this class
   automatically per a community forum thread).

**Root cause (confirmed, not guessed):** [`thomasloven/lovelace-card-mod` issue #389](https://github.com/thomasloven/lovelace-card-mod/issues/389)
— "Card mod not working in new section view?" — closed inconclusive, with community
confirmation that the **Sections view (used by this whole dashboard) is still
experimental and many cards/plugins don't fully hook into it.** card-mod is installed
at v4.2.1 (latest, confirmed via HACS) — not a version problem.

**Diagnostic test that proved this (live, Jan pasted it):** the exact same `card_mod:
style:` block, but applied **directly on the "Home" heading card** (not via the theme),
with an obvious red-background/yellow-border test style:

```yaml
- type: heading
  heading: Home
  heading_style: title
  icon: mdi:home
  card_mod:
    style: |
      ha-card {
        background: red !important;
        border: 4px solid yellow !important;
      }
```

**Result:** worked instantly, no reload even needed. This isolates the problem to
theme-level propagation specifically — card-mod itself, and heading cards themselves,
are both fine.

**Conclusion / chosen approach:** per-card `card_mod` on each of the 6 heading cards,
using the real Expanse panel style (cyan border `rgba(94,234,212,0.22)`, background
`rgba(6,20,22,0.86)`, corner brackets via `::before`/`::after` in `#5eead4`, uppercase/
bold/letter-spaced text) — more repetition than the hoped-for single theme location, but
proven to work. The `card-mod-theme`/`card-mod-card` lines already in `expanse.yaml`
(from Step 3, the font experiment) stay in the file but are not relied on for heading
styling going forward.

**Follow-up attempt (same evening):** before giving up on the one-place theme approach,
tried registering card-mod as a **frontend module** rather than just a dashboard
resource, per a note in card-mod's own README: doing this via `configuration.yaml`
gives "enhanced speed in applying card-mod to cards, especially when using card-mod
themes." HA core confirmed at `2026.7.1` (well above card-mod's minimum `2026.2.0`), so
not a version issue. Added to `configuration.yaml`, under the existing `frontend:` key
(not a new one — YAML doesn't allow duplicate top-level keys):

```yaml
frontend:
  themes: !include_dir_merge_named themes
  extra_module_url:
    - /hacsfiles/lovelace-card-mod/card-mod.js?hacstag=190927524421
```

Restarted HA (required after changing `extra_module_url`). **Result: still no effect** —
screenshot of the dashboard after restart + hard refresh showed the base theme colors
working fine, but the 6 heading cards still had zero border/panel/corner-bracket
styling. So the frontend-module route wasn't the fix either — theme-level
`card-mod-card` genuinely does not reach `heading` cards inside this Sections-view
dashboard, restart or not.

**Final approach adopted:** YAML anchors within the dashboard file itself. Define the
`card_mod` style once with `&heading_mod` on the first heading card, then reference it
with `card_mod: *heading_mod` on the other five. One source of truth, but scoped to the
dashboard file (not the shared theme) — proven to work, since it's just the same
per-card mechanism confirmed in the diagnostic test above. Applied to a **copy**
dashboard first (`path: home-v2`, not the live `dashboard-home`) so nothing production
was touched while testing. Trade-off noted: if this dashboard is ever resaved through
the visual UI editor, HA will expand the anchors into 6 duplicated literal blocks —
still works, just no longer DRY. Not a concern as long as edits stay in raw YAML, which
is how Jan works anyway.

**Status at end of session (2026-07-06):** anchor-based YAML given to Jan for the
`home-v2` copy dashboard, not yet confirmed live (session ended before result came
back). Next session: check whether the anchors rendered correctly, then decide whether
to port this to the real `dashboard-home`.

---

## Not done yet / still open

- Actual "Expanse-styled" dashboards (Home / Caitlyn / Camina) — still only exist as
  static mockups in `mockups/`, nothing built in HA yet. Home dashboard now has a first
  real attempt (heading-card styling via YAML anchors) on a copy (`home-v2`), pending
  confirmation.
- Sensor Bar Card Plus — not installed via HACS yet.
- Swiss Army Knife card — already installed, not yet used for anything Expanse-related.
- Hero "SECURED" panel restyle — deferred until real dashboard build.
- Once heading cards are confirmed working: style `tile` cards, then remaining panel
  types (mini-graph-card, glance, calendar, weather-card).

See `Personal/memory/open_decisions.md` (Vela's memory) for the full running log of
design decisions behind this project.
