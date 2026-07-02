# PH_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ph.admin`
Version: `v1.0.1`
Country: `PH`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `ph.admin v1.0.1` after correcting Philippines admin-level-4 scope generation.
The previous scope was derived from province polygons after Natural Earth admin-0 filtering, which excluded
valid OSM province relations for multiple island/coastal provinces. Version `v1.0.1` builds the scope from all
extracted Philippines admin-level-4 province polygons in the Geofabrik Philippines PBF; Natural Earth is retained
as provenance only.

---

# 2. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `admin-level-4 coverage union` |
| Boundary Builder | `scripts/build_ph_boundaries.py` |
| Generated Boundary | `tmp/ph_country.json` |
| Boundary Source | `OSM admin-level-4 coverage union from Geofabrik Philippines PBF` |
| OSM Source | `geofabrik:asia/philippines` |

---

# 3. Administrative Model

| Level | Runtime Label | Dataset Count |
| ----: | ------------- | ------------: |
| 4 | `admin_province` | 82 |
| 6 | `admin_municipality_city` | 861 |
| 10 | `admin_detail` | 2,851 |

The restored admin-level-4 scope includes Pangasinan, Romblon, Batanes, Samar, Leyte, Palawan, Antique, Cebu,
Surigao del Norte, Agusan del Norte, Zamboanga Sibugay, and Davao Occidental.

---

# 4. Evaluation Summary

# CADIS Lookup Mass Test Summary (PH)

## Overview
- Country: `PH`
- Dataset ID: `ph.admin`
- Dataset Version: `v1.0.1`
- Dataset Country Name: `Philippines`
- Dataset Dir: `PH/ph.admin/v1.0.1`
- Samples: `10,000`
- Throughput: `11045.560` qps (`0.905` sec)
- Overall pass rate: `100.00%` (10,000/10,000)
- Policy pass rate: `100.00%`
- Inside coverage pass rate: `100.00%`
- Runtime policy detected: `True`
- Policy version: `1.0`
- Hierarchy required: `True`
- Repair required: `False`
- Hierarchy usage samples: `0`
- Repair usage samples: `0`
- Nearby usage samples: `1`
- Offshore samples: `7`

## Scenario Comparison
| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (ok/partial/failed/unknown) |
|---|---:|---:|---:|---:|---|
| `full_policy` | 100.00% | 100.00% | 0 | 0 | 201/8,807/992/0 |
| `no_hierarchy` | 100.00% | 100.00% | 0 | 0 | 201/8,807/992/0 |
| `no_repair` | 100.00% | 100.00% | 0 | 0 | 201/8,807/992/0 |
| `no_nearby` | 100.00% | 100.00% | 0 | 0 | 194/8,806/1,000/0 |
| `osm_only` | 100.00% | 100.00% | 0 | 0 | 194/8,806/1,000/0 |

## Layer Effects
- Rescued by hierarchy: `0`
- Rescued by repair: `0`
- Rescued by nearby: `8`
- Rescued vs OSM-only: `8`

## Level 4 City Hit Rates
- Unique level-4 cities hit: `82`
- Total level-4 hits (all points): `9,001`
- Total level-4 hits (inside points): `9,000`

| City (Level 4) | Hits | Hit Rate (All Points) | Hits (Inside) | Hit Rate (Inside Points) |
|---|---:|---:|---:|---:|
| `Palawan` | 1,219 | 12.19% | 1,219 | 13.54% |
| `Cagayan` | 334 | 3.34% | 334 | 3.71% |
| `Quezon` | 327 | 3.27% | 327 | 3.63% |
| `Tawi-Tawi` | 288 | 2.88% | 288 | 3.20% |
| `Sulu` | 241 | 2.41% | 241 | 2.68% |
| `Cebu` | 214 | 2.14% | 214 | 2.38% |
| `Masbate` | 211 | 2.11% | 211 | 2.34% |
| `Negros Occidental` | 207 | 2.07% | 207 | 2.30% |
| `Occidental Mindoro` | 199 | 1.99% | 199 | 2.21% |
| `Isabela` | 178 | 1.78% | 178 | 1.98% |
| `Antique` | 171 | 1.71% | 171 | 1.90% |
| `Zamboanga del Norte` | 171 | 1.71% | 171 | 1.90% |
| `Eastern Samar` | 158 | 1.58% | 158 | 1.76% |
| `Leyte` | 157 | 1.57% | 157 | 1.74% |
| `Davao Oriental` | 154 | 1.54% | 154 | 1.71% |
| `Samar` | 152 | 1.52% | 152 | 1.69% |
| `Camarines Sur` | 145 | 1.45% | 145 | 1.61% |
| `Bukidnon` | 144 | 1.44% | 144 | 1.60% |
| `Batanes` | 141 | 1.41% | 141 | 1.57% |
| `Negros Oriental` | 139 | 1.39% | 139 | 1.54% |
| `Romblon` | 138 | 1.38% | 138 | 1.53% |
| `Bohol` | 135 | 1.35% | 135 | 1.50% |
| `Iloilo` | 124 | 1.24% | 124 | 1.38% |
| `Pangasinan` | 121 | 1.21% | 121 | 1.34% |
| `Surigao del Sur` | 120 | 1.20% | 120 | 1.33% |
| `Agusan del Sur` | 118 | 1.18% | 118 | 1.31% |
| `Zambales` | 105 | 1.05% | 105 | 1.17% |
| `Oriental Mindoro` | 98 | 0.98% | 98 | 1.09% |
| `Sultan Kudarat` | 98 | 0.98% | 98 | 1.09% |
| `Zamboanga Sibugay` | 97 | 0.97% | 97 | 1.08% |

## Distributions
**Shape Distribution**
- `[4,6]`: `5,316`
- `[4]`: `3,487`
- `[]`: `999`
- `[4,6,10]`: `194`
- `[4,10]`: `4`
**Node Source Distribution**
- `polygon`: `9,000`
- `nearby`: `1`
**Source Mix Distribution**
- `polygon`: `9,000`
- `__none__`: `999`
- `nearby`: `1`
**Policy Reason Distribution**
- `shape_status_map`: `9,001`
- `empty_shape`: `992`
- `offshore`: `7`


---

# 5. Verdict

`ph.admin v1.0.1` passes the staged evaluation with full policy and inside-boundary coverage. The Philippines
admin-level-4 scope now includes all 82 province polygons extracted from the source PBF.
