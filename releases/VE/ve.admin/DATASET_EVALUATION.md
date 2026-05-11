# VE_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ve.admin`
Version: `v1.0.0`
Country: `VE`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ve.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Venezuela dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `ve.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `VE`              |
| Country Name            | `Venezuela`       |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.50`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_ve_boundaries.py`       |
| Generated Boundary       | `tmp/ve_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 4 and 6 whose representative point is covered by the Natural Earth VE boundary.` |
| Boundary BBox            | `[-73.3529632, 0.6473964, -59.8055058, 12.255523]` |
| OSM Source               | `geofabrik:south-america/venezuela`    |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Venezuela administrative coverage represented by the selected source PBF and the Natural Earth VE country boundary. A raw full-country Natural Earth evaluation showed a small number of inside misses in areas not covered by the selected OSM administrative polygons. The administrative-coverage boundary avoids treating those uncovered areas as inside-dataset failures.

---

# 4. Administrative Model

The initial Venezuela engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_state`        | 23            | State-level and capital district coverage |
| 6     | `admin_municipality` | 333           | Municipality-level coverage |
| 7     | `admin_parish`       | 1,181         | Parish-level coverage |
| 8     | `admin_locality`     | 696           | Locality coverage where available |
| 9     | `admin_sector`       | 741           | Sector coverage where available |
| 10    | `admin_neighborhood` | 613           | Neighborhood/detail coverage where available |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 3     | 0           | Excluded because no scoped features were present |
| 5     | 2           | Excluded as special overlay coverage, not part of the initial runtime contract |

The engine uses canonical names from `name:es`, `name`, `official_name`, and `name:en`, with bounded multilingual aliases for `es` and `en`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `VE/ve.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

Sampling is uniform over land area, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value          |
| ------------------------- | -------------- |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |
| Failed Samples            | `0`            |
| Throughput                | `3082.370` QPS |
| Total Runtime             | `3.244 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 8,485 / 525 / 990 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 8,485 / 525 / 990 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 8,485 / 525 / 990 / 0 |
| no_nearby    | 99.99%    | 99.99%           | 1      | 1             | 8,474 / 525 / 1,001 / 0 |
| osm_only     | 99.99%    | 99.99%           | 1      | 1             | 8,474 / 525 / 1,001 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 11              |
| Total vs OSM-only | 11              |

Nearby fallback resolves a small number of boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `ve.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape            | Count |
| ---------------- | ----: |
| `[4,6,7]`        | 8,159 |
| `[]`             | 999   |
| `[6,7]`          | 408   |
| `[4]`            | 153   |
| `[6]`            | 114   |
| `[4,6,7,8]`      | 72    |
| `[4,6]`          | 54    |
| `[4,6,7,8,9]`    | 19    |

Empty shapes correspond to:

* `empty_shape`: 990
* `offshore`: 9

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,999 |
| admin_tree_id | 4     |
| nearby        | 2     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,995 |
| none                   | 999   |
| admin_tree_id\|polygon | 4     |
| nearby                 | 2     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,001 |
| empty_shape      | 990   |
| offshore         | 9     |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `23`
* Total level-4 hits across all samples: `8,476`
* Total level-4 hits across inside samples: `8,475`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Estado Bolívar | 2,342 | 23.42% | 2,342 | 26.02% |
| Estado Amazonas | 1,716 | 17.16% | 1,716 | 19.07% |
| Estado Apure | 730 | 7.30% | 729 | 8.10% |
| Estado Guárico | 637 | 6.37% | 637 | 7.08% |
| Estado Anzoátegui | 450 | 4.50% | 450 | 5.00% |
| Estado Delta Amacuro | 382 | 3.82% | 382 | 4.24% |
| Estado Barinas | 373 | 3.73% | 373 | 4.14% |
| Estado Falcón | 277 | 2.77% | 277 | 3.08% |
| Estado Monagas | 239 | 2.39% | 239 | 2.66% |
| Estado Lara | 201 | 2.01% | 201 | 2.23% |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ve_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Venezuela dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,6,7]`, reflecting strong state, municipality, and parish coverage across most uniformly sampled land points.
3. Levels 8, 9, and 10 provide valid lower-level detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `dd9610fe5e29e973df0654122d31e29c6f4da414`
- Cadis version:
  `0.8.50`
- Boundary builder: `scripts/build_ve_boundaries.py`
- Build boundary: `tmp/ve_country.json`
- Evaluation boundary: `tmp/ve_country.json`
- Staged dataset: `VE/ve.admin/v1.0.0`
- Source OSM SHA256:
  `702c3fce38e2db4e2eb7944e3b00e8518c77592d9ce8c028dde4bbdb138c32ce`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ve.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Venezuela levels 4, 6, 7, 8, 9, and 10

The dataset is suitable for human quality review.
