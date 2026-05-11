# MD_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `md.admin`
Version: `v1.0.0`
Country: `MD`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `md.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Moldova dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `md.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `MD`              |
| Country Name            | `Moldova`         |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.42`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_md_boundaries.py`       |
| Generated Boundary       | `tmp/md_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth MD admin-0 country boundary.` |
| Boundary BBox            | `[26.617889038000015, 45.46177398700006, 30.131576375000122, 48.486033834000054]` |
| OSM Source               | `geofabrik:europe/moldova`             |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Moldova engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_district`     | 37            | Districts, municipalities, and autonomous/special units |
| 8     | `admin_locality`     | 973           | Localities |
| 9     | `admin_neighborhood` | 1,423         | Neighborhood/detail coverage |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 3     | 1           | Excluded as a single broad Nistrenia overlay |
| 5     | 7           | Excluded as Transnistria-specific overlay coverage |

The engine uses canonical names from `name:ro`, `name:ru`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `en`, `ro`, `ru`, and `uk`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `MD/md.admin/v1.0.0`

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
| Throughput                | `10841.770` QPS |
| Total Runtime             | `0.922 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,035 / 0 / 965 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,035 / 0 / 965 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,035 / 0 / 965 / 0 |
| no_nearby    | 99.05%    | 98.94%           | 95     | 95            | 8,905 / 0 / 1,095 / 0 |
| osm_only     | 99.05%    | 98.94%           | 95     | 95            | 8,905 / 0 / 1,095 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 130             |
| Total vs OSM-only | 130             |

Nearby fallback resolves boundary-adjacent and district/locality-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `md.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape     | Count |
| --------- | ----: |
| `[4,8]`   | 8,120 |
| `[]`      | 992   |
| `[4,8,9]` | 844  |
| `[4]`     | 42    |
| `[4,9]`   | 2     |

Empty shapes correspond to:

* `empty_shape`: 965
* `offshore`: 27

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,905 |
| nearby        | 103   |
| admin_tree_id | 24    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,881 |
| none                   | 992   |
| nearby                 | 103   |
| admin_tree_id\|polygon | 24    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,008 |
| empty_shape      | 965   |
| offshore         | 27    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `37`
* Total level-4 hits across all samples: `9,008`
* Total level-4 hits across inside samples: `9,000`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Unitățile administrativ-teritoriale din stînga Nistrului | 764 | 7.64% | 761 | 8.46% |
| Găgăuzia | 476 | 4.76% | 476 | 5.29% |
| Raionul Cahul | 432 | 4.32% | 432 | 4.80% |
| Raionul Căușeni | 412 | 4.12% | 412 | 4.58% |
| Raionul Hîncești | 369 | 3.69% | 369 | 4.10% |
| Raionul Orhei | 320 | 3.20% | 320 | 3.56% |
| Raionul Ungheni | 312 | 3.12% | 312 | 3.47% |
| Raionul Fălești | 304 | 3.04% | 304 | 3.38% |
| Raionul Florești | 299 | 2.99% | 299 | 3.32% |
| Raionul Soroca | 277 | 2.77% | 277 | 3.08% |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/md_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Moldova dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,8]`, reflecting district and locality coverage.
3. Level 9 adds dense neighborhood/detail coverage where available.
4. Nearby fallback is bounded at 5 km and accounts for a small rescue effect around administrative edges.
5. Levels 3 and 5 are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `387098d9b74a694d1dbac05aeec744a877876852`
- Cadis version:
  `0.8.42`
- Boundary builder: `scripts/build_md_boundaries.py`
- Build boundary: `tmp/md_country.json`
- Evaluation boundary: `tmp/md_country.json`
- Staged dataset: `MD/md.admin/v1.0.0`
- Source OSM SHA256:
  `f4ada111bee3a9c12f3b86d1cb654282d3fea613a00743f611a4c42e93520fbb`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `md.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Moldova levels 4, 8, and 9

The dataset is suitable for human quality review.
