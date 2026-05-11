# HN_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `hn.admin`
Version: `v1.0.0`
Country: `HN`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `hn.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Honduras dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `hn.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `HN`              |
| Country Name            | `Honduras`        |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.59`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_hn_boundaries.py`       |
| Generated Boundary       | `tmp/hn_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 4 and 6 whose representative point is covered by the Natural Earth HN boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox            | `[-89.35368768855732, 12.9878699617407, -82.18364001846835, 16.51389632520692]` |
| OSM Source               | `geofabrik:central-america/honduras`   |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Honduras administrative coverage represented by materialized OSM department and municipality relations.

---

# 4. Administrative Model

The initial Honduras engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_department`   | 14            | Department-level coverage |
| 6     | `admin_municipality` | 296           | Municipality-level coverage |
| 7     | `admin_city`         | 1             | City-level coverage where available |
| 8     | `admin_district`     | 24            | District/local coverage where available |
| 9     | `admin_neighborhood` | 47            | Neighborhood coverage where available |
| 10    | `admin_detail`       | 328           | Detail coverage where available |
| 12    | `admin_micro_detail` | 1             | Micro-detail coverage where available |

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
* Dataset path: `HN/hn.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

Sampling is uniform over scoped administrative coverage, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value          |
| ------------------------- | -------------- |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |
| Failed Samples            | `0`            |
| Throughput                | `8133.880` QPS |
| Total Runtime             | `1.229 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 8,196 / 819 / 985 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 8,196 / 819 / 985 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 8,196 / 819 / 985 / 0 |
| no_nearby    | 99.90%    | 99.89%           | 10     | 10            | 8,172 / 819 / 1,009 / 0 |
| osm_only     | 99.90%    | 99.89%           | 10     | 10            | 8,172 / 819 / 1,009 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 24              |
| Total vs OSM-only | 24              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `hn.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape          | Count |
| -------------- | ----: |
| `[4,6]`        | 6,509 |
| `[4]`          | 1,628 |
| `[]`           | 999   |
| `[6]`          | 818   |
| `[4,6,8,9,10]` | 23    |
| `[4,6,8]`      | 10    |
| `[4,6,7,8]`    | 10    |
| `[4,6,9,10]`   | 1     |
| `[6,10]`       | 1     |
| `[4,6,7]`      | 1     |

Empty shapes correspond to:

* `empty_shape`: 985
* `offshore`: 14

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,991 |
| admin_tree_id | 20    |
| nearby        | 10    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,971 |
| none                   | 999   |
| admin_tree_id\|polygon | 20    |
| nearby                 | 10    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,001 |
| empty_shape      | 985   |
| offshore         | 14    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `14`
* Total level-4 hits across all samples: `8,182`
* Total level-4 hits across inside samples: `8,181`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Gracias a Dios | 2,347 | 23.47% | 2,346 | 26.07% |
| Olancho | 1,583 | 15.83% | 1,583 | 17.59% |
| Colón | 896 | 8.96% | 896 | 9.96% |
| Francisco Morazán | 586 | 5.86% | 586 | 6.51% |
| Yoro | 554 | 5.54% | 554 | 6.16% |
| El Paraíso | 463 | 4.63% | 463 | 5.14% |
| Comayagua | 355 | 3.55% | 355 | 3.94% |
| Santa Bárbara | 308 | 3.08% | 308 | 3.42% |
| Lempira | 303 | 3.03% | 303 | 3.37% |
| Copán | 228 | 2.28% | 228 | 2.53% |
| Intibucá | 180 | 1.80% | 180 | 2.00% |
| La Paz | 154 | 1.54% | 154 | 1.71% |
| Valle | 116 | 1.16% | 116 | 1.29% |
| Ocotepeque | 109 | 1.09% | 109 | 1.21% |

Hit distribution reflects uniform sampling over the scoped administrative coverage.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/hn_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Honduras dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,6]`, reflecting strong department and municipality coverage.
3. Levels 7, 8, 9, 10, and 12 provide valid lower-level detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `c962265e4301b47369f9a537b0046b2d4de1181d`
- Cadis version:
  `0.8.59`
- Boundary builder: `scripts/build_hn_boundaries.py`
- Build boundary: `tmp/hn_country.json`
- Evaluation boundary: `tmp/hn_country.json`
- Staged dataset: `HN/hn.admin/v1.0.0`
- Source OSM SHA256:
  `b5547b3786f671b60f63ad8e4c1926660f8831eda7fc89c1c90bd751fdc16796`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `hn.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Honduras levels 4, 6, 7, 8, 9, 10, and 12

The dataset is suitable for human quality review.
