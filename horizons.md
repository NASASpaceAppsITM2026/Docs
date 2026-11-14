# JPL Horizons System — Ephemeris Data Source Context for AI Agent

## Overview
- **URL:** https://ssd.jpl.nasa.gov/horizons/
- **Operator:** JPL Solar System Dynamics Group (NASA/JPL/Caltech)
- **Manual version:** 4.98e (2026-08-25)
- **API:** `GET https://ssd.jpl.nasa.gov/api/horizons.api` (API version 1.3, 2025-06)
- **File-based POST API:** `/doc/horizons_file.html`
- **Command-line:** telnet localhost:6775 (requires stunnel setup first)
- **Email:** horizons@ssd.jpl.nasa.gov (subject `BATCH-LONG`)
- **Uptime:** historically >99.9%, best-effort; recommend allowance for outages in critical planning

## How It Works
Horizons numerically computes position, velocity, and observables of solar-system objects as seen from any location, at arbitrary times. It integrates pre-computed ephemerides (e.g., DE441 for major bodies) and numerically integrates small-body orbits on demand.

### Access Interfaces (all automatable)
1. **API (GET/POST)** — recommended for machine-to-machine automation
2. **Browser app:** `https://ssd.jpl.nasa.gov/horizons/app.html`
3. **Command-line:** interactive VT100 session (telnet :6775 via stunnel)
4. **Email:** batch file interface (`BATCH-LONG` → edit → `JOB`)

### Available Objects
- 1,560,000+ asteroids, 4,075 comets, 461 natural satellites
- All planets, the Sun, 253 spacecraft
- Dynamical points: Earth-Sun L1/L2/L4/L5, Earth-Moon L1/L2/L4/L5, planetary barycenters
- **Body IDs relevant to this challenge:** Sun = `10`, Earth = `399`, Moon = `301`, Moon body center = `500@301`

### Ephemeris Types
| EPHEM_TYPE | Description | Typical Use |
|------------|-------------|-------------|
| OBSERVER | Observables (RA/DEC, AZ/EL, magnitude, visibility) | telescope/sky observation — **key for this challenge** |
| VECTORS | Cartesian state vectors and uncertainties | dynamical studies, propagation |
| ELEMENTS | Osculating orbital elements | instantaneous geometry |
| APPROACH | Close-approach tables | encounter planning / hazards |
| SPK | Binary SPK trajectory files (small bodies only) | navigation, mission-planning plug-ins |

## Critical Parameters for Lunar Landing Site Assessment
| Parameter | Value / Meaning |
|-----------|-----------------|
| `format` | `json` or `text` |
| `COMMAND` | Target body ID/name (Sun=`'10'`, Earth=`'399'`, Moon=`'301'`) |
| `CENTER` | Observing site. Must use `@` form for non-Earth bodies: `'coord@301'` for a lunar surface site. Using `'coord'` alone would resolve to an Earth location. |
| `COORD_TYPE` | `GEODETIC` (planetodetic) or `CYLINDRICAL` |
| `SITE_COORD` | `'east_longitude, latitude, altitude_km'` (degrees, degrees, km) |
| `EPHEM_TYPE` | `OBSERVER` |
| `START_TIME` / `STOP_TIME` | Ephemeris time span |
| `STEP_SIZE` | Output cadence (e.g. `'1h'`, `'10m'`, `'1d'`) |
| `QUANTITIES` | Output quantity codes |
| `OBJ_DATA` / `MAKE_EPHEM` | `YES` to get object summary + ephemeris |

### Defining a Lunar Surface Observing Site (API)
```
CENTER='coord@301'&COORD_TYPE='GEODETIC'&SITE_COORD='<east_lon>,<lat>,<alt_km>'
```
- Longitude = east longitude in degrees (0–360) — **directly compatible with QuickMap's `isLongitude360`**
- Altitude above the reference ellipsoid (km)
- Predefined lunar sites exist (e.g. `1@301` = site #1 on Moon; `*@301` lists all predefined Moon sites)

### Example API Request (Sun position from a lunar site)
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
*(URL-encode spaces as `%20`, semicolons as `%3B`)*

### Observer Quantity Codes (Most Relevant)
| Code | Quantity | Use |
|------|----------|-----|
| `1` | Astrometric RA & DEC | Inertial sky position |
| `2` | Apparent RA & DEC | Apparent sky position |
| `4` | **Azimuth & Elevation (AZ-EL)** | **Horizon position — illumination & comms** (azimuth clockwise from north: N=0,E=90,S=180,W=270; elevation vs local zenith; degrees; TOPOCENTRIC ONLY) |
| `5` | AZ/EL rates | Tracking/slew rate |
| `9` | Visual magnitude & surface brightness | Not used for AZ/EL validation (request only quantities 1 and 4) |

## Science & Formulas
### Coordinate Systems
- **ICRF** (International Celestial Reference Frame), **FK4/B1950**, Earth true-of-date, Earth ecliptic, IAU body-frame of-date, Lunar mean Earth (DE421)
- 9 coordinate systems, 4 time scales (TDB, TT, UT, Civil)

### AZ-EL Definition (Quantity 4)
- Apparent azimuth & elevation, adjusted for light-time, gravitational deflection, stellar aberration, precession, nutation
- Optional atmospheric refraction (Earth only — NOT modeled for lunar sites)
- **Azimuth:** measured clockwise from north: North(0) → East(90) → South(180) → West(270)
- **Elevation:** angle above/below plane perpendicular to local zenith
- Units: degrees
- **TOPOCENTRIC ONLY** — must specify a surface center, not body center

### Lunar / Interfering Body Presence Markers
- Solar presence: `*` daylight, `C` civil twilight, `N` nautical, `A` astronomical, ` ` night
- Lunar/IB presence: `m` Moon upper limb at/above horizon (Earth), `i` interfering body (non-Earth sites, e.g. Earth for lunar observers)
- For Moon observers, no twilight codes (C/N/A), no refraction, interfering body marker = `i` (Earth)

### Surface Targets
Horizons can compute ephemerides for surface targets on rotating bodies:
- Geodetic: `{g: E.Long, latitude, h @ } BODY`
- Cylindrical: `{c: E.Long, DXY, DZ @ } BODY`
- Example: `g: 348.8, -43.3, 0 @ 301` = crater Tycho on Moon

### Output Markers for Surface Targets
When a surface target is specified, output includes markers for:
- Whether the surface point is lit by the Sun
- Whether the point is on near- or far-side of the body relative to observer

## Agent Notes
- **Target = Sun (`10`), Center = lunar surface site** → Sun's AZ/EL = illumination geometry, solar power assessment
- **Target = Earth (`399`), Center = lunar surface site** → Earth's AZ/EL = line-of-sight for direct-to-Earth communications, Earth occultation by terrain
- Horizons gives **unobstructed** celestial positions; combine with QuickMap/QTS terrain horizon mask for **true** visibility/illumination
- Longitude convention: Horizons east longitude (0–360°) matches QuickMap `isLongitude360=true` — no sign flip needed
- Always check JSON `signature.version` (currently `1.0` in responses; API doc is version `1.3`)
- No account/password required; best-effort availability — plan for potential outages