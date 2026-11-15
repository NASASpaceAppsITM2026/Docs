# Hackathon Sprint Plan — Lunar Landing Site Navigator

**Event:** 2026 NASA Space Apps Challenge  
**Timebox:** 48 hours (2 days)  
**Judging priorities:** 1) Scientific validity, 2) Mission-planning relevance (launch-slip), 3) Creativity, 4) Presentation

## Core rule

**Correctness before features.** A working demo with validated Sun/Earth positions beats a polished app with made-up numbers. If we can't validate it against Horizons, we don't ship it.

---

## Day 1 — Core science + validation (hours 0–24)

### Morning (hours 0–8): Pipeline + science core

| Task | Time | Output |
|------|------|--------|
| Scaffold repo (`docs/`, `pipeline/`, `packages/core/`, `packages/web/`, `tests/fixtures/`) | 1 hr | Empty skeleton, typecheck passes |
| Python pipeline: pin SPICE kernels + download script | 2 hrs | `docs/DATA.md` with kernel names/versions/checksums; kernels cached in `data/raw/` |
| Python pipeline: build Chebyshev fits for Moon→Sun / Moon→Earth | 3 hrs | Precomputed coefficient files in `data/derived/` |
| TypeScript core: load Chebyshev coefficients, compute apparent positions (light-time corrected) | 2 hrs | `packages/core` ephemeris module |
| Frame discipline: `MOON_ME` → ENU, azimuth clockwise from North | 1 hr | Frame conversion tested |
| Disc angular radii from true distance (never hardcode) | 1 hr | `discRadius(distance, bodyRadius)` |

**End-of-morning check:** `pnpm -r typecheck` passes; science functions return sensible numbers for a known site (e.g., Apollo 11).

### Afternoon (hours 8–16): Validation against Horizons

| Task | Time | Output |
|------|------|--------|
| Fetch Horizons fixtures (offline-first): Apollo sites + south polar sites | 2 hrs | `tests/fixtures/horizons/*.json` with full query URLs recorded |
| Golden test: compare our AZ/EL to Horizons flat-horizon values | 3 hrs | `test:golden` passes at `< 0.01°` |
| If mismatch > 0.01°: debug frame/time-scale/light-time (do not loosen tolerance) | 2 hrs | Fixed or documented in `docs/VALIDATION.md` |

**End-of-afternoon check:** `pnpm --filter core test:golden` passes offline. We have independently validated numbers.

### Evening (hours 16–24): Terrain horizon mask + first metrics

