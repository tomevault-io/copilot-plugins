## dutch-choropleth-maps

> Use this skill when the user wants to create a map of the Netherlands showing data by gemeente, wijk, or buurt — a choropleth map. Trigger on 'map', 'choropleth', 'kaart', 'geographical visualization', 'show on a map', or any request to visualize data spatially across Dutch regions. Covers merging statistical data with PDOK geodata, CRS conversion, and rendering with geopandas, folium, and plotly. For fetching CBS data, use cbs-statline-skill first. For non-map charts, use data-viz-journalism instead.


## Purpose

Create choropleth maps of Dutch statistical data at gemeente, wijk, or buurt level. This skill handles the full pipeline: from a DataFrame with region codes to a rendered map.

## When to use

User wants to show data spatially on a map of the Netherlands — by gemeente, wijk, or buurt. Keywords: "kaart", "map", "choropleth", "geografisch", "per gemeente op de kaart", "show on a map".

## When NOT to use

- **No statistical data yet** → use `cbs-statline-skill` first to get a DataFrame with region codes, then return here.
- **Non-map charts** → use `data-viz-journalism` for bar charts, line charts, etc.

## Prerequisites

The user should have a pandas DataFrame with a column containing CBS region codes (GM, WK, or BU codes). If they don't, tell them to use `cbs-statline-skill` first.

## Required packages

```bash
uv pip install geopandas folium matplotlib plotly requests
# of: pip install geopandas folium matplotlib plotly requests
```

---

## The mapping pipeline

Follow these steps in order.

---

### Stap 1: Detecteer het geografisch niveau

Auto-detect from the region codes in the data:

```python
# Detecteer geografisch niveau op basis van regiocodes
voorbeeld_code = df['regio_code'].dropna().iloc[0].strip()

if voorbeeld_code.startswith('BU'):
    niveau, pdok_laag = 'buurt', 'buurt_gegeneraliseerd'
elif voorbeeld_code.startswith('WK'):
    niveau, pdok_laag = 'wijk', 'wijk_gegeneraliseerd'
elif voorbeeld_code.startswith('GM'):
    niveau, pdok_laag = 'gemeente', 'gemeente_gegeneraliseerd'
else:
    raise ValueError(f"Onbekend regiocode-formaat: {voorbeeld_code}")

print(f"Gedetecteerd niveau: {niveau} — laden: {pdok_laag}-grenzen")
```

---

### Stap 2: Haal geografische grenzen op

**Primaire bron: PDOK WFS** (officieel, altijd actueel)

Read `references/pdok-endpoints.md` for the complete URL template and year-availability table.

```python
import geopandas as gpd

jaar = 2024  # Gebruik hetzelfde jaar als je statistische data!

wfs_url = (
    f"https://service.pdok.nl/cbs/gebiedsindelingen/{jaar}/wfs/v1_0"
    f"?request=GetFeature&service=WFS&version=2.0.0"
    f"&typeName={pdok_laag}&outputFormat=json"
)
print(f"Downloaden van: {wfs_url}")
geo = gpd.read_file(wfs_url)
print(f"Gedownload: {len(geo)} gebieden, CRS: {geo.crs}")
```

**Fallback: cartomap/nl** (gebruik bij PDOK-timeout of buurt-niveau met 13.000+ polygonen)

```python
# Vereenvoudigde grenzen, al in WGS84 (EPSG:4326)
geo = gpd.read_file(
    f"https://cartomap.github.io/nl/wgs84/gemeente_{jaar}.geojson"
)
# Voor wijk: f"https://cartomap.github.io/nl/wgs84/wijk_{jaar}.geojson"
# Voor buurt: f"https://cartomap.github.io/nl/wgs84/buurt_{jaar}.geojson"
```

---

### Stap 3: Converteer het coördinatenstelsel

PDOK levert data in **EPSG:28992** (Rijksdriehoekscoördinaten). Webkaarten hebben **EPSG:4326** (WGS84) nodig.

