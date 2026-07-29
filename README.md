# 令和8年熊本地震（2026-07-28）被害焦点 AOI

2026 Kumamoto Earthquake — damage-focus AOI for HOT Tasking Manager / fAIr / MapSwipe.

M7.1, 2026-07-28 16:27 JST. Max JMA intensity **7** at Uki City (Toyono) and Hikawa Town (Shimaji).

## Why this exists

As of 2026-07-30 there is **no official building-damage dataset**. FDMA report #17
(2026-07-29 16:30) lists housing damage only for Fukuoka (4 partially damaged);
the Kumamoto column is blank.

There *is* now an official **slope-failure** dataset, published 2026-07-30 and included
here, and it carries an interpretation-extent polygon of 509 km² which is the closest
thing to an official affected-area boundary. See
[GSI slope-failure interpretation](#gsi-slope-failure-interpretation) below.

So this AOI is built from **proxies**, in this priority order:

1. **Observed JMA intensity** — the primary proxy for building damage
2. **Individually reported structural damage** — collapses, bridge failure, road uplift
3. **Whether post-event GSI orthophoto already exists** — you cannot map what has not been flown

Water-outage figures are included but **demoted to a reference attribute**: a broken
distribution main does not imply building damage.

## Data

All files are served over HTTPS with CORS, so they can be loaded straight into QGIS, JOSM,
iD, a STAC browser, or `fetch()`.

| URL | Content |
|---|---|
| <https://yuiseki.github.io/earthquake-kumamoto-2026-07/data/aoi.json> | 16 municipality polygons with damage proxies. Boundaries from OSM via Nominatim |
| <https://yuiseki.github.io/earthquake-kumamoto-2026-07/data/gsi-ortho-coverage.json> | Measured footprint of the GSI post-event orthophoto (48 tiles at z13) |
| <https://yuiseki.github.io/earthquake-kumamoto-2026-07/data/gsi-oblique-photos-20260729.geojson> | 567 GSI oblique-photo locations, tidied: image links extracted from HTML, timestamps as ISO 8601 |
| <https://yuiseki.github.io/earthquake-kumamoto-2026-07/data/gsi-slope-failure-20260730.geojson> | GSI slope-failure / debris-flow / deposit interpretation, 21 polygons, `class` decoded from the published legend |
| <https://yuiseki.github.io/earthquake-kumamoto-2026-07/data/gsi-interpretation-extent.geojson> | GSI interpretation extent, 509 km². The closest thing to an official affected-area polygon |
| <https://yuiseki.github.io/earthquake-kumamoto-2026-07/stac/catalog.json> | Static STAC 1.1.0 catalog (2 collections, 4 items) |
| <https://yuiseki.github.io/earthquake-kumamoto-2026-07/> | Map viewer: AOI, GSI post-event ortho, slope failures, cloud gaps |

```bash
curl -s https://yuiseki.github.io/earthquake-kumamoto-2026-07/data/aoi.json \
  | jq -r '.features[].properties
           | [.name_en, .priority_tier, .mainshock_shindo, .post_event_ortho_coverage_pct]
           | @tsv'
```

## Static STAC catalog

STAC core has no way to express an XYZ tile service as an asset. The
**[`web-map-links`](https://github.com/stac-extensions/web-map-links) extension (v1.2.0)**
does: it defines link relations `xyz`, `wmts`, `tilejson`, `pmtiles` and `wms`. That is a
stable published extension, unlike `tiled-assets`, which is still at proposal stage. So the
GSI orthophoto is registered as an Item whose tile template is a `rel: "xyz"` link:

```json
{
  "rel": "xyz",
  "href": "https://maps.gsi.go.jp/xyz/20260729kumamoto_yatsushiro_0729do_sokuho/{z}/{x}/{y}.png",
  "type": "image/png"
}
```

```
stac/catalog.json
├── gsi-r8-kumamoto-aerial/collection.json          GSI aerial imagery and derived products
│   ├── gsi-ortho-yatsushiro-20260729.json          geometry = the 48 measured tiles (MultiPolygon)
│   ├── gsi-oblique-yatsushiro-20260729.json        567 photo points, no tile link
│   └── gsi-slope-failure-yatsushiro-20260730.json  derived_from the orthophoto item
└── kumamoto-r8-aoi/collection.json                 this AOI
    └── damage-focus-aoi-20260730.json
```

All seven files validate against STAC 1.1.0 (`pystac.validate()`, 0 errors).

Note this still does not make the imagery ingestible by OpenAerialMap: OAM's
`item_assets` require a COG GeoTIFF, and a tile template is not one. Getting it into OAM
would need tiles → COG → a STAC item pointing at the COG. The catalog here is for
discovery and for QGIS / JOSM / iD consumption, not for OAM ingestion.

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
  "mainshock_shindo": "7",
  "mainshock_shindo_source": "消防庁 第17報 (2026-07-29 16:30)",
  "aftershock_max_shindo": "4",
  "n_shindo_ge4": 3,
  "n_shindo_ge3": 11,
  "n_felt": 56,
  "aftershock_window": "2026-07-28T16:27:00+09:00 .. 2026-07-30T06:25:00+09:00",
  "reported_structural_damage": [
    "住宅座屈2件（消防庁第17報の救助対応中事案）"
  ],
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

## GSI slope-failure interpretation

Published 2026-07-30, interpreted from the 2026-07-29 orthophoto (interpretation finished
2026-07-29 23:50 JST). Served as a **GeoJSON polygon layer at z2**, not raster tiles:

```
https://maps.gsi.go.jp/xyz/20260729kumamoto_syamenhoukai_dosekiryu_taiseki_yatsushiro/2/3/1.geojson
```

The upstream features carry **only styling attributes** (`_fillColor` and friends), no
semantics. The meaning is in the colours, so `class` here is decoded against the published
legend:

| `class` | Upstream fill | Features | Area |
|---|---|---:|---:|
| `slope_failure_debris_deposit` 斜面崩壊・土石流・堆積範囲 | `#ff3232` | 8 | 0.012 km² |
| `unreadable_due_to_cloud` 雲による未判読範囲 | `#646464` | 12 | 1.283 km² |
| `interpretation_extent` 判読範囲 | `#3388ff` outline | 1 | 509.0 km² |

By municipality (centroid test):

| | slope failure | cloud gap |
|---|---|---|
| Yatsushiro 八代市 | 6 features, 8,744 m² | 5 features, 0.569 km² |
| Ashikita 芦北町 | 1 feature, 1,476 m² | 7 features, 0.714 km² |
| Hikawa 氷川町 | 1 feature, 1,593 m² | none |

Two things matter operationally.

**The cloud gaps are holes in the data, not damage.** 1.28 km² inside the flown area could
not be interpreted because cloud covered it. fAIr cannot read those pixels either. Those
12 polygons need either a re-flight or ground truth. Coverage measured by tile response
(`post_event_ortho_coverage_pct` in `aoi.json`) does **not** account for this.

**The slope failures are small and few.** 8 features totalling 0.012 km², against 509 km²
interpreted. GSI states the interpretation is not field-verified, may miss real failures
and may include failures unrelated to this earthquake, and only shows features roughly
30 m or larger in length or width. So this is a lower bound, but it does suggest this
earthquake did not produce widespread slope failure in the flown area.

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

```
# Slope failure / debris flow / deposit interpretation, published 2026-07-30 — GeoJSON at z2
https://maps.gsi.go.jp/xyz/20260729kumamoto_syamenhoukai_dosekiryu_taiseki_yatsushiro/2/3/1.geojson
```

GSI also lists preliminary vertical photos
(`20260729kumamoto_yatsushiro_0729suichoku_sokuho`), but that endpoint did not respond to
probing in either `.png` or `.geojson`. See
<https://www.gsi.go.jp/BOUSAI/20260728_kumamoto_earthquake.html>.

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

**[CC0 1.0 Universal](LICENSE)** (public domain dedication) for everything produced in this
repository: the compiled attribute tables, the measured ortho footprint, the tidied oblique
photo layer, the STAC metadata, the viewer and the build scripts. Use it however you like,
no attribution required.

Two upstream inputs carry their own terms and are **not** covered by the CC0 dedication:

| Input | License | Where it appears |
|---|---|---|
| Administrative boundary geometry, from OpenStreetMap via Nominatim | **ODbL 1.0** | the `geometry` of every feature in `aoi.json` |
| GSI aerial imagery, photo locations and slope-failure interpretation | **[国土地理院コンテンツ利用規約](https://www.gsi.go.jp/kikakuchousei/kikakuchousei40182.html)** — redistribution is explicitly permitted as long as the source is credited to 国土地理院. Hidenori also cites [Japan PDL 1.0](https://www.digital.go.jp/resources/open_data/public_data_license_v1.0) (CC BY 4.0 equivalent) for GSI open data | `gsi-oblique-photos-*.geojson`, `gsi-slope-failure-*.geojson`, `gsi-interpretation-extent.geojson`, and the imagery the STAC items point at |

So: the numbers and the analysis are CC0, the administrative polygons stay ODbL, and the
GSI layers need attribution to 国土地理院 (redistribution itself is allowed). If you only need the attributes and supply your own boundaries,
nothing here constrains you.
