---
name: frontend-conventions
version: "1.2.0"
description: |
  Project-specific frontend conventions for the Lunar Landing Site Navigator.
  Encodes the design system (deep-space palette with derived semantic colors),
  URL-serializable state model, offline-capable science core,
  instant-approximate-then-refine UX, design review gates (Floor/Reflex/Finish),
  slop detection, accessibility requirements, and shareable link encoding.
  This is NOT a generic frontend skill — it encodes this app's specific
  conventions.
domain: ["frontend", "web-application", "user-experience", "design-system"]
work_mode: ["application-development", "ui-ux"]
compatibility:
  javascript: "ES2020+"
  packages: ["react", "vite", "typescript", "tailwindcss", "@tailwindcss/vite", "three", "@react-three/fiber", "@react-three/drei"]
  network: "optional online AI/collaboration; offline-capable science core"
author: "K-Dense Inc. / project-specific"
license: "MIT"
references:
  - "QuickMap shareable link pattern (defs param)"
  - "AGENTS.md science conventions (frames, AZ-EL, upper-limb convention)"
  - "Impeccable design skill (critique/audit/polish/harden, slop detector)"
  - "Hallmark design skill (Floor/Reflex/Finish gates, slop linting)"
  - "Tailwind CSS v4 + Vite installation: https://tailwindcss.com/docs/installation/using-vite"
---

# Frontend Conventions — Lunar Landing Site Navigator

## Purpose

Teach an AI agent the project-specific frontend conventions for the Lunar
Landing Site Navigator web app. This skill is NOT a generic "how to code
frontend" skill (there are already many for React, Tailwind, Next.js, etc.).
It encodes the design system, state model, architecture constraints, and UX
patterns specific to this project.

## When to apply

- Building or modifying UI components for the landing site navigator
- Designing the state model and URL serialization
- Implementing the map-first, click-to-compute interaction
- Creating the Sun/Earth AZ-EL plot with terrain silhouette
- Implementing accessibility (colorblind-safe, keyboard nav, chart alternatives)
- Encoding shareable link state
- Enforcing the offline-capable science core and optional online states
- Running design review gates before shipping UI changes

## Architecture constraint

**Offline-capable science core.** The deployed app is static files. Core
calculations use precomputed ephemeris and terrain assets; they must not
depend on runtime calls to Horizons, NASA, QuickMap, or an AI provider. A
network failure must not block core site/date analysis. If AI or live
collaboration is later enabled, make the online state explicit and keep those
services optional.

## Design system

**Color palette (deep-space, colorblind-safe):**

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

**Semantic color usage:**
- **Warning** (`#f59e0b`, darker `#d97706`): Sun or Earth near the horizon mask, marginal illumination/communication windows, data at edge of validity.
- **Error** (`#dc2626`, darker `#b91c1c`): Invalid coordinates, out-of-range date, missing tiles, computation failure.
- **Success** (`#16a34a`, darker `#15803d`): Valid site, successful horizon mask computation, validated against Horizons.
- **Info** (`#3b82f6`, darker `#1d4ed8`): Helper text, "refining" state, assumptions disclosed.

**Typography:** readable on a projector and mobile. Technical detail one click
deeper; copy understandable by a non-expert.

**Contrast:** All text meets WCAG AA minimum (4.5:1) against its background.
Test on a projector in a lit room — the app will be demoed in presentation
settings.

## Styling: Tailwind CSS v4 + Vite

Tailwind CSS v4 uses a CSS-first configuration with Vite. Follow the official
installation guide:
https://tailwindcss.com/docs/installation/using-vite

**Installation (Vite):**
```bash
npm install tailwindcss @tailwindcss/vite
```

**`vite.config.ts`:**
```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [react(), tailwindcss()],
})
```

**CSS entry (`src/index.css`):**
```css
@import "tailwindcss";
```

**Project conventions:**
- Use the semantic color variables from `DESIGN.md` — define them as Tailwind
  theme extensions in `tailwind.config.js` (optional, since v4 is CSS-first)
  or as CSS custom properties in `src/index.css`
- Prefer utility classes over custom CSS; reserve custom CSS for complex
  3D canvas overlays and chart SVG styling
- No arbitrary values (`h-[47px]`) unless documented in `DESIGN.md`
- Use `oklch` or hex colors from the palette; avoid Tailwind's default
  color scale unless it matches `DESIGN.md`