```python
print(f"Huidig CRS: {geo.crs}")

# Voor webkaarten (Folium, Plotly): converteren naar WGS84
geo_wgs84 = geo.to_crs(epsg=4326)

# Voor statische kaarten (geopandas): RD New behouden voor minste vervorming
# geo_rd = geo  # EPSG:28992 behoudt Nederlandse verhoudingen
```

Read `references/crs-guide.md` for details on when to use which CRS.

---

### Stap 4: Koppel statistische data aan geodata

```python
# Normaliseer de koppelsleutels — PDOK gebruikt 'statcode'
geo['statcode'] = geo['statcode'].str.strip()
df['regio_code'] = df['regio_code'].str.strip()

samengevoegd = geo.merge(df, left_on='statcode', right_on='regio_code', how='left')

# Controleer de koppelkwaliteit
n_gekoppeld = samengevoegd[samengevoegd['waarde'].notna()].shape[0]
n_totaal = geo.shape[0]
print(f"Gekoppeld: {n_gekoppeld}/{n_totaal} gebieden ({n_gekoppeld/n_totaal*100:.1f}%)")

# Toon ongekoppelde gebieden — mogelijk een jaarmismatch
ongekoppeld = samengevoegd[samengevoegd['waarde'].isna()]
if len(ongekoppeld) > 0:
    print(f"\n⚠️  {len(ongekoppeld)} ongekoppelde gebieden:")
    print(ongekoppeld[['statcode', 'statnaam']].head(10).to_string(index=False))
    print("\n→ Controleer of geodatajaar overeenkomt met statjaar!")
```

**Kritieke valkuil — gemeentelijke herindelingen**: Nederland fuseer regelmatig gemeenten. GM-codes veranderen daarbij. Gebruik **altijd geodata uit hetzelfde jaar als je statistische data**. Zie `references/pdok-endpoints.md` voor het jaar-specifieke URL-patroon.

---

### Stap 5: Render de kaart

Kies een optie op basis van het beoogde gebruik:

---

#### Optie A: Statische kaart met geopandas (voor print / rapporten)

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(1, 1, figsize=(10, 12))

samengevoegd.plot(
    column='waarde',
    cmap='YlOrRd',
    legend=True,
    legend_kwds={
        'label': 'Eenheid hier invullen',
        'shrink': 0.6,
        'orientation': 'vertical'
    },
    missing_kwds={'color': 'lightgrey', 'label': 'Geen data'},
    ax=ax,
    edgecolor='white',
    linewidth=0.3
)

ax.set_axis_off()
ax.set_title(
    'De bevinding als kop, niet de variabele',
    fontsize=16, pad=20, fontweight='bold'
)
ax.annotate(
    'Bron: CBS StatLine, PDOK gebiedsindelingen',
    xy=(0.02, 0.02), xycoords='axes fraction',
    fontsize=9, color='grey'
)

plt.tight_layout()
import os; os.makedirs('output', exist_ok=True)
plt.savefig('output/kaart.png', dpi=300, bbox_inches='tight')
plt.savefig('output/kaart.svg', bbox_inches='tight')
print("Kaart opgeslagen als output/kaart.png en output/kaart.svg")
```

---

#### Optie B: Interactieve kaart met Folium (voor webartikelen)

```python
import folium

# Gebruik de WGS84-versie voor Folium
samengevoegd_wgs84 = samengevoegd.to_crs(epsg=4326)

m = folium.Map(
    location=[52.1, 5.3],
    zoom_start=7,
    tiles='cartodbpositron'
)

folium.Choropleth(
    geo_data=samengevoegd_wgs84.to_json(),
    data=df,
    columns=['regio_code', 'waarde'],
    key_on='feature.properties.statcode',
    fill_color='YlOrRd',
    fill_opacity=0.7,
    line_opacity=0.2,
    legend_name='Eenheid hier invullen',
    nan_fill_color='lightgrey',
    nan_fill_opacity=0.4
).add_to(m)

