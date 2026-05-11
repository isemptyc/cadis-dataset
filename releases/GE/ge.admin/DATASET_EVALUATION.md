# GE_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ge.admin`
Version: `v1.0.0`
Country: `GE`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ge.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Georgia dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `ge.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `GE`              |
| Country Name            | `Georgia`         |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.39`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_ge_boundaries.py`       |
| Generated Boundary       | `tmp/ge_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth GE admin-0 country boundary.` |
| Boundary BBox            | `[39.985976355436605, 41.04411082000006, 46.69480310100005, 43.57584259000002]` |
| OSM Source               | `geofabrik:europe/georgia`             |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Georgia engine exposes these OSM administrative levels:

| Level | Runtime Label         | Dataset Count | Notes |
| ----: | --------------------- | ------------: | ----- |
| 4     | `admin_region`        | 12            | Regions, autonomous republics, and Tbilisi |
| 6     | `admin_municipality`  | 77            | Municipalities and districts |
| 8     | `admin_locality`      | 90            | Locality areas |
| 10    | `admin_neighborhood`  | 109           | Neighborhood/detail features |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 3     | 2           | Excluded as broad disputed-region overlay coverage |
| 7     | 1           | Excluded as a single scoped feature |

The engine uses canonical names from `name:ka`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `ab`, `en`, `ka`, `os`, and `ru`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `GE/ge.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

Sampling is uniform over land area, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value           |
| ------------------------- | --------------- |
| Overall Pass Rate         | `100.00%`       |
| Inside Coverage Pass Rate | `100.00%`       |
| Policy Pass Rate          | `100.00%`       |
| Failed Samples            | `0`             |
| Throughput                | `14624.350` QPS |
| Total Runtime             | `0.684 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 8,592 / 432 / 976 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 8,592 / 432 / 976 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 8,592 / 432 / 976 / 0 |
| no_nearby    | 98.81%    | 98.68%           | 119    | 119           | 8,451 / 432 / 1,117 / 0 |
| osm_only     | 98.81%    | 98.68%           | 119    | 119           | 8,451 / 432 / 1,117 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 141             |
| Total vs OSM-only | 141             |

Nearby fallback resolves boundary-adjacent and administrative-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `ge.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape        | Count |
| ------------ | ----: |
| `[4,6]`      | 8,259 |
| `[]`         | 994   |
| `[6]`        | 276   |
| `[4,6,8]`   | 161   |
| `[6,8]`     | 155   |
| `[4,10]`    | 59    |
| `[4,6,8,10]` | 50  |
| `[4,6,10]`  | 24    |

Additional low-count shapes observed in the full report were `[4]` and `[6,10]`.

Empty shapes correspond to:

* `empty_shape`: 976
* `offshore`: 18

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,883 |
| nearby        | 123   |
| admin_tree_id | 27    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,856 |
| none                   | 994   |
| nearby                 | 123   |
| admin_tree_id\|polygon | 27    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,006 |
| empty_shape      | 976   |
| offshore         | 18    |

---

# 10. Level-4 Coverage

* Unique level-4 region units hit: `12`
* Total level-4 hits across all samples: `8,574`
* Total level-4 hits across inside samples: `8,566`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| კახეთის მხარე | 1,467 | 14.67% | 1,464 | 16.27% |
| აფხაზეთის ავტონომიური რესპუბლიკა | 1,133 | 11.33% | 1,133 | 12.59% |
| სამეგრელო-ზემო სვანეთი | 1,002 | 10.02% | 1,002 | 11.13% |
| სამცხე-ჯავახეთი | 872 | 8.72% | 870 | 9.67% |
| ქვემო ქართლი | 847 | 8.47% | 847 | 9.41% |
| იმერეთის მხარე | 805 | 8.05% | 805 | 8.94% |
| მცხეთა-მთიანეთი | 734 | 7.34% | 733 | 8.14% |
| რაჭა-ლეჩხუმი და ქვემო სვანეთი | 600 | 6.00% | 599 | 6.66% |
| შიდა ქართლი | 430 | 4.30% | 430 | 4.78% |
| აჭარის ავტონომიური რესპუბლიკა | 367 | 3.67% | 366 | 4.07% |
| გურია | 260 | 2.60% | 260 | 2.89% |
| თბილისი | 57 | 0.57% | 57 | 0.63% |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ge_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Georgia dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,6]`, reflecting region and municipality coverage.
3. Levels 8 and 10 add locality and neighborhood detail where available.
4. Nearby fallback is bounded at 5 km and accounts for a small rescue effect around administrative edges.
5. Levels 3 and 7 are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `6f71b221efa0efe8dde38af46654cf0095b53944`
- Cadis version:
  `0.8.39`
- Boundary builder: `scripts/build_ge_boundaries.py`
- Build boundary: `tmp/ge_country.json`
- Evaluation boundary: `tmp/ge_country.json`
- Staged dataset: `GE/ge.admin/v1.0.0`
- Source OSM SHA256:
  `b83911459bb7ea5d1dcd0f0ed4c143cf48fc8fd2313e5ad2cefadb2e52c87a02`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ge.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Georgia levels 4, 6, 8, and 10

The dataset is suitable for human quality review.
