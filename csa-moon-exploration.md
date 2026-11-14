# Canadian Space Agency (CSA) Moon Exploration — Data Source Context for AI Agent

## Overview
- **URL:** CSA Moon Exploration portal (`https://www.asc-csa.gc.ca/eng/astronomy/moon-exploration/`)
- **Operator:** Canadian Space Agency (CSA / ASC)
- **Type:** Mission portal covering past, current, and future lunar missions

*(Note: URL provided in the NASA Lunar Landing Site Navigator challenge description. The following is based on the challenge description and the portal content.)*

## How It Works
The CSA Moon Exploration portal provides information about:
- Canada's past, current, and future missions to the Moon
- Canadian technologies for lunar exploration
- Canadian scientific research on the Moon
- Canada's participation in the **Artemis program**

## What Data It Shows
- Mission descriptions and timelines
- Canadian lunar technology contributions (robotics, rovers, instruments)
- Science research priorities and findings
- Artemis program involvement and partnership details

## Science Notes for the Agent
- Relevant for **mission context, partnerships, and Artemis-aligned landing-site rationale**
- Useful for understanding Canadian rover/robotics capabilities and south-pole interest that may inform landing site selection
- **Not** a source of ephemeris, terrain tiles, or celestial position data

## Agent Usage Pattern
- Cross-reference CSA/Artemis context for mission-aligned site justification
- Identify Canadian-contributed instruments or rovers that may operate at selected sites
- Align landing site choices with Artemis program objectives and international partner capabilities

## Integration with Other Sources
| Source | Role |
|--------|------|
| CSA Moon Exploration | Mission context, Artemis alignment, Canadian tech/science |
| QuickMap | Terrain imagery + DEM + coordinates |
| Horizons | Sun/Earth celestial positions (AZ/EL) |
| Moon Trek | 3D horizon-position visualization reference |

*(CSA portal URL: https://www.asc-csa.gc.ca/eng/astronomy/moon-exploration/. The portal provides mission context, Artemis alignment, and Canadian lunar technology contributions. No programmatic API confirmed — if needed, inspect the portal endpoints manually.)*