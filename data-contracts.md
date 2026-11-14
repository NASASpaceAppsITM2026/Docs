# docs/data-contracts.md — Data Contracts for Pipeline Data Releases and App Consumption

Status: **draft contract** (written per the data-contracts prompt in `docs/KICKOFF.md`, Prompt C). This file is the cross-repo contract for how the pipeline repository publishes lunar navigation data and how the app repository consumes it. Each repository keeps its own copy at `docs/data-contracts.md`; the human coordinator owns keeping the copies in sync.

This contract exists so that two different agents working in two different repositories — one producing data releases, one consuming them — can produce and consume compatible files without hand-copying anything.

## 1. Scope

Three artifact families are versioned here:

1. **`manifest.json`** — release manifest for a pipeline data release.
2. **`ephemeris_v1`** — the versioned ephemeris artifact (Chebyshev fits of Moon→Sun and Moon→Earth).
3. **`horizon_masks_v1`** — the versioned horizon-mask artifact (LOLA-based terrain horizon masks).

Plus the consumer side:

4. **`data.lock.json`** — release pin + sync-and-verify behavior used by the app repository.

Non-goals: this contract does not specify the science calculations (see `docs/SCIENCE.md`), the validation method (see `docs/VALIDATION.md`), or dataset provenance (see `docs/DATA.md`).

## 2. Conventions

### 2.1 Units in field names (non-negotiable)

Every numeric field carries its unit in the field name. No bare numbers with ambiguous units. Radians exist only inside tight math helpers and are converted at boundaries.

| Concept | Field name convention | Value |
|---|---|---|
| Time (internal SPICE) | `epochTdbSec` | TDB seconds past J2000 |
| Time (recorded) | `createdUtc`, `syncedAtUtc` | ISO-8601 UTC, `Z` suffix |
| Longitude | `eastLonDeg` | degrees east-positive |
| Latitude | `latDeg` | degrees, north-positive |
| Elevation | `elevationDeg` | degrees (local horizontal reference) |
| Horizon azimuth step | `azimuthStepDeg` | degrees |
| Local frame azimuth | `azimuthDeg` (internal) | degrees, clockwise from North |
| Body radius | `radiusKm` | kilometers |
| Moon sphere radius | `sphereRadiusKm` | kilometers (1737.4) |
| Observer height | `observerHeightKm` | kilometers |
| Max ray range | `maxRayRangeKm` | kilometers |
| DEM resolution | `resolutionM` | meters per pixel |
| Distance | `distanceKm` | kilometers |
| Fit error (direction) | `fitErrorDirectionDeg` | degrees |
| Fit error (distance) | `fitErrorDistanceKm` | kilometers |

### 2.2 Frames, time, coordinates

- Moon body-fixed frame: `MOON_ME` (mean-Earth / mean-rotation axis, DE421-aligned DE440 pair — see `docs/DATA.md`).
- Time scale: `TDB` internally; UTC ISO-8601 `Z` for UI and user inputs.
- Longitude: east-positive. Azimuth: clockwise from North. Elevation: relative to the local horizontal plane (perpendicular to the local zenith), not the terrain slope.
- No atmosphere: no refraction anywhere.
- Moon sphere radius: **1737.4 km**.

## 3. manifest.json — pipeline data release

The release manifest is the producer's record of a data release.

### 3.1 Required fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `releaseVersion` | string | yes | Semantic version, e.g. `"1.0.0"`. TODO(decide) for the first assigned value. |
| `createdUtc` | string | yes | ISO-8601 UTC timestamp with `Z` suffix, e.g. `"2026-11-14T00:00:00Z"`. |
| `pipelineGitSha` | string | yes | Full 40-character commit SHA of the pipeline repository the release was built from. |
| `inputs` | array | yes | Input datasets used to build the release, with checksums. |
| `outputs` | array | yes | Output artifacts shipped by this release, with checksums. |

### 3.2 Input entries

Each input entry has a `type` discriminator and the checksum of the pinned input:

- Spice kernel input (`type: "spice_kernel"`): `name` (kernel file name) + `sha256`.
- DEM input (`type: "dem"`): `productName`, `version`, `sha256`, optional `labelSha256`, `resolutionM`.

