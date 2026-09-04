# kg-class-demo

A single self-contained `index.html` page with three tabs:

1. **Hirakata Weather** – a 7-day forecast for Hirakata, Osaka Prefecture, Japan.
2. **Hirakata vs Bergen** – the same 7 days side by side for Hirakata, Japan and Bergen, Norway:
   summary-difference cards, a high-temperature line chart with the day-by-day gap shaded, a
   grouped rainfall bar chart, and a day-by-day card grid.
3. **Norway Earthquakes** – 50 years of earthquakes in and around Norway (M ≥ 3.0, 1976–2026),
   shown as an epicentre map, a year/magnitude scatter, a per-year bar chart, stat cards
   and a "strongest events" table.

No build step, no dependencies. All charts and the map are hand-drawn inline SVG; all data is
baked into the file, so it works offline once loaded.

## Viewing it

- **Live:** [DEMO](https://compiledex.github.io/kg-class-demo/)
- **Local:** open `index.html` in any browser.

## Data sources

| Data | Source | Notes |
|------|--------|-------|
| Weather forecast | [Open-Meteo](https://open-meteo.com/) | Hirakata and Bergen, snapshot taken 4 Sep 2026; values are hard-coded, not live. |
| Earthquakes | [USGS FDSN event catalogue](https://earthquake.usgs.gov/fdsnws/event/1/) | Region box 58–71.5°N, 3–31°E; magnitude ≥ 3.0. Includes mainland, North Sea, Norwegian Sea and border zones. |
| Coastline | [Natural Earth](https://www.naturalearthdata.com/) 1:110m (via [world.geo.json](https://github.com/johan/world.geo.json)) | Low-resolution outlines for Norway, Sweden and Finland. |

### Caveats

- The forecast is a fixed snapshot – it will not update. Regenerate it for a current forecast.
- Earthquake catalogue completeness for small magnitudes varies over time. The 1980s–90s spike
  in yearly counts is mostly improved detection plus mining-induced tremors near Kiruna–Gällivare,
  not a real increase in seismic activity.
- Epicentres are projected with a simple equirectangular (plate carrée) projection with a
  cos-latitude correction – fine for a schematic map, not survey-grade.

## Regenerating the data

The page is assembled by a small Python script that fetches the raw data and inlines it:

```bash
# 1. Earthquakes -> CSV
curl -s "https://earthquake.usgs.gov/fdsnws/event/1/query?format=csv&starttime=1976-01-01&endtime=2026-09-04&minmagnitude=3&minlatitude=58&maxlatitude=71.5&minlongitude=3&maxlongitude=31&orderby=time" -o no_quakes.csv

# 2. Coastlines
for c in NOR SWE FIN; do
  curl -sL "https://raw.githubusercontent.com/johan/world.geo.json/master/countries/$c.geo.json" -o "$c.json"
done

# 3. Weather forecasts (Hirakata, then Bergen)
DAILY="weather_code,temperature_2m_max,temperature_2m_min,apparent_temperature_max,precipitation_sum,precipitation_probability_max,wind_speed_10m_max,relative_humidity_2m_mean"
curl -s "https://api.open-meteo.com/v1/forecast?latitude=34.8144&longitude=135.6497&daily=$DAILY&timezone=auto&forecast_days=7" -o hirakata.json
curl -s "https://api.open-meteo.com/v1/forecast?latitude=60.3913&longitude=5.3221&daily=$DAILY&timezone=auto&forecast_days=7" -o bergen.json

# 4. Build index.html
python3 build_index.py
```

(`build_index.py` and the intermediate `quakes.json` / `land.json` / `hir.json` / `ber.json`
are kept in the scratchpad used to author this demo, not committed.)

## License

Educational demo. Data belongs to its respective providers (USGS – public domain;
Open-Meteo – CC BY 4.0; Natural Earth – public domain).
