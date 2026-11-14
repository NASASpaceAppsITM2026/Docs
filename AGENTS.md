# AGENTS.md — Lunar Landing Site Navigator

Instructions for any AI coding agent working in this repository. Read this fully before making changes.
Direct instructions from the human in chat override this file. If a rule here conflicts with a task, say so and ask.

## 1. Project in one paragraph

A web app that lets mission planners, educators and the public evaluate lunar landing sites (focus: south pole) for a given date: where the Sun and Earth are relative to the local horizon (terrain included), how much power is available, when direct-to-Earth communication is possible, and what happens if the date or site changes. The user clicks a lunar map, types coordinates, or picks a known site, and gets results **instantly and fully client-side**. Built for the 2026 NASA Space Apps Challenge "Lunar Landing Site Navigator". The challenge also references NASA Moon Trek (3D terrain/horizon visualization reference) and the CSA Moon Exploration portal (mission/Artemis context); these are secondary sources, not part of the core ephemeris/terrain computation.

Judging priorities, in order: **scientific validity**, relevance to mission planning (launch-slip decisions), creativity, presentation. When in doubt, choose correctness over polish.

## 2. Non-negotiable principles

1. **Correctness before features.** Never ship or claim a number that is not validated against an independent reference (JPL Horizons, published LOLA illumination results, Quickmap). Cut scope before cutting validation.
2. **Never invent facts.** Do not guess dataset names, file names, URLs, SPICE kernel filenames, API parameters, coordinates of landing sites, or mission dates. If you cannot verify it from an authoritative source in this session, write `TODO(verify): ...` and tell the human. Authoritative sources: NASA PDS Geosciences Node (LOLA), NAIF (SPICE kernels and docs), JPL Horizons manual and API docs, Lunar QuickMap (https://quickmap.lroc.im-ldi.com/; the old https://quickmap.lroc.asu.edu/ host is retiring), mission agency pages.
3. **Offline-capable core; optional online features.** The deployed core is a static app: science calculations use precomputed assets and must not require runtime calls to Horizons, NASA, QuickMap, or an AI provider. A network failure must not break core site/date analysis. Live collaboration and an optional AI assistant may use separately documented online services, only with the user's knowledge; the offline core and shareable scenario links must remain usable without them.
4. **Reproducible.** Every derived data file must be rebuildable from a documented script plus pinned inputs (versions and checksums recorded in `docs/DATA.md`).
5. **Honest uncertainty.** Surface resolution limits, assumptions, and conventions in the UI. Do not present false precision.
6. **Small, verifiable steps.** Prefer a working vertical slice over broad scaffolding.

## 3. Repository layout (target)

```
AGENTS.md
docs/
  SCIENCE.md       conventions, frames, formulas, references (source of truth for science)
  DATA.md          dataset provenance, versions, checksums, licenses
  VALIDATION.md    validation method and latest results
  DECISIONS.md     short ADR log; append, do not rewrite history
pipeline/          Python: kernel handling, Chebyshev fits, DEM tiling, derived rasters
packages/core/     TypeScript: pure science library (no DOM, no network)
packages/web/      TypeScript + Vite: the app
data/raw/          downloaded inputs (gitignored, fetched by script)
data/derived/      generated assets consumed by the app (large files via LFS/DVC or release assets, not plain git)
tests/fixtures/    small committed reference outputs (e.g. cached Horizons responses)
```

If the repo does not match this yet, follow it when adding files and note deviations in `docs/DECISIONS.md`.

## 4. Commands

Keep this section accurate. When you add or change tooling, update these lines in the same change. Do not leave stale commands.

```
# Python pipeline (uv)
uv sync
uv run pytest
uv run ruff check pipeline tests scripts
uv run ruff format --check pipeline tests scripts

# TypeScript (pnpm workspaces, app repo)
pnpm install
pnpm -r typecheck
pnpm -r lint
pnpm -r test
pnpm --filter web dev
pnpm --filter web build
pnpm --filter core test:golden
```

Before declaring a task done: typecheck, lint, and tests pass, and the app builds. Run what you changed; do not claim tests passed without running them.

## 5. Science conventions (authoritative; details in `docs/SCIENCE.md`)

Violating these silently produces plausible-looking wrong answers. Follow them exactly.

**Frames and coordinates**
- Moon body-fixed frame: **mean-Earth / mean-rotation axis** (SPICE `MOON_ME`), because LOLA products and Horizons surface sites use it. Do not mix with the principal-axis frame (`MOON_PA`) without an explicit, documented conversion.
- Longitude is **east-positive**, 0–360° or −180–180° (state which at every API boundary). Latitude is planetocentric/geodetic on the lunar reference sphere; reference radius **1737.4 km**. Confirm the convention each dataset uses before combining data.
- Local frame: East-North-Up at the site. **Azimuth is measured clockwise from North**; elevation is relative to the local horizontal plane (perpendicular to local zenith), not the terrain slope.
- Horizon elevation at a given azimuth is the maximum of: terrain skyline angle, and the curvature horizon of the sphere (dip from site height). There is **no atmosphere**: no refraction anywhere.

**Time**
- Internally use SPICE ephemeris time (TDB seconds past J2000). UI and inputs use UTC with an explicit ISO-8601 `Z` suffix. Never use naive local times. Never use JavaScript `Date` for science math; use integer/float epoch values from the core library.
- The high-precision lunar orientation model is valid only for roughly 1550–2650 (TODO(verify): confirm exact window from NAIF/SPICE kernel documentation). Restrict supported dates to the shipped ephemeris window and fail loudly outside it.

**Ephemeris**
- Source: SPICE kernels from NAIF (DE440-family planetary ephemeris plus lunar orientation kernels). Verify exact filenames on NAIF and pin them in `docs/DATA.md`.
- Sun and Earth directions are **apparent** (light-time and stellar aberration corrected), expressed in `MOON_ME`, then rotated into the site's ENU frame. Match the conventions Horizons uses when we validate against it.
- Runtime ephemeris is a set of **Chebyshev polynomial fits** of Moon→Sun and Moon→Earth vectors over the supported window. Fit the **aberrated** (apparent) direction, not the geometric one — stellar aberration from the Moon's heliocentric motion is about 20″ (~0.006°), more than half the 0.01° tolerance budget if omitted. Fit error budget must be recorded in `docs/VALIDATION.md` (target: well under 0.001° in direction).

**Pinned Horizons parameters for this project**
- API: `https://ssd.jpl.nasa.gov/api/horizons.api` (API doc v1.3, 2025-06; manual v4.98e, 2026-08-25 at `https://ssd.jpl.nasa.gov/horizons/manual.html`)
- Target IDs: Sun=`10`, Earth=`399`, Moon=`301`, Moon body center=`500@301`
- Surface site: `CENTER='coord@301'`, `COORD_TYPE='GEODETIC'`, `SITE_COORD='<east_lon>,<lat>,<alt_km>'` (east longitude degrees, 0–360; altitude km above reference ellipsoid). Non-Earth bodies MUST use the `@` form (`coord@301`, not `coord`); `coord` alone would resolve to an Earth location.
- `EPHEM_TYPE=OBSERVER`; observer quantity codes: `1`=astrometric RA/DEC, `2`=apparent RA/DEC, `4`=azimuth/elevation (topocentric only), `9`=visual magnitude/surface brightness
- Longitude: east-positive degrees — matches QuickMap `isLongitude360=true`, so no sign flip at the API boundary
- Validate every JSON response: check `"signature":{"version":...}` (responses currently carry version `1.0`) to detect format drift

**Discs, not points**
- Sun and Earth are finite discs. Compute angular radii from true distance and body radii (never hardcode). Report visibility as **fraction of disc above the horizon mask**, not just center elevation.
- Always state the sunrise/sunset convention used: upper limb vs disc center. Default for operational metrics: **upper limb**. Expose the convention in the UI and API.

**Events and windows**
- Find sunset, sunrise, Earthset, Earthrise and eclipse boundaries with **root-finding** on a continuous function (e.g. bisection/Brent on `elevation − horizon(azimuth)`), not by thresholding a sampled grid. Sampling is only for bracketing and plotting.
- Countdowns ("time until sunset / Earthset") derive from these exact events.

**Metrics**
- Power proxy = solar irradiance (scaled by 1/r² from distance) × cosine of incidence on a configurable panel orientation × visible fraction of the Sun disc. Document the assumed constants.
- Direct-to-Earth window = Earth visible above the mask (with configurable minimum elevation and antenna height). Also compute "Earth visible AND sunlit".
- Longest continuous darkness is a first-class metric (battery/thermal survival).
- Earth blocking the Sun (lunar eclipse) must be modeled as a disc overlap; do not ignore it.

**Terrain**
- Horizon masks come from LOLA digital elevation models, ray-marched over 360 azimuths with sphere curvature drop included. Include observer height as a parameter. Verify product names and resolution on PDS before use, and record them in `docs/DATA.md`.
- DEM resolution limits real accuracy. Snap user clicks to the DEM grid and display the effective resolution.

**Key data sources (observed, undocumented — may change)**
| Source | Endpoint | Use | Status |
|--------|----------|-----|--------|
| Lunar QuickMap map config | `https://quickmap.lroc.im-ldi.com/maps/lroc` | Layer/projection/terrain config; `isLongitude360=true` | Observed, undocumented, may change |
| QuickMap tiles | `https://lroc-tiles.quickmap.io/tiles/{layer}/{projection}/{z}/{x}/{y}.jpg` | Raster imagery (WAC/NAC mosaics) | Observed, undocumented, may change |
| QuickMap terrain (Cesium heightmap) | `https://files.actgate.com/terrain/lroc/cz_cached_qts_demstack` | Terrain elevation | Observed, undocumented, may change |
| QuickMap Terrain Shadows (QTS) | `https://qts.quickmap.io/fcgi-bin/fprovweb.exe?_xtype=text/html&target=MOON&cmd_script=qts_js` | Illumination/shadow reference | Observed, undocumented, may change |
| JPL Horizons API | `https://ssd.jpl.nasa.gov/api/horizons.api` | Ephemeris (validation reference only — no runtime calls) | Documented, stable |
| JPL Horizons manual | `https://ssd.jpl.nasa.gov/horizons/manual.html` | Parameter and quantity-code documentation | Documented, stable |

**Secondary sources (context only, not for ephemeris/terrain computation)**
- NASA Moon Trek (`https://trek.nasa.gov/moon/`): 3D terrain/horizon visualization reference
- CSA Moon Exploration portal (`https://www.asc-csa.gc.ca/eng/astronomy/moon-exploration/`): mission context, Artemis alignment, Canadian lunar technology contributions

## 6. Validation requirements

A science change is not complete without validation evidence.

- **Golden tests vs JPL Horizons**: for Apollo sites plus randomly sampled points (including several south polar sites at **−89.5° latitude, never exactly −90°** where azimuth is degenerate) and **several dates spread across the supported window** (not just 2026-11-13/14), compare our Sun and Earth azimuth/elevation to Horizons observer tables (apparent, airless, surface site on body 301, targets 10 and 399). Horizons reports elevation relative to the plane perpendicular to the local zenith, so compare against our **flat-horizon** values, not the terrain-masked ones. Request `CSV_FORMAT=YES` and `TIME_DIGITS=FRACSEC` (default precision is minutes, insufficient for validation). Request only quantity `4` (azimuth/elevation) and `1` (astrometric RA/DEC if useful), **not** quantity `9` (magnitude). Horizons reports elevation relative to the plane perpendicular to the local zenith, so compare against our **flat-horizon** values, not the terrain-masked ones.
- Target agreement: **< 0.01°**. If we miss, find the cause (frame, time scale, light-time, site definition); do not loosen the tolerance without recording why in `docs/VALIDATION.md`.
- **CI must not call Horizons.** Fetch once with a script, commit the small response fixtures in `tests/fixtures/`, and test against those. Record the query parameters in each fixture.
- Terrain/illumination results are cross-checked against Quickmap and published LOLA illumination studies at known sites. Also cross-check terrain shadows against QuickMap Terrain Shadows (QTS, linked from the QuickMap map config). Record the comparison.
- Property tests: azimuth wraps correctly, elevation in [−90°, 90°], events are monotone-ordered, disc fraction in [0, 1], results are invariant to unit-preserving refactors.
- Never edit a test threshold, delete a failing test, or regenerate a fixture just to make a build pass. Explain the discrepancy to the human instead.

## 7. Engineering conventions

- **Languages**: TypeScript (strict) for `packages/*`, Python 3.11+ for `pipeline/`. The science core is TypeScript first. Move a hot path (e.g. horizon-mask ray-marching) to Rust/WASM only after profiling shows it is needed, behind the same interface. Record the decision in `docs/DECISIONS.md`.
- **Styling**: Tailwind CSS v4 + Vite (see `docs/frontend-conventions/SKILL.md` for installation and conventions). Dark mode by default (deep-space theme); no light toggle unless required.
- **Design system**: `docs/frontend-conventions/DESIGN.md` records the visual system (palette, typography, spacing, components). Every UI change should reference or update it.
- **Design review gates**: Floor / Reflex / Finish (see `docs/frontend-conventions/SKILL.md`). Floor gates never ship; Reflex gates may be overruled with a reason; Finish gates are craft detail.
- **Units in names**: `elevationDeg`, `distanceKm`, `epochTdbSec`. Radians only inside tight math helpers, converted at boundaries. No bare numbers with ambiguous units.
- **Pure core**: `packages/core` has no DOM, no network, no global state, no `Date.now()`. Same inputs, same outputs. Heavy work runs in Web Workers from `packages/web`.
- **No magic constants**: every physical constant lives in one module with a source comment and units.
- **State is shareable**: scenario inputs (sites, dates, constraints, selected view) serialize to a URL. Do not put secrets, private notes, or large blobs in the URL. Live shared workspaces are a separate optional online feature.
- **AI is an optional operator, not a science engine**: expose a small, typed set of app actions. An AI may propose actions and summarize engine results, but may not calculate scientific metrics or apply scenario-changing actions without clear user approval. Keep the action interface independent of any one AI provider; do not assume NASA AI access exists.
- **Team changes are accountable**: if live collaboration is implemented, identify the person or assistant responsible for each change, make AI changes reviewable, and support undo or restore. Document service access, data handling, and offline behavior before implementation.
- **Data**: never commit large binaries to plain git. Never edit `data/raw/`. Derived files are produced only by scripts in `pipeline/`.
- **Dependencies**: justify new dependencies in the PR description. Prefer small, maintained libraries. Pin versions. Check licenses.
- **Errors**: fail loudly on out-of-range dates, invalid coordinates, or missing tiles; surface a clear message in the UI rather than a wrong number.
- **Style**: follow existing formatters and linters; do not reformat unrelated code. Comments explain *why* and cite sources for science.

## 8. UI and UX principles

- Map-first, click-to-compute. Three inputs stay in sync: map click, typed lat/lon or site name, curated site list.
- A result appears within a fraction of a second. Show an instant approximate answer, then refine in a worker with a visible "refining" state. Never block the UI thread.
- Signature visuals: Sun/Earth azimuth-elevation plot with the terrain silhouette; a timeline of sunlit and Earth-visible windows with countdowns; side-by-side comparison of 2–4 scenarios with a clear verdict.
- The slip-window finder (candidate sites × dates checked against visible user constraints) and landing-area robustness check (how results vary across an explicit landing area) are the main decision features; protect their validation and make failed or indeterminate results understandable.
- Shareable links are the first collaboration feature. Live multi-person workspaces and AI operation are optional online layers and must not be prerequisites for core use.
- AI can translate requests into visible, reviewable app actions and explain returned results. It must not supply or alter computed science values. Ask before adding an external or paid provider.
- Accessibility is required: colorblind-safe palette, full keyboard navigation, text alternatives for every chart, sufficient contrast, readable on a projector, usable on mobile.
- Show assumptions and a "Verify in Horizons" action that generates the exact query used for validation.
- Copy must be understandable by a non-expert, with technical detail one click deeper.

## 9. How to work in this repo

1. Read `docs/SCIENCE.md`, `docs/DECISIONS.md`, and the code you will touch before changing anything.
2. State a short plan for non-trivial work, then implement in small increments with tests.
3. Prefer the smallest change that solves the task. Do not refactor, rename, or reformat unrelated code. Do not add features that were not requested.
4. When science is involved: cite the source in code comments or `docs/SCIENCE.md`, add or update golden tests, and update `docs/VALIDATION.md` with results.
5. When you make an architectural or tooling decision, append a short entry to `docs/DECISIONS.md` (context, decision, consequence).
6. If requirements are ambiguous or a verification step is impossible in your environment, stop and ask or leave a clearly marked `TODO(verify)`. Do not fill the gap with an assumption.
7. Report honestly: what you changed, what you ran, what passed, what you could not verify.

**Ask the human first before**: changing the frame, time-scale, or sunset convention; adding a runtime network dependency; swapping the DEM or ephemeris source; loosening a validation tolerance; deleting data or history; adding a paid or external service.

**Never**: commit secrets or API keys; fabricate data or citations; hardcode results to satisfy a test; silently catch and swallow errors in science code; use browser storage for anything other than non-critical per-user preferences.

## 10. Git and review

- Branch per task. Commits follow Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `chore:`).
- Each pull request states: what changed, how it was validated, known limitations. Science changes include before/after validation numbers.
- Keep PRs focused and small enough to review in minutes.

