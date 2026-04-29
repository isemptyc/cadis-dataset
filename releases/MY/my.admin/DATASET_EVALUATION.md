# MY_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `my.admin`
Version: `v1.0.0`
Country: `MY`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates the `my.admin v1.0.0` dataset under Cadis Runtime.

The evaluation validates:

* Malaysia country-scope isolation from the combined Malaysia/Singapore/Brunei OSM extract
* Runtime policy behavior for the Malaysia administrative model
* Boundary and offshore behavior under mixed inside/outside sampling
* Deterministic supplementation for Malaysian state geometry missing from OSM geometry assembly

OSM data is not modified by Cadis. The dataset encodes a deterministic runtime interpretation of the available administrative geometry and hierarchy.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `my.admin` |
| Dataset Version | `v1.0.0` |
| Country | `MY` |
| Country Name | `Malaysia` |
| Policy Version | `1.0` |
| Cadis Version | `0.8.10` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Minimum Cadis Version | `0.8.10` |

---

# 3. Dataset Scope

Scope: `full ISO2 country`

The source OSM extract is the combined Geofabrik regional extract:

* `geofabrik:asia/malaysia-singapore-brunei`

Because the PBF contains multiple countries, the build and evaluation both use a tracked Natural Earth boundary builder:

* Country boundary source: `ne_10m_admin_0_countries.dbf`
* Selection rule: ISO2 `MY`, country name `Malaysia`
* Boundary bbox: `[99.64522806000008, 0.8513703410001057, 119.27808678500003, 7.35578034100007]`
* Country polygon count: `17`

The same builder embeds Natural Earth admin-1 supplements from `ne_10m_admin_1_states_provinces.shp` using `adm0_a3 == 'MYS'`. The engine uses these supplements only for missing level-4 Malaysian states. This is required because the OSM geometry build did not emit all Malaysia state polygons from the combined regional extract; without the supplement, Sabah inside-boundary samples returned empty hierarchy results.

---

# 4. Runtime Policy

| Policy Field | Value |
| --- | --- |
| Allowed Levels | `[4, 5, 6, 7, 8, 10]` |
| Hierarchy Parent Level | `4` |
| Hierarchy Child Levels | `[5, 6, 7, 8, 10]` |
| Repair Layer | Disabled |
| Nearby Policy | Enabled |
| Nearby Max Distance | `2.0 km` |
| Offshore Max Distance | `20.0 km` |
| Name Schema | `multilingual_v1` |
| Alias Languages | `zh`, `ar`, `en`, `ms`, `ta` |

Level meanings:

| Level | Label |
| --- | --- |
| 4 | State / Federal Territory |
| 5 | Division |
| 6 | District |
| 7 | Local Authority |
| 8 | Subdistrict |
| 10 | Neighborhood / Section |

---

# 5. Test Methodology

| Field | Value |
| --- | --- |
| Total samples | `50,000` |
| Inside ratio | `0.9` |
| Inside samples | `45,000` |
| Outside samples | `5,000` |
| Lookup mode | Runtime |

The test intentionally injects approximately 10% out-of-country points to validate boundary rejection, offshore classification, and cross-border isolation.

---

# 6. Performance Metrics

| Metric | Value |
| --- | --- |
| Throughput | `10380.510` QPS |
| Total Runtime | `4.817 sec` |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| --- | ---: | ---: | ---: | ---: |
| `full_policy` | `100.00%` | `100.00%` | `0` | `0` |
| `no_hierarchy` | `100.00%` | `100.00%` | `0` | `0` |
| `no_repair` | `100.00%` | `100.00%` | `0` | `0` |
| `no_nearby` | `99.49%` | `99.43%` | `256` | `256` |
| `osm_only` | `99.49%` | `99.43%` | `256` | `256` |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| --- | ---: |
| Hierarchy | `0` |
| Repair | `0` |
| Nearby | `316` |
| Total vs OSM-only | `316` |

