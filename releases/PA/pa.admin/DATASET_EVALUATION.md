# PA_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `pa.admin`
Version: `v1.0.0`
Country: `PA`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `pa.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Panama dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `pa.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `PA`              |
| Country Name            | `Panama`          |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.62`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_pa_boundaries.py`       |
| Generated Boundary       | `tmp/pa_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 4 and 6 whose representative point is covered by the Natural Earth PA boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox            | `[-83.04975196079556, 7.004717105528309, -77.16221067591296, 9.633856712251589]` |
| OSM Source               | `geofabrik:central-america/panama`     |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Panama administrative coverage represented by materialized OSM province/comarca and district relations.

---

# 4. Administrative Model

The initial Panama engine exposes these OSM administrative levels:

| Level | Runtime Label               | Dataset Count | Notes |
| ----: | --------------------------- | ------------: | ----- |
| 4     | `admin_province`            | 7             | Province/comarca-level coverage |
| 6     | `admin_district`            | 82            | District-level coverage |
| 7     | `admin_comarca_subdivision` | 0             | Exposed by policy; no scoped nodes materialized in this build |
| 8     | `admin_corregimiento`       | 408           | Corregimiento-level coverage |
| 9     | `admin_neighborhood`        | 1             | Neighborhood coverage where available |
| 10    | `admin_detail`              | 570           | Detail coverage where available |
| 11    | `admin_micro_detail`        | 2             | Micro-detail coverage where available |

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
* Dataset path: `PA/pa.admin/v1.0.0`

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
| Throughput                | `2580.150` QPS |
| Total Runtime             | `3.876 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 5,180 / 3,834 / 986 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 5,180 / 3,834 / 986 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 5,180 / 3,834 / 986 / 0 |
| no_nearby    | 99.95%    | 99.94%           | 5      | 5             | 5,162 / 3,834 / 1,004 / 0 |
| osm_only     | 99.95%    | 99.94%           | 5      | 5             | 5,162 / 3,834 / 1,004 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 18              |
| Total vs OSM-only | 18              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `pa.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape        | Count |
| ------------ | ----: |
| `[6,8]`      | 3,094 |
| `[4,6,8]`   | 2,252 |
| `[4]`        | 1,774 |
| `[]`         | 999   |
| `[4,6]`      | 982   |
| `[6]`        | 731   |
| `[4,8]`      | 155   |
| `[6,8,10]`  | 9     |
| `[4,6,8,10]` | 4    |

Empty shapes correspond to:

* `empty_shape`: 986
* `offshore`: 13

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,996 |
| nearby        | 5     |
| admin_tree_id | 2     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,994 |
| none                   | 999   |
| nearby                 | 5     |
| admin_tree_id\|polygon | 2     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,001 |
| empty_shape      | 986   |
| offshore         | 13    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `7`
* Total level-4 hits across all samples: `5,167`
* Total level-4 hits across inside samples: `5,167`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Veraguas | 2,451 | 24.51% | 2,451 | 27.23% |
| Ngäbe-Buglé | 916 | 9.16% | 916 | 10.18% |
| Coclé | 564 | 5.64% | 564 | 6.27% |
| Panamá Oeste | 431 | 4.31% | 431 | 4.79% |
| Emberá-Wounaan | 416 | 4.16% | 416 | 4.62% |
| Herrera | 233 | 2.33% | 233 | 2.59% |
| Naso Tjër Di | 156 | 1.56% | 156 | 1.73% |

Hit distribution reflects uniform sampling over the scoped administrative coverage.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/pa_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Panama dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shapes are `[6,8]` and `[4,6,8]`, reflecting strong district and corregimiento coverage with partial level-4 parent coverage in the scoped OSM hierarchy.
3. Levels 9, 10, and 11 provide valid lower-level detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `60ae50480913810002cc0f2e6b79e7e20f5b255e`
- Cadis version:
  `0.8.62`
- Boundary builder: `scripts/build_pa_boundaries.py`
- Build boundary: `tmp/pa_country.json`
- Evaluation boundary: `tmp/pa_country.json`
- Staged dataset: `PA/pa.admin/v1.0.0`
- Source OSM SHA256:
  `60100857a2bea72628b3ed484c8896afc96a4155329225cf43b2892af5bddbb1`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `pa.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Panama levels 4, 6, 8, 9, 10, and 11, with level 7 reserved in policy for comarca subdivisions when scoped nodes materialize

The dataset is suitable for human quality review.
