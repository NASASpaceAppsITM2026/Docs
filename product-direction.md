# Product Direction — Lunar Landing Site Navigator

## The app in one sentence

Help a mission team decide whether a lunar landing plan still works when the landing date or touchdown location changes.

The app shows sunlight and Earth communication opportunities at a selected site and date. Its main planning features compare possible dates and nearby sites, then explain the trade-offs in language that both specialists and non-specialists can understand.

## What users do

1. Select a site on the map, enter coordinates, or choose a saved site.
2. Set the planned date and the conditions the mission needs, such as enough sunlight or a limit on the longest dark period.
3. Check the current plan, explore delayed dates, or compare alternatives.
4. Share a link or, when online collaboration is enabled, invite teammates to work in the same scenario.

The Sun/Earth geometry and terrain calculations remain deterministic and validated. The interface and any AI assistant explain or organize those calculations; they do not replace them.

## Features, in order

### 1. Slip-window finder

Answer: “If our landing date moves, which dates and nearby sites still meet our needs?” The user chooses measurable limits. The app checks a set of candidate sites over a range of dates and marks each result as meeting, missing, or not enough information to decide. It should also point out the next suitable date at the selected site and suitable alternatives for the original date.

This is a planning aid, not an optimizer that silently decides mission priorities. Show the limits used, the candidates checked, and the reason a result passed or failed.

### 2. Landing-area robustness

Answer: “Would the plan still work if the lander touched down a little away from the target?” Check points within a user-selected landing area or provided landing ellipse, then show how the results vary. A plain-language result might say that most checked points meet the sunlight limit, while some do not.

Do not imply a probability of landing at a point unless a defensible landing distribution was supplied. Label the area size, sample method, terrain-data resolution, and uncertainty. Report a range or percentiles only when their meaning and sampling basis are clear.

### 3. Shareable scenarios

A link should reopen the same sites, dates, limits, and view. Links contain no credentials, private notes, or large datasets. This works without accounts or live collaboration and is the first sharing feature to build.

### 4. Optional AI assistant

The user can ask for a change in everyday language, for example: “Show me backup dates two weeks later with at least this much sunlight.” The assistant turns the request into a visible, editable set of app actions. The user reviews and approves changes before they affect the shared scenario.

The assistant may select sites, change dates or limits, run the slip finder or landing-area check, compare results, and summarize results returned by the app. It may not calculate scientific values, invent missing results, or claim that a plan is safe. Every numeric answer comes from the deterministic engine; uncertainty and unmet constraints remain visible.

Keep the assistant optional. The core app must still work when the AI provider is unavailable or the user is offline. Use a provider-neutral interface around the approved app actions so another model or an authorized institutional AI could be connected later. Do not assume NASA provides an accessible AI service; confirm access, security, and terms before integrating one.

### 5. Live team workspace

When online collaboration is enabled, several people can view and update one scenario. Show who made each change. AI-generated changes are proposals until a person approves them. Define what happens when two people edit the same setting, and provide a way to undo or restore earlier scenario states.

Live collaboration requires an online service to synchronize changes. It is a later feature, separate from the offline app and shareable links. Never put private scenario data or service credentials in a URL.

## Architecture and trust

```text
Offline-capable app
  validated science engine -> metrics and events -> map, comparisons, reports
  saved scenario state -> shareable link

Optional online services, only when enabled
  AI provider adapter -> approved app actions -> user-reviewed changes
  collaboration service -> shared scenario changes and participant identity
```

- The science engine owns the numbers. AI providers and collaboration services cannot change its formulas or bypass its checks.
- The static app and science calculations remain useful without a network.
- Online features must be clearly identified and must not be required to inspect a result or use the core planning workflow.
- Ask for permission before adding an external or paid AI provider or a hosted collaboration service. Document data sent, retention, access control, and failure behavior before implementation.

## Why this app, given existing tools?

Do not claim that terrain views, sunlight analysis, 3D views, site ranking, or AI operation are unique. The initial landscape notes identify substantial overlap:

| Tool or category | Existing overlap to account for | Possible distinction to test |
|------------------|---------------------------------|-------------------------------|
| Lunar QuickMap | Lunar maps, terrain and illumination/visibility analysis, and 3D context | A guided, date-and-site decision workflow with explicit limits and side-by-side outcomes |
| NASA PGDA visibility products | Long-term Sun/Earth visibility maps | Date-specific launch-slip comparisons rather than relying only on long-term averages |
| NASA Moon Trek | 3D lunar terrain and horizon visualization | A direct path from a mission question to comparable site/date results |
| Professional mission-analysis tools | Illumination analysis and multi-criteria site selection | A simpler shared workflow for a hackathon/demo audience, not a claim of higher-fidelity analysis |
| New landing-site APIs and ML projects | Safe, sunlit, or accessible site screening | Connecting date-dependent power and communication windows with landing-location sensitivity |

These are leads from the project's initial research notes, not a current feature-by-feature audit. Recheck each product's current capabilities and test the actual workflow before using the table as an external product claim. Do not say that another tool lacks a feature based only on not finding it.
Research starting points from the supplied notes: [QuickMap overview](https://files.actgate.com/quickmap/LROC_QuickMap_Jan2020.pdf), [NASA PGDA visibility product](https://pgda.gsfc.nasa.gov/products/69), [lunar site-selection study](https://pmc.ncbi.nlm.nih.gov/articles/PMC6398960), [landing-site API example](https://dev.to/irisdatalabs/our-lunar-landing-site-api-goes-live-today-1hig), and [ML site-ranking example](https://dev.to/alan_scottencinas/lunarsite-an-end-to-end-ml-pipeline-for-lunar-south-pole-site-selection-2lda). These are starting points, not endorsements or proof that a feature is absent elsewhere.

The intended distinction is the connected decision workflow: compare dates and candidate sites against explicit mission needs, show sensitivity to touchdown location, and let a team review the same scenario.

Treat the market gap as a hypothesis, not a proven fact. The initial research found little direct user feedback; that is not evidence that users have no complaints. Before making a “first,” “only,” or “no other tool” claim, test current competitor workflows and ask likely users to complete a task such as comparing a delayed date at two candidate sites. Ask where they got stuck, what information they trust, and what they currently do by hand. Record what was tested and what remains unknown. Direct user feedback has not yet been established in these project notes.

## What not to build first

- Do not add AI before the calculations and the slip/landing-area results are validated.
- Do not build accounts or live collaboration before shareable scenarios and the core workflow are useful.
- Do not make runtime access to NASA, QuickMap, Horizons, or an AI service a requirement for core calculations.
- Do not rank sites using hidden or invented mission priorities. Let users see and change the criteria.
