# AF_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `af.admin`
Version: `v1.0.0`
Country: `AF`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `af.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Afghanistan dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `af.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `AF`              |
| Country Name            | `Afghanistan`     |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.63`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_af_boundaries.py`       |
| Generated Boundary       | `tmp/af_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 4 and 5 whose representative point is covered by the Natural Earth AF boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox            | `[60.51969761033871, 29.37956731022669, 74.88739679670022, 38.48847022400434]` |
| OSM Source               | `geofabrik:asia/afghanistan`           |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Afghanistan administrative coverage represented by materialized OSM province and district relations. The country envelope is excluded from the initial runtime contract.

---

# 4. Administrative Model

The initial Afghanistan engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_province`     | 34            | Province-level coverage |
| 5     | `admin_district`     | 397           | District-level coverage |
| 7     | `admin_city_district` | 53           | City district coverage where available |
| 10    | `admin_detail`       | 1             | Detail coverage where available |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:fa`, and `name:ps`, with bounded multilingual aliases for `en`, `fa`, and `ps`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `AF/af.admin/v1.0.0`

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
| Throughput                | `3129.040` QPS |
| Total Runtime             | `3.196 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,006 / 0 / 994 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,006 / 0 / 994 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,006 / 0 / 994 / 0 |
| no_nearby    | 100.00%   | 100.00%          | 0      | 0             | 9,000 / 0 / 1,000 / 0 |
| osm_only     | 100.00%   | 100.00%          | 0      | 0             | 9,000 / 0 / 1,000 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 6               |
| Total vs OSM-only | 6               |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `af.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape      | Count |
| ---------- | ----: |
| `[4,5]`    | 8,965 |
| `[]`       | 1,000 |
| `[4,5,7]` | 32    |
| `[4]`      | 2     |
| `[4,5,10]` | 1    |

Empty shapes correspond to:

* `empty_shape`: 994
* `offshore`: 6

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 9,000 |
| admin_tree_id | 7     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,993 |
| none                   | 1,000 |
| admin_tree_id\|polygon | 7     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,000 |
| empty_shape      | 994   |
| offshore         | 6     |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `34`
* Total level-4 hits across all samples: `9,000`
* Total level-4 hits across inside samples: `9,000`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Herat Province | 760 | 7.60% | 760 | 8.44% |
| Helmand Province | 755 | 7.55% | 755 | 8.39% |
| Kandahar Province | 734 | 7.34% | 734 | 8.16% |
| Farah Province | 692 | 6.92% | 692 | 7.69% |
| Badakhshan Province | 588 | 5.88% | 588 | 6.53% |
| Ghor Province | 556 | 5.56% | 556 | 6.18% |
| Nimruz Province | 553 | 5.53% | 553 | 6.14% |
| Faryab Province | 320 | 3.20% | 320 | 3.56% |
| Badghis Province | 311 | 3.11% | 311 | 3.46% |
| Ghazni Province | 310 | 3.10% | 310 | 3.44% |
| Baghlan Province | 291 | 2.91% | 291 | 3.23% |
| Bamyan Province | 263 | 2.63% | 263 | 2.92% |
| Zabul Province | 255 | 2.55% | 255 | 2.83% |
| Paktika Province | 251 | 2.51% | 251 | 2.79% |
| Balkh Province | 243 | 2.43% | 243 | 2.70% |
| Daykundi Province | 237 | 2.37% | 237 | 2.63% |
| Sar-e Pol Province | 228 | 2.28% | 228 | 2.53% |
| Takhar Province | 192 | 1.92% | 192 | 2.13% |
| Samangan Province | 181 | 1.81% | 181 | 2.01% |
| Jowzjan Province | 158 | 1.58% | 158 | 1.76% |
| Maidan Wardak Province | 139 | 1.39% | 139 | 1.54% |
| Nuristan Province | 134 | 1.34% | 134 | 1.49% |
| Uruzgan Province | 132 | 1.32% | 132 | 1.47% |
| Kunduz Province | 122 | 1.22% | 122 | 1.36% |
| Nangarhar Province | 97 | 0.97% | 97 | 1.08% |
| Parwan Province | 73 | 0.73% | 73 | 0.81% |
| Kunar Province | 66 | 0.66% | 66 | 0.73% |
| Khost Province | 65 | 0.65% | 65 | 0.72% |
| Paktia Province | 64 | 0.64% | 64 | 0.71% |
| Laghman Province | 58 | 0.58% | 58 | 0.64% |

Hit distribution reflects uniform sampling over the scoped administrative coverage.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/af_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Afghanistan dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,5]`, reflecting strong province and district coverage.
3. Levels 7 and 10 provide valid lower-level detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `087d4b77de58a1ab474012a2b2a6747879c8b499`
- Cadis version:
  `0.8.63`
- Boundary builder: `scripts/build_af_boundaries.py`
- Build boundary: `tmp/af_country.json`
- Evaluation boundary: `tmp/af_country.json`
- Staged dataset: `AF/af.admin/v1.0.0`
- Source OSM SHA256:
  `1a474b5fc4de466cd3baa57a1ebddecd823bcfba916e7f7767534ba8f6287fda`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `af.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Afghanistan levels 4, 5, 7, and 10

The dataset is suitable for human quality review.
