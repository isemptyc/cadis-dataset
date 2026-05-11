# LT_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `lt.admin`
Version: `v1.0.0`
Country: `LT`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `lt.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Lithuania dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `lt.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `LT`              |
| Country Name            | `Lithuania`       |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.31`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_lt_boundaries.py`       |
| Generated Boundary       | `tmp/lt_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth LT admin-0 country boundary.` |
| Boundary BBox            | `[20.924568700365, 53.88684112599999, 26.80072025600009, 56.44260243700002]` |
| OSM Source               | `geofabrik:europe/lithuania`           |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Lithuania engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_county`       | 10            | Counties |
| 5     | `admin_municipality` | 60            | Municipalities |
| 6     | `admin_eldership`    | 504           | Elderships |
| 8     | `admin_settlement`   | 4,426         | Settlements and populated places |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 10    | 54          | Excluded as sparse city/neighborhood and border-detail coverage |

The engine uses canonical names from `name:lt`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `be`, `lt`, `pl`, `ru`, and `uk`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `LT/lt.admin/v1.0.0`

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
| Throughput                | `8521.250` QPS |
| Total Runtime             | `1.174 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,024 / 3 / 973 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,024 / 3 / 973 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,024 / 3 / 973 / 0 |
| no_nearby    | 99.19%    | 99.10%           | 81     | 81            | 8,921 / 3 / 1,076 / 0 |
| osm_only     | 99.19%    | 99.10%           | 81     | 81            | 8,921 / 3 / 1,076 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 103             |
| Total vs OSM-only | 103             |

## Interpretation

* OSM-only success rate: `99.19%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves boundary-adjacent samples that otherwise miss polygon containment.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `lt.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape       | Count |
| ----------- | ----: |
| `[4,5,6]`   | 7,058 |
| `[4,5,6,8]` | 1,718 |
| `[]`        | 997   |
| `[4,5,8]`   | 146   |
| `[4,5]`     | 52    |
| `[4,6]`     | 18    |
| `[4]`       | 7     |
| `[8]`       | 1     |
| `[5,8]`     | 1     |
| `[5,6]`     | 1     |
| `[4,6,8]`   | 1     |

Empty shapes correspond to:

* `empty_shape`: 973
* `offshore`: 24

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,924 |
| nearby        | 79    |
| admin_tree_id | 15    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,909 |
| none                   | 997   |
| nearby                 | 79    |
| admin_tree_id\|polygon | 15    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,003 |
| empty_shape      | 973   |
| offshore         | 24    |

---

# 10. Level-4 Coverage

* Unique level-4 county units hit: `10`
* Total level-4 hits across all samples: `9,000`
* Total level-4 hits across inside samples: `8,995`

| Level-4 Unit             | Hits  | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------------------ | ----: | ----------------------: | --------------------: | -------------------------: |
| Vilniaus apskritis       | 1,287 | 12.87%                  | 1,286                 | 14.29%                     |
| Šiaulių apskritis        | 1,200 | 12.00%                  | 1,199                 | 13.32%                     |
| Kauno apskritis          | 1,119 | 11.19%                  | 1,119                 | 12.43%                     |
| Panevėžio apskritis      | 1,103 | 11.03%                  | 1,103                 | 12.26%                     |
| Utenos apskritis         | 956   | 9.56%                   | 955                   | 10.61%                     |
| Alytaus apskritis        | 788   | 7.88%                   | 788                   | 8.76%                      |
| Klaipėdos apskritis      | 703   | 7.03%                   | 702                   | 7.80%                      |
| Marijampolės apskritis   | 622   | 6.22%                   | 622                   | 6.91%                      |
| Telšių apskritis         | 616   | 6.16%                   | 616                   | 6.84%                      |
| Tauragės apskritis       | 606   | 6.06%                   | 605                   | 6.72%                      |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/lt_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Lithuania dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,5,6]`, reflecting county, municipality, and eldership coverage.
3. Settlement-level coverage appears through level 8 where OSM boundaries are available.
4. Nearby fallback is bounded and accounts for the only meaningful rescue effect.
5. Level 10 is intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `c77bcbc14ecf97287935110b508a92cf0a8ca0c9`
- Cadis version:
  `0.8.31`
- Boundary builder: `scripts/build_lt_boundaries.py`
- Build boundary: `tmp/lt_country.json`
- Evaluation boundary: `tmp/lt_country.json`
- Staged dataset: `LT/lt.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `lt.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Lithuania levels 4, 5, 6, and 8

The dataset is suitable for human quality review.
