# VN_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `vn.admin`
Version: `v1.0.0`
Country: `VN`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `vn.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the selected Vietnam dataset scope
* Quantifies structural completeness under randomized lookup stress testing
* Documents deterministic policy effects
* Validates boundary isolation against the declared evaluation boundary
* Provides reproducible integrity metrics for release review

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value     |
| ----------------------- | --------- |
| Dataset ID              | `vn.admin` |
| Dataset Version         | `v1.0.0`  |
| Country                 | `VN`      |
| Country Name            | `Vietnam` |
| Policy Version          | `1.0`     |
| Hierarchy Required      | `True`    |
| Repair Required         | `False`   |
| Runtime Policy Detected | `True`    |

## 2.1 Scope

| Field                    | Value |
| ------------------------ | ----- |
| Dataset scope            | `OSM admin-level-4 coverage union` |
| Boundary builder         | `build_vn_boundaries.py` |
| Build boundary JSON      | `vn_country.json` |
| Evaluation boundary JSON | `vn_country.json` |
| Selection rule           | Union all extracted Vietnam `admin_level=4` province/municipality polygons after Natural Earth VN admin-0 filtering |
| Scoped boundary bbox     | `[102.1438643, 8.1810738, 109.6665477, 23.3926032]` |
| Level-4 polygon count    | `28` |

The build and evaluation used the same scoped boundary. Natural Earth components not covered by extracted OSM level-4 province/municipality polygons are outside this `v1.0.0` evaluation scope.

---

# 3. Test Methodology

## 3.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside points: `9,000`
* Outside points: `1,000`
* Lookup mode: `runtime`
* Dataset dir: `VN/vn.admin/v1.0.0`

The test intentionally injects outside-boundary samples to validate rejection behavior, offshore classification, and policy-layer containment.

---

# 4. Performance Metrics

| Metric                    | Value          |
| ------------------------- | -------------- |
| Throughput                | `13140.620` QPS |
| Total Runtime             | `0.761 sec`    |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |

No policy or coverage failures were observed in this run.

---

# 5. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| ------------ | --------- | ---------------- | ------ | ------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             |
| no_nearby    | 100.00%   | 100.00%          | 0      | 0             |
| osm_only     | 100.00%   | 100.00%          | 0      | 0             |

---

# 6. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 10              |
| Total vs OSM-only | 10              |

## Interpretation

* OSM-only success rate was already `100.00%` against the scoped evaluation boundary.
* Nearby fallback had a small positive effect on status classification, rescuing 10 samples from empty-shape classification.
* No hierarchy or repair rescue was required in this evaluation run.
* No explicit repair layer is required for this release candidate.

---

# 7. Structural Distribution

## 7.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| [4,6] | 6,875 |
| [4]   | 2,127 |
| []    | 998   |

Empty shapes correspond to expected outside-boundary samples and offshore classifications:

* `empty_shape`: 990
* `offshore`: 8

## 7.2 Node Source Distribution

| Source  | Count |
| ------- | ----: |
| polygon | 9,000 |
| nearby  | 2     |

### Source Mix

| Mix     | Count |
| ------- | ----: |
| polygon | 9,000 |
| none    | 998   |
| nearby  | 2     |

---

# 8. Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,002 |
| empty_shape      | 990   |
| offshore         | 8     |

---

# 9. Level-4 Coverage

* Unique level-4 units hit: `28`
* Total level-4 hits (all samples): `9,002`
* Total level-4 hits (inside samples): `9,000`

Hit distribution reflects uniform area sampling over the scoped boundary, not population-weighted sampling.

## 9.1 Level-4 Hit Distribution

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ---------------------: | --------------------: | ------------------------: |
| Tỉnh Lâm Đồng | 992 | 9.92% | 991 | 11.01% |
| Tỉnh Cà Mau | 600 | 6.00% | 600 | 6.67% |
| Tỉnh Nghệ An | 555 | 5.55% | 555 | 6.17% |
| Tỉnh Gia Lai | 523 | 5.23% | 523 | 5.81% |
| Tỉnh Đắk Lắk | 477 | 4.77% | 477 | 5.30% |
| Tỉnh Quảng Ngãi | 417 | 4.17% | 417 | 4.63% |
| Tỉnh Lào Cai | 396 | 3.96% | 396 | 4.40% |
| Tỉnh Sơn La | 386 | 3.86% | 386 | 4.29% |
| Tỉnh Tuyên Quang | 368 | 3.68% | 367 | 4.08% |
| Tỉnh Thanh Hóa | 361 | 3.61% | 361 | 4.01% |
| Tỉnh Quảng Ninh | 334 | 3.34% | 334 | 3.71% |
| Thành phố Đà Nẵng | 330 | 3.30% | 330 | 3.67% |
| Tỉnh Điện Biên | 296 | 2.96% | 296 | 3.29% |
| Tỉnh Lai Châu | 273 | 2.73% | 273 | 3.03% |
| Tỉnh Phú Thọ | 266 | 2.66% | 266 | 2.96% |
| Hà Tĩnh | 260 | 2.60% | 260 | 2.89% |
| Thành phố Đồng Nai | 257 | 2.57% | 257 | 2.86% |
| Tỉnh Thái Nguyên | 254 | 2.54% | 254 | 2.82% |
| Thành phố Hải Phòng | 224 | 2.24% | 224 | 2.49% |
| Tỉnh Lạng Sơn | 214 | 2.14% | 214 | 2.38% |
| Tỉnh Cao Bằng | 202 | 2.02% | 202 | 2.24% |
| Thành phố Cần Thơ | 197 | 1.97% | 197 | 2.19% |
| Tỉnh Vĩnh Long | 193 | 1.93% | 193 | 2.14% |
| Tỉnh Tây Ninh | 173 | 1.73% | 173 | 1.92% |
| Tỉnh Đồng Tháp | 144 | 1.44% | 144 | 1.60% |
| Tỉnh Hưng Yên | 121 | 1.21% | 121 | 1.34% |
| Tỉnh Bắc Ninh | 110 | 1.10% | 110 | 1.22% |
| Hà Nội | 79 | 0.79% | 79 | 0.88% |

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-boundary samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes were limited to expected outside-boundary behavior.
* The same scoped boundary was used for build and evaluation.
* No evidence was observed that nearby fallback created cross-border escalation.

---

# 11. Structural Observations

1. The first release contract exposes levels `4` and `6`.
2. Level `8` was excluded because scoped source coverage is sparse.
3. Level `9` was excluded because the raw probe found many neighborhood polygons without stable parent linkage.
4. Geometry integrity is sufficient for full scoped-boundary coverage.
5. Partial `[4]` outcomes are expected where level-6 polygons are absent or not hit.
6. Nearby fallback usage is minimal and bounded.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine commit: `b20f9c00b84e5293eb019bd8e3003b8c882e2d27`
* Cadis version: `0.8.19`
* Source OSM file: `vietnam-260509.osm.pbf`
* Source OSM SHA-256: `ad40bad0cad3183b500d38586483c131e26726dc7b3dc2b796fbf9043b942d4f`
* Source OSM replication timestamp: `2026-05-09T20:21:02Z`
* Boundary builder: `build_vn_boundaries.py`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 13. Conclusion

The `vn.admin v1.0.0` release candidate demonstrates:

* Full coverage under the declared scoped boundary
* Deterministic runtime policy behavior
* No required repair layer
* Minimal nearby fallback usage
* Strict boundary isolation in the sampled run

