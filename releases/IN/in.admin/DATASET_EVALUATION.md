# IN_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `in.admin`
Version: `v1.0.2`
Country: `IN`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `in.admin v1.0.2` after correcting the India admin-level-4 scope generation.
The previous scope was derived from India state/union-territory polygons after Natural Earth admin-0 filtering,
which excluded the valid OSM union territory relation for Lakshadweep. Version `v1.0.2` builds the scope from
all extracted India admin-level-4 state and union territory polygons in the Geofabrik India PBF; Natural Earth
is retained as provenance only.

---

# 2. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `admin-level-4 coverage union` |
| Boundary Builder | `scripts/build_in_boundaries.py` |
| Generated Boundary | `tmp/in_country.json` |
| Boundary Source | `OSM admin-level-4 coverage union from Geofabrik India PBF` |
| OSM Source | `geofabrik:asia/india` |

---

# 3. Administrative Model

| Level | Runtime Label | Dataset Count |
| ----: | ------------- | ------------: |
| 4 | `admin_state_union_territory` | 35 |
| 5 | `admin_district` | 753 |
| 6 | `admin_subdistrict` | 6,276 |

`Lakshadweep` is present as `in_r2027460`.

---

# 4. Evaluation Summary

# CADIS Lookup Mass Test Summary (IN)

## Overview
- Country: `IN`
- Dataset ID: `in.admin`
- Dataset Version: `v1.0.2`
- Dataset Country Name: `India`
- Dataset Dir: `IN/in.admin/v1.0.2`
- Samples: `10,000`
- Throughput: `5882.220` qps (`1.700` sec)
- Overall pass rate: `100.00%` (10,000/10,000)
- Policy pass rate: `100.00%`
- Inside coverage pass rate: `100.00%`
- Runtime policy detected: `True`
- Policy version: `1.0`
- Hierarchy required: `True`
- Repair required: `False`
- Hierarchy usage samples: `0`
- Repair usage samples: `0`
- Nearby usage samples: `2`
- Offshore samples: `4`

## Scenario Comparison
| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (ok/partial/failed/unknown) |
|---|---:|---:|---:|---:|---|
| `full_policy` | 100.00% | 100.00% | 0 | 0 | 8,397/608/995/0 |
| `no_hierarchy` | 100.00% | 100.00% | 0 | 0 | 8,397/608/995/0 |
| `no_repair` | 100.00% | 100.00% | 0 | 0 | 8,397/608/995/0 |
| `no_nearby` | 99.99% | 99.99% | 1 | 1 | 8,391/608/1,001/0 |
| `osm_only` | 99.99% | 99.99% | 1 | 1 | 8,391/608/1,001/0 |

## Layer Effects
- Rescued by hierarchy: `0`
- Rescued by repair: `0`
- Rescued by nearby: `6`
- Rescued vs OSM-only: `6`

## Level 4 City Hit Rates
- Unique level-4 cities hit: `34`
- Total level-4 hits (all points): `9,001`
- Total level-4 hits (inside points): `9,000`

| City (Level 4) | Hits | Hit Rate (All Points) | Hits (Inside) | Hit Rate (Inside Points) |
|---|---:|---:|---:|---:|
| `Rajasthan` | 1,016 | 10.16% | 1,016 | 11.29% |
| `Madhya Pradesh` | 893 | 8.93% | 893 | 9.92% |
| `Maharashtra` | 843 | 8.43% | 843 | 9.37% |
| `Uttar Pradesh` | 720 | 7.20% | 720 | 8.00% |
| `Gujarat` | 538 | 5.38% | 538 | 5.98% |
| `Karnataka` | 488 | 4.88% | 488 | 5.42% |
| `Andhra Pradesh` | 443 | 4.43% | 443 | 4.92% |
| `Odisha` | 422 | 4.22% | 422 | 4.69% |
| `Tamil Nadu` | 358 | 3.58% | 358 | 3.98% |
| `Chhattisgarh` | 354 | 3.54% | 354 | 3.93% |
| `Lakshadweep` | 325 | 3.25% | 325 | 3.61% |
| `Telangana` | 276 | 2.76% | 276 | 3.07% |
| `Bihar` | 270 | 2.70% | 270 | 3.00% |
| `Assam` | 239 | 2.39% | 239 | 2.66% |
| `West Bengal` | 231 | 2.31% | 231 | 2.57% |
| `Jharkhand` | 211 | 2.11% | 211 | 2.34% |
| `Ladakh` | 202 | 2.02% | 202 | 2.24% |
| `Punjab` | 157 | 1.57% | 157 | 1.74% |
| `Himachal Pradesh` | 157 | 1.57% | 157 | 1.74% |
| `Uttarakhand` | 152 | 1.52% | 152 | 1.69% |
| `Haryana` | 145 | 1.45% | 145 | 1.61% |
| `Kerala` | 121 | 1.21% | 120 | 1.33% |
| `Jammu and Kashmir` | 121 | 1.21% | 121 | 1.34% |
| `Manipur` | 65 | 0.65% | 65 | 0.72% |
| `Meghalaya` | 63 | 0.63% | 63 | 0.70% |
| `Mizoram` | 56 | 0.56% | 56 | 0.62% |
| `Nagaland` | 46 | 0.46% | 46 | 0.51% |
| `Tripura` | 29 | 0.29% | 29 | 0.32% |
| `Andaman and Nicobar Islands` | 27 | 0.27% | 27 | 0.30% |
| `Sikkim` | 14 | 0.14% | 14 | 0.16% |

## Distributions
**Shape Distribution**
- `[4,5,6]`: `8,393`
- `[]`: `999`
- `[4]`: `334`
- `[4,5]`: `267`
- `[4,6]`: `7`
**Node Source Distribution**
- `polygon`: `8,999`
- `nearby`: `2`
**Source Mix Distribution**
- `polygon`: `8,999`
- `__none__`: `999`
- `nearby`: `2`
**Policy Reason Distribution**
- `shape_status_map`: `9,001`
- `empty_shape`: `995`
- `offshore`: `4`


---

# 5. Verdict

`in.admin v1.0.2` passes the staged evaluation with full policy and inside-boundary coverage. The India
admin-level-4 scope now includes all 35 state/union-territory polygons extracted from the source PBF, including
Lakshadweep.
