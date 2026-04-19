## mission-planning

> Satellite mission planning domain knowledge and scientific conventions


# Domain Knowledge — Satellite Mission Planning

**Model Decision: Use when working on orbit, visibility, scheduling, SAR, or mission planning logic**

## Core Concepts

- **TLE (Two-Line Element)**: Standard format for describing satellite orbits. Source: CelesTrak.
- **Orbit Propagation**: Predicting satellite position over time using SGP4/SDP4 (via `orbit-predictor`).
- **Ground Track**: The path traced on Earth's surface by the point directly below the satellite.
- **Visibility/Pass**: Time window when a satellite is above a ground target's elevation mask.
- **Elevation Mask**: Minimum angle above horizon (degrees) for a target to "see" the satellite.
- **CZML**: JSON-based format for time-dynamic 3D visualization in Cesium.

## Satellite Imaging

- **SAR (Synthetic Aperture Radar)**: Active imaging, works day/night and through clouds.
  - Modes defined in `config/sar_modes.yaml` (Spotlight, Stripmap, ScanSAR, etc.).
  - SAR has specific look angles, swath widths, and resolution parameters.
- **Optical**: Passive imaging, requires sunlight and clear weather.
  - Typically narrower FOV (sensor_fov_half_angle_deg ~1°) vs SAR (~30°).
- **Sensor FOV**: Half-angle in degrees defining the camera cone on the ground target.

## Scheduling & Optimization

- **MissionScheduler**: Uses PuLP (linear programming) or greedy algorithms to optimize acquisition schedules.
- **AlgorithmType**: Enum selecting optimization strategy (greedy, priority-weighted, ILP, etc.).
- **Roll/Pitch**: Satellite can slew to point its sensor. Constrained by max angles and slew rates (°/s).
- **Settle Time**: Time for satellite attitude to stabilize after a slew maneuver.
- **Conflict Resolution**: When multiple targets are in view simultaneously, scheduler must choose.
- **Quality Scoring**: Multi-criteria model combining geometry, timing, priority, and weather factors.
- **Weight Presets**: Pre-configured scoring weights (in `WEIGHT_PRESETS`) for different mission types.

## Ground Infrastructure

- **Ground Stations**: Defined in `config/ground_stations.yaml`. Used for downlink scheduling.
- **Targets**: Defined per-mission with lat/lon/elevation_mask/priority. Priority scale: 1 (lowest) to 5 (highest).
- **Batch Policies**: Rules for grouping acquisitions, defined in `config/batch_policies.yaml`.

## Units & Conventions

- Latitude/Longitude: decimal degrees, WGS84 datum.
- Altitudes: kilometers (orbital), meters (ground).
- Angles: degrees at all API boundaries.
- Time: UTC always. ISO-8601 format for serialization.
- Duration: hours for mission planning, seconds for maneuver timing.
- Slew rates: degrees per second.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/panos-dim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
