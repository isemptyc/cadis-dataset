# RO_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ro.admin`
Version: `v1.0.0`
Country: `RO`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ro.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Romania dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `ro.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `RO`              |
| Country Name            | `Romania`         |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.34`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_ro_boundaries.py`       |
| Generated Boundary       | `tmp/ro_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth RO admin-0 country boundary.` |
| Boundary BBox            | `[20.24282596900008, 43.6500499480001, 29.699554884000065, 48.27483225600007]` |
| OSM Source               | `geofabrik:europe/romania`             |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Romania engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_county`       | 42            | Counties and Bucharest |
| 8     | `admin_municipality` | 3,165         | Cities, towns, communes, and municipalities |
| 9     | `admin_locality`     | 19            | Sparse localities, including Bucharest sectors |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 7     | 7           | Excluded as metropolitan association coverage |
| 10    | 4           | Excluded as sparse neighborhood / border detail |

The engine uses canonical names from `name:ro`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `en`, `hu`, `ro`, `ru`, and `uk`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `RO/ro.admin/v1.0.0`

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
| Throughput                | `11604.000` QPS |
| Total Runtime             | `0.862 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,018 / 1 / 981 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,018 / 1 / 981 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,018 / 1 / 981 / 0 |
| no_nearby    | 99.73%    | 99.70%           | 27     | 27            | 8,972 / 1 / 1,027 / 0 |
| osm_only     | 99.73%    | 99.70%           | 27     | 27            | 8,972 / 1 / 1,027 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 46              |
| Total vs OSM-only | 46              |

## Interpretation

* OSM-only success rate: `99.73%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves a small number of boundary-adjacent samples that otherwise miss polygon containment.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `ro.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape     | Count |
| --------- | ----: |
| `[4,8]`   | 8,945 |
| `[]`      | 1,000 |
| `[4]`     | 35    |
| `[4,9]`   | 15    |
| `[4,8,9]` | 4     |
| `[8]`     | 1     |

Empty shapes correspond to:

* `empty_shape`: 981
* `offshore`: 19

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,973 |
| nearby        | 27    |
| admin_tree_id | 15    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,958 |
| none                   | 1,000 |
| nearby                 | 27    |
| admin_tree_id\|polygon | 15    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,000 |
| empty_shape      | 981   |
| offshore         | 19    |

---

# 10. Level-4 Coverage

* Unique level-4 county units hit: `42`
* Total level-4 hits across all samples: `8,999`
* Total level-4 hits across inside samples: `8,997`

| Level-4 Unit       | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Suceava            | 335  | 3.35%                   | 335                   | 3.72%                      |
| Timiș              | 326  | 3.26%                   | 326                   | 3.62%                      |
| Bihor              | 321  | 3.21%                   | 321                   | 3.57%                      |
| Arad               | 302  | 3.02%                   | 302                   | 3.36%                      |
| Caraș-Severin      | 298  | 2.98%                   | 298                   | 3.31%                      |
| Cluj               | 296  | 2.96%                   | 296                   | 3.29%                      |
| Tulcea             | 287  | 2.87%                   | 285                   | 3.17%                      |
| Hunedoara          | 287  | 2.87%                   | 287                   | 3.19%                      |
| Dolj               | 275  | 2.75%                   | 275                   | 3.06%                      |
| Alba               | 266  | 2.66%                   | 266                   | 2.96%                      |
| Mureș              | 260  | 2.60%                   | 260                   | 2.89%                      |
| Argeș              | 248  | 2.48%                   | 248                   | 2.76%                      |
| Buzău              | 246  | 2.46%                   | 246                   | 2.73%                      |
| Constanța          | 239  | 2.39%                   | 239                   | 2.66%                      |
| Neamț              | 236  | 2.36%                   | 236                   | 2.62%                      |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ro_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Romania dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,8]`, reflecting county and municipality/commune coverage.
3. Level 9 adds sparse local detail, including Bucharest sectors where available.
4. Nearby fallback is bounded and accounts for a small rescue effect.
5. Levels 7 and 10 are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `96892505571567bfab099e36fbe07dd1efc63afb`
- Cadis version:
  `0.8.34`
- Boundary builder: `scripts/build_ro_boundaries.py`
- Build boundary: `tmp/ro_country.json`
- Evaluation boundary: `tmp/ro_country.json`
- Staged dataset: `RO/ro.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ro.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Romania levels 4, 8, and 9

The dataset is suitable for human quality review.