- Dark mode by default — the app is deep-space themed; do not add a light
  toggle unless required

## DESIGN.md

Maintain a `docs/frontend-conventions/DESIGN.md` that records the visual
system decisions (palette, typography, spacing, component conventions) so
changes stay on-brand. Every UI change should reference or update it, per
the Hallmark convention of recording design direction in a versioned file.

## State model

**Scenario inputs serialize to the URL:** sites, dates, constraints, selected
view, and the inputs needed to reproduce a comparison. Do not put secrets,
private notes, large blobs, or shared-room credentials in the URL. A live
workspace is separate from a shareable scenario link.

Follow the QuickMap shareable link pattern: `prjExtent`, `proj`, `stack`,
`defs`-style encoded state. Users should be able to copy-paste a URL and
reproduce the exact scenario.

## Interaction model

**Map-first, click-to-compute.** Three inputs stay in sync:
1. Map click
2. Typed lat/lon or site name
3. Curated site list

**Performance:** A result appears within a fraction of a second. Show an
instant approximate answer, then refine in a Web Worker with a visible
"refining" state. Never block the UI thread.

**Heavy work:** Runs in Web Workers from `packages/web`. `packages/core` has
no DOM, no network, no global state, no `Date.now()`.

## Optional AI and collaboration

- Treat AI as an optional operator of the app, not as the source of science
  values. It may translate a request into a small, typed set of app actions and
  explain results returned by the engine.
- Show the structured request and proposed changes. Require the user to
  approve changes to dates, sites, constraints, or the shared scenario.
- Keep actions independent of a particular AI vendor. Do not assume that a
  NASA AI service is available; confirm access and terms before connecting one.
- Show who changed shared state. Treat AI actions as proposals, not as a human
  decision. Provide undo/restore and a visible way to handle conflicting edits.
- AI and live collaboration may require a network service, but neither may be
  required to use the offline science workflow. Never expose provider keys in
  browser code, URLs, or logs.

## Signature visuals

1. **Sun/Earth AZ-EL plot with terrain silhouette** — azimuth/elevation with
   the terrain horizon mask overlaid
2. **Timeline of sunlit and Earth-visible windows with countdowns**
3. **Side-by-side comparison of 2–4 scenarios** with a clear verdict

**Decision features:**
The **slip-window finder** checks a bounded set of candidate sites and dates
against visible user limits. Show meeting, missing, and unknown outcomes, and
make the reason for each result inspectable. The **landing-area robustness
check** shows how results vary across an explicit landing area; disclose its
size, sampling method, and terrain resolution. Do not label sample percentages
as landing probabilities without a justified probability model.

## 3D visualization (Web)

The app uses **Three.js + React Three Fiber (`@react-three/fiber`)** with
`@react-three/drei` for helpful abstractions, running in React + Vite.

**Why Three.js / R3F (not CesiumJS for the core app):**
- QuickMap already provides Cesium-based 3D as a reference/external source —
  this app's 3D is for stylized, interactive visualization of Sun/Earth
  position, terrain silhouette, and horizon mask, not geospatial precision
- Three.js + R3F integrates cleanly with React and Vite, and is easier to
  extend with custom 3D models (lunar lander, terrain panels, Sun/Earth discs)
- Lighter bundle than CesiumJS for a client-side-only, offline-first static
  deployment
- `@react-three/drei` provides `OrbitControls`, `Stars`, `Sphere`, and
  `UseTexture` helpers that speed up a polished lunar globe

**When to consider CesiumJS:**
- If geospatial precision (terrain tiles, WMS/WMTS layers, globe projection)
  becomes a core requirement beyond stylized visualization — then CesiumJS can
  be added as a separate view alongside the Three.js scene

**3D models:**
- Build custom low-poly lunar terrain, lander, and celestial bodies in-house
  (Blender or similar), export as GLTF/GLB
- Keep models low-poly and texture-baked for performance on mobile and
  projectors
- Use `@react-three/drei`'s `<UseTexture>` and `<OrbitControls>` for scene
  interaction
- The Moon is a sphere with a terrain displacement map or a stylized
  low-poly surface; Sun and Earth are emissive spheres with angular radii
  computed from true distance (see `celestial-ephemeris` skill)

