# 令和8年熊本地震（2026-07-28）被害焦点 AOI

2026 Kumamoto Earthquake — damage-focus AOI for HOT Tasking Manager / fAIr / MapSwipe.

M7.1, 2026-07-28 16:27 JST. Max JMA intensity **7** at Uki City (Toyono) and Hikawa Town (Shimaji).

## Why this exists

As of 2026-07-30 there is **no official building-damage dataset**. FDMA report #17
(2026-07-29 16:30) lists housing damage only for Fukuoka (4 partially damaged);
the Kumamoto column is blank. Neither GSI nor MLIT publishes priority-affected-area
polygons.

So this AOI is built from **proxies**, in this priority order:

1. **Observed JMA intensity** — the primary proxy for building damage
2. **Individually reported structural damage** — collapses, bridge failure, road uplift
3. **Whether post-event GSI orthophoto already exists** — you cannot map what has not been flown

Water-outage figures are included but **demoted to a reference attribute**: a broken
distribution main does not imply building damage.

## Data

| File | Content |
|---|---|
| [`public/data/aoi.json`](public/data/aoi.json) | 16 municipality polygons with damage proxies. Boundaries from OSM via Nominatim |
| [`public/data/gsi-ortho-coverage.json`](public/data/gsi-ortho-coverage.json) | Measured footprint of the GSI post-event orthophoto (48 tiles at z13) |

Live: `https://yuiseki.github.io/earthquake-kumamoto-2026-07/data/aoi.json`

## The operational finding

**Post-event imagery does not cover where the damage most likely is.**

Coverage measured by probing `https://maps.gsi.go.jp/xyz/20260729kumamoto_yatsushiro_0729do_sokuho/{z}/{x}/{y}.png`
with 40 sampled points per municipality at z15.

| Municipality | JMA intensity | Post-event ortho | Reported structural damage |
|---|---|---:|---|
| Hikawa 氷川町 | **7** | **52%** | 2 houses buckled |
| Yatsushiro 八代市 | 6+ | **35%** | factory chimney collapse; Uyanagi bridge (Pref. Rd 42) collapse; up to 84 cm crustal deformation |
| Ashikita 芦北町 | 6- | **51%** | — |
| Kamiamakusa 上天草市 | 6- | 6% | — |
| **Uki 宇城市** | **7** | **3%** | road uplift on National Rd 3 |
| **Kashima 嘉島町** | not listed | **0%** | **shopping-mall 2nd floor collapse, 2 deaths** |
| Kumamoto 熊本市 | 6+ | **0%** | — |
| Uto 宇土市 | 6+ | **0%** | — |
| Misato 美里町 | 6+ | **0%** | — |
| Mashiki 益城町 | 6+ | **0%** | — |
| Kosa 甲佐町 | not listed | **0%** | 1 death |
| Mifune 御船町 | 6- | 0% | — |
| Koshi 合志市 / Ozu 大津町 / Nishihara 西原村 | 6- | 0% | — |
| Amakusa 天草市 | 5+ | 0% | — |

GSI has flown only the **Yatsushiro district** (Yatsushiro / Hikawa / Ashikita).
The whole northern cluster is uncovered, including the other intensity-7 municipality
(Uki, 3%) and the municipality with the worst reported structural failure and 2 deaths
(Kashima, 0%).

**Practical consequence**: fAIr / MapSwipe can start immediately on Hikawa, Ashikita and
Yatsushiro. For Uki, Kumamoto-south, Uto, Kashima, Mashiki and Kosa a new flight is
needed before any imagery-based damage detection is possible.

Two of the three damage clusters are outside current imagery:

- **Coastal north–south axis** (~50 km): Kumamoto-south → Uto → Uki → Hikawa → Yatsushiro
- **Inland east** (roughly the 2016 Kumamoto earthquake belt): Kashima, Mashiki, Mifune, Kosa, Nishihara
- **Islands and south**: Kamiamakusa, Amakusa, Ashikita

## Properties

```json
{
  "muni_code": "434680",
  "name": "氷川町",
  "name_en": "Hikawa",
  "priority_tier": 1,
  "max_shindo": "7",
  "reported_structural_damage": ["住宅座屈2件（消防庁第17報の救助対応中事案）"],
  "deaths": 0,
  "post_event_ortho": "partial",
  "post_event_ortho_coverage_pct": 52,
  "post_event_ortho_method": "z15 tile probe, 13/25 sampled tiles returned 200",
  "open_shelters_2026_07_29": 0,
  "water_outage_households_2026_07_29": 10500,
  "notes": "…"
}
```

`priority_tier` 1 = intensity 7, or fatalities, or a reported structural collapse.
2 = intensity 6+, or fatalities without a listed intensity. 3 = intensity 6- / 5+.

`null` means not published. No value here is estimated or interpolated.

## Caveats

- **Municipality granularity is coarse for an AOI.** These are administrative polygons,
  not damage extents. Use them to prioritise, then cut finer AOIs inside them
- Kashima and Kosa do **not appear** in FDMA report #17's intensity-5-or-above table,
  yet account for all 3 confirmed deaths. `max_shindo` is `null` for them, not zero
- FDMA #17 also notes, for Kumamoto Prefecture, "many injured whose severity and
  disaster-relatedness is under investigation, including 13 dead" — the casualty figures
  will rise
- Coverage percentages are sampled, not exhaustive. The ortho footprint follows flight
  strips, so it is sparse inside its own bbox

## GSI imagery endpoints (verified 2026-07-30)

```
# Orthophoto (preliminary), Yatsushiro district, flown 2026-07-29 — PNG raster tiles
https://maps.gsi.go.jp/xyz/20260729kumamoto_yatsushiro_0729do_sokuho/{z}/{x}/{y}.png
bbox 130.4297,32.1756,130.7812,32.6209

# Oblique photos — NOT raster tiles. A GeoJSON point layer of photo locations.
https://maps.gsi.go.jp/xyz/20260729kumamoto_yatsushiro_0729naname/{z}/{x}/{y}.geojson
567 points, 2026-07-29 12:10:37–13:54:31 JST, bbox 130.4467,32.2619,130.7381,32.5719
Each point links a JPG under https://saigai.gsi.go.jp/1/R8_kumamoto/0729yatsushiro_naname/
```

GSI also lists a slope-failure / sediment-deposit interpretation (published 2026-07-30)
and preliminary vertical photos, but their tile endpoints did not respond to probing;
IDs unconfirmed. See <https://www.gsi.go.jp/BOUSAI/20260728_kumamoto_earthquake.html>.

## Sources

- FDMA report #17, 2026-07-29 16:30 — intensity, casualties, rescue cases
- MLIT report #5, 2026-07-29 12:00 — water outage by municipality
- Kumamoto Prefecture disaster portal `portal.bousai.pref.kumamoto.jp` — open shelters
  (redistribution requires prior contact with the prefecture; only counts are used here)
- GSI <https://www.gsi.go.jp/BOUSAI/20260728_kumamoto_earthquake.html> — imagery
  (Japan PDL 1.0, equivalent to CC BY 4.0)
- NHK 2026-07-29 13:00 — Hikawa car-camping evacuees
- Boundaries: OpenStreetMap via Nominatim (ODbL)

## License

Code and compiled attributes: CC0 1.0.
Boundary geometry derives from OpenStreetMap and remains under **ODbL**.
