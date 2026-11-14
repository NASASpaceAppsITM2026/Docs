# Lunar Review Checklist — Astronomer

Distilled from the K-Dense Astronomer AGENTS.md profile for lunar landing site navigation review. Use this when reviewing ephemeris, coordinate, and observational PRs. Load the full profile on demand for deep dives.

## Before approving any ephemeris/observation PR, verify:

### Coordinate systems & frames
- [ ] RA/DEC output uses ICRF or FK4/B1950 reference frame, stated explicitly
- [ ] Azimuth measured clockwise from North (N=0, E=90, S=180, W=270)
- [ ] Elevation relative to local horizontal plane (perpendicular to local zenith), NOT terrain slope
- [ ] No atmospheric refraction modeled for lunar sites (Earth-only correction)
- [ ] Lunar/Interfering body presence marker = `i` (Earth) for Moon observers, not `m` (Moon)

### Horizons query parameters
- [ ] `COMMAND='10'` for Sun, `COMMAND='399'` for Earth
- [ ] `CENTER='coord@301'` for lunar surface site (not `coord` alone)
- [ ] `COORD_TYPE='GEODETIC'` for planetodetic coordinates
- [ ] `SITE_COORD='<east_lon>,<lat>,<alt_km>'` with east-positive longitude (0–360°)
- [ ] `EPHEM_TYPE=OBSERVER` for sky position queries
- [ ] `QUANTITIES='1,4'` (astrometric RA/DEC + azimuth/elevation), not `9`
- [ ] `CSV_FORMAT=YES` and `TIME_DIGITS=FRACSEC` (default precision is minutes)
- [ ] `TIME_TYPE` appropriate (UT for observer tables)

### Output interpretation
- [ ] Solar presence symbol (`*`, `C`, `N`, `A`, ` `) interpreted correctly — no twilight codes for non-Earth sites
- | Lunar/IB presence marker `i` (Earth as interfering body) for Moon observers
- [ ] Rise/transit/set markers (`r`, `e`, `t`, `s`) only when print interval ≤ 30 min or RTS-only option
- [ ] Apparent vs astrometric RA/DEC distinction respected

### Validation hygiene
- [ ] Fixtures avoid −90° latitude (azimuth degenerate) — use −89.5°
- [ ] Fixtures spread across date window
- [ ] Agreement target: `< 0.01°` on flat-horizon AZ/EL (not terrain-masked)
- [ ] CI does not call live Horizons; fixtures committed

**If any check fails: do not approve. Fix or document in `docs/VALIDATION.md` before proceeding.**
