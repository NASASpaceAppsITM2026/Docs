---
name: terrain-illumination
version: "1.0.0"
description: |
  Lunar terrain horizon masking and illumination computation for the Lunar
  Landing Site Navigator. Encodes LOLA DEM access, ray-marching over 360
  azimuths, sphere curvature drop, observer height, terrain skyline angle,
  disc-above-mask fraction, permanently shadowed regions, upper-limb sunrise
  convention, eclipse disc-overlap, and longest continuous darkness.
domain: ["lunar-science", "remote-sensing", "terrain-analysis", "illumination"]
work_mode: ["computational", "field-analog"]
compatibility:
  python: "3.11+"
  packages: ["numpy", "scipy", "rasterio", "gdal", "geopandas"]
  network: "offline-first (DEM pre-downloaded; no runtime PDS calls)"
author: "K-Dense Inc. / project-specific"
license: "MIT"
references:
  - "PDS Geosciences Node LRO LOLA: https://pds-geosciences.wustl.edu/missions/lro/lola.htm"
  - "SLDEM2015 High-resolution Lunar Topography: https://pgda.gsfc.nasa.gov/products/54"
  - "QuickMap Terrain Shadows (QTS): https://qts.quickmap.io/fcgi-bin/fprovweb.exe?_xtype=text/html&target=MOON&cmd_script=qts_js"
  - "QuickMap map config: https://quickmap.lroc.im-ldi.com/maps/lroc"
---

# Terrain Illumination — Horizon Masks & Lunar Lighting

## Purpose

Teach an AI agent how to compute terrain horizon masks, illumination windows,
and shadow metrics for lunar landing sites. This skill is project-specific:
it encodes the Lunar Landing Site Navigator's terrain/illumination rules,
including south polar permanently shadowed regions and upper-limb conventions.

## When to apply

- Building horizon masks from LOLA DEMs
- Ray-marching terrain over 360 azimuths with sphere curvature
- Computing disc-above-mask fraction for Sun/Earth visibility
- Modeling permanently shadowed regions (PSR) at the lunar poles
- Determining upper-limb sunrise/sunset (default operational convention)
- Modeling eclipse disc-overlap (Earth blocking the Sun)
- Computing longest continuous darkness for battery/thermal survival

## Terrain data (LOLA)

Source: PDS Geosciences Node LRO LOLA Reduced Data Record (RDR) and Gridded
Data Record (GDR).

Key products:
- **SLDEM2015**: high-resolution lunar topography (reference radius 1737.4 km).
  **WARNING: covers only ±60° latitude — does NOT cover the lunar poles.**
  Useless for south pole site assessment. Use LOLA polar DEM products (80°S)
  for south pole terrain.
- **LOLA GDR**: gridded elevation with scale/offset in labels — read the label
  before converting height values
- **LOLA polar DEM (80°S)**: polar terrain for south pole horizon masks

Verify product names and resolution on PDS before use; record them in
`docs/DATA.md`.

## Horizon mask computation

**Ray-marching over 360 azimuths:** for each azimuth angle, step outward from
the site and compute the terrain elevation angle relative to the local horizon
plane. The horizon elevation at azimuth A is the maximum terrain angle
encountered along that ray.

**Sphere curvature drop:** include the curvature horizon of the lunar sphere
(dip from site height) in addition to terrain skyline angle. The Moon has no
atmosphere — no refraction anywhere.

**Observer height:** include as a parameter; it shifts the effective horizon.

## Disc-above-mask fraction

Sun and Earth are finite discs. Compute visibility as the **fraction of disc
above the horizon mask**, not just whether the center is above the mask. This
matters near the horizon where partial disc visibility determines illumination
and communication windows.

## Sunrise/sunset convention

**Default for operational metrics: upper limb.** The upper limb of the Sun (or
Earth) crossing the horizon defines sunrise/sunset. Disc-center convention is
an alternative — state which convention is used in the UI and API.

## South polar illumination

- **Permanently shadowed regions (PSR):** areas where the Sun never rises
  above the horizon mask over the supported ephemeris window. First-class
  metric for landing site assessment.
- **Longest continuous darkness:** maximum duration of unbroken darkness at a
  site — critical for battery/thermal survival analysis.
- **Earth blocking the Sun (lunar eclipse):** model as disc overlap; do not
  ignore it.

## Cross-checks

Cross-check terrain/illumination results against:
- QuickMap Terrain Shadows (QTS) at known sites
- Published LOLA illumination studies at known sites
- Record comparisons in `docs/VALIDATION.md`

## Code examples

### Horizon mask ray-march (conceptual)
```
for azimuth in 0..360 step 1°:
    max_elevation = -90°
    for distance in step_out_from_site:
        terrain_height = dem.sample(position_at(distance, azimuth))
        angle = atan((terrain_height - site_height) / ground_distance)
        angle -= curvature_drop(ground_distance)  # sphere dip
        max_elevation = max(max_elevation, angle)
    horizon_mask[azimuth] = max_elevation
```

### Disc-above-mask fraction
```
disc_fraction = area_of_sun_disc_above_mask(azimuth, elevation, angular_radius)
```

## Validation

- Cross-check against QuickMap and QTS at known sites
- Cross-check against published LOLA illumination studies
- Property tests: elevation in [−90°, 90°], disc fraction in [0, 1]

## Failure modes to avoid

- Forgetting sphere curvature drop (overestimates horizon near poles)
- Using disc-center instead of upper-limb without stating convention
- Ignoring observer height parameter
- Treating Sun/Earth as points instead of finite discs
- Not recording DEM product name and resolution in DATA.md
