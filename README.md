# EarthLens: Earth Intelligence

NASA Space Apps Challenge 2026, Preliminary Round submission.

EarthLens is a web application for exploring long-term environmental trends using real NASA data. It combines refreshable NASA data sources, a world trend map, and a browser-side statistical engine that reports the direction, size, and significance of change at each location.

Live demo: <add link here>
Team: <add team name here>

## Features

- World trend map and table for 16 sampled coordinates, with location search and sorting
- Four NASA POWER variables: air temperature at 2 m, precipitation, relative humidity at 2 m, and wind speed at 10 m (1981-2025)
- NASA GISTEMP v4 annual land-ocean temperature anomalies in eight latitude bands (1880-2025)
- Adjustable time intervals and confidence levels (90%, 95%, 99%)
- Side-by-side comparison of up to four locations against a common baseline
- Add any coordinate (up to 40 study points) and fetch its NASA data directly
- Import your own annual CSV data, processed locally and clearly labeled as not independently verified
- Save a workspace on the device, or export portable JSON projects, results CSV, observations CSV, SVG charts, and standalone HTML reports
- Data sources page with provenance, coverage, and last successful fetch time

## Data sources

| Source | Content | Coverage |
| --- | --- | --- |
| NASA POWER / MERRA-2 | Temperature, precipitation, humidity, wind (model grid-cell values) | 1981-2025 |
| NASA GISTEMP v4 | Temperature anomalies relative to 1951-1980 | 1880-2025 |
| Natural Earth | Map geometry (public domain) | n/a |

All built-in scientific values are actual NASA downloads. POWER values are model grid-cell samples, not whole-region averages. The app shows the latest published complete years and does not claim real-time conditions. No NASA API key is needed.

## Statistical methods

- Linear trend with Newey-West HAC uncertainty (default)
- Ordinary least squares with Student's t inference
- Mann-Kendall test with Sen's slope
- Benjamini-Hochberg q-values across all eligible locations to control the false discovery rate
- At least eight annual observations are required, and missing years are never interpolated

Statistical significance does not establish causality, practical importance, or future change.

## Tech stack

- Frontend: HTML, CSS, and vanilla JavaScript in `public/`
- Server: Vinext (Next.js on Vite) running as a Cloudflare Worker
- Data refresh: `server/nasa.mjs` with fixed NASA endpoints, validation, and cache fallback
- Tooling: pnpm, TypeScript, Tailwind CSS, Drizzle ORM, Wrangler
- Data snapshot script: Python (`fetch_data.py`)

## Getting started

Requirements: Node.js 22.13 or newer and pnpm 11.25.0. Internet access is needed to install dependencies and fetch fresh NASA data.

```sh
pnpm install --frozen-lockfile
pnpm dev
```

Then open http://localhost:5173.

To build and run the compiled app:

```sh
pnpm build
pnpm start
```

The NASA API routes need the Worker runtime, so opening `index.html` directly or using a plain static file server will not work.

## API routes

- `GET /api/nasa/power?lat=23.8&lon=90.4` fetches NASA POWER data for a coordinate
- `GET /api/nasa/gistemp` fetches the NASA GISTEMP table

Add `refresh=1` to bypass the server cache.

## Refreshing the data snapshots

```sh
python fetch_data.py
```

This rewrites the NASA snapshots in `public/assets`. It does not change the map geometry.

## Tests

```sh
node tests/test-stats.cjs
node tests/test-import.cjs
node tests/test-nasa.mjs
node tests/test-ui.cjs
node tests/test-workspace.cjs
node tests/test-feeds.cjs
python tests/reference-check.py
```

The UI and workspace tests need jsdom (set `EARTHLENS_JSDOM_PATH` if it is installed elsewhere). The Python reference check needs NumPy, SciPy, and statsmodels.

## Project structure

```
app/api/nasa/     NASA API routes (POWER and GISTEMP)
server/           NASA fetching, validation, and caching
public/           Interface, statistics engine, and bundled NASA data
frontend/         HTML served by the server root route
tests/            Statistical, import, UI, workspace, and NASA refresh tests
build/, scripts/  Framework and Worker integration
fetch_data.py     Reproducible data snapshot downloader
```

Keep `frontend/index.html` and `public/index.html` in sync when editing the interface.

## Acknowledgements

Data provided by NASA POWER and NASA GISS. Map geometry from Natural Earth. NASA affiliation or endorsement is not implied.

## License

<add license here>
