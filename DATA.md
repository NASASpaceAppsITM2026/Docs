# DATA.md — Dataset Provenance, Versions, Checksums, Licenses

All datasets used by the Lunar Landing Site Navigator are precomputed and
shipped as static assets in `data/derived/`. Python pipeline scripts in
`pipeline/` download and process them. This file records the exact versions,
checksums, and licenses so the data pipeline is reproducible.

## SPICE kernels (NAIF)

Used by the Python pipeline to build Chebyshev coefficient files for
Moon→Sun and Moon→Earth apparent ephemeris. **Fit the aberrated (apparent)
direction, not geometric — stellar aberration from the Moon's heliocentric
motion is about 20″ (~0.006°), more than half the 0.01° tolerance budget if omitted.**

| Kernel | Purpose | Version | URL | SHA-256 | License |
|--------|---------|---------|-----|---------|---------|
| `naif0012.tls` | Leap seconds (LSK) | 12 | https://naif.jpl.nasa.gov/pub/naif/generic_kernels/lsk/ | TODO(verify): compute after download | Public domain (NAIF) |
| `de440.bsp` | Planetary ephemeris (SPK) | DE440 | https://naif.jpl.nasa.gov/pub/naif/generic_kernels/spk/planets/ | TODO(verify) | Public domain (NAIF) |
| `moon_pa_de440_1900_2050.bpc` | Moon orientation (PCK) | DE440 | https://naif.jpl.nasa.gov/pub/naif/generic_kernels/pck/ | TODO(verify) | Public domain (NAIF) |
| `moon_080317.tf` | Moon frame kernel (FK) | — | https://naif.jpl.nasa.gov/pub/naif/generic_kernels/fk/ | TODO(verify) | Public domain (NAIF) |

> **TODO(verify):** Kernel filenames are unverified. `moon_pa_de440_1900_2050.bpc` contradicts the 1550–2650 orientation window claimed in AGENTS.md. `moon_080317.tf` may be a DE421-era frame kernel, not DE440. Horizons uses the DE421 mean-Earth frame; the DE440 frame kernel may have a DE421-aligned variant. Confirm exact filenames and checksums on NAIF before building Chebyshev coefficients, since a frame mismatch is a possible source of the last 0.001°.

## Lunar topography (LOLA / PDS Geosciences Node)

Used to build horizon masks via ray-marching over 360 azimuths.

| Product | Purpose | Version | URL | SHA-256 | License |
|---------|---------|---------|-----|---------|---------|
| SLDEM2015 | High-resolution lunar DEM (reference radius 1737.4 km) | — | https://pgda.gsfc.nasa.gov/products/54 | TODO(verify) | Public domain (NASA) |
| LOLA GDR | Gridded Data Record, terrain shape map | — | https://ode.rsl.wustl.edu/moon/pagehelp/Content/Missions_Instruments/LRO/LOLA/GDR/Intro.htm | TODO(verify) | Public domain (NASA) |
| LOLA polar DEM (80°S) | Polar terrain for south pole horizon masks | — | https://pgda.gsfc.nasa.gov/ (LOLA polar products) | TODO(verify) | Public domain (NASA) |

> **NOTE:** SLDEM2015 spans only ±60° latitude — it does **not** cover the lunar poles and is useless for south pole site assessment. Use LOLA polar DEM products (80°S and nearby) for south pole terrain. LOLA GDR height values are scaled and offset — read the product label before converting. Record the scale/offset here.

## Horizons validation fixtures (JPL)

Committed snapshots of Horizons API responses used for golden tests. Not shipped with the app — used only in CI/tests.

| Fixture | Site | Date range | Query URL | SHA-256 |
|---------|------|------------|-----------|---------|
| `apollo11_sun_2026.json` | Apollo 11 | 2026-11-13 to 2026-11-14 | TODO(verify): record full URL | TODO(verify) |
| `apollo11_earth_2026.json` | Apollo 11 | 2026-11-13 to 2026-11-14 | TODO(verify) | TODO(verify) |
| `south_pole_sun_2026.json` | South polar site | 2026-11-13 to 2026-11-14 | TODO(verify) | TODO(verify) |
| `south_pole_earth_2026.json` | South polar site | 2026-11-13 to 2026-11-14 | TODO(verify) | TODO(verify) |

> Each fixture must record the full Horizons API query parameters so the test can be reproduced independently.

## QuickMap references (not shipped)

Used as cross-check references, not bundled with the app.

| Source | Endpoint | Use |
|--------|----------|-----|
| Map config | `https://quickmap.lroc.im-ldi.com/maps/lroc` | Layer/projection/terrain config; `isLongitude360=true` |
| Terrain (Cesium heightmap) | `https://files.actgate.com/terrain/lroc/cz_cached_qts_demstack` | Cross-check terrain elevation |
| QTS shadows | `https://qts.quickmap.io/fcgi-bin/fprovweb.exe?_xtype=text/html&target=MOON&cmd_script=qts_js` | Illumination/shadow reference |

## How to update data

1. Run `pipeline/download_kernels.py` to fetch new SPICE kernels.
2. Record new versions + SHA-256 in this file.
3. Run `pipeline/build_chebyshev.py` to regenerate `data/derived/` coefficient files.
4. Run `pipeline/process_dem.py` if DEM source changed.
5. Re-run `pnpm --filter core test:golden` and update `docs/VALIDATION.md` with before/after numbers.
6. Commit updated `data/derived/` + this file + `docs/VALIDATION.md`.

## License

All NASA/NAIF/PDS data listed here is public domain (NASA open data policy).
Citation for QuickMap data: "Lunar QuickMap (https://lunar.quickmap.io), a collaboration between NASA, Intuitive Machines & Applied Coherent Technology Corp."
