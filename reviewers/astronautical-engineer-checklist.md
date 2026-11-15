# Lunar Review Checklist — Astronautical Engineer

Distilled from the K-Dense Astronautical Engineer AGENTS.md profile for lunar landing site navigation review. Use this when reviewing system architecture, V&V, and mission-design PRs. Load the full profile on demand for deep dives.

## Before approving any system/mission PR, verify:

### Architecture & V&V
- [ ] Frame/unit ICD mismatches checked (km vs m, deg vs rad, UTC vs TAI)
- [ ] Frame/epoch/time-scale consistency across all components
- [ ] NASA-STD V&V principles applied (analysis-test-similarity hierarchy)
- [ ] GMAT/STK/SPICE mission design conventions referenced where applicable
- [ ] No light/dark mode toggle, animations, or non-essential polish before core science validated

### Offline-first architecture
- [ ] Core calculations do not require runtime calls to Horizons, NASA, or QuickMap
- [ ] All ephemeris and terrain data precomputed and shipped as static assets
- [ ] Core site/date workflow works offline; AI provider failures do not block manual controls
- [ ] Python pipeline used only at build-time (kernel download, Chebyshev fits, DEM processing), never at runtime

### Mission-planning relevance (judging priority #2)
- [ ] Slip-window finder checks documented candidate sites, dates, and visible user limits
- [ ] Pass/fail/unknown results explain which constraints were met or missing
- [ ] Landing-area robustness states the checked area, sampling method, and terrain resolution
- [ ] Power proxy: solar irradiance (1/r²) × cosine incidence × visible disc fraction
- [ ] Direct-to-Earth window: Earth visible above terrain mask (configurable min elevation + antenna height)
- [ ] Eclipse disc-overlap modeled (Earth blocking Sun), not ignored
- [ ] Longest continuous darkness considered for battery/thermal survival

### Error handling
- [ ] Fail loudly on out-of-range dates, invalid coordinates, missing tiles
- [ ] Clear error messages in UI rather than wrong numbers
- [ ] No silently caught and swallowed errors in science code

### Reproducibility
- [ ] Kernel versions and checksums recorded in `docs/DATA.md`
- [ ] DEM product names and resolutions recorded in `docs/DATA.md`
- [ ] Every derived data file rebuildable from documented script + pinned inputs
- [ ] `docs/VALIDATION.md` updated with before/after numbers for any science change

### AI chat
- [ ] AI actions are allow-listed, visible, and require human approval before changing scenario state
- [ ] AI explanations use values returned by the deterministic engine; no AI-generated science values
- [ ] AI provider is optional, with data handling and access controls documented
- [ ] Chat panel is keyboard-accessible with text alternatives
- [ ] Approved AI changes can be undone manually

**If any check fails: do not approve. Fix or document in `docs/VALIDATION.md` before proceeding.**
