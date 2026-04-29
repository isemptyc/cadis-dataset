# LU_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `lu.admin`
Version: `v1.0.0`
Country: `LU`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `lu.admin v1.0.0` dataset under Cadis Runtime.

The Luxembourg dataset scope is the full ISO2 country boundary. 

Generated boundary:

- bbox: `[5.714927205000038, 49.44132436200003, 6.502579387000139, 50.17497467100007]`

No sovereign territory subset or mainland-only exclusion is applied.

---

# 2. Dataset Identity

| Field                   | Value        |
| ----------------------- | ------------ |
| Dataset ID              | `lu.admin`   |
| Dataset Version         | `v1.0.0`     |
| Country                 | `LU`         |
| Country Name            | `Luxembourg` |
| Policy Version          | `1.0`        |
| Hierarchy Required      | `True`       |
| Repair Required         | `False`      |
| Runtime Policy Detected | `True`       |
| Name Schema             | `multilingual_v1` |

---

# 3. Source Inputs

| Field | Value |
| ----- | ----- |
| OSM Source | `geofabrik:europe/Luxembourg` |
| OSM File | `luxembourg-260426.osm.pbf` |
| Replication Timestamp UTC | `2026-04-26T20:21:04Z` |
| OSM SHA256 | `b18debbe55d0b3dca0b9ee5eaded920efe13c7dd04759ab83953bda51ce868c3` |
| Engine Commit | `32187d0dfbbb9cb27f85c9304a759558ee187444` |
| Cadis Version Evaluated | `0.8.7` |

---

# 4. Structural Model

Luxembourg is modeled with these administrative levels:

| Level | Meaning |
| ----- | ------- |
| `6` | canton |
| `8` | municipality |
| `9` | locality / quarter subdivision |

Dataset-scoped hierarchy artifact generation produced:

| Metric | Value |
| ------ | ----- |
| Nodes | `310` |
| Edges | `295` |
| Unresolved edges | `0` |
| Level 6 count | `13` |
| Level 8 count | `99` |
| Level 9 count | `198` |

Allowed runtime shapes:

- `[6]`
- `[6,8]`
- `[6,8,9]`
- `[6,9]`
- `[8]`
- `[8,9]`
- `[9]`

Complete shapes are `[6,8]` and `[6,8,9]`. Sparse shapes are accepted as partial.

---

# 5. Test Methodology

Runtime mass validation used:

- lookup mode: `runtime`
- samples: `10,000`
- inside ratio: `0.9`
- expected inside points: `9,000`
- expected outside points: `1,000`
- max attempts: `1,000,000`
- batch size: `1`

Batch size `1` is intentional for this evaluation because the current local runtime batch path did not emit a complete per-point report for Luxembourg during the first run. Per-point runtime evaluation produced the valid 10,000-sample report used here.

---

# 6. Performance Metrics

| Metric | Value |
| ------ | ----- |
| Throughput | `7529.130` QPS |
| Total Runtime | `1.328 sec` |
| Overall Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Failed Samples | `0` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| -------- | --------- | ---------------- | ------ | ------------- |
| full_policy | `100.00%` | `100.00%` | `0` | `0` |
| no_hierarchy | `100.00%` | `100.00%` | `0` | `0` |
| no_repair | `100.00%` | `100.00%` | `0` | `0` |
| no_nearby | `94.69%` | `94.10%` | `531` | `531` |
| osm_only | `94.69%` | `94.10%` | `531` | `531` |

Layer contribution:

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | `0` |
| Repair | `0` |
| Nearby | `680` |
| Total vs OSM-only | `680` |

Nearby fallback is material for Luxembourg because sampled Natural Earth boundary points can fall outside OSM administrative polygons near the border. The LU runtime policy uses a bounded `5.0 km` nearby fallback.

Cadis also reports `offshore` for some expected-outside samples. For landlocked Luxembourg this label should be read as a generic near-country fallback classification, not as a maritime offshore location. These points are outside the LU country boundary but within the configured fallback band. In an installation that also has neighboring `DE`, `BE`, and `FR` datasets, many of these same points are expected to resolve through the neighboring country dataset rather than remain LU fallback classifications.

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape | Count |
| ----- | ----- |
| `[6,8]` | `4,966` |
| `[6,8,9]` | `3,971` |
| `[]` | `965` |
| `[6]` | `74` |
| `[8]` | `16` |
| `[6,9]` | `7` |
| `[8,9]` | `1` |

Empty shapes correspond to expected outside samples and near-country fallback samples:

- `empty_shape`: `847`
- `offshore`: `118` near-LU out-of-country samples; for landlocked LU this is cross-border adjacency fallback, not literal offshore geography

## 8.2 Node Source Distribution

| Source | Count |
| ------ | ----- |
| polygon | `8,473` |
| nearby | `562` |
| admin_tree_id | `29` |

## 8.3 Source Mix

| Mix | Count |
| --- | ----- |
| polygon | `8,444` |
| none | `965` |
| nearby | `562` |
| admin_tree_id\|polygon | `29` |

---

# 9. Boundary Isolation Validation

The test used 10% forced out-of-bound samples against the same generated LU boundary used for evaluation. No failed samples were observed.

Full policy results:

- `9,055` ok outcomes
- `98` partial outcomes
- `847` failed outcomes for expected outside samples
- `118` `offshore` outcomes representing near-LU out-of-country fallback classifications
- `0` unknown outcomes

No evidence was observed that hierarchy or nearby layers created incorrect in-LU escalation in this run. The `offshore` outcomes are acceptable standalone-LU fallback behavior and are expected to be superseded by neighboring dataset resolution when `DE`, `BE`, and `FR` datasets are installed.

---

# 10. Conclusion

The `lu.admin v1.0.0` dataset is acceptable for publish.

The dataset demonstrates:

- complete 10,000-sample runtime validation
- 100% inside-boundary coverage under full policy
- deterministic hierarchy artifacts with no unresolved dataset-scoped edges
- no required repair layer
- bounded nearby fallback that closes Natural Earth/OSM border drift gaps
- expected near-boundary fallback behavior for landlocked LU, where `offshore` denotes near-country out-of-bound points rather than maritime geography
- strict boundary containment in the sampled run
