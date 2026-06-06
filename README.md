# Region Conquest

A mobile-friendly territory capture game playable in any browser. Choose a map, claim provinces, and outsmart the AI to control the most connected territory.

## Gameplay

- Players take turns claiming provinces on the map
- The goal is to maximize **connected territory** (fewer separate groups = stronger position)
- The AI scores moves based on adjacency and piece reduction
- Game ends when all provinces are claimed — highest territory wins

## Maps

| Map | Regions |
|-----|---------|
| 🇹🇷 Turkey | 81 provinces |
| 🇺🇸 USA | 49 states |
| 🇫🇷 France | 95 departments |

## Modes

- **2 Player** — local multiplayer on the same device
- **Easy / Normal / Hard** — single player vs AI

## Tech Stack

- Vanilla HTML + CSS + JavaScript (single file, no framework)
- [D3.js v7](https://d3js.org/) for map rendering
- GeoJSON for province/state boundaries
- SVG-based interactive map

## How to Play

Just open `files/BolgeKapma_Mobil.html` in a browser — no server or install needed.

All assets (D3.js, GeoJSON files) are bundled locally so the game works fully offline.

## Project Structure

```
files/
  BolgeKapma_Mobil.html          # Main game file
  d3.v7.min.js                   # D3.js (local)
  tr-cities-utf8.json            # Turkey GeoJSON
  us-states.json                 # USA GeoJSON
  departements-version-simplifiee.geojson  # France GeoJSON
turkiye_komsu_iller.csv          # Turkey province adjacency data
```