Known kernel names and checksums are recorded in `docs/DATA.md`; the release manifest must use the same names. Known values: `naif0012.tls` (`678E32BDB5A744117A467CD9601CD6B373F0E9BC9BBDE1371D5EEE39600A039B`), `de440.bsp` (`A4CE9BF9B3282BECC9F4B2AC3CEBE03A2AE7599981AABD7265FD8482FFF7C4B5`), `moon_pa_de440_200625.bpc` (`60CD55AA401EA2EA97360636F567554BFE4E37BB829F901B4460A455DFAF783F`), `moon_de440_250416.tf` (`A47C71E9C9F33796BDAFB2C9D69A7EE447B6016ECAD80F71CD6F3E479F9CF768`); DEM product `LDEM_80S_20M` (`DB44C3B2444ACFF1AF528301C8A6988934B490249ACEFF0AC55668BDF3EAB365`).

### 3.3 Output entries

Each output entry references one versioned artifact:

| Field | Type | Required | Notes |
|---|---|---|---|
| `artifact` | string | yes | Artifact identifier, e.g. `"ephemeris_v1"`, `"horizon_masks_v1"`. |
| `fileName` | string | yes | File name inside the release. TODO(decide). |
| `sha256` | string | yes | SHA-256 checksum of the artifact file. |
| `bytes` | integer | yes | Size of the artifact file in bytes. |

### 3.4 Example

```json
{
  "releaseVersion": "1.0.0",
  "createdUtc": "2026-11-14T00:00:00Z",
  "pipelineGitSha": "0000000000000000000000000000000000000000000",
  "inputs": [
    { "type": "spice_kernel", "name": "de440.bsp", "sha256": "A4CE9BF9B3282BECC9F4B2AC3CEBE03A2AE7599981AABD7265FD8482FFF7C4B5" }
  ],
  "outputs": [
    { "artifact": "ephemeris_v1", "fileName": "ephemeris_v1.json", "sha256": "REPLACE_WITH_ACTUAL_SHA256", "bytes": 0 }
  ]
}
```

`REPLACE_WITH_ACTUAL_SHA256` is a placeholder; real releases must carry the actual artifact checksum.

## 4. ephemeris_v1

### 4.1 File shape

```json
{
  "schemaVersion": 1,
  "header": {
    "schemaVersion": 1,
    "kernels": [
      { "name": "naif0012.tls", "sha256": "678E32BDB5A744117A467CD9601CD6B373F0E9BC9BBDE1371D5EEE39600A039B" },
      { "name": "de440.bsp", "sha256": "A4CE9BF9B3282BECC9F4B2AC3CEBE03A2AE7599981AABD7265FD8482FFF7C4B5" },
      { "name": "moon_pa_de440_200625.bpc", "sha256": "60CD55AA401EA2EA97360636F567554BFE4E37BB829F901B4460A455DFAF783F" },
      { "name": "moon_de440_250416.tf", "sha256": "A47C71E9C9F33796BDAFB2C9D69A7EE447B6016ECAD80F71CD6F3E479F9CF768" }
    ],
    "frame": "MOON_ME",
    "aberrationCorrection": "apparent",
    "timeScale": "TDB",
    "units": {
      "time": "TDB seconds",
      "direction": "degrees",
      "distance": "kilometers"
    },
    "supportedStartEpochTdbSec": 0.0,
    "supportedEndEpochTdbSec": 0.0,
    "fitErrorDirectionDeg": 0.001,
    "fitErrorMeasuredDirectionDeg": 0.0
  },
  "segments": [
    {
      "bodyPair": "MOON_TO_SUN",
      "startEpochTdbSec": 0.0,
      "endEpochTdbSec": 0.0,
      "chebyshev": {
        "order": 0,
        "direction": [ 0.0 ],
        "distanceKm": [ 0.0 ]
      }
    },
    {
      "bodyPair": "MOON_TO_EARTH",
      "startEpochTdbSec": 0.0,
      "endEpochTdbSec": 0.0,
      "chebyshev": {
        "order": 0,
        "direction": [ 0.0 ],
        "distanceKm": [ 0.0 ]
      }
    }
  ]
}
```

