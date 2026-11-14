---
name: quickmap-client
version: "1.0.0"
description: |
  Lunar QuickMap (LROC) client conventions for the Lunar Landing Site Navigator.
  Encodes the map config endpoint, ZXY tile server pattern, isLongitude360=true
  compatibility, URL state encoding (prjExtent, proj, stack, defs), terrain
  provider, and QTS terrain shadows cross-check.
domain: ["remote-sensing", "lunar-science", "web-mapping", "data-access"]
work_mode: ["computational", "data-access"]
compatibility:
  javascript: "ES2020+"
  packages: ["cesium", "openlayers", "react"]
  network: "fetch-once-then-precompute (offline-first app; no runtime tile calls at demo)"
author: "K-Dense Inc. / project-specific"
license: "MIT"
references:
  - "Lunar QuickMap: https://quickmap.lroc.im-ldi.com/"
  - "QuickMap map config: https://quickmap.lroc.im-ldi.com/maps/lroc"
  - "QuickMap tiles: https://lroc-tiles.quickmap.io/tiles/"
  - "QuickMap terrain: https://files.actgate.com/terrain/lroc/cz_cached_qts_demstack"
  - "QuickMap Terrain Shadows (QTS): https://qts.quickmap.io/fcgi-bin/fprovweb.exe?_xtype=text/html&target=MOON&cmd_script=qts_js"
  - "QuickMap docs: http://docs.quickmap.io/"
---

# QuickMap Client — Lunar Imagery & Terrain Access

## Purpose

Teach an AI agent how to work with Lunar QuickMap data sources and conventions
for the Lunar Landing Site Navigator. This skill encodes the project-specific
QuickMap integration patterns, not generic web mapping.

## When to apply

- Reading the QuickMap map config JSON endpoint
- Constructing ZXY tile URLs for lunar imagery layers
- Working with isLongitude360=true longitude convention (0–360° east)
- Encoding/decoding shareable link state (prjExtent, proj, stack, defs)
- Accessing the Cesium terrain heightmap provider
- Cross-checking terrain shadows against QTS
- Selecting data extraction layers (defaultDataExtractSources)

## Map config endpoint

**GET** `https://quickmap.lroc.im-ldi.com/maps/lroc`

Returns JSON (`apiVersion: 2.0`) defining planet, projections, layers, terrain
provider, data sources, units, citations, and help links.

Key fields used by this project:
| Field | Value | Use |
|---|---|---|
| `planetId` | `1` | Moon |
| `isLongitude360` | `true` | 0–360° east longitude |
| `defaultProjectionId` | `10` | Default map projection |
| `projectionIds` | `[16, 26, 27, 7, 10, 19, 17, 22]` | Available projections |
| `demSourceId` | `2848` | Default DEM source |
| `defaultCharts` | `qts_demstack, gld100i, lsldem2015` | Default DEM stacks |
| `defaultProfileSources` | `qts_demstack, gld100i, lsldem2015` | Elevation profile sources |
| `defaultDataExtractSources` | `lp_h, lk_oma, dlre_hparam, ...` | Extractable data layers |
| `terrainProvider` | `https://files.actgate.com/terrain/lroc/cz_cached_qts_demstack` | Cesium heightmap |
| `terrainProviderOptions` | `heightScale: 0.302948, heightOffset: -9093` | Vertical scale/offset |
| `defaultClampToGround` | `true` | Features clamped to terrain |
| `isSkyBoxEnabled` | `true` | Starry skybox in 3D mode |

## Tile server (ZXY raster tiles)

Pattern:
```
https://lroc-tiles.quickmap.io/tiles/{layer}/{projection}/{z}/{x}/{y}.jpg
```

Example observed: `.../tiles/wac_nac_nacroi/lunar-orthons/0/0/0.jpg`

Layers include WAC/NAC mosaics and derived products. Precompute and cache
tiles needed for the supported landing site regions — do not rely on runtime
tile fetching at demo time.

## URL state encoding (shareable links)

QuickMap encodes map state in URL parameters:
```
?prjExtent=-1737400,-1831172.71,1737400,1831172.71&earthShadowEnabled=true&proj=10&stack=3314&defs=N4IgzGCMAsIFygPYAcCGBjAlgFwJ70gF9Cg
```

| Parameter | Meaning |
|---|---|
| `prjExtent` | Projected map bounds in meters (Moon radius ≈ 1,737,400 m) |
| `earthShadowEnabled` | Earth shadow toggle in 3D illumination |
| `proj` | Projection ID |
| `stack` | Layer stack ID |
| `defs` | Serialized layer/definition state (compressed) |

This project's frontend should follow the same pattern: full scenario state
(serializable to URL, no secrets or large blobs).

## Terrain provider (Cesium)

**URL:** `https://files.actgate.com/terrain/lroc/cz_cached_qts_demstack`

Cesium quantized-mesh heightmap for 3D globe terrain. Options:
- `heightScale: 0.302948` — vertical exaggeration
- `heightOffset: -9093` — vertical offset

## QuickMap Terrain Shadows (QTS)

**URL:**
```
https://qts.quickmap.io/fcgi-bin/fprovweb.exe?_xtype=text/html&target=MOON&cmd_script=qts_js
```

Linked from QuickMap's map config. Used as a cross-check for terrain shadow
and illumination results at known sites.

## Data extraction layers

Available via `defaultDataExtractSources`:
- `lp_h` — Lunar Prospector hydrogen (polar ice indicator)
- `lk_oma` — Diviner / LRO thermal
- `dlre_hparam` — Diviner radiometer
- `lp_fe` — Lunar Prospector iron
- `ldrm_16` — LRO laser altimeter
- `lm3_gp_*` — Moon Mineralogy Mapper
- `qts_demstack*` — QTS DEM and slope

These are reference layers for site assessment; precompute derived rasters
offline.

## Validation

- Cross-check terrain/illumination against QTS at known sites
- Cross-check against published LOLA illumination studies
- Verify `isLongitude360=true` matches Horizons east longitude (no sign flip)

## Failure modes to avoid

- Assuming longitude is west-positive or −180–180 without checking
- Relying on runtime tile fetching for the demo (offline-first requirement)
- Using the retiring ASU host (`quickmap.lroc.asu.edu`) — use `.im-ldi.com`
- Not recording DEM product names and resolutions in `docs/DATA.md`
