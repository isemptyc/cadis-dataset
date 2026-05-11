# CU_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `cu.admin`
Version: `v1.0.0`
Country: `CU`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `cu.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Cuba dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `cu.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `CU`              |
| Country Name            | `Cuba`            |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.54`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_cu_boundaries.py`       |
| Generated Boundary       | `tmp/cu_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 4 and 6 whose representative point is covered by the Natural Earth CU boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox            | `[-85.16596257503407, 19.629530231433268, -73.92101221606362, 23.47969657320431]` |
| OSM Source               | `geofabrik:central-america/cuba`       |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Cuba administrative coverage represented by materialized OSM province and municipality relations. The level-5 Mariel special development zone overlay is excluded from the initial runtime contract.

---

# 4. Administrative Model

The initial Cuba engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_province`     | 15            | Province-level coverage |
| 6     | `admin_municipality` | 168           | Municipality-level coverage |
| 7     | `admin_district`     | 11            | District/submunicipal coverage where available |
| 8     | `admin_locality`     | 142           | Locality coverage where available |
| 9     | `admin_neighborhood` | 125           | Neighborhood coverage where available |
| 10    | `admin_detail`       | 9             | Detail coverage where available |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 5     | 1           | Excluded as a special development zone overlay |
| 11    | 1           | Excluded because it did not materialize in the scoped dataset build |

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
* Dataset path: `CU/cu.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

Sampling is uniform over scoped administrative coverage, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value           |
| ------------------------- | --------------- |
| Overall Pass Rate         | `100.00%`       |
| Inside Coverage Pass Rate | `100.00%`       |
| Policy Pass Rate          | `100.00%`       |
| Failed Samples            | `0`             |
| Throughput                | `16465.730` QPS |
| Total Runtime             | `0.607 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 8,880 / 135 / 985 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 8,880 / 135 / 985 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 8,880 / 135 / 985 / 0 |
| no_nearby    | 99.91%    | 99.90%           | 9      | 9             | 8,856 / 135 / 1,009 / 0 |
| osm_only     | 99.91%    | 99.90%           | 9      | 9             | 8,856 / 135 / 1,009 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 24              |
| Total vs OSM-only | 24              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `cu.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape     | Count |
| --------- | ----: |
| `[4]`     | 4,386 |
| `[4,6]`   | 4,261 |
| `[]`      | 1,000 |
| `[6]`     | 135   |
| `[4,6,7]` | 93    |
| `[4,6,9]` | 85    |
| `[4,6,8]` | 36    |
| `[4,9]`   | 1     |

Empty shapes correspond to:

* `empty_shape`: 985
* `offshore`: 15

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,991 |
| nearby        | 9     |
| admin_tree_id | 1     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,990 |
| none                   | 1,000 |
| nearby                 | 9     |
| admin_tree_id\|polygon | 1     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,000 |
| empty_shape      | 985   |
| offshore         | 15    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `15`
* Total level-4 hits across all samples: `8,865`
* Total level-4 hits across inside samples: `8,865`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Camagüey | 1,175 | 11.75% | 1,175 | 13.06% |
| Pinar del Río | 1,106 | 11.06% | 1,106 | 12.29% |
| Matanzas | 972 | 9.72% | 972 | 10.80% |
| Villa Clara | 700 | 7.00% | 700 | 7.78% |
| Granma | 693 | 6.93% | 693 | 7.70% |
| Ciego de Ávila | 674 | 6.74% | 674 | 7.49% |
| Holguín | 639 | 6.39% | 639 | 7.10% |
| Guantánamo | 510 | 5.10% | 510 | 5.67% |
| Sancti Spíritus | 506 | 5.06% | 506 | 5.62% |
| Las Tunas | 441 | 4.41% | 441 | 4.90% |

Hit distribution reflects uniform sampling over the scoped administrative coverage.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/cu_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Cuba dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shapes are `[4]` and `[4,6]`, reflecting broad province and municipality coverage.
3. Levels 7, 8, 9, and 10 provide valid lower-level detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `1d0088addbb28ca50bf1aee796350943e16b37e4`
- Cadis version:
  `0.8.54`
- Boundary builder: `scripts/build_cu_boundaries.py`
- Build boundary: `tmp/cu_country.json`
- Evaluation boundary: `tmp/cu_country.json`
- Staged dataset: `CU/cu.admin/v1.0.0`
- Source OSM SHA256:
  `ed2edc50da2d97837bacedbcaa33aa158092f71620ff7c3252ef30335ed1bb82`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `cu.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Cuba levels 4, 6, 7, 8, 9, and 10

The dataset is suitable for human quality review.
