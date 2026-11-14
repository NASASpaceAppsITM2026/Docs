---
name: horizons-validation
version: "1.0.0"
description: |
  Validation procedure for the Lunar Landing Site Navigator ephemeris engine
  against JPL Horizons. Encodes API query construction, observer quantity codes,
  golden test fixtures, <0.01° tolerance, signature.version checking, and the
  offline-first fixture rule (CI must not call Horizons).
domain: ["validation", "astrodynamics", "ephemeris", "testing"]
work_mode: ["computational", "quality-assurance"]
compatibility:
  python: "3.11+"
  packages: ["pytest", "requests", "numpy"]
  network: "fetch-once-then-commit (no live network in CI)"
author: "K-Dense Inc. / project-specific"
license: "MIT"
references:
  - "JPL Horizons API v1.3: https://ssd.jpl.nasa.gov/api/horizons.api"
  - "JPL Horizons Manual v4.98e: https://ssd.jpl.nasa.gov/horizons/manual.html"
---

# Horizons Validation — Golden Test Procedure

## Purpose

Teach an AI agent how to validate the project's ephemeris computations against
JPL Horizons as the independent reference. This skill encodes the exact query
construction, tolerance thresholds, fixture management, and offline-first CI
rule required by the Lunar Landing Site Navigator.

## When to apply

- Validating Sun/Earth azimuth/elevation from lunar surface sites
- Adding new golden test fixtures
- Checking tolerance agreement (< 0.01°)
- Verifying JSON signature.version on API responses
- Ensuring CI does not make live network calls to Horizons

## Horizons API query construction

**Endpoint:** `GET https://ssd.jpl.nasa.gov/api/horizons.api`

**Parameters for lunar surface observer ephemeris:**
| Parameter | Value | Meaning |
|---|---|---|
| `format` | `json` | JSON output |
| `OBJ_DATA` | `YES` | Return object summary data |
| `MAKE_EPHEM` | `YES` | Generate ephemeris |
| `EPHEM_TYPE` | `OBSERVER` | Observables (AZ/EL, RA/DEC, magnitude) |
| `COMMAND` | `'10'` or `'399'` | Sun=`10`, Earth=`399` |
| `CENTER` | `'coord@301'` | User-defined lunar surface site (must use `@` form for non-Earth bodies) |
| `COORD_TYPE` | `GEODETIC` | Planetodetic coordinates |
| `SITE_COORD` | `'<east_lon>,<lat>,<alt_km>'` | East longitude deg, lat deg, alt km |
| `START_TIME` / `STOP_TIME` | ISO date | Ephemeris time span |
| `STEP_SIZE` | e.g. `'1h'` | Output cadence |
| `QUANTITIES` | `'1,4'` | RA/DEC + azimuth/elevation only (not magnitude) |

**Body IDs:** Sun=`10`, Earth=`399`, Moon=`301`, Moon body center=`500@301`

**URL encoding:** spaces as `%20`, semicolons as `%3B`

## Quantity codes (observer table)

| Code | Quantity | Use |
|---|---|---|
| `1` | Astrometric RA & DEC | Inertial sky position |
| `2` | Apparent RA & DEC | Apparent sky position |
| `4` | Azimuth & Elevation | Horizon position — illumination & comms (topocentric only) |
| `9` | Visual magnitude & surface brightness | Not used for AZ/EL validation (request only quantities 1 and 4) |

## Golden test fixtures

**Fixture locations:** `tests/fixtures/horizons/`

Each fixture must contain:
1. The full Horizons API query URL (recorded)
2. The raw JSON or text response
3. The query parameters used
4. The date range and site coordinates

**Fixtures to maintain:**
- Apollo landing sites (Apollo 11, 12, 14, 15, 16, 17)
- Several south polar sites (various latitudes near −90°)
- Multiple dates across the supported ephemeris window

**Rule:** CI must not call Horizons. Fetch once with a script, commit the small
response fixtures, and test against those. Record the query parameters in each
fixture.

## Tolerance

**Target agreement: < 0.01°** on Sun and Earth azimuth/elevation.

Compare against Horizons **flat-horizon** values (elevation relative to the
plane perpendicular to the local zenith), not terrain-masked values.

If agreement is missed, find the cause (frame, time scale, light-time, site
definition) — do not loosen the tolerance without recording why in
`docs/VALIDATION.md`.

## Response validation

Always check the JSON `"signature"` object:
```json
{
  "signature": {
    "source": "NASA/JPL Horizons API",
    "version": "1.0"
  }
}
```
Responses currently carry version `1.0`; the API doc is version `1.3`. Detect
format drift by monitoring this field.

## Code example

### Golden test (pytest)
```python
import json
import pytest
import math

def angular_distance_deg(az1, el1, az2, el2):
    """Great-circle angular separation between two sky directions (degrees)."""
    az1, el1, az2, el2 = map(math.radians, [az1, el1, az2, el2])
    return math.degrees(math.acos(
        math.sin(el1) * math.sin(el2) +
        math.cos(el1) * math.cos(el2) * math.cos(az1 - az2)
    ))

def test_sun_azel_apollo11():
    with open("tests/fixtures/horizons/apollo11_sun_2026.json") as f:
        data = json.load(f)
    
    # Read site coordinates from the fixture header (Horizons predefined site
    # "Apollo-11 @ 301"), not hand-typed values.
    site_lat, site_lon, site_alt = read_site_from_header(data["result"])
    
    # Extract AZ/EL from Horizons observer table (flat-horizon values)
    horizons_az, horizons_el = parse_observer_table(data["result"])
    
    # Compute with project ephemeris engine using the same site
    project_az, project_el = compute_sun_az_el(
        lat=site_lat, lon=site_lon, alt=site_alt,
        epoch="2026-11-13T12:00:00Z"
    )
    
    # Compare with angular separation for AZ (handles 0°/360° wrap) and
    # absolute difference for EL.
    assert angular_distance_deg(project_az, project_el, horizons_az, horizons_el) < 0.01
    assert abs(project_el - horizons_el) < 0.01
```

## Validation requirements (from AGENTS.md §6)

A science change is not complete without validation evidence:

- Golden tests vs JPL Horizons for Apollo + south polar sites + multiple dates
- Target agreement < 0.01°
- CI must not call Horizons; fixtures committed
- Terrain/illumination cross-checked against QuickMap and QTS
- Property tests: azimuth wraps, elevation in [−90°, 90°], disc fraction in [0, 1]
- Never edit a test threshold or regenerate a fixture just to make a build pass

## Failure modes to avoid

- Comparing terrain-masked values to flat-horizon Horizons output
- Loosening tolerance without documenting why in VALIDATION.md
- Making live Horizons calls in CI
- Regenerating fixtures to force a passing build
- Using body center (Moon `301`) instead of surface site (`CENTER='coord@301'`) for topocentric quantities