# Tooltips toevoegen
folium.GeoJson(
    samengevoegd_wgs84,
    tooltip=folium.GeoJsonTooltip(
        fields=['statnaam', 'waarde'],
        aliases=['Gemeente:', 'Waarde:'],
        localize=True
    ),
    style_function=lambda x: {'fillOpacity': 0, 'weight': 0}
).add_to(m)

os.makedirs('output', exist_ok=True)
m.save('output/kaart.html')
print("Interactieve kaart opgeslagen als output/kaart.html")
```

---

#### Optie C: Interactief met Plotly (voor dashboards)

```python
import plotly.express as px

samengevoegd_wgs84 = samengevoegd.to_crs(epsg=4326)

fig = px.choropleth_mapbox(
    samengevoegd_wgs84,
    geojson=samengevoegd_wgs84.geometry.__geo_interface__,
    locations=samengevoegd_wgs84.index,
    color='waarde',
    hover_name='statnaam',
    hover_data={'waarde': ':.1f'},
    mapbox_style='carto-positron',
    center={'lat': 52.1, 'lon': 5.3},
    zoom=6,
    color_continuous_scale='YlOrRd',
    title='De bevinding als titel'
)
fig.update_layout(margin={"r": 0, "t": 40, "l": 0, "b": 0})

os.makedirs('output', exist_ok=True)
fig.write_html('output/kaart.html')
print("Plotly-kaart opgeslagen als output/kaart.html")
```

---

## Journalism rules for maps

- **Titel formuleert de bevinding**, niet de variabele: "Noord-Groningen loopt achter op energietransitie" — niet "Aardgasaansluitingen per gemeente"
- **Kleurschalen**: gebruik sequentieel (YlOrRd, YlGnBu) voor waarden/aantallen; gebruik divergerend (RdBu) voor afwijking van gemiddelde
- **Legenda altijd aanwezig** met duidelijke eenheden
- **Grijze gebieden** = geen data — verberg ze niet
- **Bronvermelding**: "Bron: CBS StatLine, PDOK gebiedsindelingen"
- **Buurt/wijk-kaarten**: overweeg om alleen één gemeente ingezoomd te tonen — landsdekkende buurtkaarten zijn onleesbaar

---

## Colour scale selection

```python
# Sequentieel — voor hoeveelheden (bijv. % zonnepanelen)
cmap = 'YlOrRd'    # Geel → Oranje → Rood (hoog = donker)
cmap = 'YlGnBu'   # Geel → Groen → Blauw

# Divergerend — voor afwijking van gemiddelde
cmap = 'RdBu_r'   # Rood = boven gemiddelde, Blauw = onder gemiddelde

# Afwijking van gemiddelde berekenen voor divergerende schaal:
samengevoegd['afwijking'] = samengevoegd['waarde'] - samengevoegd['waarde'].mean()
# Gebruik dan column='afwijking' en cmap='RdBu_r'
```

---

## Output format

The map output must include:

1. **Kaartbestand** — PNG + SVG voor statisch; HTML voor interactief
2. **Alt-tekst** — één zin die beschrijft wat de kaart toont
3. **Bronvermelding** — "Bron: CBS StatLine, PDOK gebiedsindelingen {jaar}"

Voorbeeld alt-tekst: "Choroplethkaart van Nederland op gemeenteniveau. Donkere kleuren geven een hoog percentage woningen met zonnepanelen aan. Zeeland en Noord-Brabant vallen op als koplopers."

---

## Extended reference

For complete PDOK WFS URL templates per level and year, the 2023 endpoint migration, field name mapping, gegeneraliseerd vs. niet-gegeneraliseerd, cartomap/nl fallback URLs, and rate limits:

→ Read `references/pdok-endpoints.md`

For CRS details (EPSG:28992, EPSG:4326, EPSG:3857) and when to convert:

→ Read `references/crs-guide.md`

---
> Source: [linksmith/dutch-choropleth-maps](https://github.com/linksmith/dutch-choropleth-maps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
