# VALIDATION.md — Validation Method and Latest Results

All science changes require validation evidence. This file records the method,
tolerances, and latest results. Updated after every data or code change that
affects ephemeris or terrain computations.

## Validation method

### Golden tests vs JPL Horizons

Compare our computed Sun and Earth azimuth/elevation against Horizons observer
tables for:

- **Apollo sites** (Apollo 11, 12, 14, 15, 16, 17)
- **South polar sites** (several points at **−89.5° latitude, never exactly −90°** where azimuth is degenerate)
- **Several dates spread across the supported ephemeris window** (not just one date pair)

**Comparison basis:** Horizons flat-horizon values (elevation relative to the
plane perpendicular to the local zenith), **not** terrain-masked values.

**Target agreement: < 0.01°** on both azimuth and elevation.

**Request parameters:** `CSV_FORMAT=YES`, `TIME_DIGITS=FRACSEC` (default precision
is minutes, insufficient for validation), quantities `1` and `4` only
(astrometric RA/DEC + azimuth/elevation), **not** quantity `9` (magnitude).

If agreement is missed, investigate the cause (frame, time scale, light-time,
site definition). Do not loosen the tolerance without recording why here.

### Fixtures

Committed in `tests/fixtures/horizons/`. Each fixture contains:

1. Full Horizons API query URL (recorded)
2. Raw JSON/text response
3. Query parameters used
4. Date range and site coordinates

**Rule:** CI must not call Horizons. Fetch once with a script, commit fixtures,
test against those.

### Cross-checks

Terrain/illumination results cross-checked against:

- QuickMap (terrain imagery/DEM)
- QuickMap Terrain Shadows (QTS) illumination analysis
- Published LOLA illumination studies at known sites

Record each comparison here with the site, date, and result.

### Property tests

- Azimuth wraps correctly [0°, 360°)
- Elevation in [−90°, 90°]
- Disc fraction in [0, 1]
- Events are monotone-ordered
- Results invariant to unit-preserving refactors

## Validation plan for decision features

These are required checks before the features are presented as mission-planning
results. No result should be claimed until the tests are implemented and run.

### Slip-window finder

- Use fixed, documented candidate sites, date ranges, constraints, and engine
  inputs in offline fixtures.
- Test that each result is classified as meeting, missing, or unknown; missing
	data must never be treated as a pass.
- Test exact threshold boundaries, date-range edges, empty candidate lists,
  invalid limits, and stable ordering when results tie.
- For every result, retain the criteria and inputs that produced it so a user
	can inspect why it passed or failed.
- Validate the underlying Sun/Earth and terrain calculations using the existing
	Horizons and terrain cross-check rules. Horizons validates celestial geometry,
	not the site-ranking policy.

### Landing-area robustness

- Test deterministic sample locations for a simple synthetic terrain fixture
	with expected variation across the landing area.
- Record the area shape and size, point-generation method, terrain resolution,
	missing-data handling, and any supplied landing-position distribution.
- Do not describe the fraction of checked points as a landing probability
	unless those points are weighted by a justified probability distribution.
- Check that reported ranges or percentiles are repeatable for fixed inputs and
	that finer sampling does not create an unexplained change in conclusions.
- Report insufficient terrain coverage or resolution as unknown/limited, not as
	a successful robustness result.

### AI chat

- Test with a mocked provider; automated tests and CI must not require a live AI
	service or send scenario data to an external provider.
- Verify that only documented app actions can be requested and that malformed,
	out-of-range, or unsupported requests are rejected safely.
- Verify that scenario-changing actions require human approval and that the AI
	can only summarize values returned by the science engine.
- Test that the UI exposes the structured request and the results used in its
	explanation, including unknowns and unmet limits.
- Verify that the chat panel is accessible via keyboard and that chat messages
	have text alternatives.

### Product and usability claims

Scientific validation does not establish that the workflow is easier than
competitors. Before making market or usability claims, record which current
competitor workflows were tried, who gave feedback, and what tasks they found
difficult. Treat untested claims as hypotheses.

## Latest results

