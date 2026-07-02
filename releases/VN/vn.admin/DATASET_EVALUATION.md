# VN_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `vn.admin`
Version: `v1.0.1`
Country: `VN`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `vn.admin v1.0.1` after correcting Vietnam admin-level-4 scope generation.
The previous scope was derived from province/municipality polygons after Natural Earth admin-0 filtering, which
excluded valid OSM admin-level-4 units. Version `v1.0.1` builds the scope from all extracted Vietnam admin-level-4
province/municipality polygons in the Geofabrik Vietnam PBF; Natural Earth is retained as provenance only.

---

# 2. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `admin-level-4 coverage union` |
| Boundary Builder | `scripts/build_vn_boundaries.py` |
| Generated Boundary | `tmp/vn_country.json` |
| Boundary Source | `OSM admin-level-4 coverage union from Geofabrik Vietnam PBF` |
| OSM Source | `geofabrik:asia/vietnam` |

---

# 3. Administrative Model

| Level | Runtime Label | Dataset Count |
| ----: | ------------- | ------------: |
| 4 | `admin_province_municipality` | 33 |
| 6 | `admin_district` | 3,253 |

The restored admin-level-4 scope includes An Giang, Huế, Quảng Trị, Ninh Bình, and Thành phố Hồ Chí Minh.

---

# 4. Evaluation Summary

# CADIS Lookup Mass Test Summary (VN)

## Overview
- Country: `VN`
- Dataset ID: `vn.admin`
- Dataset Version: `v1.0.1`
- Dataset Country Name: `Vietnam`
- Dataset Dir: `VN/vn.admin/v1.0.1`
- Samples: `10,000`
- Throughput: `12509.680` qps (`0.799` sec)
- Overall pass rate: `100.00%` (10,000/10,000)
- Policy pass rate: `100.00%`
- Inside coverage pass rate: `100.00%`
- Runtime policy detected: `True`
- Policy version: `1.0`
- Hierarchy required: `True`
- Repair required: `False`
- Hierarchy usage samples: `0`
- Repair usage samples: `0`
- Nearby usage samples: `3`
- Offshore samples: `12`

## Scenario Comparison
| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (ok/partial/failed/unknown) |
|---|---:|---:|---:|---:|---|
| `full_policy` | 100.00% | 100.00% | 0 | 0 | 5,916/3,099/985/0 |
| `no_hierarchy` | 100.00% | 100.00% | 0 | 0 | 5,916/3,099/985/0 |
| `no_repair` | 100.00% | 100.00% | 0 | 0 | 5,916/3,099/985/0 |
| `no_nearby` | 100.00% | 100.00% | 0 | 0 | 5,903/3,097/1,000/0 |
| `osm_only` | 100.00% | 100.00% | 0 | 0 | 5,903/3,097/1,000/0 |

## Layer Effects
- Rescued by hierarchy: `0`
- Rescued by repair: `0`
- Rescued by nearby: `15`
- Rescued vs OSM-only: `15`

## Level 4 City Hit Rates
- Unique level-4 cities hit: `33`
- Total level-4 hits (all points): `9,003`
- Total level-4 hits (inside points): `9,000`

| City (Level 4) | Hits | Hit Rate (All Points) | Hits (Inside) | Hit Rate (Inside Points) |
|---|---:|---:|---:|---:|
| `Tỉnh Lâm Đồng` | 898 | 8.98% | 898 | 9.98% |
| `Thành phố Hồ Chí Minh` | 698 | 6.98% | 698 | 7.76% |
| `Tỉnh Cà Mau` | 547 | 5.47% | 547 | 6.08% |
| `Tỉnh An Giang` | 521 | 5.21% | 521 | 5.79% |
| `Tỉnh Gia Lai` | 462 | 4.62% | 461 | 5.12% |
| `Tỉnh Quảng Ngãi` | 414 | 4.14% | 414 | 4.60% |
| `Tỉnh Đắk Lắk` | 404 | 4.04% | 404 | 4.49% |
| `Tỉnh Nghệ An` | 393 | 3.93% | 392 | 4.36% |
| `Tỉnh Quảng Trị` | 381 | 3.81% | 381 | 4.23% |
| `Tỉnh Sơn La` | 267 | 2.67% | 267 | 2.97% |
| `Tỉnh Tuyên Quang` | 252 | 2.52% | 251 | 2.79% |
| `Thành phố Đà Nẵng` | 249 | 2.49% | 249 | 2.77% |
| `Tỉnh Quảng Ninh` | 247 | 2.47% | 247 | 2.74% |
| `Thành phố Đồng Nai` | 242 | 2.42% | 242 | 2.69% |
| `Tỉnh Thanh Hóa` | 241 | 2.41% | 241 | 2.68% |
| `Tỉnh Lào Cai` | 226 | 2.26% | 226 | 2.51% |
| `Tỉnh Vĩnh Long` | 201 | 2.01% | 201 | 2.23% |
| `Tỉnh Điện Biên` | 199 | 1.99% | 199 | 2.21% |
| `Hà Tĩnh` | 196 | 1.96% | 196 | 2.18% |
| `Tỉnh Lạng Sơn` | 178 | 1.78% | 178 | 1.98% |
| `Thành phố Huế` | 178 | 1.78% | 178 | 1.98% |
| `Thành phố Cần Thơ` | 177 | 1.77% | 177 | 1.97% |
| `Tỉnh Lai Châu` | 172 | 1.72% | 172 | 1.91% |
| `Tỉnh Ninh Bình` | 164 | 1.64% | 164 | 1.82% |
| `Tỉnh Phú Thọ` | 163 | 1.63% | 163 | 1.81% |
| `Tỉnh Thái Nguyên` | 155 | 1.55% | 155 | 1.72% |
| `Thành phố Hải Phòng` | 150 | 1.50% | 150 | 1.67% |
| `Tỉnh Tây Ninh` | 140 | 1.40% | 140 | 1.56% |
| `Tỉnh Cao Bằng` | 137 | 1.37% | 137 | 1.52% |
| `Tỉnh Đồng Tháp` | 110 | 1.10% | 110 | 1.22% |

## Distributions
**Shape Distribution**
- `[4,6]`: `5,904`
- `[4]`: `3,099`
- `[]`: `997`
**Node Source Distribution**
- `polygon`: `9,000`
- `nearby`: `3`
- `admin_tree_id`: `1`
**Source Mix Distribution**
- `polygon`: `8,999`
- `__none__`: `997`
- `nearby`: `3`
- `admin_tree_id|polygon`: `1`
**Policy Reason Distribution**
- `shape_status_map`: `9,003`
- `empty_shape`: `985`
- `offshore`: `12`


---

# 5. Verdict

`vn.admin v1.0.1` passes the staged evaluation with full policy and inside-boundary coverage. The Vietnam
admin-level-4 scope now includes all 33 province/municipality polygons extracted from the source PBF.
