# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

Learning project. No code yet. The full brief is in `dagster-geospatial-project-prompt.md`.
Update this file when phases add real commands and structure.

## Goals

1. Learn **Dagster** in depth: assets, partitions, sensors, asset checks, backfills, idempotency, deployment.
2. Learn **geospatial** fundamentals: vector and raster processing, projections, GeoPandas, rasterio, GDAL, QGIS.
3. Make a credible portfolio piece for an application to Overstory (vegetation intelligence for electric utilities, uses Dagster).

## Working mode (important)

- The user is a senior Python / ML engineer (computer vision, Earth observation).
- Understanding comes before speed. Before each step, explain the concept and the reason for the choice in a few sentences.
- Work in small, testable steps. The user writes the code. Do not generate full implementations.
- For key parts (asset definition, partitioning, asset check, sensor): let the user write a first version, or give a skeleton to complete. Then review it.
- Point out pitfalls and alternatives (Airflow, Prefect) when they help the learning.
- Check Dagster APIs and commands against the current official documentation (`dg` CLI, scaffolding). Do not rely on memory.
- The user works on a VPS over SSH, with VS Code and a Jupyter server.

## Project: vegetation proximity risk map for power lines

Public data in, risk map per power-line segment out. Pilot area: Morbihan (France), partitioned by municipality or grid square. Targets can change during the project.

### Target asset graph

1. `osm_power_lines`: `power=line` features from OpenStreetMap (Geofabrik or Overpass), cleaned, stored as GeoParquet.
2. `line_segments`: lines split into ~100 m segments with deterministic IDs.
3. `corridor_buffers`: buffers around segments (e.g. 15 m and 30 m).
4. `canopy_height_tiles`: Meta/WRI canopy height tiles (AWS Open Data), read by window from COGs. Do not download everything.
5. `segment_vegetation_stats`: per segment max height, mean height, share of area above a height threshold.
6. `segment_risk_score`: simple, explainable score.
7. `risk_map`: GeoPackage export for QGIS, plus a small web map.

### Dagster requirements

- Partitions by geographic area. A new territory is a new partition.
- Idempotency: a rerun of a partition gives the same result. Write atomically (temp file, then rename).
- Asset checks: no segment with missing values, valid geometries, plausible heights (0–80 m), complete tile coverage.
- Backfill demo: a change to the threshold or scoring method recomputes only assets 5–7.
- At least one sensor (e.g. new OSM data update).
- Asset metadata in the UI: segment count, median height, duration.
- Business logic is in plain functions with pytest tests, independent of Dagster.

### Geospatial rules

- Use Lambert-93 (EPSG:2154) for all distances, buffers, and lengths. Use WGS84 only for data exchange.
- Document data limitations (OSM quality, canopy resolution, data date).
- Explain each spatial operation (reprojection, buffer, zonal statistics) when it first appears.

## Stack and conventions

- Python with `uv` and `pyproject.toml`. Lint with `ruff`. Test with `pytest`.
- GeoPandas, rasterio, shapely, pyogrio. Add no unnecessary dependencies.
- Data goes in `data/` (gitignored). Never commit large files.
- Docker and `docker compose` come at the end (Phase 5).
- Small, clear commits.

## Phases

- **Phase 0**: this file.
- **Phase 1**: scaffold the Dagster project with the current official method. Add a trivial asset. Confirm `dagster dev` works.
- **Phase 2**: prototype the computation on ONE municipality in a notebook or script, without Dagster.
- **Phase 3**: turn each step into an asset, then add partitions.
- **Phase 4**: asset checks, tests, sensor, backfill demo.
- **Phase 5**: Docker, README (graph diagram, partitioning and idempotency choices, "Airflow equivalent" section).

Stop after each phase and summarize before the next one.
