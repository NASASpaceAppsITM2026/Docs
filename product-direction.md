# Product Direction — Lunar Landing Site Navigator

## The app in one sentence

Help a mission planner decide whether a lunar landing plan still works when the landing date or touchdown location changes.

The app shows sunlight and Earth communication opportunities at a selected site and date. An AI assistant translates natural-language requests into scenario changes, runs comparisons, and explains the trade-offs — all while the 3D lunar visualization remains the primary view.

## What users do

1. Start in the 3D lunar scene — the Moon, Earth, and Sun are the main visual focus.
2. Tell the AI what the mission needs, for example: "Find me dates two weeks later with at least six hours of sunlight."
3. The AI sets the site, date, and constraints, runs the analysis, and updates the visualization.
4. Fine-tune the result manually if desired, or share a link to reproduce the scenario.

The Sun/Earth geometry and terrain calculations remain deterministic and validated. The AI operates the app; it does not calculate scientific values.

## Visual direction

The Moon is the main subject of the app and the first visual focus. Open on a
large, interactive lunar model with the selected landing site visible. Show
Earth and the Sun as contextual 3D bodies, with their directions and apparent
disc sizes driven by the selected site and date.

Offer two related views using the same scene: a Moon overview for spatial
context, and a site-level view showing Earth and Sun against the local terrain
horizon. Compress distances in the overview so all three bodies fit on screen;
label it “not to scale.” The site-level view must communicate the computed
azimuth, elevation, terrain obstruction, and disc visibility rather than
implying that the composition is an exact-scale solar-system model.

Keep the map available for precise site selection, but make it a secondary
panel or view. Date controls, sunlight/communication timeline, comparison
results, and constraints should support and explain the 3D scene without
competing with the Moon for visual priority.

## Features, in order

### 1. Slip-window finder

Answer: “If our landing date moves, which dates and nearby sites still meet our needs?” The user chooses measurable limits. The app checks a set of candidate sites over a range of dates and marks each result as meeting, missing, or not enough information to decide. It should also point out the next suitable date at the selected site and suitable alternatives for the original date.

This is a planning aid, not an optimizer that silently decides mission priorities. Show the limits used, the candidates checked, and the reason a result passed or failed.

### 2. Landing-area robustness

Answer: “Would the plan still work if the lander touched down a little away from the target?” Check points within a user-selected landing area or provided landing ellipse, then show how the results vary. A plain-language result might say that most checked points meet the sunlight limit, while some do not.

Do not imply a probability of landing at a point unless a defensible landing distribution was supplied. Label the area size, sample method, terrain-data resolution, and uncertainty. Report a range or percentiles only when their meaning and sampling basis are clear.

### 3. AI chat

A chat panel at the bottom of the screen lets the user describe mission needs in natural language. The AI parses the request, proposes scenario changes (site, date, constraints), and applies them after the user approves. The chat also summarizes results, explains why a site-date combination passes or fails, and suggests alternatives.

The AI may select sites, change dates or limits, run the slip finder or landing-area check, and summarize results returned by the app. It may not calculate scientific values, invent missing results, or claim that a plan is safe. Every numeric answer comes from the deterministic engine; uncertainty and unmet constraints remain visible.

The chat is the primary input method, but manual controls remain available for fine-tuning. Keep the AI optional — the core app must still work when the AI provider is unavailable or the user is offline. Use a provider-neutral interface so another model could be connected later.

### 4. Shareable scenarios

A link should reopen the same sites, dates, limits, and view. Links contain no credentials, private notes, or large datasets.

## Architecture and trust

```text
Single-user app
  validated science engine -> metrics and events -> 3D scene, charts, reports
  AI chat -> approved app actions -> scenario changes -> science engine
  saved scenario state -> shareable link
```

- The science engine owns the numbers. The AI operates the app; it cannot change formulas or bypass checks.
- The AI chat is the primary interface, but manual controls remain for fine-tuning.
- The static app and science calculations remain useful without a network or AI provider.
- Ask for permission before adding an external or paid AI provider. Document data sent, retention, access control, and failure behavior before implementation.

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

The intended distinction is the connected decision workflow: an AI assistant translates natural-language mission needs into scenario changes, compares dates and candidate sites against explicit limits, shows sensitivity to touchdown location, and explains results — all centered around an immersive 3D lunar visualization.

Treat the market gap as a hypothesis, not a proven fact. The initial research found little direct user feedback; that is not evidence that users have no complaints. Before making a “first,” “only,” or “no other tool” claim, test current competitor workflows and ask likely users to complete a task such as comparing a delayed date at two candidate sites. Ask where they got stuck, what information they trust, and what they currently do by hand. Record what was tested and what remains unknown. Direct user feedback has not yet been established in these project notes.

## What not to build first

- Do not make runtime access to NASA, QuickMap, Horizons, or an AI service a requirement for core calculations.
- Do not let the AI calculate scientific values — every number comes from the deterministic engine.
- Do not rank sites using hidden or invented mission priorities. Let users see and change the criteria.
- Do not hide manual controls behind the AI — the user must always be able to fine-tune manually.
