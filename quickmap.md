# Lunar QuickMap (LROC) — Data Source Context for AI Agent

## Overview
- **URL:** https://quickmap.lroc.im-ldi.com/
- **Retiring URL:** https://quickmap.lroc.asu.edu/
- **Operator:** NASA (LRO/LROC data), currently hosted at Intuitive Machines; collaboration between NASA, Intuitive Machines, and Applied Coherent Technology Corp. (ACT)
- **Data license:** Public domain (NASA open data policy). Citation requested: "Lunar QuickMap (https://lunar.quickmap.io), a collaboration between NASA, Intuitive Machines & Applied Coherent Technology Corp."
- **User guide:** http://docs.quickmap.io/

## How It Works
### Technology Stack
- Frontend: React + CesiumJS (3D globe) + OpenLayers (2D map) + Lodash
- Build tool: rolldown / Vite
- Error tracking: Sentry
- Required browser features: WebGL, Promise, Blob, ArrayBuffer, OffscreenCanvas (Chrome, Firefox, Edge, Safari 17+)

### Data Architecture
QuickMap is a tile-driven lunar web map. All map behavior is driven by a single map-config endpoint.

**Map config:** `GET https://quickmap.lroc.im-ldi.com/maps/lroc`
Returns JSON (`apiVersion: 2.0`) defining the planet, projections, layers, terrain provider, data sources, units, citations, and help links.

**Tile server (ZXY raster tiles):**
```
https://lroc-tiles.quickmap.io/tiles/{layer}/{projection}/{z}/{x}/{y}.jpg
```
Example observed: `.../tiles/wac_nac_nacroi/lunar-orthons/0/0/0.jpg`

**Terrain (Cesium quantized-mesh heightmap):**
```
https://files.actgate.com/terrain/lroc/cz_cached_qts_demstack
```

**Approximate terrain heights (JSON):**
```
https://files.actgate.com/approximateTerrainHeights.lunar.json
```

**QuickMap Terrain Shadows (QTS) — illumination / shadow analysis:**
```
https://qts.quickmap.io/fcgi-bin/fprovweb.exe?_xtype=text/html&target=MOON&cmd_script=qts_js
```

### Map Configuration Facts (from `/maps/lroc`)
| Field | Value | Relevance to Agent |
|-------|-------|-------------------|
| Planet / body | Moon (`planetId: 1`) | Lunar-only map |
| `isLongitude360` | `true` | Longitude stored 0–360° east — matches Horizons east-longitude convention |
| `distanceUnits` | kilometers | UI distance unit |
| `areaUnits` | square kilometer | UI area unit |
| `defaultProjectionId` | `10` | Default map projection (matches URL `proj=10`) |
| `projectionIds` | `[16, 26, 27, 7, 10, 19, 17, 22]` | Available lunar projections |
| `demSourceId` | `2848` | Default DEM source |
| `defaultCharts` | `qts_demstack, gld100i, lsldem2015` (+ slope) | Default terrain/DEM stacks |
| `defaultProfileSources` | `qts_demstack, gld100i, lsldem2015` | Elevation profile sources |
| `defaultDataExtractSources` | `lp_h, lk_oma, dlre_hparam, lp_fe, ldrm_16, lm3_gp_*, qts_demstack*, ...` | Layers available for data extraction |
| `booleanTileServerId` | `2` | Boolean/mask tile server |
| `graticuleLayerId` | `null` | No default graticule overlay |
| `terrainProvider` | `https://files.actgate.com/terrain/lroc/cz_cached_qts_demstack` | Cesium terrain heightmap |
| `terrainProviderOptions` | `heightScale: 0.302948, heightOffset: -9093` | Vertical exaggeration / offset for terrain |
| `defaultClampToGround` | `true` | Features clamped to terrain |
| `isSkyBoxEnabled` | `true` | Starry skybox in 3D mode |

### URL State Encoding (Shareable Links)
Observed URL format:
```
?prjExtent=-1737400,-1831172.71,1737400,1831172.71&earthShadowEnabled=true&proj=10&stack=3314&defs=N4IgzGCMAsIFygPYAcCGBjAlgFwJ70gF9Cg
```

| Parameter | Meaning |
|-----------|---------|
| `prjExtent` | Projected map bounds in meters (Moon radius ≈ 1,737,400 m) |
| `earthShadowEnabled` | Toggle for Earth shadow in 3D illumination |
| `proj` | Projection ID |
| `stack` | Layer stack ID |
| `defs` | Serialized layer/definition state (compressed, base64-like) |

### UI Features Observed
- Zoom controls (`+`, `–`), info (`?`, `i`), scale bar (e.g. `500 km`)
- Live position readout: `Lat`, `Lon` (0–360° east), `Res` (m/px) — example: `Lat 2.49191, Lon 291.73378, Res 5209.60 m/px`
- Layer Browser (redesigned — only active map layer shown at a time)
- Visibility Analysis panel
- Accuracy statement per layer
- 3D globe mode (Cesium) with terrain clamping
- Sky/star search (queries SIMBAD API for bright stars near a clicked point)
- Right-click context-menu feature identification (lat/lon lookup)
- Notifications panel (Alt+T)

## What Data It Shows
- High-resolution lunar imagery (LRO LROC WAC/NAC mosaics)
- Topographic / DEM data (GLD100, LOLA-derived, QTS DEM stacks)
- Slope and hillshade layers
- Science data layers extractable via `defaultDataExtractSources` (LP hydrogen, Diviner, LRO laser altimeter, etc.)
- Coordinate readout and measurement tools

## Science Notes for the Agent
- QuickMap answers **"where is the site and what does the terrain look like"** (imagery + DEM + coordinates).
- QuickMap also has visibility/illumination analysis; verify its current date-specific capabilities directly before claiming a workflow gap.
- Longitude convention compatibility with Horizons is a key integration advantage (both use 0–360° east longitude).
- Use the app's validated local engine for its calculations. Use QuickMap and QTS as external terrain/illumination references where appropriate; Horizons remains an independent ephemeris validation reference, not the app's runtime calculation path.

## Agent Usage Pattern
1. Select a candidate site in the app or use QuickMap to inspect terrain and coordinates.
2. Run the app's local calculations for the selected site and date.
3. Use committed Horizons fixtures to validate celestial geometry; use QuickMap/QTS and published studies to cross-check terrain illumination.
4. Compare dates and candidate sites with visible user constraints, then inspect the reason behind each result.
5. Use QuickMap's current analysis and visualization tools as a competitor/reference workflow; note what it already does before describing the app's distinction.