The artifact schema version is carried in `schemaVersion` (kept in sync between the file envelope and `header.schemaVersion` by release tooling). Consumers must check the schema version before reading; an unsupported version is a hard error (see §6).

### 4.2 Header fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `schemaVersion` | integer | yes | Artifact schema version = 1 for this contract. |
| `kernels` | array of `{ name, sha256 }` | yes | SPICE kernels used for the fit, with checksums. |
| `frame` | string | yes | Moon body frame; value `MOON_ME` for this contract. |
| `aberrationCorrection` | string | yes | `apparent` for v1 (fit the aberrated/apparent direction, not geometric). |
| `timeScale` | string | yes | `TDB`. |
| `units` | object | yes | `{ "time": "TDB seconds", "direction": "degrees", "distance": "kilometers" }`. |
| `supportedStartEpochTdbSec` | number | yes | First supported epoch (TDB seconds past J2000). **TODO(decide)**: exact value. Documented rough window is ≈ 1550–2650. |
| `supportedEndEpochTdbSec` | number | yes | Last supported epoch (TDB seconds past J2000). **TODO(decide)**: exact value. |
| `fitErrorDirectionDeg` | number | yes | Fit error budget in degrees. Target from `docs/VALIDATION.md`: < 0.001°. |
| `fitErrorMeasuredDirectionDeg` | number | no | Measured fit error vs SPICE. **TODO(decide)**: measured value for the first release. |

### 4.3 Segment layout

`segments` is an array of contiguous segments covering the supported epoch window for each body pair.

Each segment:

| Field | Type | Required | Notes |
|---|---|---|---|
| `bodyPair` | string | yes | `MOON_TO_SUN` or `MOON_TO_EARTH`. |
| `startEpochTdbSec` | number | yes | Segment start (TDB seconds past J2000). |
| `endEpochTdbSec` | number | yes | Segment end (TDB seconds past J2000). |
| `chebyshev` | object | yes | `{ "order": <int>, "direction": <coefficients>, "distanceKm": <coefficients> }`. |

Open layout decisions (see §8): whether one segment per body pair is sufficient or the window must be split; the Chebyshev polynomial `order`; the meaning of `direction` coefficients (Moon→body vector components in the `MOON_ME` frame vs azimuth/elevation vs RA/Dec); representation of distance (`distanceKm` coefficients).

## 5. horizon_masks_v1

### 5.1 File shape

```json
{
  "schemaVersion": 1,
  "header": {
    "schemaVersion": 1,
    "demProductName": "LDEM_80S_20M",
    "demVersion": "1.0",
    "demSha256": "DB44C3B2444ACFF1AF528301C8A6988934B490249ACEFF0AC55668BDF3EAB365",
    "resolutionM": 20,
    "sphereRadiusKm": 1737.4,
    "observerHeightKm": 0.0,
    "azimuthStepDeg": 0.0,
    "maxRayRangeKm": 0.0,
    "status": "ok"
  },
  "sites": [ ... ]
}
```

### 5.2 Header fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `schemaVersion` | integer | yes | Artifact schema version = 1 for this contract. |
| `demProductName` | string | yes | DEM product name, e.g. `LDEM_80S_20M`. |
| `demVersion` | string | yes | DEM version, e.g. `1.0` (`LRO-L-LOLA-4-GDR-V1.0`). |
| `demSha256` | string | yes | SHA-256 of the DEM product. |
| `resolutionM` | number | yes | DEM resolution in meters per pixel (e.g. 20). |
| `sphereRadiusKm` | number | yes | Moon sphere radius, 1737.4. |
| `observerHeightKm` | number | yes | Observer height above the sphere. **TODO(decide)**: value. |
| `azimuthStepDeg` | number | yes | Azimuth sampling step. **TODO(decide)**: value. |
| `maxRayRangeKm` | number | yes | Maximum ray range for horizon masks. **TODO(decide)**: value. |
| `status` | string | yes | Coverage status: `ok`, `missing_coverage`, or `insufficient_coverage`. |

### 5.3 Per-site masks

Each site entry:

| Field | Type | Required | Notes |
|---|---|---|---|
| `id` | string | yes | Site identifier. |
| `eastLonDeg` | number | yes | Site east longitude, degrees. |
| `latDeg` | number | yes | Site latitude, degrees. |
| `elevationDeg` | array of numbers | yes | Horizon elevations in degrees, one per azimuth sample. **TODO(decide)**: azimuth indexing convention for the array (element index ↔ azimuth; start azimuth; inclusive end). |
| `status` | string | no | Per-site status override when coverage is missing/insufficient. |

## 6. data.lock.json and sync-and-verify behavior

The app repository pins a pipeline release in `data.lock.json` and syncs artifacts through a sync script. Format:

| Field | Type | Required | Notes |
|---|---|---|---|
| `lockSchemaVersion` | integer | yes | Lock file schema version. |
| `releaseVersion` | string | yes | Semver release version pinned from the pipeline repository. |
| `source` | object | yes | `{ "repository": "...", "releaseTag": "..." }` identifying the pipeline release. |
| `artifacts` | array | yes | Per-artifact pin: `{ "artifact", "fileName", "sha256", "bytes" }`. |
| `syncedAtUtc` | string | yes | ISO-8601 UTC timestamp of the last successful sync. |

Sync-and-verify behavior (must hold):

1. The sync script reads `data.lock.json` and resolves the pinned release (`releaseTag` = `v<releaseVersion>` convention is expected; exact convention **TODO(decide)**).
2. The sync script downloads the pinned artifacts.
3. The sync script computes SHA-256 and byte size of each downloaded artifact and compares against the lock entries.
4. **Checksum mismatch → fail loudly**: abort the sync, raise an error naming the artifact, the expected checksum and the actual checksum, and do not proceed to use or cache the mismatched data. Never silently fall back to another release or another artifact.
5. Missing release or missing artifact → fail loudly.
6. The app must refuse to consume artifacts whose schema version is newer than the supported consumer version (fail loudly).

## 7. Versioning rules

- Data artifact schema versions start at `1` (ephemeris, masks) and release-manifest format version `1`. These are the contracts defined in this file.
- **A breaking change requires a new schema version** (e.g. `ephemeris_v1` → `ephemeris_v2`) **and a human decision**. No breaking change is shipped silently.
- Breaking changes include: removal or renaming of a required field, type changes, and semantic changes (frame, time scale, aberration correction, supported epoch window, units, azimuth/elevation convention, status enum, fit methodology).
- Non-breaking changes: additive optional fields only; schema version stays; release version gets a MINOR bump.
- Release version (`releaseVersion`, semver) rules: data fixes reproducing the same contract → PATCH bump; additive data changes → MINOR bump; breaking contract change → MAJOR bump, new schema version, and human decision required.
- Science-semantic changes (frame, time scale, aberration correction, epoch window, sunlit convention) are breaking by default and additionally require the human approval flow described in `docs/AGENTS.md`.

## 8. Open items — TODO(decide)

Listed at the end as required. Each becomes a decision before the corresponding artifact ships.

- Assigned `releaseVersion` value for the first data release and release tag naming convention (`v<version>` vs `<version>`).
- Physical file names and release packaging layout (single release zip vs loose files) for `ephemeris_v1` and `horizon_masks_v1`.
- Exact supported ephemeris window: `supportedStartEpochTdbSec` and `supportedEndEpochTdbSec` values (documented rough window ≈ 1550–2650).
- Measured fit error vs SPICE: `fitErrorMeasuredDirectionDeg` value for the first release (target < 0.001° in direction).
- Ephemeris segment layout: number of segments per body pair, segment boundaries, Chebyshev polynomial order, direction-coefficient basis, distance representation.
- Horizon masks: shipped values for `observerHeightKm`, `azimuthStepDeg`, and `maxRayRangeKm`; azimuth-indexing convention for `elevationDeg[]`.
- Horizon masks: shipped `status` value for healthy coverage and final per-site status override semantics.
- `data.lock.json`: final field set (e.g. whether `bytes` and `source` are required), lock schema version, lock file location.
- Whether ephemeris and masks artifacts are single files as drafted or split per body pair / per site.
