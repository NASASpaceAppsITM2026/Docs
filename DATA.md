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
| `naif0012.tls` | Leap seconds (LSK) | NAIF0012 (12) | https://naif.jpl.nasa.gov/pub/naif/generic_kernels/lsk/naif0012.tls | `678E32BDB5A744117A467CD9601CD6B373F0E9BC9BBDE1371D5EEE39600A039B` | Public domain (NAIF) |
| `de440.bsp` | Planetary ephemeris (SPK, DE440 family) | DE440 | https://naif.jpl.nasa.gov/pub/naif/generic_kernels/spk/planets/de440.bsp | `A4CE9BF9B3282BECC9F4B2AC3CEBE03A2AE7599981AABD7265FD8482FFF7C4B5` | Public domain (NAIF) |
| `moon_pa_de440_200625.bpc` | Moon orientation (PCK, DE440-based) | DE440 | https://naif.jpl.nasa.gov/pub/naif/generic_kernels/pck/moon_pa_de440_200625.bpc | `60CD55AA401EA2EA97360636F567554BFE4E37BB829F901B4460A455DFAF783F` | Public domain (NAIF) |
| `moon_de440_250416.tf` | Moon frame kernel (FK, DE440-based) | DE440 (2025-04-16) | https://naif.jpl.nasa.gov/pub/naif/generic_kernels/fk/satellites/moon_de440_250416.tf | `A47C71E9C9F33796BDAFB2C9D69A7EE447B6016ECAD80F71CD6F3E479F9CF768` | Public domain (NAIF) |