Observed layer usage:

| Source | Samples |
| --- | ---: |
| Polygon | `44,759` |
| Nearby | `224` |
| Admin tree ID | `26` |
| Offshore | `92` |

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[4,6]` | `22,713` |
| `[4,5,6]` | `19,944` |
| `[]` | `5,017` |
| `[4,6,7]` | `1,054` |
| `[4]` | `500` |
| `[4,5,6,7]` | `412` |
| `[4,6,8]` | `135` |
| `[4,6,7,10]` | `61` |
| `[4,5,6,8]` | `47` |
| `[6]` | `37` |
| `[4,6,7,8]` | `29` |
| `[4,5]` | `21` |
| `[5,6]` | `7` |
| `[4,7]` | `4` |
| `[4,8]` | `4` |
| `[10]` | `3` |
| `[5,6,8]` | `3` |
| `[8]` | `2` |
| `[4,5,6,8,10]` | `2` |
| `[6,7,8]` | `2` |
| `[4,6,10]` | `1` |
| `[4,6,7,8,10]` | `1` |
| `[6,7]` | `1` |

Empty shapes correspond to:

| Reason | Count |
| --- | ---: |
| `empty_shape` | `4,925` |
| `offshore` | `92` |

Outside-labeled samples are intentional and do not indicate inside coverage gaps.

---

# 10. Level-4 Coverage

| Metric | Value |
| --- | ---: |
| Unique level-4 units hit | `16` |
| Total level-4 hits, all samples | `44,928` |
| Total level-4 hits, inside samples | `44,909` |

| Level-4 Unit | Hits | Hit Rate, All Samples | Hits, Inside | Hit Rate, Inside |
| --- | ---: | ---: | ---: | ---: |
| `Sarawak` | `12,645` | `25.29%` | `12,636` | `28.08%` |
| `Sabah` | `7,813` | `15.63%` | `7,811` | `17.36%` |
| `Pahang` | `6,703` | `13.41%` | `6,703` | `14.90%` |
| `Perak` | `3,752` | `7.50%` | `3,751` | `8.34%` |
| `Johor` | `3,595` | `7.19%` | `3,590` | `7.98%` |
| `Kelantan` | `2,827` | `5.65%` | `2,827` | `6.28%` |
| `Terengganu` | `2,453` | `4.91%` | `2,453` | `5.45%` |
| `Kedah` | `1,726` | `3.45%` | `1,726` | `3.84%` |
| `Selangor` | `1,400` | `2.80%` | `1,400` | `3.11%` |
| `Negeri Sembilan` | `1,296` | `2.59%` | `1,295` | `2.88%` |
| `Melaka` | `296` | `0.59%` | `296` | `0.66%` |
| `Pulau Pinang` | `187` | `0.37%` | `187` | `0.42%` |
| `Perlis` | `158` | `0.32%` | `157` | `0.35%` |
| `Kuala Lumpur` | `44` | `0.09%` | `44` | `0.10%` |
| `Labuan` | `23` | `0.05%` | `23` | `0.05%` |
| `Putrajaya` | `10` | `0.02%` | `10` | `0.02%` |

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Full policy mode produced zero failed samples.
* Nearby fallback rescued coastal adjacency samples without cross-border escalation.
* The admin-1 supplement restored full sampled inside coverage for Sabah and the other missing level-4 units.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine commit: `13f52b24dfb40f2ca89dfe7a95e6182de8ece543`
* Cadis version: `0.8.10`

The staged package was generated from a clean `cadis_dataset_engine` working tree.

---

# 13. Conclusion

The `my.admin v1.0.0` dataset demonstrates:

* Full sampled inside-boundary coverage under policy mode
* Strict country-scope isolation from a combined regional OSM extract
* Complete level-4 Malaysia state/federal-territory coverage after deterministic Natural Earth admin-1 supplementation
* No repair-layer requirement
* Minimal, bounded nearby fallback use