**Performance:**
- Render the 3D scene in a Web Worker-isolated canvas when possible, or at
  minimum off the main React render thread
- Target 60 FPS on mobile and projector displays
- Do not load 3D models at runtime — bundle GLB assets or precompute them into
  `data/derived/` (offline-first rule)

## Design review gates (Floor / Reflex / Finish)

Before any UI change is considered done, it must clear these tiers, adapted
from Hallmark's gate system and Impeccable's critique/audit:

**Floor (never ships if failed):**
- Contrast ratios meet WCAG AA (4.5:1) against background
- Keyboard focus indicators visible on every interactive element
- Respects `prefers-reduced-motion` (no forced animation)
- No invented facts or numbers — every displayed value is computed or cited
- No broken mobile layout (usable on a phone in portrait)

**Reflex (defaults a model falls into; may be overruled with a reason):**
- Avoids overused UI patterns (generic cards, rounded-square icon tiles, purple-to-blue gradients)
- No empty frames or hero sections that miss the screen
- No "thanks" without a real action behind the form
- Color palette uses only the defined semantic colors above

**Finish (craft detail, never blocks a ship but should be addressed):**
- Spacing and rhythm consistent across panels
- Font hierarchy clear (headings vs body vs labels)
- Chart text alternatives present
- Hover/focus states on interactive elements

Run an `impeccable`-style slop detector check (62 deterministic rules) on UI
files before commit: flags cramped padding, skipped headings, and other
machine-made tells. No API key required.

## Accessibility

- Colorblind-safe palette (test with protanopia/deuteranopia simulation)
- Full keyboard navigation
- Text alternatives for every chart
- Sufficient contrast, readable on a projector
- Usable on mobile

## Verification action

Include a **"Verify in Horizons"** action that generates the exact Horizons
query used for validation (see `horizons-validation` skill for query format).

## Honest uncertainty

Show surface resolution limits, assumptions, and conventions in the UI. Do not
present false precision. Snap user clicks to the DEM grid and display the
effective resolution.

## Edge-case hardening

Harden the UI against edge cases that are first-class in this domain:

- **Polar sites:** Earth and Sun near the horizon, libration keeps Earth near
  the horizon line at the poles — AZ-EL plot must handle near-zero elevation
- **Eclipse boundaries:** Earth disc overlapping Sun disc — model disc
  overlap, do not ignore it
- **Disc-above-mask fraction:** Sun/Earth partially occluded by terrain —
  show partial visibility, not just center-above/below
- **Overflow:** long site names, many comparison scenarios, wide AZ-EL
  ranges — no clipped text or broken layout
- **Errors:** invalid coordinates, out-of-range dates, missing tiles —
  surface a clear message rather than a wrong number (fail loudly)

## Code example: URL state serialization (conceptual)

```typescript
interface ScenarioState {
  sites: Array<{ lat: number; lon: number; name?: string }>;
  date: string;        // ISO-8601 UTC with Z suffix
  constraints: {
    minSunElevation?: number;
    minEarthElevation?: number;
    requireSunlitEarth?: boolean;
  };
  view: "az-el" | "timeline" | "comparison";
}

(rank sites × dates against user constraints) are core differentiators. The
slip-window finder checks a bounded set of candidate sites and dates against
visible user limits. Show meeting, missing, and unknown outcomes, and make the
reason for each result inspectable. The landing-area robustness check shows
how results vary across an explicit landing area; disclose its size, sampling
method, and terrain resolution. Do not label sample percentages as landing
probabilities without a justified probability model.
function encodeState(state: ScenarioState): string {
  // base64url-encode JSON, similar to QuickMap's `defs` param
}

// Decode from URL
function decodeState(encoded: string): ScenarioState {
  // validate and snap coordinates to DEM grid
}
```

## Failure modes to avoid

- Making runtime network calls to Horizons or QuickMap at demo time
- Using hardcoded results instead of computed values
- Presenting false precision (e.g., sub-meter terrain claims from coarse DEM)
- Ignoring the upper-limb sunrise/sunset convention in UI displays
- Breaking URL shareability (state not reconstructable from link)
- Missing keyboard navigation or chart text alternatives
- Using colors outside the defined semantic palette
- Skipping Floor design review gates before shipping UI changes
