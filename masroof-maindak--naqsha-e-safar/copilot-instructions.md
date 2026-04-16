## naqsha-e-safar

> This project aims to identify and classify **public transit deserts** in Lahore

# Context: Public Transit Deserts in Lahore

## Overview

This project aims to identify and classify **public transit deserts** in Lahore
— areas with **high population density** but **low access to public transport**.
The work combines open geographic and demographic data sources and applies
**graph-based machine learning** to analyze spatial accessibility and urban
mobility equity.

## Data Sources

- **WorldPop** – Provides gridded population density data for Lahore. Used to
  estimate demand for public transit.
- **CityLines** – Contains public transport routes and stop locations (bus,
  metro, etc.). Used to assess transit coverage.
- **OpenStreetMap (OSM)** – Supplies the road and pedestrian network topology.
  Used to compute travel distances and build the spatial graph.

## Core Problem

Traditional methods measure transit access via straight-line distance or simple
buffer zones. These ignore real network connectivity and spatial context. This
project instead models the city as a **graph**, where:

- Nodes represent spatial units (e.g., H3 hexagons or grid cells).
- Edges represent adjacency or road-network connectivity.
- Node features include population density, distance to nearest transit stop,
  road density, and other local metrics.

A **Graph Neural Network (GNN)** is trained to classify each node (region) as:

- “Transit desert” — high demand, low accessibility.
- “Transit served” — adequate proximity to public transport.

## Objectives

1. Build a spatial graph representation of Lahore integrating demographic and
   transport data.
2. Train and evaluate a GNN to identify underserved regions.
3. Compare results to baseline distance-based and tabular ML models.
4. Produce an interpretable accessibility map highlighting potential areas for
   new transit routes or stops.

## Expected Output

- A reproducible pipeline (data preprocessing → graph construction → model
  training → evaluation).
- Transit accessibility maps of Lahore.
- A paper-style report and presentation summarizing findings.

## Implementation Notes

- Environment Management: `uv` is used for dependency management and running
  scripts/notebooks.
- Code Quality: GitHub Actions are set up for automated checks:
  - Prettier for Markdown file formatting.
  - Ruff for Python linting and formatting.
- Data Storage: OSMnx processing outputs (e.g., network graphs, POIs) are stored
  in the `data/osmnx` directory.
- Frameworks: OSMnx and GeoPandas for spatial processing.

## Summary

In short, this project combines **WorldPop**, **CityLines**, and
**OpenStreetMap** data to detect **transit deserts in Lahore** using **graph
neural networks**, generating actionable insight for equitable urban transport
planning.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/masroof-maindak) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
