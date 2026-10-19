# WildfireTwinLA

A browser-based wildfire digital twin for exploring fire spread, comparing prevention strategies, and replaying public wildfire observations on a 3D map.

**[Try the live demo → wildfiretwinla.vercel.app](https://wildfiretwinla.vercel.app/)**

The current demo focuses on the **2026 Norte Fire near San Juan Capistrano and San Clemente in Southern California**. It combines terrain, satellite-derived vegetation, fuel models, buildings, and archived weather and hotspot observations in one shared simulation grid.

Built for the **InnovateLA × USC Viterbi Dual-Use Hackathon**, with a focus on disaster preparedness and response.

## Explore the demo

- **Respond:** Replay the Norte Fire using archived public observations. Follow reported ignitions, satellite hotspot detections, changing winds, and evacuation-order records alongside the modeled spread.
- **Plan:** Explore prevention scenarios for the same area. Compare manual fuel treatments and home hardening with a budget-constrained planning algorithm, and inspect their modeled effect on community risk.
- **Emergency contact simulation:** Watch simulated outreach scheduling as homes become threatened. The public demo uses synthetic households and runs its dispatcher in a browser worker.

The hosted demo runs in the browser without a backend. Its replay data is bundled with the app. Live data feeds require a separately configured backend and are disabled on the standalone site.

> WildfireTwinLA is an experimental, non-official decision-support demo. Modeled spread and alerts are not forecasts or evacuation orders. Follow local authorities for emergency instructions. Community boundaries are approximate; satellite detections and provisional fire extents must not be treated as official perimeters.

## How it works

The Three.js frontend renders terrain, buildings, vegetation, fire, and smoke. A GPU cellular automaton models spread using slope, wind, fuel, and ember spotting. Shared TypeScript algorithms support arrival-time analysis, risk evaluation, and treatment planning.

The current area uses a **512 × 512 grid with 25 m cells in EPSG:32611**. The offline Python pipeline aligns geospatial inputs to this grid. The FastAPI backend supports recorded replay, optional live ingestion, incident clustering, WebSocket delivery, resident opt-in, and optional Retell integration.

| Input | Sources used by the project |
| --- | --- |
| Terrain | USGS 3DEP elevation |
| Vegetation and fuel | Copernicus Sentinel-2 L2A and LANDFIRE FBFM40 |
| Buildings and roads | OpenStreetMap |
| Hotspot observations | NASA FIRMS VIIRS/MODIS; optional GOES-18 ingestion |
| Weather | Archived HRRR winds via Open-Meteo; optional live weather polling |
| Incident and comparison data | NIFC WFIGS records and explicitly labeled provisional extents where an official perimeter is unavailable |

Norte is the default dataset. The earlier Eaton replay remains available for compatibility. See the [Norte scenario and data boundaries](docs/norte-fire.md) for provenance and limitations.

## Run locally

Use **Node.js 24+**, **Python 3.11+**, and [uv](https://docs.astral.sh/uv/). The committed Norte data is sufficient to run the default demo; no API keys or GIS downloads are required.

### Frontend

From the repository root:

```bash
cd web
npm ci
npm run dev
```

Open the local address printed by Vite. In development, the frontend tries the backend on port `8010` and falls back to its bundled replay when the backend is unavailable.

### Optional backend

In a separate terminal, from the repository root:

```bash
cd server
uv sync --frozen --extra dev
.venv/bin/uvicorn app.main:app --reload --port 8010
```

Run from `server/` so the default replay and grid paths resolve correctly. Pollers and real calls are disabled by default. Check `/healthz` and `/readyz` for service status.

For live ingestion or Retell configuration, follow the [server README](server/README.md) and its `.env.example`. Keep credentials in backend environment variables or an untracked `.env` file; never put secrets in frontend `VITE_` variables.

### Data pipeline

```bash
cd data-pipeline
uv sync --frozen --extra dev
```

The [pipeline guide](data-pipeline/README.md) covers source downloads, grid generation, community rasterization, and replay generation. Refreshing external datasets is optional for running the committed demo.

## Build and validate

Run each command from the repository root:

```bash
# Frontend type check and production build
npm --prefix web run build

# Shared simulation and planning tests
npm --prefix packages/core test

# Norte scenario and calibration tests
node --experimental-transform-types --test analysis/norte.test.ts
python -m unittest discover -s analysis -p 'test_*.py'

# Backend tests and lint (after backend setup)
(cd server && .venv/bin/pytest -q && .venv/bin/ruff check .)

# Pipeline tests and lint (after pipeline setup)
(cd data-pipeline && .venv/bin/pytest -q && .venv/bin/ruff check .)
```

For the standalone production build, leave `VITE_API_URL` unset. The repository's `vercel.json` builds from the repository root and publishes `web/dist`; the frontend also imports shared code and replay data outside `web/`. See the [deployment guide](docs/deploy.md).

## Repository guide

| Directory | Purpose |
| --- | --- |
| `web/` | Three.js interface, GPU simulation, replay, and browser dispatcher |
| `packages/core/` | Shared grid contracts, spread analysis, and planning algorithms |
| `server/` | FastAPI API, ingestion, replay, alerts, and optional outreach integration |
| `data-pipeline/` | Offline geospatial preprocessing and dataset generation |
| `analysis/` | Calibration, validation, and Norte scenario experiments |
| `data/norte/` | Archived public inputs and scenario data |
| `docs/` | Design, data provenance, deployment, and team contracts |

## Further reading

- [Technical design](docs/technical-design.md)
- [Backend architecture](docs/backend-design.md)
- [Norte Fire scenario](docs/norte-fire.md)
- [Shared interfaces and team plan](docs/team-plan.md)
- [Terrain wind-field format](docs/wind-field.md)
- [Backend delivery and compliance notes](docs/backend-delivery-and-compliance.md)
