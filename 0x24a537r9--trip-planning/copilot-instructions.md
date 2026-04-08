## trip-planning

> Useful sources for research (add specific URLs or tools as discovered):

## Research Tools & Sources

Useful sources for research (add specific URLs or tools as discovered):

- Trails: AllTrails, HikingProject, state park trail finders
- Hot springs: ranger district websites for seasonal conditions, recreation.gov
- Tide tables: NOAA Tides and Currents (tidesandcurrents.noaa.gov)
- Road conditions: state DOT sites, USFS road condition reports
- Fungi / naturalist: iNaturalist observation data by region and month
- Accommodations: Booking.com, Airbnb, Hotels.com, direct motel websites
- Passes: Recreation.gov (America the Beautiful Pass), state park annual passes

## Map Production

The plan must include a KML file (`plan/route.kml`) per `specs/14-map-deliverable.md`.

### How to generate the KML

Produce `plan/route.kml` as a standard KML 2.2 file containing:

1. **Route lines** — `<LineString>` placemarks tracing the northbound and southbound paths along actual roads (10–30 coordinate points is sufficient; this is spatial orientation, not turn-by-turn navigation)

2. **Lodging pins** — one `<Point>` placemark per night, labeled `Night 1 — [Town]`, etc.

3. **Key area pins** — `<Point>` placemarks for each significant natural area in the plan, labeled descriptively (e.g., `Humbug Mountain — coastal hiking`)

4. **Style** — use distinct `<Style>` blocks with color differentiation for lodging vs. natural areas vs. route lines

### Offline delivery to the traveler

Include in the plan document:

```
## Using the Map Offline

1. **Google My Maps (pre-trip):** Go to mymaps.google.com → Create → Import → upload `route.kml`.
   Gives a shareable, visual, editable map.

2. **Gaia GPS (offline in the field):** In Gaia GPS app → Import → select `route.kml`.
   All pins are visible offline once imported. Recommended for field use.

3. **Google Maps:** Open `route.kml` in Google Drive, then "Open with Google My Maps."
   Save the map areas for offline use in the Google Maps app.
```

### Google Maps segment links

For each day's primary driving segment, include a Google Maps URL in the plan:
`https://www.google.com/maps/dir/[origin]/[destination]`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/0x24a537r9)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/0x24a537r9)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
