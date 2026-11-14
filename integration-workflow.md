# Lunar Landing Site Navigator — Integrated Agent Workflow

## Recommended Data-Flow Pipeline

### Step 1: Select Candidate Site (QuickMap)
- Select a site in the app, enter coordinates, or choose a saved candidate.
- Use QuickMap as an external reference for terrain context and coordinates;
	do not require it at runtime for the app's science calculations.

### Step 2: Compute Celestial Positions (Horizons)
- The app's science core computes Sun and Earth positions from the shipped,
	precomputed ephemeris and converts them to local azimuth/elevation.
- Horizons is an independent validation reference. Fetch fixtures separately;
	never make live Horizons calls a requirement for using the app or running CI.

### Step 3: Apply Terrain Horizon Mask (QuickMap + QTS)
- The app uses its locally available, documented terrain data and horizon mask
	to compute sunlight and Earth visibility.
- Cross-check terrain and illumination results against QuickMap/QTS and
	published LOLA studies; these services are references, not runtime
	dependencies.

### Step 4: Visualize in 3D (QuickMap Cesium / Moon Trek)
- Inspect the app's plots and timelines for the selected site and date.
- Use QuickMap or Moon Trek as external visual references where useful; do not
	treat a visual match alone as scientific validation.

### Step 5: Mission Context (CSA / Artemis)
- Add mission context only when a specific, verified mission requirement calls
	for it. CSA and Artemis materials provide context, not ephemeris or terrain
	calculations.

### Step 6: Compare plans and share
- Run the slip-window finder across the selected candidate sites, dates, and
	visible mission limits.
- Check landing-area robustness only when the landing area, sampling method,
	and terrain resolution are stated.
- Share a URL for a reproducible scenario. Use a live workspace only when an
	explicitly enabled collaboration service is available.
- An optional AI assistant may propose app actions and summarize computed
	results. The user reviews changes; the deterministic engine supplies every
	scientific value.

## Key Compatibility Gotchas
- **Longitude:** Horizons east longitude (0–360°) matches QuickMap `isLongitude360=true` — no sign flip
- **Latitude:** Both use standard geodetic latitude (positive north)
- **Altitude:** Horizons `SITE_COORD` altitude is above ellipsoid in km; QuickMap `Res` is m/px
- **Body IDs:** Sun `10`, Earth `399`, Moon `301`, Moon center `500@301`
- **AZ-EL is topocentric only** — must set a surface `CENTER`, not body center
- **API version:** Check JSON `signature.version` (currently `1.0`; API doc is `1.3`)

## Optional online services

The core app, scenario links, and deterministic calculations must remain useful
without these services. Before adding an AI provider or collaboration backend,
document what data leaves the browser, access controls, retention, cost, outage
behavior, and how a user disables the feature. Never place service credentials
in browser code or shared URLs.