# CR_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `cr.admin`
Version: `v1.0.0`
Country: `CR`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `cr.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Costa Rica dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `cr.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `CR`              |
| Country Name            | `Costa Rica`      |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.53`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_cr_boundaries.py`       |
| Generated Boundary       | `tmp/cr_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 4 and 6 whose representative point is covered by the Natural Earth CR boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox            | `[-87.09100014133432, 5.502516640969175, -82.43272402932725, 11.217330099992434]` |
| OSM Source               | `geofabrik:central-america/costa-rica` |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Costa Rica administrative coverage represented by materialized OSM province and canton relations. The raw extract includes country envelopes and neighboring administrative relations; those are excluded from the Costa Rica runtime contract.

---

# 4. Administrative Model

The initial Costa Rica engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_province`     | 6             | Materialized province-level coverage |
| 6     | `admin_canton`       | 84            | Canton-level coverage |
| 8     | `admin_district`     | 495           | District-level coverage |
| 10    | `admin_neighborhood` | 509           | Neighborhood/local detail coverage where available |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 5     | 1           | Excluded as a San José metropolitan overlay |

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
* Dataset path: `CR/cr.admin/v1.0.0`

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
| Throughput                | `7322.790` QPS |
| Total Runtime             | `1.366 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 7,412 / 1,601 / 987 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 7,412 / 1,601 / 987 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 7,412 / 1,601 / 987 / 0 |
| no_nearby    | 99.91%    | 99.90%           | 9      | 9             | 7,390 / 1,601 / 1,009 / 0 |
| osm_only     | 99.91%    | 99.90%           | 9      | 9             | 7,390 / 1,601 / 1,009 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 22              |
| Total vs OSM-only | 22              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `cr.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape         | Count |
| ------------- | ----: |
| `[4,6,8]`     | 5,736 |
| `[4]`         | 1,609 |
| `[6,8]`       | 1,598 |
| `[]`          | 998   |
| `[4,6,8,10]` | 42    |
| `[4,8]`       | 8     |
| `[4,6]`       | 6     |
| `[6]`         | 2     |

Empty shapes correspond to:

* `empty_shape`: 987
* `offshore`: 11

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,991 |
| admin_tree_id | 19    |
| nearby        | 11    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,972 |
| none                   | 998   |
| admin_tree_id\|polygon | 19    |
| nearby                 | 11    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,002 |
| empty_shape      | 987   |
| offshore         | 11    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `6`
* Total level-4 hits across all samples: `7,401`
* Total level-4 hits across inside samples: `7,399`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Guanacaste | 2,499 | 24.99% | 2,499 | 27.77% |
| Limón | 1,989 | 19.89% | 1,989 | 22.10% |
| Alajuela | 1,374 | 13.74% | 1,372 | 15.24% |
| San José | 716 | 7.16% | 716 | 7.96% |
| Heredia | 415 | 4.15% | 415 | 4.61% |
| Cartago | 408 | 4.08% | 408 | 4.53% |

Hit distribution reflects uniform sampling over the scoped administrative coverage.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/cr_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Costa Rica dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,6,8]`, reflecting strong province, canton, and district coverage across most scoped land points.
3. Level 10 provides valid lower-level detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `76523a9e55ac30f14c2a9311d96390b54ec6658d`
- Cadis version:
  `0.8.53`
- Boundary builder: `scripts/build_cr_boundaries.py`
- Build boundary: `tmp/cr_country.json`
- Evaluation boundary: `tmp/cr_country.json`
- Staged dataset: `CR/cr.admin/v1.0.0`
- Source OSM SHA256:
  `f413dd698972333d335e79a3acaba86cbf0dcc5c32caa321ae0d208b2b3689f1`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `cr.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Costa Rica levels 4, 6, 8, and 10

The dataset is suitable for human quality review.
