# KICKOFF.md — Starting the Lunar Landing Site Navigator (multi-repo)

This is a script for the human coordinator. Agents do not run it; you paste the prompts into them, one step at a time.

Read order for every agent, in every repo: `AGENTS.md` → `docs/product-direction.md` → `docs/development-process.md` → `docs/data-contracts.md` (once it exists).

First milestone: **one Apollo site and one date where our Sun azimuth/elevation matches Horizons within 0.01°.** No new features until that passes.

---

## 1. Repos

Keep the number small. Every extra repo adds syncing work in a 48-hour sprint.

| Repo | Contains | Agents working here |
|------|----------|---------------------|
| `lunar-navigator-pipeline` (Python, uv) | SPICE kernel handling, Chebyshev fits, DEM processing, horizon masks, data release packaging | Ephemeris pipeline, terrain pipeline |
| `lunar-navigator-app` (TypeScript, pnpm) | `packages/core` science library, `packages/web` UI, Horizons fixtures, golden tests | Core ephemeris, validation, terrain core, web shell, planning features |
| `Docs` | The shared docs, prompts, this file | You |

Every repo gets its own copy of `AGENTS.md`, `docs/product-direction.md` and `docs/data-contracts.md`, plus a short "What this repo does and does not do" section at the top of its `AGENTS.md`. Agents only read the `AGENTS.md` in the folder they start from.

**Repos talk through versioned data files, not shared code.** The pipeline repo publishes a data release (a tagged zip with a manifest and checksums). The app repo pins that release in `data.lock.json` and a sync script verifies the checksums. Nobody copies files by hand.

---

## 2. Order of work

```
1  Research prompt (R)             any agent with web access, edits docs/DATA.md only
2  Data-contracts prompt (C)       writes docs/data-contracts.md; you review and copy it to every repo
3  Bootstrap prompts (P, Q)        one agent per repo, in parallel
4  Parallel workstreams (W1-W5)    after both bootstraps merge
5  First validation gate           before any planning feature
6  Planning features and UI        slip-window finder first
```

### Branches and worktrees

- **One branch per task, per repo** (for example `ephemeris-fit`). It lets you review before merging and keeps `main` working for the repos that depend on it. If a run goes badly, delete the branch.
- Subagents inside one agent session share that task's branch.
- **Two agents in the same repo at the same time** need separate branches and separate folders (`git worktree add ../app-terrain terrain-core`). Otherwise they overwrite each other's files.
- Merge by pull request, small and often. You review contracts, dataset choices and validation results.

---

## 3. Prompts

### Prompt R — Research (agent with web access; edits only `docs/DATA.md`)

```text
Read AGENTS.md and docs/DATA.md. Do not edit any file except docs/DATA.md.

Verify from authoritative sources only (NAIF, NASA PDS Geosciences Node, PGDA) and fill in:
1. Exact SPICE kernel filenames and versions for: leap seconds, DE440-family planetary ephemeris,
   the lunar orientation kernel that covers our supported date window, and the matching lunar frame kernel.
   State whether a DE421-aligned mean-Earth frame variant exists and which one matches Horizons conventions.
2. The exact LOLA product(s) covering the south pole (80S and nearby): resolution, projection,
   height scale and offset, file format, download URL.
3. SHA-256 checksums only if you actually downloaded the files; otherwise write TODO(verify).
Give the source URL for each item. Mark anything you could not confirm as TODO(verify). Do not guess.
```

### Prompt C — Data contracts (any agent; one branch; you review the result)

```text
Read AGENTS.md and docs/product-direction.md. Create only docs/data-contracts.md. Write no code.

Define, precisely enough that two different agents in two repos produce and consume compatible files:
1. manifest.json for a pipeline data release: releaseVersion (semver), createdUtc, pipelineGitSha,
   input datasets with checksums (kernels, DEM product), output files with checksums.
2. ephemeris_v1: header (schemaVersion, kernel names and checksums, frame MOON_ME, aberration correction used,
   time scale TDB, units, supported start and end epoch, fit error budget measured vs SPICE) and the segment
   layout for Chebyshev coefficients of Moon->Sun and Moon->Earth direction and distance.
3. horizon_masks_v1: header (DEM product name, version, checksum, resolution, sphere radius 1737.4 km,
   observer height, azimuth step, maximum ray range, status for missing or insufficient coverage) and per-site
   masks (id, eastLonDeg, latDeg, elevationDeg[]).
4. The data.lock.json format used by the app repo to pin a release, and the sync-and-verify behavior
   (fail loudly on checksum mismatch).
5. Versioning rules: a breaking change requires a new schema version and a human decision.
Use units in field names. Mark anything not yet decided as TODO(decide) and list those at the end.
Do not invent datasets, filenames or coordinates.
```

### Prompt P — Bootstrap, pipeline repo (one agent)

```text
Read AGENTS.md, docs/product-direction.md, docs/development-process.md and docs/data-contracts.md.
Goal: scaffold lunar-navigator-pipeline. Do NOT implement ephemeris fitting, DEM processing or any science.

Tasks:
1. Python 3.11+ project managed with uv: pipeline/ephemeris/, pipeline/terrain/, pipeline/release/, tests/.
2. Make these work and document them in AGENTS.md: install, lint, test.
3. pipeline/release/: a manifest writer and a schema validator for docs/data-contracts.md (with unit tests
   using small synthetic examples).
4. Stub entry points that raise NotImplemented: download_kernels, build_ephemeris_fits, build_horizon_masks.
5. A script that packages outputs plus manifest into a release zip and prints the checksums.
Rules: no network calls in tests; no invented URLs, filenames or coordinates (use TODO(verify)).
Finish with a report: files created, commands run and their results, open questions.
```

