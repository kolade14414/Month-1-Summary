# Month 1 Summary

Month 1 covered three GIS projects: two scoped and planned (flood exposure in Lagos, pipeline vandalism in the Niger Delta) and one carried through to a completed analysis (CO2 storage hub coverage).

| Project | Focus | Status |
|---|---|---|
| Lagos Low-Lying Settlements Flood Exposure Mapping | Flood exposure of settlements near rivers | Project definition and data feasibility done |
| Detection of Pipeline Vandalism in Nigeria | Remote sensing of pipeline tapping and sabotage | Project definition and data feasibility done |
| CO2 Storage Hub Coverage Analysis | Planning zones beyond 100 km of any storage hub | Analysis complete (Week 4) |

---

## 1. Lagos Low-Lying Settlements Flood Exposure Mapping

**Question.** Which settlements in Lagos State sit on low-elevation land close to rivers and waterways, and are therefore at elevated flood risk?

**Data.**
- GRID3: ward boundaries and settlement extents
- WorldPop: population
- OpenStreetMap: rivers and waterways
- Copernicus GLO-30 DEM via OpenTopography: elevation, with slope derived in QGIS
- ESA WorldCover: land use/land cover
- CHIRPS: rainfall
- HDX: waste management sites and administrative boundaries

**Planned method.**
1. Reproject all layers to UTM Zone 31N (EPSG:32631), keeping a WGS84 master copy.
2. Clip to the Lagos State AOI.
3. Derive elevation and slope from the DEM.
4. Buffer rivers at 100 m, 250 m and 500 m.
5. Reclassify elevation into low/medium/high bands relative to local river bank elevation.
6. Overlay settlements on the low-elevation, river-proximity zones.
7. Join population to exposed settlements.
8. Produce a composite exposure map and a ranked list.

**Expected output.** A flood exposure map and a list of at-risk settlements and wards ranked by population at risk, with a possible later web GIS app.

---

## 2. Detection of Pipeline Vandalism in Nigeria

**Question.** Where along the Niger Delta pipeline network are vandalism and illegal tapping most likely, based on surface disturbance, thermal anomalies, spill signatures, and proximity to settlements and creeks?

**Data.**
- Pipeline routes: NNPC / NOSDRA / OpenStreetMap
- Spill records: NOSDRA
- Sentinel-2 (optical) and Sentinel-1 (SAR)
- VIIRS / MODIS active fire and VIIRS night-time lights
- GRID3 settlements, OSM waterways, ESA WorldCover, Copernicus DEM
- ACLED conflict events

**Planned method.**
1. Buffer the pipeline route (50 m, 100 m, 250 m) to define corridors.
2. Geolocate historical NOSDRA spills to build a baseline.
3. Run multi-temporal NDVI/NDWI change detection on Sentinel-2.
4. Run Sentinel-1 backscatter change detection for ground disturbance.
5. Overlay active-fire and night-lights data to flag artisanal refining clusters.
6. Cross-reference with settlement proximity, creek access and ACLED events.
7. Produce a ranked hotspot map for field verification.

**Expected output.** A ranked hotspot map of pipeline segments, cross-referenced with NOSDRA records, with a possible later near-real-time dashboard.

---

## 3. CO2 Storage Hub Coverage Analysis (Week 4)

> All numbers come from a synthetic, illustrative dataset. The scripts and outputs are reproducible from `scripts/week4_coverage_analysis.py`.

**Question.** Which planning zones are more than 100 km from any candidate CO2 storage hub?

**Method.**
1. Buffer the storage hubs by 100 km and dissolve into one shape (QGIS Buffer).
2. Run Difference (zones minus buffer) to isolate uncovered areas.
3. Calculate the uncovered area and share per zone.

The same steps are scripted in Python.

**Result.** Three zones have more than 30% of their area beyond 100 km of any hub, all in the north-east corner of the study area.

| Zone | Uncovered area | Uncovered share |
|---|---|---|
| Z12 | 5,994 km² | 99.9% |
| Z08 | 2,396 km² | 39.9% |
| Z11 | 2,121 km² | 35.3% |

Seven zones are fully covered. Z09 (3.5%) and Z10 (8.9%) have small gaps and are not flagged.

**Checks performed.**
- **Map review:** uncovered areas sit on the periphery, furthest from all five hubs.
- **Row count:** 12 zones in, 5 out. Difference drops fully covered features, so the 7 covered zones vanish instead of returning 0%. Reconciled as 5 + 7 = 12 and asserted in the script.
- **Hand check:** Z12's centre is 147.6 km from the nearest hub.
- **Empty geometry:** none returned.
- **Sensitivity:** re-ran at 80, 90, 100, 110 and 120 km.

| Buffer | Zones flagged |
|---|---|
| 80 km | Z08, Z10, Z11, Z12 |
| 90 km | Z08, Z11, Z12 |
| **100 km** | **Z08, Z11, Z12** |
| 110 km | Z12 |
| 120 km | Z12 |

Z12 is flagged at every distance. Z08 and Z11 depend on the chosen 100 km threshold.

**Emitters.** Of 41 emitters, one lies beyond 100 km: E18, a gas processing plant, 138.3 km from the nearest hub in Z12. It reports 0.72 Mt CO2/yr, 2.9% of the 24.4 Mt reported in total. 11 emitters report no CO2 figure, so that percentage is a floor.

**Next steps.**
- Replace straight-line distance with pipeline route distance.
- Chase the 11 missing CO2 figures before quoting emissions shares.
- Add hub storage capacity, since a hub within 100 km is useless if it holds 1 Mt.

---

## Takeaways

- Data feasibility and CRS decisions (UTM 31N for metric work, WGS84 for sharing) were settled up front for the Lagos and Niger Delta projects.
- The Week 4 analysis showed that the answer can hinge on one parameter (the buffer distance) and that GIS tools can silently drop features, so row counts and sensitivity runs are part of the workflow.
- Next: execute the Lagos and pipeline workflows from planning into analysis.
