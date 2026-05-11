# RS_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `rs.admin`
Version: `v1.0.0`
Country: `RS`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `rs.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Serbia dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `rs.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `RS`              |
| Country Name            | `Serbia`          |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.35`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_rs_boundaries.py`       |
| Generated Boundary       | `tmp/rs_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth RS admin-0 country boundary.` |
| Boundary BBox            | `[18.84497847500006, 42.23494482000011, 22.98457076000011, 46.17387522400013]` |
| OSM Source               | `geofabrik:europe/serbia`              |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Serbia engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 6     | `admin_district`     | 25            | Districts and Belgrade |
| 8     | `admin_municipality` | 147           | Municipalities and cities |
| 9     | `admin_locality`     | 4,694         | Settlements and locality-level areas |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 4     | 2           | Excluded as broad regional coverage |
| 7     | 28          | Excluded as small city overlay coverage |
| 10    | 329         | Excluded as sparse neighborhood detail |

The engine uses canonical names from `name:sr-Latn`, `name:sr`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `en`, `hr`, `hu`, `ro`, `sr`, and `sr-latn`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `RS/rs.admin/v1.0.0`

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
| Throughput                | `9037.930` QPS |
| Total Runtime             | `1.106 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,006 / 17 / 977 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,006 / 17 / 977 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,006 / 17 / 977 / 0 |
| no_nearby    | 98.72%    | 98.58%           | 128    | 128           | 8,856 / 17 / 1,127 / 0 |
| osm_only     | 98.72%    | 98.58%           | 128    | 128           | 8,856 / 17 / 1,127 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 150             |
| Total vs OSM-only | 150             |

## Interpretation

* OSM-only success rate: `98.72%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves boundary-adjacent and settlement-edge samples that otherwise miss polygon containment.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `rs.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape     | Count |
| --------- | ----: |
| `[6,8,9]` | 6,676 |
| `[6,9]`   | 2,285 |
| `[]`      | 991   |
| `[6,8]`   | 21    |
| `[8]`     | 16    |
| `[6]`     | 10    |
| `[9]`     | 1     |

Empty shapes correspond to:

* `empty_shape`: 977
* `offshore`: 14

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,873 |
| nearby        | 136   |
| admin_tree_id | 10    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,863 |
| none                   | 991   |
| nearby                 | 136   |
| admin_tree_id\|polygon | 10    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,009 |
| empty_shape      | 977   |
| offshore         | 14    |

---

# 10. Level-4 Coverage

No level-4 city hit rates are reported for this dataset because the Serbia runtime contract does not expose level 4.

The primary top-level runtime parent is level 6.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/rs_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Serbia dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[6,8,9]`, reflecting district, municipality, and settlement coverage.
3. Level 9 provides dense settlement/locality coverage and materially improves lookup detail.
4. Nearby fallback is bounded and accounts for a moderate rescue effect around polygon edges.
5. Levels 7 and 10 are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `e6a356544067c6a3ffb122fde567b3b3cad39b28`
- Cadis version:
  `0.8.35`
- Boundary builder: `scripts/build_rs_boundaries.py`
- Build boundary: `tmp/rs_country.json`
- Evaluation boundary: `tmp/rs_country.json`
- Staged dataset: `RS/rs.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `rs.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Serbia levels 6, 8, and 9

The dataset is suitable for human quality review.
