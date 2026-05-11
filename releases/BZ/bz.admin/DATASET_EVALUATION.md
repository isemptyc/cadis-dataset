# BZ_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `bz.admin`
Version: `v1.0.0`
Country: `BZ`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `bz.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Belize dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `bz.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `BZ`              |
| Country Name            | `Belize`          |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.52`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_bz_boundaries.py`       |
| Generated Boundary       | `tmp/bz_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 4, 7, 8, 9, 10, and 11 whose representative point is covered by the Natural Earth BZ boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox            | `[-89.2254958375798, 15.888206183371535, -87.86056419753437, 18.493912157709524]` |
| OSM Source               | `geofabrik:central-america/belize`     |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Belize administrative coverage represented by materialized OSM administrative relations. The raw extract includes country envelopes and neighboring Guatemala/Mexico administrative relations; those are excluded from the Belize runtime contract. Two Belize level-4 district relations are present in the raw extract but do not materialize as scoped multipolygons from this clipped PBF, so this release is scoped to the materialized administrative coverage.

---

# 4. Administrative Model

The initial Belize engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_district`     | 4             | Materialized district-level coverage |
| 7     | `admin_municipality` | 4             | Municipality-level coverage |
| 8     | `admin_town`         | 17            | Town-level coverage |
| 9     | `admin_village`      | 136           | Village/local coverage |
| 10    | `admin_neighborhood` | 46            | Neighborhood coverage where available |
| 11    | `admin_detail`       | 8             | Detail coverage where available |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 3           | Excluded as country-level envelopes in the raw extract |
| 6     | 8           | Excluded because these are neighboring Guatemala/Mexico municipalities |

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
* Dataset path: `BZ/bz.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

Sampling is uniform over scoped administrative coverage, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value           |
| ------------------------- | --------------- |
| Overall Pass Rate         | `100.00%`       |
| Inside Coverage Pass Rate | `100.00%`       |
| Policy Pass Rate          | `100.00%`       |
| Failed Samples            | `0`             |
| Throughput                | `16771.060` QPS |
| Total Runtime             | `0.596 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 8,816 / 236 / 948 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 8,816 / 236 / 948 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 8,816 / 236 / 948 / 0 |
| no_nearby    | 99.88%    | 99.87%           | 12     | 12            | 8,755 / 236 / 1,009 / 0 |
| osm_only     | 99.88%    | 99.87%           | 12     | 12            | 8,755 / 236 / 1,009 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 61              |
| Total vs OSM-only | 61              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `bz.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape        | Count |
| ------------ | ----: |
| `[4]`        | 8,035 |
| `[]`         | 989   |
| `[4,8]`      | 354   |
| `[9]`        | 192   |
| `[4,9]`      | 168   |
| `[4,8,10]`  | 79    |
| `[4,7]`      | 68    |
| `[4,8,9]`   | 45    |

Empty shapes correspond to:

* `empty_shape`: 948
* `offshore`: 41

## 9.2 Node Source Distribution

| Source  | Count |
| ------- | ----: |
| polygon | 8,991 |
| nearby  | 20    |

## 9.3 Source Mix Distribution

| Mix     | Count |
| ------- | ----: |
| polygon | 8,991 |
| none    | 989   |
| nearby  | 20    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,011 |
| empty_shape      | 948   |
| offshore         | 41    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `4`
* Total level-4 hits across all samples: `8,775`
* Total level-4 hits across inside samples: `8,765`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Stann Creek | 2,760 | 27.60% | 2,758 | 30.64% |
| Cayo | 2,513 | 25.13% | 2,508 | 27.87% |
| Orange Walk | 2,360 | 23.60% | 2,358 | 26.20% |
| Corozal | 1,142 | 11.42% | 1,141 | 12.68% |

Hit distribution reflects uniform sampling over the scoped administrative coverage.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/bz_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Belize dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4]`, reflecting broad district-level coverage where level-4 polygons materialize from the source extract.
3. Levels 7, 8, 9, 10, and 11 provide valid lower-level detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `ffa38bc4b3e3d3148aec89e8d4262306d0a8c0f5`
- Cadis version:
  `0.8.52`
- Boundary builder: `scripts/build_bz_boundaries.py`
- Build boundary: `tmp/bz_country.json`
- Evaluation boundary: `tmp/bz_country.json`
- Staged dataset: `BZ/bz.admin/v1.0.0`
- Source OSM SHA256:
  `9f5512497c0e56ae4e8afe24110dd40931b19ba3c5e52b28c7f7b64d7626dfe0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `bz.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Belize levels 4, 7, 8, 9, 10, and 11

The dataset is suitable for human quality review.