Golden tests pass against JPL Horizons fixtures across 6 site/body combinations (145 samples each, every 10th sample over a 24-hour window). Tolerance: < 0.01° on angular separation, |ΔAZ|, and |ΔEL|.

| Fixture | Site | Target | Max angular separation | Max |ΔAZ| | Max |ΔEL| | Pass? |
|---------|------|--------|----------------------|--------|--------|-------|
| apollo11_Sun_2026 | Apollo 11 (0.674°N, 23.45°E) | Sun | 0.000001° | 0.000001° | 0.000001° | Yes |
| apollo11_Earth_2026 | Apollo 11 (0.674°N, 23.45°E) | Earth | 0.000011° | 0.000001° | 0.000010° | Yes |
| south_pole_Sun_2026 | South Pole 89.5S (0°E) | Sun | 0.000001° | 0.000000° | 0.000000° | Yes |
| south_pole_Earth_2026 | South Pole 89.5S (0°E) | Earth | 0.000020° | 0.000001° | 0.000019° | Yes |
| south_pole_summer_Sun_2026 | South Pole 89.5S (180°E) | Sun | 0.000001° | 0.000001° | 0.000000° | Yes |
| south_pole_summer_Earth_2026 | South Pole 89.5S (180°E) | Earth | 0.000025° | 0.000001° | 0.000025° | Yes |

**Worst case across all fixtures:** max angular separation 0.000025°, max |ΔAZ| 0.000001°, max |ΔEL| 0.000025°. All within 0.01° tolerance with >2 orders of magnitude margin.

Ephemeris fit artifact (`ephemeris_v1.json`, built from SPICE kernels in RawData/):
- Window: 2026-01-01 to 2027-01-01 (TDB)
- Fit error measured vs SPICE: 1.478779e-06° (target: well under 0.001°)
- 366 segments (48-hour segments), 2 body pairs (MOON_TO_SUN, MOON_TO_EARTH)
- Chebyshev order 10, MOON_ME frame, apparent (LT+S) aberration correction

### Error budget

| Component | Target error | Measured error |
|-----------|-------------|----------------|
| Ephemeris direction (Chebyshev fit vs SPICE) | < 0.001° | 1.478779e-06° |
| AZ/EL vs Horizons (golden test) | < 0.01° | 0.000025° |
| Disc angular radius | < 0.001° | — |

## Data version validated against

| Dataset | Version | Checksum |
|---------|---------|----------|
| SPICE kernels (naif0012.tls, de440.bsp, moon_pa_de440_200625.bpc, moon_de440_250416.tf) | Pinned per docs/DATA.md | SHA-256 verified at build time |
| LOLA DEM (ldem_80s_20m.img) | LDEM_80S_20M v1.0 | DB44C3B2444ACFF1AF528301C8A6988934B490249ACEFF0AC55668BDF3EAB365 |
| Horizons API | signature.version = 1.0 (doc v1.3) | — |

**Artifacts produced:**
- `ephemeris_v1.json` — SHA-256: 9a3aa1496ae8683b888fa056ce7af1081816c4da5e28334d9eeb905543851312 (release zip 1.0.0: 6a0f92dc61b45abe85166ba97c2d29abfbf166f1bf3dc38ec83b7760e97d722a)
- `horizon_masks_v1.json` — SHA-256: 549c17c53001fcdd71f420c8a1f8e146b6ad8669cba5f2d02b939f268848ce9c

## How to add a validation run

1. Fetch Horizons response for a new site/date using `CSV_FORMAT=YES`, `TIME_DIGITS=FRACSEC`, `QUANTITIES='1,4'`, `CENTER='coord@301'`.
2. Commit fixture to `tests/fixtures/horizons/` with full query URL recorded.
3. Run `pnpm --filter core test:golden`.
4. Add a row to the "Latest results" table above with ΔAZ and ΔEL.
5. If Δ > 0.01°, investigate and document the cause here before proceeding.

## Rules

- Never edit a test threshold just to make a build pass.
- Never delete a failing test.
- Never regenerate a fixture to force a pass.
- Explain discrepancies to the human instead of hiding them.
