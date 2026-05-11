# BS_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `bs.admin`
Version: `v1.0.0`
Country: `BS`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `bs.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Bahamas dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `bs.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `BS`              |
| Country Name            | `Bahamas`         |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.51`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_bs_boundaries.py`       |
| Generated Boundary       | `tmp/bs_country.json`                  |
| Boundary Source          | `OSM administrative relations`         |
| Boundary Selection Rule  | `Union OSM administrative relation areas at level 8 from the selected Bahamas PBF.` |
| Boundary BBox            | `[-80.7001941, 20.7059846, -72.5794998, 27.4734551]` |
| OSM Source               | `geofabrik:central-america/bahamas`    |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Bahamas administrative coverage represented by OSM level-8 district relations. The raw Geofabrik extract also contains neighboring country and province relations; those are excluded from the Bahamas runtime contract.

---

# 4. Administrative Model

The initial Bahamas engine exposes these OSM administrative levels:

| Level | Runtime Label    | Dataset Count | Notes |
| ----: | ---------------- | ------------: | ----- |
| 8     | `admin_district` | 33            | District-level coverage |
| 9     | `admin_locality` | 2             | Local detail coverage where available |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 4           | Excluded because these are country envelopes in the raw extract, including neighboring countries |
| 4     | 4           | Excluded because these are neighboring Cuba province relations in the raw extract |

The engine uses canonical names from `name:en`, `name`, and `official_name`, with bounded multilingual aliases for `en`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `BS/bs.admin/v1.0.0`

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
| Throughput                | `19942.930` QPS |
| Total Runtime             | `0.501 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,009 / 0 / 991 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,009 / 0 / 991 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,009 / 0 / 991 / 0 |
| no_nearby    | 99.95%    | 99.94%           | 5      | 5             | 8,995 / 0 / 1,005 / 0 |
| osm_only     | 99.95%    | 99.94%           | 5      | 5             | 8,995 / 0 / 1,005 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 14              |
| Total vs OSM-only | 14              |

Nearby fallback resolves a small number of boundary-adjacent and island geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `bs.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[8]` | 9,000 |
| `[]`  | 1,000 |

Empty shapes correspond to:

* `empty_shape`: 991
* `offshore`: 9

## 9.2 Node Source Distribution

| Source  | Count |
| ------- | ----: |
| polygon | 8,995 |
| nearby  | 5     |

## 9.3 Source Mix Distribution

| Mix     | Count |
| ------- | ----: |
| polygon | 8,995 |
| none    | 1,000 |
| nearby  | 5     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,000 |
| empty_shape      | 991   |
| offshore         | 9     |

---

# 10. Administrative Coverage

* Level-8 district nodes: `33`
* Level-9 detail nodes: `2`
* Total hierarchy nodes: `35`
* Dataset hierarchy edges: `2`
* Unresolved `is_in` edges: `0`

This dataset has no level-4 administrative model; the highest exposed Bahamas administrative level is level 8.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/bs_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Bahamas dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[8]`, reflecting district-level administrative coverage across uniformly sampled land points.
3. Level 9 exists only for two local detail polygons and did not materially affect random land-area sampling.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around island administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `eb526cc6a74667683f2369d13d512b180948831c`
- Cadis version:
  `0.8.51`
- Boundary builder: `scripts/build_bs_boundaries.py`
- Build boundary: `tmp/bs_country.json`
- Evaluation boundary: `tmp/bs_country.json`
- Staged dataset: `BS/bs.admin/v1.0.0`
- Source OSM SHA256:
  `20d2d33afae78b839e3f6e3db05158e5a48ec62dd9e22ce6186855aa681dc1c8`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `bs.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Bahamas levels 8 and 9

The dataset is suitable for human quality review.
