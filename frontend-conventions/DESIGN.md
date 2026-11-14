# DESIGN.md — Lunar Landing Site Navigator Visual System

This file records the design system decisions for the Lunar Landing Site
Navigator frontend. Every UI change should reference or update it, per the
Hallmark convention of recording design direction in a versioned file.

## Palette (deep-space, colorblind-safe)

| Role | Hex | Use |
|------|-----|-----|
| Background (deep space) | `#0f0e13` | App background, darkest surface |
| Surface (card/elevated) | `#1d1b22` | Panels, modals, elevated containers |
| Text (primary) | `#f7f7f8` | Body text, headings, high-contrast labels |
| Accent (solar/amber) | `#f59e0b` | Primary actions, links, solar highlights |
| Warning | `#f59e0b` / `#d97706` | Caution states, marginal visibility |
| Error | `#dc2626` / `#b91c1c` | Invalid input, failed validation, out-of-range |
| Success | `#16a34a` / `#15803d` | Valid state, confirmed computation |
| Info | `#3b82f6` / `#1d4ed8` | Neutral information, helper text |
| Muted text | `#a1a1aa` | Secondary labels, disabled states |
| Border | `#27272a` | Subtle dividers on dark surfaces |

**Do not introduce new hues.** If a new semantic color is needed, add it here
first with a rationale, then use it consistently.

## Typography

- **Headings:** sans-serif, uppercase tracking-tight, accent color (`#f59e0b`)
  for section titles; primary text (`#f7f7f8`) for page titles
- **Body:** sans-serif, primary text (`#f7f7f8`), 1rem base, 1.5 line height
- **Labels/captions:** muted text (`#a1a1aa`), 0.875rem, letter-spacing wide
- **Monospace (coordinates, epochs):** primary text on surface background,
  suitable for lat/lon, ISO-8601 timestamps, query strings

Readable on a projector in a lit room and on mobile portrait.

## Spacing & layout

- Use a 4px grid (4, 8, 12, 16, 24, 32, 48, 64)
- Panels: surface background (`#1d1b22`), border (`#27272a`), rounded corners
- 3D lunar scene: primary, unframed viewport; the Moon is the dominant object
- Map: secondary site-selection panel or view, visually subordinate to the 3D scene
- Sidebar/inset panels: surface background with subtle border

## Components

- **Buttons:** accent (`#f59e0b`) background with primary text for primary
  actions; surface background with primary text for secondary; error (`#dc2626`)
  for destructive actions
- **Inputs:** surface background, border (`#27272a`), primary text,
  accent focus ring (`#f59e0b`)
- **Badges/tags:** surface background, muted text, small rounded padding
- **Celestial colors:** use the solar accent for the Sun and info blue for Earth.
- **3D scene:** Moon is the primary subject; Earth and Sun are contextual bodies
  whose apparent direction and disc size reflect the selected site and date.
- **Overview scale:** compress distances only when necessary to show all three
  bodies; clearly label the scene “not to scale.”
- **Site selection:** the map remains available as a secondary control and
  synchronizes with the selected-site marker in the 3D scene.
- **Charts:** use accent for solar, info blue (`#3b82f6`) for Earth, and muted
  terrain colors; always include text alternatives.
- **Decision results:** label outcomes as meeting, missing, or unknown; never
  rely on color alone to communicate a result or risk.
- **AI proposals:** show the interpreted request and proposed scenario changes
  before applying them; provide clear approve and dismiss actions.
- **Shared workspaces:** attribute changes to a participant, show conflicts,
  and make undo or restore discoverable.

## Motion

- Respect `prefers-reduced-motion` — disable or reduce all animation
- Eased transitions only (no bouncing, no staggered reveals)
- "Refining" state: subtle pulse or spinner, not distracting

## Projector safety

- All text WCAG AA minimum (4.5:1) against its background
- Avoid pure white (`#ffffff`) — use near-white (`#f7f7f8`) to reduce glare
- Accent amber (`#f59e0b`) is visible against deep space background

## Changes to this file

Any addition of a new color, component, or spacing value must be recorded here
first, then implemented consistently across the app.
