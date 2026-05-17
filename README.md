# Building Codes Map

Interactive map of building codes by jurisdiction. **v1 covers the United Kingdom** — England, Scotland, Wales, and Northern Ireland — with link-out to the official government sources.

Built with [Astro](https://astro.build), [MapLibre GL JS](https://maplibre.org/), and [Tailwind CSS v4](https://tailwindcss.com/).

## What it does

- Renders a clean, light basemap (OpenFreeMap Positron) zoomed to the UK.
- Click a nation → side panel lists every Approved Document / Technical Handbook / Technical Booklet that applies, with one-click links to the canonical PDF on the relevant government site.
- Address search bar (top centre) — type a UK address, the map flies there and auto-selects the containing nation.
- "Ask about codes" floating chat button (bottom right) — **UI only**, no LLM wired up yet. See [Wiring up the chat](#wiring-up-the-chat) below.

## Run it

> Requires Node.js 18+ (LTS recommended).

```bash
npm install
npm run dev
```

Open <http://localhost:4321>.

```bash
npm run build      # static site → dist/
npm run preview    # serve the build locally
```

The entire app is statically generated. No server, no API keys, no tracking.

## Project structure

```
.
├── public/data/
│   ├── uk-nations.geojson     # 4 nation polygons (ONS, ultra-generalised)
│   └── uk-codes.json          # link-out metadata per region
├── src/
│   ├── components/
│   │   ├── Map.astro          # MapLibre + region click handling
│   │   ├── Sidebar.astro      # Slide-in panel listing codes
│   │   ├── AddressSearch.astro# Nominatim search bar
│   │   └── ChatStub.astro     # LLM chat UI (stubbed)
│   ├── layouts/Layout.astro
│   ├── pages/index.astro
│   └── styles/global.css
└── astro.config.mjs
```

## Data sources

| Region | Source | Licence |
|---|---|---|
| England — Approved Documents A–T, 7 | [gov.uk/approved-documents](https://www.gov.uk/government/collections/approved-documents) | OGL v3.0 |
| Scotland — Technical Handbooks | [gov.scot](https://www.gov.scot/policies/building-standards/) | OGL v3.0 |
| Wales — Approved Documents (Wales) | [gov.wales](https://www.gov.wales/building-regulations-guidance-approved-documents) | OGL v3.0 |
| Northern Ireland — Technical Booklets | [finance-ni.gov.uk](https://www.finance-ni.gov.uk/articles/building-regulations-technical-booklets) | OGL v3.0 |
| 4-nation boundaries | [ONS Open Geography Portal](https://geoportal.statistics.gov.uk/) — Countries (Dec 2023) BUC | OGL v3.0 |
| Basemap | [OpenFreeMap](https://openfreemap.org/) Positron style | ODbL |
| Address search | [Nominatim](https://nominatim.openstreetmap.org/) (OSM) | ODbL |

The app **does not host or re-publish** any code documents. All links resolve to the canonical PDF on the original government site.

## Wiring up the chat

The chat panel currently echoes back a stub. To wire it to an LLM:

1. Add an Astro API route at `src/pages/api/chat.ts` (or a serverless function on your host).
2. The client already knows the active region — extend the `bcm-chat-form` submit handler in `ChatStub.astro` to POST `{ message, region }` to your endpoint.
3. On the server, fetch the region's `documents[].url` PDFs, extract text, build a RAG prompt, call Claude / OpenAI / etc., and stream the response back.

Cost-wise: PDF extraction can be done once at build time and cached as plain text per document.

## Roadmap

- **v0.2** — add a second country (Australia → NCC + Victorian variations; or USA → ICC link-out)
- **v0.3** — drill-down: counties / local authorities in England, councils in Scotland
- **v0.4** — section-level deep links inside each PDF (where the gov publishes them)
- **v0.5** — wire up the LLM chat with RAG over indexed code text

## Licence

Source code: MIT (see `LICENSE` if added).
Data: each upstream source's licence applies (OGL v3.0 for government data, ODbL for OSM-derived).