### Prompt Q — Bootstrap, app repo (one agent)

```text
Read AGENTS.md, docs/product-direction.md, docs/development-process.md and docs/data-contracts.md.
Goal: scaffold lunar-navigator-app. Do NOT implement science or UI features yet.

Tasks:
1. pnpm workspace with packages/core and packages/web, tests/fixtures/, scripts/.
2. Make these work and document them in AGENTS.md: install, typecheck, lint, test, build,
   `pnpm --filter core test:golden` (may be a skipped placeholder for now).
3. packages/core/src/contracts.ts: typed, documented contracts with units in names:
   LunarSite, SkyState, HorizonMask, Constraint, Verdict ('meets' | 'misses' | 'unknown'; missing data is never 'meets'),
   ScenarioState (versioned, JSON-serializable, URL-safe, no secrets), plus function signatures that throw NotImplemented:
   skyAt, horizonMaskFor, visibleDiscFraction, sunlitAndEarthVisibleWindows, evaluateConstraints, slipWindowFinder.
4. scripts/sync_data and data.lock.json: pin a pipeline release and verify checksums per docs/data-contracts.md.
   Use a small synthetic release for the test.
5. One smoke test per package.
Rules: no network calls in code or tests; do not invent datasets, filenames, URLs or coordinates.
Finish with a report: files created, commands run and results, open questions.
```

**Gate:** both repos pass install, typecheck/lint, tests and build on a clean clone, and you have read the contracts. Changing a contract after this point is your decision, because every workstream depends on it.

---

## 4. Parallel workstreams

Start W1 to W4 together. Start W5 after the first validation gate.

| ID | Repo | Owns | Done when |
|----|------|------|-----------|
| W1 Ephemeris pipeline | pipeline | `pipeline/ephemeris/` | Produces `ephemeris_v1` with **aberrated (apparent)** directions and a measured fit error; needs verified kernel names (Prompt R) |
| W2 Core ephemeris | app | `packages/core/src/ephemeris/` | `skyAt` evaluates the fits for any site and time in the window; tested first against a synthetic fit, then the real one |
| W3 Validation | app | `scripts/fetch_horizons.*`, `tests/fixtures/horizons/`, `packages/core/test/golden/` | Fixtures committed with full query URLs (`CENTER='coord@301'`, quantities 1 and 4, `CSV_FORMAT=YES`, `TIME_DIGITS=FRACSEC`); golden test compares by angular separation and wrap-aware azimuth within 0.01° |
| W4 Terrain | pipeline (+ core reader in app) | `pipeline/terrain/`, `packages/core/src/terrain/` | Horizon masks for candidate sites with exact sphere geometry, observer height, stated range and resolution; synthetic-terrain tests; needs the verified polar DEM product |
| W4b Web shell | app | `packages/web/` | Moon-centered 3D scene with Moon, Earth, Sun, and selected site; secondary map/coordinate/site selector; date controls and result view using mocked core; URL-serializable `ScenarioState`; accessibility basics |
| W5 Planning | app | `packages/core/src/planning/` | `slipWindowFinder` with meets/misses/unknown and reasons, bounded candidates and dates, boundary and tie tests |

Dependencies: W3 can start immediately, because its fixtures do not depend on W1 or W2. W2 starts against a synthetic fit file that follows the contract. W4b uses mocks.

### Workstream prompt template

```text
Read AGENTS.md, docs/product-direction.md, docs/development-process.md and docs/data-contracts.md first.

Repo: <repo>   Workstream: <ID and name>   Branch: <branch>
You own only: <paths>. Do not edit anything else. If a contract or shared file needs to change, stop and ask.

Goal: <one sentence>
Definition of done: <from the table>, tests passing, docs updated.

Rules:
- Never invent datasets, filenames, URLs, coordinates or mission facts. Use TODO(verify) and report it.
- No network calls in the app or in tests. Fixtures are fetched by scripts and committed.
- Never loosen a tolerance, delete a failing test or regenerate a fixture to make a build pass.
- Keep units in names and follow AGENTS.md section 5 exactly.
- Small commits (Conventional Commits). Run typecheck, lint and tests before reporting.

Final report (required): what changed, commands run and their output, what you could not verify,
assumptions made, and any science or validation result with numbers. Be honest about failures.
```

---

## 5. Handing data between repos

1. The pipeline agent finishes a build, tags a release, and records its checksums in the manifest.
2. You (or the app-repo agent, on its own branch) update `data.lock.json` to the new version and run the sync script.
3. Golden tests run in the app repo against the new data. If they fail, the release does not get used.
4. Record the validated release version in `docs/VALIDATION.md`.

---

## 6. First validation gate

- `pnpm --filter core test:golden` passes offline for Apollo sites and −89.5° polar sites across several dates.
- `docs/VALIDATION.md` has real numbers (ΔAZ, ΔEL, fit error).
- If agreement misses 0.01°, stop adding features and find the cause (frame, time scale, light-time, aberration, site definition). Do not loosen the tolerance.

After the gate: slip-window finder (W5), timeline and comparison views, shareable links, AI chat panel. Landing-area robustness comes after those, as described in `docs/product-direction.md`.

---

## 7. Rules for running several agents

- One owned path set per agent; reviews and merges are yours.
- Do not start an agent on a task whose inputs are unverified (kernels, DEM). Run Prompt R first.
- If agents disagree about a science convention, `AGENTS.md` section 5 wins; if it is silent, ask the human.
- Keep the Moon-centered 3D scene as the primary interface; cut decorative model detail before demoting or removing it. Never cut Horizons validation, offline core use, or honest uncertainty.