**Verified against NAIF sources (https://naif.jpl.nasa.gov/naif/); all SHA-256 checksums for the kernels listed above are resolved from this session's downloads (see the checksums block below).**

- Leap seconds (LSK): `naif0012.tls` / `naif0012.tls.pc`; version **NAIF0012** (12); `latest_leapseconds.tls` → `naif0012.tls`. NAIF readme: required for accurate UTC↔ET conversions on/after 2017-01-01. SHA-256 (computed after download): `678E32BDB5A744117A467CD9601CD6B373F0E9BC9BBDE1371D5EEE39600A039B`.
- DE440-family planetary ephemeris: `de440.bsp` (DE440; Park et al. 2021, AJ 161, 105). NAIF `aa_summaries.txt`: Moon (301) w.r.t. Earth Barycenter (3), Earth (399) and Sun (10); ET ≈ **1549 DEC 31 – 2650 JAN 25** (≈ the supported 1550–2650 window). SHA-256 (from `D:\Projects\NASA2026\RawData\de440.bsp`): `A4CE9BF9B3282BECC9F4B2AC3CEBE03A2AE7599981AABD7265FD8482FFF7C4B5`.
- Moon orientation (PCK): `moon_pa_de440_200625.bpc` (DE440-based; NAIF coverage ≈ **1550–2650**; created 2021 Jun 25; source NAVIO `de440.nio`; ref. Park et al. 2021). Paired with lunar frame kernel `moon_de440_250416.tf` (2025-04-16 revision; path `pub/naif/generic_kernels/fk/satellites`).
- Moon frame kernel (FK): `moon_de440_250416.tf` (2025-04-16 revision; previous `moon_de440_220930.tf`, 2022-09-30); frames `MOON_PA` → `MOON_PA_DE440` (31008), `MOON_ME` → `MOON_ME_DE440_ME421` (31009); frame-association kernels `moon_assoc_pa.tf` / `moon_assoc_me.tf`.
- DE421-aligned mean-Earth frame: yes — such a variant exists. The DE-421-based set (`moon_pa_de421_1900-2050.bpc` + `moon_080317.tf`, generic `MOON_ME` = `MOON_ME_DE421`) and the DE440-based `MOON_ME_DE440_ME421` (aligned with the DE421 mean-Earth frame) both realize it. **This DE421-aligned mean-Earth frame is the one that matches Horizons conventions** (Horizons uses `MOON_ME`, East-longitude positive, pole/equ = MOON_ME, radii 1737.4/1737.4/1737.4 km).

Checksums computed from actual downloads this session: `naif0012.tls` = `678E32BDB5A744117A467CD9601CD6B373F0E9BC9BBDE1371D5EEE39600A039B`; `moon_080317.tf` = `78732477B96F9863E7B0D65BCEE3C22B8707CA5ED0DB56D1173319CB2E8C7993`; `moon_de440_250416.tf` = `A47C71E9C9F33796BDAFB2C9D69A7EE447B6016ECAD80F71CD6F3E479F9CF768`; `moon_pa_de421_1900-2050.bpc` = `656F90616403D75A75F0CD6C8830FC5B44F8CB4FACB5CCB8915E752B397520CF`; `de440.bsp` = `A4CE9BF9B3282BECC9F4B2AC3CEBE03A2AE7599981AABD7265FD8482FFF7C4B5`; `moon_pa_de440_200625.bpc` = `60CD55AA401EA2EA97360636F567554BFE4E37BB829F901B4460A455DFAF783F`.

> **TODO(verify):** resolved — SHA-256 for `de440.bsp` and `moon_pa_de440_200625.bpc` computed after download from `D:\Projects\NASA2026\RawData\`. The earlier `moon_pa_de440_1900_2050.bpc` entry did not exist; the DE421-oriented pair (`moon_pa_de421_1900-2050.bpc` + `moon_080317.tf`) is retained for the DE421-aligned-frame comparison with Horizons.

## Lunar topography (LOLA / PDS Geosciences Node)

Used to build horizon masks via ray-marching over 360 azimuths.

| Product | Purpose | Version | URL | SHA-256 | License |
|---------|---------|---------|-----|---------|---------|
| SLDEM2015 | High-resolution lunar DEM (reference radius 1737.4 km) | — | https://pgda.gsfc.nasa.gov/products/54 | TODO(verify) | Public domain (NASA) |
| LOLA GDR | Gridded Data Record, terrain shape map | — | https://ode.rsl.wustl.edu/moon/pagehelp/Content/Missions_Instruments/LRO/LOLA/GDR/Intro.htm | TODO(verify) | Public domain (NASA) |
| LOLA GDR DEM (GDRDEM), Terrain Shape Map — south pole | Polar terrain for south pole horizon masks | PDS3, DATA_SET_ID `LRO-L-LOLA-4-GDR-V1.0` | https://pds-geosciences.wustl.edu/lro/lro-l-lola-3-rdr-v1/lrolol_1xxx/data/lola_gdr/polar/img/LDEM_80S_20M.IMG | TODO(verify) | Public domain (NASA) |
| LOLA GDR DEM — nearby southern polar products | `LDEM_85S_10M` (85°S, 10 m), `LDEM_75S_30M` (75°S, 30 m), `LDEM_60S_60M`, `LDEM_45S_100M`, `LDEM_87S_5M` | PDS3, DATA_SET_ID `LRO-L-LOLA-4-GDR-V1.0` | https://pds-geosciences.wustl.edu/lro/lro-l-lola-3-rdr-v1/lrolol_1xxx/data/lola_gdr/polar/img/ | TODO(verify) | Public domain (NASA) |
| LOLA GDR DEM — adjusted large-area TIF (Barker et al. 2023) | Adjusted DEM, south pole (80–90°S) | — | https://pgda.gsfc.nasa.gov/data/LOLA_20mpp/LDEM_80S_20MPP_ADJ.TIF | TODO(verify) | Public domain (NASA) |

> **South-pole LOLA product (verified).** The exact LOLA product covering the south pole is the **LOLA Gridded Data Record – Terrain Shape Map (GDRDEM), product `LDEM_80S_20M`** (PDS3; DATA_SET_ID `LRO-L-LOLA-4-GDR-V1.0`; producer LRO_LOLA_TEAM, NASA Goddard; source paper: Barker et al. 2023, *Planet. Sci. J.* 4, 183, https://doi.org/10.3847/PSJ/acf3e1; data DOI: https://doi.org/10.60903/gsfcpgda-lola-spole; product page: https://pgda.gsfc.nasa.gov/products/90).
>
> - **Resolution:** 20 m/pixel (true at the pole); 30,400 × 30,400 pixels; latitude range 80°S–90°S (MAX/MIN latitude −80/−90°).
> - **Projection:** Polar Stereographic (`MAP_PROJECTION_TYPE` = "POLAR STEREOGRAPHIC"); latitude type PLANETOCENTRIC; body-fixed rotating coordinate system = **MEAN EARTH/POLAR AXIS OF DE421**; A/B/C axis radius = 1737.4 km; positive longitude = EAST; center latitude −90°, center longitude 0°; map scale = 20 m/pix.
> - **Height scale/offset** (from the PDS3 label): `SCALING_FACTOR = 0.5`, `OFFSET = 1737400` m; `HEIGHT = (DN × SCALING_FACTOR)`; `PLANETARY_RADIUS = (DN × SCALING_FACTOR) + OFFSET`; topography = planetary radius − geoid radius. (Reference radius: 1737.4 km.)
> - **File format:** PDS3 `UNCOMPRESSED_FILE` (fixed-length binary; `IMAGE` = HEIGHT, LSB_INTEGER, 16 bits, UNIT = METER; `^IMAGE_MAP_PROJECTION`, `MAP_PROJECTION_TYPE` = "POLAR STEREOGRAPHIC", `^DATA_SET_MAP_PROJECTION` = "DSMAP_POLAR.CAT") + detached label; also `_float` (floating-point) and `_jp2` (JPEG2000) variants. The Barker et al. (2023) adjusted products (`LDEM_80S_20MPP_ADJ.TIF`) are Cloud Optimized GeoTIFFs with south polar stereographic X/Y in meters, MOON_ME frame of DE421, LDEM height in meters.
> - **Download URL:** `https://pds-geosciences.wustl.edu/lro/lro-l-lola-3-rdr-v1/lrolol_1xxx/data/lola_gdr/polar/img/LDEM_80S_20M.IMG` (1,848,320,000 bytes; verified HTTP 200); adjusted TIF: `https://pgda.gsfc.nasa.gov/data/LOLA_20mpp/LDEM_80S_20MPP_ADJ.TIF`.

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
