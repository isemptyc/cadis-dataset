# ME_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `me.admin`
Version: `v1.0.0`
Country: `ME`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `me.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Montenegro dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `me.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `ME`              |
| Country Name            | `Montenegro`      |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.33`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_me_boundaries.py`       |
| Generated Boundary       | `tmp/me_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth ME admin-0 country boundary.` |
| Boundary BBox            | `[18.433530721000068, 41.85236237200007, 20.355170532000102, 43.547885641000036]` |
| OSM Source               | `geofabrik:europe/montenegro`          |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Montenegro engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 6     | `admin_municipality` | 25            | Municipalities, capital city, and old royal capital |
| 8     | `admin_settlement`   | 216           | Settlements and cities |
| 9     | `admin_locality`     | 72            | Local subareas and neighborhoods where available |

The engine uses canonical names from `name:sr-Latn`, `name:sr`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `cnr-latn`, `en`, `ru`, `sr`, and `sr-latn`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `ME/me.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate:

* Boundary rejection behavior
* Offshore classification
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over land area, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value           |
| ------------------------- | --------------- |
| Overall Pass Rate         | `100.00%`       |
| Inside Coverage Pass Rate | `100.00%`       |
| Policy Pass Rate          | `100.00%`       |
| Failed Samples            | `0`             |
| Throughput                | `17257.390` QPS |
| Total Runtime             | `0.579 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,032 / 11 / 957 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,032 / 11 / 957 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,032 / 11 / 957 / 0 |
| no_nearby    | 96.29%    | 95.88%           | 371    | 371           | 8,620 / 11 / 1,369 / 0 |
| osm_only     | 96.29%    | 95.88%           | 371    | 371           | 8,620 / 11 / 1,369 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 412             |
| Total vs OSM-only | 412             |

## Interpretation

* OSM-only success rate: `96.29%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves boundary-adjacent and coastal/terrain-edge samples that otherwise miss polygon containment.
* Montenegro uses a 5 km nearby radius because the initial 2 km policy left one inside-boundary sample unresolved.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `me.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape     | Count |
| --------- | ----: |
| `[6]`     | 7,968 |
| `[]`      | 990   |
| `[6,8]`   | 943   |
| `[6,9]`   | 52    |
| `[6,8,9]` | 36    |
| `[9]`     | 11    |

Empty shapes correspond to:

* `empty_shape`: 957
* `offshore`: 33

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,630 |
| nearby        | 380   |
| admin_tree_id | 9     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,622 |
| none                   | 990   |
| nearby                 | 379   |
| admin_tree_id\|polygon | 8     |
| admin_tree_id\|nearby  | 1     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,010 |
| empty_shape      | 957   |
| offshore         | 33    |

---

# 10. Level-6 Coverage

* Included level-6 municipality units: `25`
* Level-6 is the required parent scope for the Montenegro runtime policy.
* The evaluation summary does not emit a dedicated level-6 hit-rate table; the observed structural shapes show level-6 coverage in nearly all non-empty outcomes.

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/me_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Montenegro dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[6]`, reflecting robust municipality coverage with sparse lower-level polygons in uniformly sampled terrain.
3. Settlement and locality detail appears through levels 8 and 9 where OSM boundaries are available.
4. Nearby fallback is bounded and accounts for the only meaningful rescue effect.
5. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `622f47cf9a2a61a56299fef0e44c807666137ff2`
- Cadis version:
  `0.8.33`
- Boundary builder: `scripts/build_me_boundaries.py`
- Build boundary: `tmp/me_country.json`
- Evaluation boundary: `tmp/me_country.json`
- Staged dataset: `ME/me.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `me.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Montenegro levels 6, 8, and 9

The dataset is suitable for human quality review.
