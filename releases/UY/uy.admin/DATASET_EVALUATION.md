# UY_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `uy.admin`
Version: `v1.0.0`
Country: `UY`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `uy.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Uruguay dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `uy.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `UY`              |
| Country Name            | `Uruguay`         |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.49`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_uy_boundaries.py`       |
| Generated Boundary       | `tmp/uy_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth UY admin-0 country boundary.` |
| Boundary BBox            | `[-58.43936113199993, -34.97340260199991, -53.1108361419999, -30.096869811999937]` |
| OSM Source               | `geofabrik:south-america/uruguay`      |
| OSM Snapshot Timestamp   | `2026-05-09T20:21:02Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Uruguay engine exposes these OSM administrative levels:

| Level | Runtime Label      | Dataset Count | Notes |
| ----: | ------------------ | ------------: | ----- |
| 4     | `admin_department` | 19            | Departments |
| 8     | `admin_locality`   | 620           | Locality/municipal coverage |
| 10    | `admin_detail`     | 128           | Detail coverage where available |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 3     | 0           | Excluded because no scoped features were present |
| 5     | 0           | Excluded because no scoped features were present |
| 6     | 0           | Excluded because no scoped features were present |
| 7     | 0           | Excluded because no scoped features were present |
| 9     | 0           | Excluded because no scoped features were present |

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
* Dataset path: `UY/uy.admin/v1.0.0`

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
| Throughput                | `19904.820` QPS |
| Total Runtime             | `0.502 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,020 / 1 / 979 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,020 / 1 / 979 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,020 / 1 / 979 / 0 |
| no_nearby    | 99.12%    | 99.02%           | 88     | 88            | 8,911 / 1 / 1,088 / 0 |
| osm_only     | 99.12%    | 99.02%           | 88     | 88            | 8,911 / 1 / 1,088 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 109             |
| Total vs OSM-only | 109             |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `uy.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape      | Count |
| ---------- | ----: |
| `[4]`      | 8,842 |
| `[]`       | 1,019 |
| `[4,8]`   | 126   |
| `[4,8,10]` | 12   |
| `[8]`      | 1     |

Empty shapes correspond to:

* `empty_shape`: 979
* `offshore`: 40

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,912 |
| nearby        | 69    |
| admin_tree_id | 1     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,911 |
| none                   | 1,019 |
| nearby                 | 69    |
| admin_tree_id\|polygon | 1     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 8,981 |
| empty_shape      | 979   |
| offshore         | 40    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `19`
* Total level-4 hits across all samples: `8,980`
* Total level-4 hits across inside samples: `8,978`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Tacuarembó | 781 | 7.81% | 781 | 8.68% |
| Salto | 771 | 7.71% | 771 | 8.57% |
| Paysandú | 736 | 7.36% | 736 | 8.18% |
| Cerro Largo | 648 | 6.48% | 648 | 7.20% |
| Durazno | 597 | 5.97% | 597 | 6.63% |
| Rocha | 572 | 5.72% | 572 | 6.36% |
| Artigas | 565 | 5.65% | 565 | 6.28% |
| Florida | 551 | 5.51% | 551 | 6.12% |
| Treinta y Tres | 512 | 5.12% | 512 | 5.69% |
| Lavalleja | 508 | 5.08% | 508 | 5.64% |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/uy_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Uruguay dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4]`, reflecting broad department-level coverage across most uniformly sampled land points.
3. Levels 8 and 10 provide valid lower-level detail where available, but they are sparse in sampled lookup results.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `3d38ba56b67938db802748abacbdac0fbfacfa58`
- Cadis version:
  `0.8.49`
- Boundary builder: `scripts/build_uy_boundaries.py`
- Build boundary: `tmp/uy_country.json`
- Evaluation boundary: `tmp/uy_country.json`
- Staged dataset: `UY/uy.admin/v1.0.0`
- Source OSM SHA256:
  `4d91fc548866a9488f42a19620292f3d8a5465482d57ee6458219b9826baa2b0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `uy.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Uruguay levels 4, 8, and 10

The dataset is suitable for human quality review.