| Task | Time | Output |
|------|------|--------|
| **Terrain horizon mask: ray-marching over 360 azimuths from LOLA DEM** | 4 hrs | `horizonMask()` function, DEM tiling |
| Power proxy: irradiance (1/r²) × cosine incidence × visible Sun disc fraction | 2 hrs | `powerProxy()` function |
| Direct-to-Earth window: Earth visible above horizon mask | 2 hrs | `dteWindow()` function |
| Eclipse disc-overlap (Earth blocking Sun) | 1 hr | `eclipseOverlap()` function |
| **Date sweep: same site, ±N days heatmap** (launch-slip judging priority #2) | 2 hrs | `dateSweep()` function + UI-ready data |
| ~~Longest continuous darkness placeholder~~ | ~~1 hr~~ | ~~Stub with TODO~~ — dropped, not needed for MVP |

**End-of-day check:** Core science library computes Sun/Earth position, power, DTE, eclipse, horizon mask, and date sweep — all validated against committed Horizons fixtures. No UI yet, but the numbers are real.

---

## Day 2 — UI + demo + submission (hours 24–48)

### Morning (hours 24–32): Minimal UI slice

| Task | Time | Output |
|------|------|--------|
| React + Vite + Tailwind v4 scaffold | 1 hr | `packages/web` dev server runs |
| Deep-space palette from `DESIGN.md` | 1 hr | Colors applied |
| Moon-centered 3D opening scene with Moon, Earth, Sun, and selected site | 4 hrs | Main app view, wired to mocked or validated core results |
| Secondary map, coordinate entry, and saved-site selector | 2 hrs | Precise site selection stays synchronized with the 3D marker |
| AI chat panel at bottom: input, action proposals, approve/dismiss | 2 hrs | Natural-language scenario control |
| URL-serializable state (shareable link) | 1 hr | Copy-paste URL reproduces scenario |
| "Verify in Horizons" action (generates exact query) | 1 hr | Button that outputs query string |
| Snap clicks to DEM grid; display effective resolution | 1 hr | Honest uncertainty in UI |

**End-of-morning check:** The Moon is the main view, Earth and Sun provide site/date context, and users can select a site from the secondary map or coordinate controls. Core results and the scenario URL work offline.

### Afternoon (hours 32–40): Planning features + visual refinement

| Task | Time | Output |
|------|------|--------|
| Slip-window finder: candidate sites and dates against visible user limits | 3 hrs | Results show pass/fail/unknown and why |
| Timeline of sunlit/Earth-visible windows with countdowns | 1 hr | Chart SVG/Canvas |
| Side-by-side comparison of 2 scenarios | 1 hr | Comparison view |
| Refine the 3D scene: overview/site camera, terrain horizon, celestial discs | 1 hr | Keep the 3D scene primary; trim decorative detail if time is short |
| Accessibility: keyboard nav, chart text alternatives, colorblind-safe | 1 hr | Passes Floor gate |
| Design review gates (Floor/Reflex/Finish) | 1 hr | No AI-slop patterns |

**End-of-afternoon check:** App is demoable end-to-end: select a site → inspect the Moon-centered 3D scene and metrics → compare scenarios → share URL. Offline core confirmed (airplane-mode test).

### Evening (hours 40–48): Submission

| Task | Time | Output |
|------|------|--------|
| Final validation report: `docs/VALIDATION.md` with before/after numbers | 1 hr | Documented accuracy |
| `docs/DECISIONS.md`: key ADRs (Three.js vs Cesium, tailwind v4, upper-limb convention) | 1 hr | Decision log |
| README: how to run, how to validate, judging criteria mapping | 1 hr | Submission-ready |
| Build static assets (`pnpm --filter web build`) | 1 hr | Deployable bundle |
| Airplane-mode test: verify no runtime network calls | 1 hr | Confirmed offline |
| Pitch deck / demo script (3 min) | 2 hrs | Presentation ready |

**End-of-hackathon:** Submitted. Working demo, validated science, documented decisions.

---

## MVP scope (what we WILL build)

| In scope | Later or stretch scope |
|----------|------------------------|
| Validated single-site Sun/Earth geometry and terrain-aware metrics | Full south-polar site catalog |
| Moon-centered 3D scene with Moon, Earth, Sun, selected site, and a site-level horizon view | High-detail custom lander model or exact-scale three-body distances |
| Slip-window finder across a bounded set of candidate sites and dates, using visible user limits | Landing-area robustness across a user-defined radius or landing ellipse; only after the point results are validated |
| Power proxy, configurable direct-to-Earth visibility, and eclipse disc-overlap | Multi-day slip animation and real-time terrain editing |
| Horizon mask from precomputed LOLA DEM | CesiumJS geospatial precision |
| AI chat panel: natural-language input for scenario changes | User accounts or a hosted backend |
| URL shareable links that reproduce scenario inputs | Landing-area robustness across a user-defined radius or landing ellipse; only after the point results are validated |
| "Verify in Horizons" action; no live Horizons calls required | CesiumJS geospatial precision |
| Clear assumptions, limitations, and pass/fail/unknown results | Mutation testing and full QA suite |

The AI chat is a core MVP feature: it translates natural-language requests into
scenario changes and explains results. It never calculates scientific values —
every number comes from the deterministic engine. Manual controls remain
available for fine-tuning.

---

## Quality gates (compressed for hackathon)

| Gate | Standard | When |
|------|----------|------|
| Typecheck | Zero errors | Before each demo |
| Golden tests | `< 0.01°` vs Horizons fixtures | After Day 1 afternoon |
| Offline test | Core workflow works in airplane mode; optional online features are unavailable without blocking it | Day 2 afternoon |
| Design review Floor gate | Contrast WCAG AA, keyboard nav, no invented facts | Before submission |
| Validation report | `docs/VALIDATION.md` with actual numbers | Before submission |

**Skip for hackathon:** mutation testing, property-based tests, full QA suite, CI/CD pipeline, multi-site catalog, user auth.

---

## If we fall behind (triage order)

1. **Cut decorative 3D detail and polish** first; preserve the Moon-centered scene as the primary interface
2. **Cut landing-area robustness** if its terrain sampling or landing distribution cannot be justified
3. **Reduce the candidate-site set or date range** rather than hide how the slip finder reaches its results
4. **Never cut:** validation against Horizons, the bounded slip-window finder, offline core use, or honest uncertainty

---

## What NOT to do

- Do not invent coordinates, dates, or mission facts — write `TODO(verify)` and ask
- Do not loosen the `< 0.01°` tolerance to make tests pass
- Do not require runtime network calls to Horizons, QuickMap, or NASA for core calculations
- Do not let the AI calculate scientific values — every number comes from the deterministic engine
- Do not spend time on light/dark mode toggle, animations, or non-essential polish
- Do not present false precision (sub-meter claims from coarse DEM)
- Do not claim that no competing tool offers a feature without checking its current workflow
- Do not imply direct user research has been done until feedback has been collected and recorded
