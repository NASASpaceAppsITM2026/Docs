# Lunar Review Checklist — Astrodynamicist

Distilled from the K-Dense Astrodynamicist AGENTS.md profile for lunar landing site navigation review. Use this when reviewing ephemeris, frame, and trajectory PRs. Load the full profile on demand for deep dives.

## Before approving any ephemeris PR, verify:

### Frames & time (non-negotiable)
- [ ] Moon body-fixed frame is `MOON_ME` (mean-Earth / mean-rotation axis), NOT `MOON_PA` (principal axis) — unless an explicit, documented conversion is present
- [ ] Longitude is east-positive (0–360° or −180–180°), stated at every API boundary — matches QuickMap `isLongitude360=true`
- [ ] Time internally uses TDB seconds past J2000; UI uses UTC with ISO-8601 `Z` suffix; never JavaScript `Date` for science math
- [ ] Surface site uses `CENTER='coord@301'` (not `coord` alone, which resolves to Earth)
- [ ] Chebyshev fits use **aberrated (apparent)** direction, not geometric — stellar aberration (~20″ / ~0.006°) is more than half the 0.01° tolerance budget
- [ ] Lunar orientation model window verified against NAIF docs (AGENTS.md claims ~1550–2650, marked TODO(verify))

### Validation
- [ ] Golden tests vs JPL Horizons fixtures at `< 0.01°` AZ/EL agreement
- [ ] Fixtures use `CSV_FORMAT=YES`, `TIME_DIGITS=FRACSEC`, quantities `1` and `4` only (not `9`)
- [ ] Fixtures avoid exactly −90° latitude (azimuth degenerate) — use −89.5°
- [ ] Fixtures spread across the date window, not just one date pair
- [ ] CI does not call Horizons; fixtures are committed to `tests/fixtures/horizons/`
- [ ] JSON `signature.version` checked on every response (currently `1.0`)

### Ephemeris correctness
- [ ] Light-time and stellar aberration corrections applied (apparent position)
- [ ] Angular radii computed from true distance and body radii — never hardcoded
- [ ] Disc-above-mask fraction computed, not just center elevation
- [ ] Fit error budget recorded in `docs/VALIDATION.md` (target: well under 0.001° in direction)

### Failure modes to reject
- [ ] No mixing `MOON_ME` and `MOON_PA` without explicit conversion
- [ ] No comparing TEME output to GCRF/ITRF without frame transformation
- [ ] No stale TLEs or wrong epoch for science math
- [ ] No hardcoded angular radii
- [ ] No JavaScript `Date` for TDB/TT computations

**If any check fails: do not approve. Fix or document in `docs/VALIDATION.md` before proceeding.**
