# CZ_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `cz.admin`
Version: `v1.0.0`
Country: `CZ`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `cz.admin v1.0.0` dataset under Cadis Runtime.

The Czech Republic dataset scope is the full ISO2 country boundary, scoped with the Natural Earth CZ admin-0 polygon.

Generated boundary:

- bbox: `[12.076140991000045, 48.557915752, 18.837433716000106, 51.04001230900009]`

No sovereign territory subset or mainland-only exclusion is applied.

---

# 2. Dataset Identity

| Field                   | Value              |
| ----------------------- | ------------------ |
| Dataset ID              | `cz.admin`         |
| Dataset Version         | `v1.0.0`           |
| Country                 | `CZ`               |
| Country Name            | `Czech Republic`   |
| Policy Version          | `1.0`              |
| Hierarchy Required      | `True`             |
| Repair Required         | `False`            |
| Runtime Policy Detected | `True`             |
| Name Schema             | `multilingual_v1`  |

Runtime policy levels:

- level 4: regions / capital city
- level 6: SO ORP
- level 7: SO POU
- level 8: municipalities

Multilingual aliases are enabled for:

- `cs`
- `de`

---

# 3. Source Data Observations

Initial OSM hierarchy probing found the following administrative relation counts inside the CZ Natural Earth boundary:

| Admin Level | Count  |
| ----------- | -----: |
| 2           | 1      |
| 3           | 8      |
| 4           | 14     |
| 5           | 86     |
| 6           | 227    |
| 7           | 392    |
| 8           | 6,237  |
| 9           | 141    |
| 10          | 12,994 |

The runtime dataset intentionally exposes levels `4, 6, 7, 8`. Level 5 districts and lower cadastral / municipal subdivision levels are not exposed in this v1 dataset.

Three Polish border relations were observed during build probing and are explicitly excluded by the engine:

| Feature ID     | Name       | Reason          |
| -------------- | ---------- | --------------- |
| `cz_r6076233`  | Wolanow    | Poland relation |
| `cz_r6418836`  | Wrzosowka  | Poland relation |
| `cz_r6419556`  | Bielice    | Poland relation |

---

# 4. Test Methodology

## 4.1 Sampling Strategy

| Field                  | Value      |
| ---------------------- | ---------- |
| Samples                | `100,000`  |
| Sampling mode          | inside/outside stress testing |
| Expected inside points | `90,000`   |
| Expected outside points| `10,000`   |

The same generated CZ boundary was used for both dataset build scoping and evaluation sampling.

---

# 5. Performance Metrics

| Metric                    | Value         |
| ------------------------- | ------------- |
| Throughput                | `8305.470` QPS |
| Total Runtime             | `12.040 sec`  |
| Overall Pass Rate         | `100.00%`     |
| Inside Coverage Pass Rate | `100.00%`     |
| Policy Pass Rate          | `100.00%`     |

No failed samples were reported.

---

# 6. Scenario Comparison

| Scenario       | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| -------------- | --------: | ---------------: | -----: | ------------: |
| `full_policy`  | 100.00%   | 100.00%          | 0      | 0             |
| `no_hierarchy` | 100.00%   | 100.00%          | 0      | 0             |
| `no_repair`    | 100.00%   | 100.00%          | 0      | 0             |
| `no_nearby`    | 99.17%    | 99.08%           | 826    | 826           |
| `osm_only`     | 99.17%    | 99.08%           | 826    | 826           |

---

# 7. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------: |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 1,109           |
| Total vs OSM-only | 1,109           |

Interpretation:

- Geometry and polygon parent coverage are strong for the exposed model.
- Hierarchy supplementation was available but did not need to rescue sampled points.
- No repair layer was required.
- Nearby fallback resolved bounded edge/coastal-adjacent sampling misses near the country boundary.

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape       | Count  |
| ----------- | -----: |
| `[4,6,7,8]` | 88,060 |
| `[]`        | 9,979  |
| `[4,6,7]`   | 1,075  |
| `[4,6]`     | 604    |
| `[4,7,8]`   | 181    |
| `[4]`       | 84     |
| `[6,7,8]`   | 10     |
| `[4,7]`     | 4      |
| `[4,6,8]`   | 3      |

Empty shapes correspond to:

- `empty_shape`: 9,713
- `offshore`: 266

## 8.2 Node Source Distribution

| Source          | Count  |
| --------------- | -----: |
| polygon         | 89,178 |
| nearby          | 843    |
| admin_tree_id   | 215    |

## 8.3 Source Mix Distribution

| Mix                     | Count  |
| ----------------------- | -----: |
| polygon                 | 88,963 |
| none                    | 9,979  |
| nearby                  | 843    |
| admin_tree_id\|polygon  | 215    |

---

# 9. Policy Reason Distribution

| Reason           | Count  |
| ---------------- | -----: |
| shape_status_map | 90,021 |
| empty_shape      | 9,713  |
| offshore         | 266    |

---

# 10. Level-4 Coverage

- Unique level-4 units hit: `14`
- Total level-4 hits, all points: `90,011`
- Total level-4 hits, inside points: `89,967`

| Level-4 Unit              | Hits   | Hit Rate | Inside Hits | Inside Hit Rate |
| ------------------------- | -----: | -------: | ----------: | --------------: |
| `Stredocesky kraj`        | 12,516 | 12.52%   | 12,516      | 13.91%          |
| `Jihocesky kraj`          | 11,496 | 11.50%   | 11,490      | 12.77%          |
| `Plzensky kraj`           | 8,796  | 8.80%    | 8,796       | 9.77%           |
| `Jihomoravsky kraj`       | 8,329  | 8.33%    | 8,323       | 9.25%           |
| `Kraj Vysocina`           | 7,784  | 7.78%    | 7,784       | 8.65%           |
| `Ustecky kraj`            | 6,167  | 6.17%    | 6,159       | 6.84%           |
| `Olomoucky kraj`          | 6,098  | 6.10%    | 6,098       | 6.78%           |
| `Moravskoslezsky kraj`    | 5,890  | 5.89%    | 5,883       | 6.54%           |
| `Kralovehradecky kraj`    | 5,373  | 5.37%    | 5,369       | 5.97%           |
| `Pardubicky kraj`         | 5,172  | 5.17%    | 5,171       | 5.75%           |
| `Zlinsky kraj`            | 4,390  | 4.39%    | 4,386       | 4.87%           |
| `Karlovarsky kraj`        | 3,807  | 3.81%    | 3,801       | 4.22%           |
| `Liberecky kraj`          | 3,645  | 3.65%    | 3,643       | 4.05%           |
| `Praha`                   | 548    | 0.55%    | 548         | 0.61%           |

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

- no failed samples were observed
- no boundary-leak failure was observed
- expected outside samples were classified as empty or offshore where appropriate
- hierarchy and nearby layers did not produce cross-border escalation in this run

This confirms strict boundary containment within the CZ dataset.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `06e7802d73118a2a43e47dd92b6d5247e01e445c`
- Cadis version prepared for release:
  `0.8.8`
- OSM PBF:
  `czech-republic-260426.osm.pbf`
- OSM source:
  `geofabrik:europe/Czech Republic`

The staged dataset was generated from a clean `cadis_dataset_engine` working tree.

---

# 13. Conclusion

The `cz.admin v1.0.0` dataset demonstrates:

- full inside-boundary coverage in the 100,000-sample evaluation run
- strict boundary isolation
- strong polygon coverage across the selected administrative model
- no repair-layer dependency
- no hierarchy rescue dependency in the sampled run
- bounded nearby fallback behavior
