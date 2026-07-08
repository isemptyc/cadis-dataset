# TW_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `tw.admin`
Version: `v1.0.6`
Country: `TW`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates the rebuilt `tw.admin v1.0.6` dataset under Cadis Runtime.

This rebuild uses `cadis_dataset_engine` commit `7746762f`, which adds a curated Qijin District (`旗津區`) level-7 repair polygon to the Taiwan engine. The rebuild verifies that Qijin is present in the staged runtime geometry layer and resolves correctly through Cadis Runtime.

OSM data is not modified by Cadis. The dataset encodes a deterministic runtime interpretation of available administrative geometry and hierarchy, with a curated polygon added for a known missing district geometry.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `tw.admin` |
| Dataset Version | `v1.0.6` |
| Country | `TW` |
| Country Name | `Taiwan` |
| Policy Version | `1.0` |
| Cadis Version | `0.10.6` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Minimum Cadis Version | `0.10.6` |
| Engine Commit | `7746762fa3aa2877a70436a0e40d0623689c7938` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

Scope: `full Taiwan Geofabrik extract administrative coverage`

The source OSM extract is:

* `geofabrik:asia/taiwan`

The source snapshot recorded in the release manifest is:

| Field | Value |
| --- | --- |
| OSM file | `taiwan-latest.osm.pbf` |
| OSM replication timestamp | `2025-10-19T20:21:00Z` |
| OSM SHA-256 | `6b899702570a6554c5e2bcdd30bd569c5685943acea10c054eed34843e3c215a` |

Evaluation sampled against the Natural Earth Taiwan boundary with bbox:

`[118.27955162900003, 21.90460846600007, 122.00538170700008, 25.28742096600007]`

---

# 4. Runtime Policy

| Policy Field | Value |
| --- | --- |
| Allowed Levels | `[4, 7, 8]` |
| Allowed Shapes | `[4]`, `[4,7]`, `[4,7,8]`, `[4,8]` |
| Hierarchy Parent Level | `4` |
| Hierarchy Child Levels | `[7, 8]` |
| Repair Layer | Disabled |
| Nearby Policy | Enabled |
| Nearby Max Distance | `2.0 km` |
| Offshore Max Distance | `20.0 km` |
| Name Schema | `multilingual_v1` |

Feature counts in the staged runtime artifacts:

| Level | Built Polygon Count |
| --- | ---: |
| `4` | `21` |
| `7` | `170` |
| `8` | `198` |

---

# 5. Qijin Regression Validation

The rebuilt staged dataset includes the curated Qijin District geometry:

| Field | Value |
| --- | --- |
| Feature ID | `tw_r2106668` |
| OSM ID | `r2106668` |
| Name | `旗津區` |
| English Name | `Qijin District` |
| Parent | `tw_r2127079` / `高雄市` |
| Level | `7` |
| Source | `manual_repair` |

The staged artifact verification found `tw_r2106668` in:

* `geometry_meta.json`
* `taiwan_admin.json`
* `TW_feature_meta_by_index.json`

Runtime lookup for `22.5905, 120.2890` returns:

| Rank | Level | Feature ID | Name | Source |
| --- | ---: | --- | --- | --- |
| `0` | `4` | `r2127079` | `高雄市` | `admin_tree_id` |
| `1` | `7` | `tw_r2106668` | `旗津區` | `polygon` |

The lookup status is `ok`, confirming Qijin is now resolved from the polygon layer.

---

# 6. Test Methodology

| Field | Value |
| --- | --- |
| Total samples | `10,000` |
| Inside ratio | `0.9` |
| Inside samples | `9,000` |
| Outside samples | `1,000` |
| Lookup mode | `Runtime` |
| Evaluation boundary | Natural Earth `TW` boundary |
| Dataset dir | staged `TW/tw.admin/v1.0.6` release directory |

The test intentionally injects approximately 10% out-of-scope points to validate boundary rejection, offshore classification, and cross-border isolation.

---

# 7. Performance Metrics

| Metric | Value |
| --- | ---: |
| Throughput | `4275.990 QPS` |
| Total Runtime | `2.339 sec` |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |

---

# 8. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (ok/partial/failed/unknown) |
| --- | ---: | ---: | ---: | ---: | --- |
| `full_policy` | `100.00%` | `100.00%` | `0` | `0` | `9,037/0/963/0` |
| `no_hierarchy` | `100.00%` | `100.00%` | `0` | `0` | `9,037/0/963/0` |
| `no_repair` | `100.00%` | `100.00%` | `0` | `0` | `9,037/0/963/0` |
| `no_nearby` | `98.90%` | `98.78%` | `110` | `110` | `8,892/0/1,108/0` |
| `osm_only` | `98.90%` | `98.78%` | `110` | `110` | `8,892/0/1,108/0` |

---

# 9. Layer Contribution Analysis

| Layer | Rescued Samples |
| --- | ---: |
| Hierarchy | `0` |
| Repair | `0` |
| Nearby | `145` |
| Total vs OSM-only | `145` |

Interpretation:

* The staged Taiwan runtime achieves full policy and inside-boundary pass rates.
* Hierarchy and repair scenario deltas are zero for this random sample.
* Nearby fallback rescues coastal or boundary-adjacent samples that OSM-only geometry leaves empty.
* The Qijin fix is validated by targeted lookup because the 10,000-point random sample is not guaranteed to hit that small district footprint.

