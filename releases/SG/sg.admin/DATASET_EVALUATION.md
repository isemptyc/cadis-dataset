# SG_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `sg.admin`
Version: `v1.0.0`
Country: `SG`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates the `sg.admin v1.0.0` dataset under Cadis Runtime.

The evaluation validates:

* Singapore country-scope isolation from the combined Malaysia/Singapore/Brunei OSM extract
* Runtime policy behavior for the Singapore administrative model
* Boundary and offshore behavior under mixed inside/outside sampling
* Reproducibility metadata for the staged release package

OSM data is not modified by Cadis. The dataset encodes a deterministic runtime interpretation of the available administrative geometry and hierarchy.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `sg.admin` |
| Dataset Version | `v1.0.0` |
| Country | `SG` |
| Country Name | `Singapore` |
| Policy Version | `1.0` |
| Cadis Version | `0.8.9` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Minimum Cadis Version | `0.8.9` |

---

# 3. Dataset Scope

Scope: `full ISO2 country`

The source OSM extract is the combined Geofabrik regional extract:

* `geofabrik:asia/malaysia-singapore-brunei`

Because the PBF contains multiple countries, the build and evaluation both use a tracked Natural Earth boundary builder:

* Boundary source: `ne_10m_admin_0_countries.dbf`
* Selection rule: ISO2 `SG`, country name `Singapore`
* Boundary bbox: `[103.64039147200003, 1.26430898600006, 104.00342858200003, 1.4486351580000587]`
* Polygon count: `1`

---

# 4. Runtime Policy

| Policy Field | Value |
| --- | --- |
| Allowed Levels | `[6, 11]` |
| Allowed Shapes | `[6]`, `[6, 11]`, `[11]` |
| `ok` Shape | `[6, 11]` |
| `partial` Shapes | `[6]`, `[11]` |
| Hierarchy Parent Level | `6` |
| Hierarchy Child Levels | `[11]` |
| Repair Layer | Disabled |
| Nearby Policy | Enabled |
| Nearby Max Distance | `2.0 km` |
| Offshore Max Distance | `20.0 km` |

The administrative model contains three level-6 regions and seven level-11 neighborhood-level entities in the staged runtime artifacts.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Evaluation workers: `1`
* Max sampling attempts: `1,000,000`

Single-worker evaluation was used because the current concurrent evaluation path only accumulated one completed future in this run. The single-worker run exercises the same runtime lookup logic and produced a complete 10,000-sample report.

---

# 6. Performance Metrics

| Metric | Value |
| --- | --- |
| Throughput | `26249.760` QPS |
| Total Runtime | `0.381 sec` |
| Overall Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Passed Samples | `10,000 / 10,000` |

No policy, coverage, or inside-boundary failures were observed.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| --- | ---: | ---: | ---: | ---: |
| `full_policy` | `100.00%` | `100.00%` | `0` | `0` |
| `no_hierarchy` | `100.00%` | `100.00%` | `0` | `0` |
| `no_repair` | `100.00%` | `100.00%` | `0` | `0` |
| `no_nearby` | `73.55%` | `70.61%` | `2,645` | `2,645` |
| `osm_only` | `73.55%` | `70.61%` | `2,645` | `2,645` |

Nearby fallback is material for Singapore because the Natural Earth country boundary and OSM administrative geometry differ around compact coastal and offshore areas.

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| --- | ---: |
| Hierarchy | `0` |
| Repair | `0` |
| Nearby | `3,151` |
| Total vs OSM-only | `3,151` |

The hierarchy layer is present and deterministic, but this sampled run did not require hierarchy rescue. The repair layer is intentionally disabled.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[6]` | `3,834` |
| `[6,11]` | `3,288` |
| `[]` | `2,878` |

Empty shapes correspond to:

* `offshore`: `2,444`
* `empty_shape`: `434`

## 9.2 Node Source Distribution

| Source | Count |
| --- | ---: |
| `polygon` | `6,415` |
| `admin_tree_id` | `809` |
| `nearby` | `707` |

## 9.3 Source Mix Distribution

| Source Mix | Count |
| --- | ---: |
| `polygon` | `5,606` |
| `none` | `2,878` |
| `admin_tree_id\|polygon` | `809` |
| `nearby` | `707` |

---

# 10. Administrative Coverage

This supplemental coverage pass used the same staged dataset, Singapore boundary, random seed, sample count, and inside/outside ratio as the release evaluation:

* Total samples: `10,000`
* Expected inside samples: `9,000`
* Expected outside samples: `1,000`

The purpose of this section is to show which administrative units were hit by sampled runtime lookups. Hit rates are land-area-sampling rates, not population-weighted rates.

## 10.1 Level-6 Coverage

All three level-6 regions were hit.

| Level-6 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| --- | ---: | ---: | ---: | ---: |
| `Northwest` | `2,509` | `25.09%` | `2,504` | `27.82%` |
| `Central` | `2,421` | `24.21%` | `2,400` | `26.67%` |
| `Northeast` | `2,180` | `21.80%` | `2,122` | `23.58%` |

## 10.2 Level-11 Coverage

All seven level-11 entities were hit.

| Level-11 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| --- | ---: | ---: | ---: | ---: |
| `Singapur East` | `2,446` | `24.46%` | `2,440` | `27.11%` |
| `Southern Waterfront` | `354` | `3.54%` | `351` | `3.90%` |
| `Singapore Central` | `334` | `3.34%` | `334` | `3.71%` |
| `Orchard Road` | `83` | `0.83%` | `83` | `0.92%` |
| `Chinatown` | `31` | `0.31%` | `31` | `0.34%` |
| `Bugis` | `31` | `0.31%` | `31` | `0.34%` |
| `Little India` | `8` | `0.08%` | `8` | `0.09%` |

The level-11 distribution is highly uneven because the sampled points are uniform over geographic area and Singapore's available level-11 OSM administrative polygons are sparse and concentrated.

---

# 11. Boundary Isolation Validation

The Singapore dataset is built from a combined regional OSM extract but scoped with the Natural Earth `SG` boundary during both build and evaluation.

The 10,000-sample stress run observed:

* `100.00%` overall pass rate
* `100.00%` policy pass rate
* `100.00%` inside coverage pass rate
* no cross-country policy failures
* no repair-layer dependency

This supports publishing the dataset as a full ISO2 Singapore dataset scoped from the regional Geofabrik extract.

---

# 12. Reproducibility

Dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine commit: `26b74c459b477de578afb2e075cf9499db71fbde`
* Cadis version: `0.8.9`
* Build image digest: `ghcr.io/isemptyc/cadis-dataset-engine@sha256:e5b134214a51368391576e5bbe6b06b403e3acf19d9d91124063caba72f1ae76`
* OSM source: `geofabrik:asia/malaysia-singapore-brunei`
* OSM replication timestamp: `2025-10-23T20:20:52Z`
* OSM file SHA256: `6e47484706611ad358b1ce6cba94f814e7f4fbde9998ceb857217c317a672b54`

---

# 13. Conclusion

The `sg.admin v1.0.0` staged dataset passes the current release evaluation gates:

* Singapore is isolated from the combined regional source extract by a tracked boundary builder.
* Runtime policy is detected and matches the intended `[6, 11]` model.
* Full-policy evaluation passes `10,000 / 10,000` samples.
* Repair is not required.
* Nearby fallback is important and bounded for Singapore coastal/offshore behavior.
