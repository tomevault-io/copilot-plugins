## global-mining-prototype

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a global mining dataset prototype repository containing geospatial and analytical data related to global mining operations. The repository currently contains:

- **global-mining-dataset.kmz**: KML/KMZ geospatial data file containing mining location data (contains doc.kml when extracted)
- **global-mining-dataset.xlsx**: Excel spreadsheet with mining dataset information
- **research_global-mining-insights-report.pdf**: Research report with global mining insights

## Working with Data Files

### KMZ Files
- KMZ files are compressed KML files (Google Earth format)
- To extract: `unzip global-mining-dataset.kmz`
- Contains geospatial data in XML format

### Excel Files
- Use appropriate Python libraries (pandas, openpyxl) or tools to read/analyze the XLSX data
- Binary format - cannot be directly read as text

## Git Repository

- Remote: https://github.com/tth4vb/studiohackathon.git
- Main branch: main
- Initial repository state with data files committed

## Development Recommendations

When working with this data:
1. Consider using Python with libraries like pandas for Excel data analysis
2. For KML/KMZ processing, consider using libraries like simplekml or fastkml
3. Extract and parse the KMZ file before processing the geographic data

---
> Source: [tth4vb/global_mining_prototype](https://github.com/tth4vb/global_mining_prototype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
