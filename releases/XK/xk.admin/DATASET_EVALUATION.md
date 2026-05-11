# XK_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `xk.admin`
Version: `v1.0.0`
Country: `XK`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `xk.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Kosovo dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `xk.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `XK`              |
| Country Name            | `Kosovo`          |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.40`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_xk_boundaries.py`       |
| Generated Boundary       | `tmp/xk_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth XK admin-0 country boundary.` |
| Boundary BBox            | `[20.024751424000016, 41.84401031600008, 21.772758422000067, 43.26307098400004]` |
| OSM Source               | `geofabrik:europe/kosovo`              |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Kosovo engine exposes these OSM administrative levels:

| Level | Runtime Label         | Dataset Count | Notes |
| ----: | --------------------- | ------------: | ----- |
| 5     | `admin_region`        | 7             | Regions |
| 6     | `admin_municipality`  | 38            | Municipalities |
| 8     | `admin_settlement`    | 165           | Settlements |
| 9     | `admin_locality`      | 56            | Locality-level areas |
| 10    | `admin_neighborhood`  | 13            | Sparse neighborhood/detail features |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 3     | 0           | Excluded because no scoped features were found |
| 4     | 0           | Excluded because no scoped features were found |
| 7     | 0           | Excluded because no scoped features were found |

The engine uses canonical names from `name:sq`, `name:sr`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `en`, `sq`, `sr`, and `sr-latn`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `XK/xk.admin/v1.0.0`

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
| Throughput                | `14698.550` QPS |
| Total Runtime             | `0.680 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,055 / 11 / 934 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,055 / 11 / 934 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,055 / 11 / 934 / 0 |
| no_nearby    | 97.36%    | 97.07%           | 264    | 264           | 8,727 / 11 / 1,262 / 0 |
| osm_only     | 97.36%    | 97.07%           | 264    | 264           | 8,727 / 11 / 1,262 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 328             |
| Total vs OSM-only | 328             |

Nearby fallback resolves boundary-adjacent and administrative-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `xk.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape        | Count |
| ------------ | ----: |
| `[5,6]`      | 7,888 |
| `[]`         | 983   |
| `[5,6,8]`   | 887   |
| `[5,6,9]`   | 207   |
| `[5,6,8,10]` | 12   |
| `[5]`        | 12    |
| `[8]`        | 7     |
| `[9]`        | 4     |

Empty shapes correspond to:

* `empty_shape`: 934
* `offshore`: 49

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,738 |
| nearby        | 279   |
| admin_tree_id | 19    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,719 |
| none                   | 983   |
| nearby                 | 279   |
| admin_tree_id\|polygon | 19    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,017 |
| empty_shape      | 934   |
| offshore         | 49    |

---

# 10. Level-4 Coverage

No level-4 city hit rates are reported for this dataset because the Kosovo runtime contract does not expose level 4.

The primary runtime parent layer is level 5 region coverage.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/xk_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Kosovo dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[5,6]`, reflecting region and municipality coverage.
3. Levels 8, 9, and 10 add settlement, locality, and neighborhood detail where available.
4. Nearby fallback is bounded at 5 km and accounts for a meaningful rescue effect around administrative edges.
5. Country-envelope and empty levels are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `51451a0dc2ec9dde16fe4c461c7d9f35be12c3fc`
- Cadis version:
  `0.8.40`
- Boundary builder: `scripts/build_xk_boundaries.py`
- Build boundary: `tmp/xk_country.json`
- Evaluation boundary: `tmp/xk_country.json`
- Staged dataset: `XK/xk.admin/v1.0.0`
- Source OSM SHA256:
  `b63cbe57cbf49ab6d4b188a686180e4680589b5e9ce9bce4de410966b7f02938`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `xk.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Kosovo levels 5, 6, 8, 9, and 10

The dataset is suitable for human quality review.
