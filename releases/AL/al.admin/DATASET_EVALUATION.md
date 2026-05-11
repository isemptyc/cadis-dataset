# AL_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `al.admin`
Version: `v1.0.0`
Country: `AL`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `al.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Albania dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `al.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `AL`              |
| Country Name            | `Albania`         |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.24`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_al_boundaries.py`       |
| Generated Boundary       | `tmp/al_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth AL admin-0 country boundary.` |
| Boundary BBox            | `[19.272032511000106, 39.637013245000034, 21.036679321000065, 42.654813538000084]` |
| OSM Source               | `geofabrik:europe/albania`             |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Albania engine exposes these OSM administrative levels:

| Level | Runtime Label                 | Probe Count | Notes |
| ----: | ----------------------------- | ----------: | ----- |
| 4     | `admin_statistical_region`    | 3           | Broad Albania statistical regions |
| 6     | `admin_county`                | 12          | Albania counties |
| 7     | `admin_municipality`          | 61          | Municipalities |
| 8     | `admin_administrative_unit`   | 372         | Administrative units |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 9     | 21          | Excluded as sparse urban subdivision coverage |
| 10    | 21          | Excluded as sparse urban neighborhood coverage |

The engine uses canonical names from `name:sq`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `mk`, `el`, and `en`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `AL/al.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate:

* Boundary rejection behavior
* Offshore classification
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over land area, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value          |
| ------------------------- | -------------- |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |
| Failed Samples            | `0`            |
| Throughput                | `11390.170` QPS |
| Total Runtime             | `0.878 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,007 / 46 / 947 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,007 / 46 / 947 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,007 / 46 / 947 / 0 |
| no_nearby    | 99.43%    | 99.37%           | 57     | 57            | 8,900 / 46 / 1,054 / 0 |
| osm_only     | 99.43%    | 99.37%           | 57     | 57            | 8,900 / 46 / 1,054 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 107             |
| Total vs OSM-only | 107             |

## Interpretation

* OSM-only success rate: `99.43%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves boundary-adjacent and coastal-edge samples.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `al.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape       | Count |
| ----------- | ----: |
| `[4,6,7,8]` | 8,919 |
| `[]`        | 994   |
| `[4]`       | 46    |
| `[4,6,7]`   | 22    |
| `[4,7,8]`   | 9     |
| `[4,6,8]`   | 5     |
| `[4,6]`     | 3     |
| `[4,7]`     | 1     |

Empty shapes correspond to:

* `empty_shape`: 947
* `offshore`: 47

## 9.2 Node Source Distribution

| Source          | Count |
| --------------- | ----: |
| polygon         | 8,946 |
| nearby          | 60    |
| admin_tree_id   | 33    |

## 9.3 Source Mix Distribution

| Mix                   | Count |
| --------------------- | ----: |
| polygon               | 8,913 |
| none                  | 994   |
| nearby                | 60    |
| admin_tree_id\|polygon | 33    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,006 |
| empty_shape      | 947   |
| offshore         | 47    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `3`
* Total level-4 hits across all samples: `9,006`
* Total level-4 hits across inside samples: `9,000`

| Level-4 Unit          | Hits  | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| --------------------- | ----: | ----------------------: | --------------------: | -------------------------: |
| Shqipëria Jugore      | 4,007 | 40.07%                  | 4,003                 | 44.48%                     |
| Shqipëria Veriore     | 3,476 | 34.76%                  | 3,474                 | 38.60%                     |
| Shqipëria Qendrore    | 1,523 | 15.23%                  | 1,523                 | 16.92%                     |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/al_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Albania dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is the full `[4,6,7,8]` hierarchy.
3. Sparse `[4]` and shorter shapes are small and policy-accepted.
4. Nearby fallback is bounded and accounts for the only meaningful rescue effect.
5. Level 9 and level 10 urban subdivisions are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `24818656c6fb9bd746fbbe5b8890d8c2bac5096d`
- Cadis version:
  `0.8.24`
- Boundary builder: `scripts/build_al_boundaries.py`
- Build boundary: `tmp/al_country.json`
- Evaluation boundary: `tmp/al_country.json`
- Staged dataset: `AL/al.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `al.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Albania levels 4, 6, 7, and 8

The dataset is suitable for human quality review.