---

# 10. Structural Distribution

## 10.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[4,8]` | `6,200` |
| `[4,7]` | `2,776` |
| `[]` | `1,008` |
| `[4]` | `9` |
| `[4,7,8]` | `7` |

## 10.2 Node Source Distribution

| Source | Count |
| --- | ---: |
| `polygon` | `8,892` |
| `admin_tree_id` | `693` |
| `nearby` | `100` |

## 10.3 Source Mix Distribution

| Source Mix | Count |
| --- | ---: |
| `polygon` | `8,199` |
| `__none__` | `1,008` |
| `admin_tree_id|polygon` | `693` |
| `nearby` | `100` |

## 10.4 Policy Reason Distribution

| Reason | Count |
| --- | ---: |
| `shape_status_map` | `8,992` |
| `empty_shape` | `963` |
| `offshore` | `45` |

---

# 11. Level-4 Coverage

* Unique level-4 units hit: `22`
* Total level-4 hits, all samples: `8,992`
* Total level-4 hits, inside samples: `8,988`

Hit distribution reflects uniform land-area sampling.

## 11.1 Level-4 Hit Rates

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| --- | ---: | ---: | ---: | ---: |
| `南投縣` | `1,060` | `10.60%` | `1,060` | `11.78%` |
| `花蓮縣` | `1,050` | `10.50%` | `1,050` | `11.67%` |
| `臺東縣` | `890` | `8.90%` | `890` | `9.89%` |
| `高雄市` | `693` | `6.93%` | `693` | `7.70%` |
| `屏東縣` | `665` | `6.65%` | `664` | `7.38%` |
| `臺中市` | `593` | `5.93%` | `593` | `6.59%` |
| `臺南市` | `540` | `5.40%` | `540` | `6.00%` |
| `嘉義縣` | `522` | `5.22%` | `522` | `5.80%` |
| `宜蘭縣` | `511` | `5.11%` | `511` | `5.68%` |
| `新北市` | `490` | `4.90%` | `490` | `5.44%` |
| `苗栗縣` | `472` | `4.72%` | `472` | `5.24%` |
| `新竹縣` | `365` | `3.65%` | `365` | `4.06%` |
| `雲林縣` | `334` | `3.34%` | `334` | `3.71%` |
| `桃園市` | `310` | `3.10%` | `310` | `3.44%` |
| `彰化縣` | `289` | `2.89%` | `289` | `3.21%` |
| `臺北市` | `75` | `0.75%` | `75` | `0.83%` |
| `新竹市` | `36` | `0.36%` | `36` | `0.40%` |
| `金門縣` | `32` | `0.32%` | `30` | `0.33%` |
| `基隆市` | `29` | `0.29%` | `29` | `0.32%` |
| `澎湖縣` | `25` | `0.25%` | `25` | `0.28%` |
| `嘉義市` | `10` | `0.10%` | `10` | `0.11%` |
| `連江縣` | `1` | `0.01%` | `n/a` | `n/a` |

---

# 12. Boundary Isolation Validation

Under stress testing with 10% forced out-of-scope samples:

* No inside-boundary coverage failure was observed.
* No policy failure was observed.
* Empty-shape and offshore outcomes were limited to expected out-of-scope or near-coastal samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation in this run.

This confirms strict boundary containment within the TW dataset for the sampled run.

---

# 13. Reproducibility

The staged package was rebuilt with:

| Artifact | Value |
| --- | --- |
| Engine commit | `7746762fa3aa2877a70436a0e40d0623689c7938` |
| Engine logic version | `v2.0` |
| Build image digest | `ghcr.io/isemptyc/cadis-dataset-engine@sha256:e5b134214a51368391576e5bbe6b06b403e3acf19d9d91124063caba72f1ae76` |
| Cadis runtime version | `0.10.6` |
| Generated at | `2026-07-08T15:17:57Z` |

Release-file checksums:

| File | Size | SHA-256 |
| --- | ---: | --- |
| `geometry.ffsf` | `1,840,300` | `c680fb8cf8201e6424274a31077000b93c36e5bca72a6361115c4174f0602f31` |
| `geometry_meta.json` | `109,207` | `0cefb4bb62e19bb4772c1cbb1745181f5c516147fcb62b676331739825965c05` |
| `hierarchy.json` | `52,266` | `14c7e833a7f25f2287787ea645bc68b823932ee0d49f2d4f9c1f8363aebb6b6f` |
| `runtime_policy.json` | `949` | `fd0cb81b59e7dc0b25540285ab30097b29c144d5794ecb75c9ca82522a8f6416` |

Evaluation artifacts were generated by `cadis_dataset_release_sop.py --evaluate` for `TW/tw.admin/v1.0.6`.

---

# 14. Release Conclusion

`tw.admin v1.0.6` is valid for release. The rebuilt dataset passes 10,000-sample runtime evaluation with 100% overall, policy, and inside coverage pass rates. The Qijin District curated geometry is present in the staged runtime geometry metadata, and a targeted runtime lookup confirms `高雄市 > 旗津區` resolves from the polygon layer.
