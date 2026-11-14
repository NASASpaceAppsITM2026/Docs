---
name: celestial-ephemeris
version: "1.0.0"
description: |
  Lunar celestial geometry and SPICE/Horizons ephemeris conventions for the
  Lunar Landing Site Navigator. Encodes MOON_ME vs MOON_PA frames, east-positive
  selenographic longitude, ENU local frame, azimuth/elevation conventions,
  SPICE kernel loading, Chebyshev polynomial fits, light-time correction, and
  Sun/Earth disc angular radius computation.
domain: ["astronomy", "space-science", "astrodynamics", "lunar-science"]
work_mode: ["computational", "mission-analysis"]
compatibility:
  python: "3.11+"
  packages: ["spiceypy", "astropy", "numpy", "scipy"]
  network: "offline-first (kernels pre-downloaded; no runtime Horizons calls)"
author: "K-Dense Inc. / project-specific"
license: "MIT"
references:
  - "JPL Horizons Manual v4.98e (2026-08-25): https://ssd.jpl.nasa.gov/horizons/manual.html"
  - "JPL Horizons API v1.3 (2025-06): https://ssd.jpl.nasa.gov/api/horizons.api"
  - "NAIF SPICE Concepts: https://naif.jpl.nasa.gov/naif/spiceconcept.html"
  - "LPI LEAG Mean Earth (ME) Coordinate System white paper"
  - "QuickMap map config: https://quickmap.lroc.im-ldi.com/maps/lroc"
---

# Celestial Ephemeris — Lunar Geometry & SPICE/Horizons Conventions

## Purpose

Teach an AI agent how to compute Sun and Earth celestial positions as seen from a
lunar surface site, using the correct frames, time scales, and SPICE/Horizons
conventions. This skill is project-specific: it encodes the Lunar Landing Site
Navigator's lunar geometry rules, not generic astronomy.

## When to apply

- Computing Sun/Earth azimuth and elevation from a lunar landing site
- Converting between selenographic longitude conventions (0–360° east vs −180–180°)
- Working with SPICE kernels (LSK, PCK, SPK, FK) and metakernels
- Fitting Chebyshev polynomials to Moon→Sun / Moon→Earth state vectors
- Computing angular radii of the Sun and Earth discs from true distance
- Applying light-time correction for apparent positions

## Frames and coordinates (project rules)

**Moon body-fixed frame:** mean-Earth / mean-rotation axis (`MOON_ME`), because
LOLA products and Horizons surface sites use it. Do not mix with the
principal-axis frame (`MOON_PA`) without an explicit, documented conversion.

**Longitude:** east-positive, 0–360° or −180–180° — state which convention at
every API boundary. QuickMap reports `isLongitude360=true` (0–360° east), which
matches Horizons `SITE_COORD` east longitude — no sign flip needed.

**Latitude:** planetocentric/geodetic on the lunar reference sphere; reference
radius **1737.4 km**.

**Local frame:** East-North-Up (ENU) at the site. Azimuth is measured
**clockwise from North** (North=0°, East=90°, South=180°, West=270°). Elevation
is relative to the local horizontal plane (perpendicular to local zenith), not
the terrain slope.

**Time:** internally SPICE ephemeris time (TDB seconds past J2000). UI and
inputs use UTC with explicit ISO-8601 `Z` suffix. Never use naive local times.
Never use JavaScript `Date` for science math.

## SPICE kernels

Load kernels via a metakernel in the correct order:

1. LSK (leap seconds): `naif0012.tls` or current
2. PCK (planetary constants): lunar orientation + radii
3. SPK (ephemeris): DE440-family planetary ephemeris + lunar ephemeris
4. FK (frames): Moon ME/PA frame definitions

Pin exact filenames and versions in `docs/DATA.md`. Verify on NAIF before use.

## Ephemeris computation

Sun and Earth directions are **apparent** (light-time and stellar aberration
corrected), expressed in `MOON_ME`, then rotated into the site's ENU frame.

Runtime ephemeris is a set of **Chebyshev polynomial fits** of Moon→Sun and
Moon→Earth vectors over the supported window. Fit error budget must be recorded
in `docs/VALIDATION.md` (target: well under 0.001° in direction).

## Discs, not points

Sun and Earth are finite discs. Compute angular radii from true distance and
body radii — never hardcode. Report visibility as **fraction of disc above the
horizon mask**, not just center elevation.

## Code examples

### Defining a lunar surface observing site for Horizons
```
CENTER='coord@301'&COORD_TYPE='GEODETIC'&SITE_COORD='<east_lon>,<lat>,<alt_km>'
```
- `east_lon`: east longitude in degrees (0–360)
- `lat`: latitude in degrees
- `alt_km`: altitude above reference ellipsoid in km

### Horizons API request (validation only — no runtime calls)
```
https://ssd.jpl.nasa.gov/api/horizons.api?
  format=json&
  OBJ_DATA=YES&MAKE_EPHEM=YES&
  EPHEM_TYPE=OBSERVER&
  COMMAND='10'&
  CENTER='coord@301'&COORD_TYPE='GEODETIC'&SITE_COORD='291.734,2.492,0'&
  START_TIME='2026-11-13'&STOP_TIME='2026-11-14'&STEP_SIZE='1h'&
  QUANTITIES='1,4'&CSV_FORMAT=YES&TIME_DIGITS=FRACSEC
```
- `COMMAND='10'`: Sun; `COMMAND='399'`: Earth; `COMMAND='301'`: Moon center
- Quantity `4`: azimuth/elevation (topocentric only)
- Always check JSON `"signature":{"version":...}` (responses carry version `1.0`)

## Validation

- Golden tests vs JPL Horizons: Apollo sites + south polar sites, multiple dates
- Target agreement: **< 0.01°** on flat-horizon AZ/EL (not terrain-masked)
- CI must not call Horizons; commit small response fixtures in `tests/fixtures/`
- Record query parameters in each fixture

## Failure modes to avoid

- Mixing `MOON_ME` and `MOON_PA` without conversion
- Comparing TEME output to GCRF/ITRF without frame transformation
- Using stale TLEs or wrong epoch for science math
- Hardcoding angular radii instead of computing from true distance
- Using JavaScript `Date` for TDB/TT computations
