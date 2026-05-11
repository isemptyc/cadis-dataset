# EC_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ec.admin`
Version: `v1.0.0`
Country: `EC`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ec.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Ecuador dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `ec.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `EC`              |
| Country Name            | `Ecuador`         |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.45`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_ec_boundaries.py`       |
| Generated Boundary       | `tmp/ec_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 4 and 6 whose representative point is covered by the Natural Earth EC boundary.` |
| Boundary BBox            | `[-81.0847665, -5.0159314, -75.192504, 1.469581]` |
| OSM Source               | `geofabrik:south-america/ecuador`      |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The selected source PBF provides strong mainland administrative coverage but does not provide matching administrative hierarchy coverage for the full Natural Earth Ecuador extent in the Galapagos longitude range. The v1.0.0 release therefore uses the tracked administrative-coverage boundary instead of the full Natural Earth country envelope.

---

# 4. Administrative Model

The initial Ecuador engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_province`     | 23            | Provinces represented in the selected source PBF |
| 6     | `admin_canton`       | 217           | Cantons |
| 8     | `admin_parish`       | 1,027         | Parishes |
| 9     | `admin_detail`       | 99            | Detail coverage where available |
| 10    | `admin_neighborhood` | 137           | Neighborhood/detail coverage |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 3     | 0           | Excluded because no scoped features were present |
| 5     | 0           | Excluded because no scoped features were present |
| 7     | 0           | Excluded because no scoped features were present |

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
* Dataset path: `EC/ec.admin/v1.0.0`

The test intentionally injects about 10% out-of-scope points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

Sampling is uniform over administrative coverage area, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value          |
| ------------------------- | -------------- |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |
| Failed Samples            | `0`            |
| Throughput                | `8111.540` QPS |
| Total Runtime             | `1.233 sec`    |

This run confirms no policy or inside-coverage failures under the scoped administrative-coverage boundary.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 8,680 / 325 / 995 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 8,680 / 325 / 995 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 8,680 / 325 / 995 / 0 |
| no_nearby    | 99.99%    | 99.99%           | 1      | 1             | 8,675 / 324 / 1,001 / 0 |
| osm_only     | 99.99%    | 99.99%           | 1      | 1             | 8,675 / 324 / 1,001 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 6               |
| Total vs OSM-only | 6               |

Nearby fallback resolves a small number of boundary-adjacent samples that otherwise miss polygon containment. No explicit repair layer is required for `ec.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape         | Count |
| ------------- | ----: |
| `[4,6,8]`     | 8,654 |
| `[]`          | 999   |
| `[4]`         | 138   |
| `[4,8]`       | 107   |
| `[4,6]`       | 56    |
| `[4,6,9]`    | 22    |
| `[4,6,8,9]`  | 19    |
| `[4,6,8,10]` | 3     |

Empty shapes correspond to:

* `empty_shape`: 995
* `offshore`: 4

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,999 |
| admin_tree_id | 21    |
| nearby        | 2     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,978 |
| none                   | 999   |
| admin_tree_id\|polygon | 21    |
| nearby                 | 2     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,001 |
| empty_shape      | 995   |
| offshore         | 4     |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `23`
* Total level-4 hits across all samples: `9,001`
* Total level-4 hits across inside samples: `9,000`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Pastaza | 1,001 | 10.01% | 1,001 | 11.12% |
| Morona Santiago | 868 | 8.68% | 868 | 9.64% |
| Orellana | 789 | 7.89% | 788 | 8.76% |
| Manabí | 742 | 7.42% | 742 | 8.24% |
| Guayas | 630 | 6.30% | 630 | 7.00% |
| Sucumbíos | 624 | 6.24% | 624 | 6.93% |
| Esmeraldas | 532 | 5.32% | 532 | 5.91% |
| Napo | 449 | 4.49% | 449 | 4.99% |
| Zamora Chinchipe | 393 | 3.93% | 393 | 4.37% |
| Loja | 387 | 3.87% | 387 | 4.30% |

Hit distribution reflects uniform administrative-coverage-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-scope samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ec_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Ecuador dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,6,8]`, reflecting province, canton, and parish coverage.
3. Level 9 and level 10 add sparse detail coverage where available.
4. Nearby fallback is bounded at 2 km and accounts for a very small rescue effect around administrative edges.
5. The release scope is administrative coverage rather than the full Natural Earth country envelope because the selected PBF does not provide matching administrative hierarchy coverage across the full Ecuador extent.
6. Dataset achieves full inside-boundary coverage under full policy mode for the scoped boundary.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `51712199161c4a47d8888e96ff7282d901f36297`
- Cadis version:
  `0.8.45`
- Boundary builder: `scripts/build_ec_boundaries.py`
- Build boundary: `tmp/ec_country.json`
- Evaluation boundary: `tmp/ec_country.json`
- Staged dataset: `EC/ec.admin/v1.0.0`
- Source OSM SHA256:
  `6721bd38148198b2f21dd90cb06c3324cc12578697045ba897fb8e2fe420ba40`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ec.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode for the scoped administrative-coverage boundary
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Ecuador levels 4, 6, 8, 9, and 10

The dataset is suitable for human quality review.