## 11. Glossary

- **ENU**: East-North-Up local frame at a surface point.
- **Mask / horizon mask**: terrain elevation angle as a function of azimuth at a site.
- **Earthset / Earthrise**: Earth crossing the local horizon as seen from the Moon (libration keeps Earth near the horizon at the poles).
- **DTE**: direct-to-Earth communication.
- **CLPS**: NASA Commercial Lunar Payload Services.
- **LOLA**: Lunar Orbiter Laser Altimeter (source of lunar topography).
- **SPICE / NAIF**: NASA's ephemeris toolkit and the group that publishes the kernels.
- **Horizons**: JPL's ephemeris service, used here as the independent validation reference.
- **QuickMap / Lunar QuickMap**: NASA LRO/LROC lunar web map (https://quickmap.lroc.im-ldi.com/), source of imagery, DEM, and terrain data; reports `isLongitude360=true`.
- **QTS / QuickMap Terrain Shadows**: illumination and shadow analysis tool linked from QuickMap's map config, used as a cross-check for terrain shadow results.

## Reviewer checklists

Lunar-specific review checklists distilled from K-Dense expert profiles. Load on demand for science review (e.g., "run the astrodynamicist checklist on the ephemeris PR"):

- `docs/reviewers/astrodynamicist-checklist.md` — frames, time scales, kernels, ephemeris validation
- `docs/reviewers/astronomer-checklist.md` — coordinate systems, Horizons query parameters, output interpretation
- `docs/reviewers/astronautical-engineer-checklist.md` — architecture, V&V, offline-first, mission-planning relevance

Full K-Dense expert profiles (~55 KB combined) are intentionally **not** loaded into AGENTS.md to avoid bloating session context. They contain off-topic content (LEO drag, TLEs, rocket equation) and unverifiable claims that conflict with the no-invention rule. Move them to `docs/reviewers/` if needed for deep dives.
