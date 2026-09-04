# CN_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `cn.admin`

Version: `v1.0.6`

Country: `CN`

Policy Version: `1.0`

Evaluation Date: `2026-09-04`

---

# 1. Purpose

This report evaluates the overall administrative coverage, structural integrity,
runtime behavior, and boundary handling of `cn.admin v1.0.6` under Cadis Runtime.

CN remains the public dataset identifier for the combined China and Tibet
source components. The dataset exposes administrative anchors at OSM levels
4, 5, and 6. Lower locality and detailed administrative levels remain outside
the runtime contract.

Version v1.0.6 restores Fujian and Tibet, together with the eight prefectures
they depend on, through validated build-scope inclusions that derive a unit from
its declared OSM subareas. With Hainan restored in v1.0.5 and Shanghai in
v1.0.4, all 31 province-level units now carry materialized level-4 geometry for
the first time. Tibet no longer depends on the level-5/6 anchor accommodation.
Existing feature geometry is unchanged. The scope remains available
administrative coverage; it does not claim complete sovereign CN territory
coverage.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `cn.admin` |
| Dataset Version | `v1.0.6` |
| Country | `CN` |
| Country Name | China |
| Policy Version | `1.0` |
| Cadis Version | `0.10.10` |
| Hierarchy Required | True |
| Repair Required | False |
| Runtime Policy Detected | True |
| Name Schema | `multilingual_v1` |
| Administrative Features | 3,213 |
| Geometry/Metadata/Hierarchy/Policy Payload | 7,572,660 bytes (7.22 MiB) |
| Package Size | Approximately 5.7 MiB compressed; exact integrity values are in the release sidecars |

The release consists of `geometry.ffsf`, `geometry_meta.json`, `hierarchy.json`,
and `runtime_policy.json`, together with its versioned release manifest and
package checksum sidecars.

---

# 3. Dataset Scope

| Field | Value |
| --- | --- |
| Scope Label | Administrative coverage from materialized China and Tibet OSM relations |
| Boundary Builder | `scripts/build_cn_boundaries.py` |
| Inclusion Configuration | `scripts/cn_scope_inclusions.json` |
| Boundary Source | Natural Earth admin-0 selection plus OSM administrative geometry |
| OSM Source | `geofabrik:asia/china+asia/tibet` |
| OSM Snapshot Timestamp | `2026-05-11T20:20:52Z` |
| Boundary BBox | `[73.50254522441283, 18.145850315254442, 134.77345103091676, 53.55881051212936]` |
| Build and Evaluation Scope | The same generated boundary is used for both |

Ordinary China level-4 administrative areas are selected when their
representative point falls within the Natural Earth CN boundary, or the existing
`ISO3166-1:alpha2=CN` condition applies. Explicitly configured and validated scope
inclusions are then admitted. Tibet contributes materialized level-5/6
administrative areas. The combined geometry is unioned, repaired if necessary,
and given the existing 0.002-degree inward buffer.

Two inclusion forms exist. Shanghai (`913067`) admits normally assembled geometry
that scope membership would otherwise reject. Hainan (`2128285`), Fujian
(`553303`) and Tibet (`153292`) are marked `derive_from_subareas`: each
contributes the union of the subareas OSM declares for it, to the declared
`subarea_depth`, because its own ring cannot be assembled from the source. Every
form validates relation identity against the configured administrative level and,
where the unit carries one, its ISO code. Missing or mismatched configured
inclusions fail the build.

**Why these units are derived.** Each depends on member ways that lie outside the
clip polygons used to cut the published regional extracts and are therefore
absent from the source:

| Unit | Depth | Missing ring depends on | Declared subareas | Contributing |
| --- | ---: | --- | ---: | ---: |
| Hainan | 1 | South China Sea maritime boundary, Spratly area | 19 | 19 |
| Fujian | 2 | Kinmen/Matsu restricted-water and coastline ways | 94 | 77 |
| Tibet | 2 | Disputed border ways | 82 | 80 |

Fujian and Tibet need depth 2 because their immediate subareas fail for the same
reason they do: all seven coastal Fujian prefectures, and Shigatse. Depth 2
reaches the county level, where 74 of 84 Fujian counties and 17 of 18 Shigatse
counties assemble. Those eight prefectures are separately materialized from their
own subareas so that level 5 remains present in the hierarchy rather than
lookups descending from province directly to district.

**Kinmen and Matsu are not reachable through this mechanism.** They are not
declared subareas of any Fujian prefecture; they appeared only in Fujian's outer
ring ways, which a derivation does not use. No Taiwan-administered territory
enters the CN dataset through these rules, and none is claimed.

**Declared scope exclusion.** Sansha (`2833102`) is named in
`excluded_subarea_relation_ids` and is outside this dataset's scope by
declaration rather than by source absence. It administers the Paracel, Spratly
and Zhongsha groups roughly 800 to 1,200 km offshore.

These build-scope rules do not add a runtime land override. Administrative
polygons can include water; their area must not be interpreted as a land-area
estimate. The runtime's existing scope flags and nearby/offshore policy remain
unchanged.

**Known exclusions and gaps:** ten Taiwan-Strait-facing Fujian counties
(Longhai, Xiang'an, Changle, Lianjiang, Pingtan, Xiuyu, Xiapu, Hui'an, Jinjiang,
Shishi) and one Shigatse county do not assemble, because their own rings use the
same absent ways. Pingtan prefecture has no assembling subarea and is therefore
absent at level 5. Other relations rejected by the normal CN selection rule
remain excluded. Detail levels below the level-6 runtime contract remain
excluded by design.

---

# 4. Administrative Model and Structural Integrity

| Level | Runtime Label | Dataset Count | Role |
| --- | --- | --- | --- |
| 4 | `admin_region` | 31 | Primary regional anchors; all province-level units |
| 5 | `admin_district` | 335 | Intermediate administrative units |
| 6 | `admin_municipality` | 2,847 | District/county-level units |

All seven non-empty combinations of levels 4, 5, and 6 are allowed by runtime
policy and map to `ok`. A non-empty result need not contain every level.
Nearby fallback is enabled within 2 km; the offshore distance limit is 20 km.
Hierarchy support is required, while geometry repair fallback is not required.

The hierarchy contains **3,213 nodes**, **3,077 parent links**, and **136 roots**,
against 157 roots in v1.0.5. Derived units attach their own direct subareas
explicitly, because spatial parent inference only pairs adjacent levels and
cannot attach a county-level unit that sits directly beneath a province.

Canonical names follow the existing engine order: `name:en`, `name`,
`official_name`, `name:zh`, `name:bo`, and `name:ru`. Multilingual aliases are
bounded to `en`, `zh`, `bo`, and `ru`.

| Language Alias | Features Carrying Alias |
| --- | --- |
| `en` | 3,202 |
| `zh` | 3,193 |
| `bo` | 226 |
| `ru` | 1,269 |

Alias counts overlap and describe source availability, not translation
completeness.

---

# 5. Test Methodology

- Runtime-mode evaluation through the standard Cadis mass tester.
- **100,000 samples**: **90,000 inside** and **10,000 outside** the generated
  administrative scope; deterministic sampling seed 42.
- One evaluation worker, 50,000-point batches, maximum 10,000,000 sampling attempts.
- Build and evaluation use the identical scoped boundary.
- Full-policy results are compared with `no_hierarchy`, `no_repair`,
  `no_nearby`, and `osm_only` scenarios.
- Independent fixed probes cover all 31 provincial-capital/municipality
  locations, every Fujian prefecture seat, Tibetan prefecture seats including
  Shigatse, the declared Sansha exclusion, and the known county gaps.
- All 3,129 existing feature records are compared against v1.0.5, and every
  added record is checked in v1.0.6.

Sampling is geographic rather than population-weighted, and the mass pass rate
describes the declared administrative coverage. **It cannot discover territory
omitted from the scope**: the ten unassembled Fujian counties lie outside the
generated boundary, so no inside sample is drawn there and their absence cannot
appear as a mass-test failure. This is the same scope circularity recorded in the
original CN investigation, and it is the reason the fixed probes below test named
locations independently of the sampled corpus. Because the scope grew again with
this release, v1.0.5 and v1.0.6 sample populations are not identical and their
aggregate counts are not a direct longitudinal performance comparison.

---

# 6. Evaluation Results

| Metric | Value |
| --- | --- |
| Overall Pass Rate | 100.00% (100,000/100,000) |
| Inside Coverage Pass Rate | 100.00% (90,000/90,000) |
| Policy Pass Rate | 100.00% |
| Validation Failures | 0 |
| Inside Validation Failures | 0 |
| Throughput | 1230.250 QPS |
| Total Runtime | 81.285 seconds |
| Runtime Lookup Status: ok | 90,029 |
| Runtime Lookup Status: partial | 0 |
| Runtime Lookup Status: failed | 9,971 |
| Empty Administrative Hierarchies | 9,998 |
| Offshore Outcomes | 27 |

Expected outside points may return `failed`; that is distinct from a validation
failure. Offshore outcomes have empty administrative hierarchies and can carry
transport/lookup status `ok`. Such status alone does not imply administrative
coverage. The evaluation used runtime mode, so the tester's `http_200` counter
is not evidence of 100,000 actual network HTTP requests.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Statuses: ok / partial / failed / unknown |
| --- | --- | --- | --- | --- | --- |
| `full_policy` | 100.000% | 100.000% | 0 | 0 | 90,029 / 0 / 9,971 / 0 |
| `no_hierarchy` | 100.000% | 100.000% | 0 | 0 | 90,029 / 0 / 9,971 / 0 |
| `no_repair` | 100.000% | 100.000% | 0 | 0 | 90,029 / 0 / 9,971 / 0 |
| `no_nearby` | 99.998% | 99.998% | 2 | 2 | 89,998 / 0 / 10,002 / 0 |
| `osm_only` | 99.998% | 99.998% | 2 | 2 | 89,998 / 0 / 10,002 / 0 |

Rates above are recomputed from exact counts rather than copied from rounded
summary percentages. Full policy, no hierarchy, and no repair have zero
validation failures. Disabling nearby behavior yields two inside failures, the
same as v1.0.4 and v1.0.5.

---

# 8. Layer Contribution Analysis

| Scenario-Delta Metric | Samples |
| --- | --- |
| Rescued by hierarchy | 0 |
| Rescued by repair | 0 |
| Rescued by nearby | 31 |
| Total versus OSM-only | 31 |

The 31-sample nearby delta measures changes in scenario outcomes, including
offshore handling; it is not the number of inside validation failures prevented.
Only two inside failures occur without nearby behavior. Direct nearby source
usage appears in four full-policy samples, while 27 outcomes are offshore.

The tester reports zero hierarchy-rescue and zero repair-rescue samples. Its
node-source distribution also contains 14 `admin_tree_id` uses. These are
different metrics: source tagging does not establish that hierarchy was needed
to change a failed scenario into a passing one.

---

# 9. Structural Distribution

## 9.1 Administrative Shapes

| Category | Samples | v1.0.5 |
| --- | --- | --- |
| `[4,5,6]` | 87,716 | 77,254 |
| `[]` | 9,998 | 10,000 |
| `[4,6]` | 1,811 | 1,838 |
| `[4]` | 352 | 358 |
| `[4,5]` | 94 | 97 |
| `[6]` | 29 | 1,524 |
| `[5,6]` | 0 | 8,929 |

The shape distribution changes materially: `[5,6]` disappears and `[6]` falls
from 1,524 to 29, while `[4,5,6]` rises by 10,462. Those were predominantly
Tibetan samples that previously had no level-4 parent to report. Complete
three-level results now cover 97.5% of inside samples.

## 9.2 Node Sources

| Category | Samples |
| --- | --- |
| `polygon` | 89,998 |
| `admin_tree_id` | 14 |
| `nearby` | 4 |

## 9.3 Source Mix

| Category | Samples |
| --- | --- |
| `polygon` | 89,984 |
| `__none__` | 9,998 |
| `admin_tree_id / polygon` | 14 |
| `nearby` | 4 |

## 9.4 Policy Reasons

| Category | Samples |
| --- | --- |
| `shape_status_map` | 90,002 |
| `empty_shape` | 9,971 |
| `offshore` | 27 |

These distributions describe sampled runtime outcomes, not the stored feature
count. Empty hierarchies are expected for outside-scope and offshore outcomes.

---

# 10. Level-4 Coverage

All **31** stored level-4 units were hit by the evaluation, covering every
province-level unit for the first time. There were **89,973** total level-4 hits,
of which **89,971** were inside-scope.

| Level-4 Unit | All Hits | Inside Hits | Inside-Sample Hit Rate |
| --- | --- | --- | --- |
| Xinjiang | 16,083 | 16,083 | 17.870% |
| Inner Mongolia | 11,575 | 11,574 | 12.860% |
| Tibet | 10,310 | 10,310 | 11.456% |
| Qinghai | 5,768 | 5,768 | 6.409% |
| Heilongjiang | 5,201 | 5,201 | 5.779% |
| Sichuan | 4,335 | 4,335 | 4.817% |
| Gansu | 3,943 | 3,943 | 4.381% |
| Yunnan | 3,322 | 3,322 | 3.691% |
| Guangxi | 2,118 | 2,118 | 2.353% |
| Jilin | 1,990 | 1,990 | 2.211% |
| Guangdong | 1,984 | 1,984 | 2.204% |
| Shaanxi | 1,937 | 1,937 | 2.152% |
| Hebei | 1,925 | 1,925 | 2.139% |
| Shandong | 1,811 | 1,811 | 2.012% |
| Hunan | 1,764 | 1,764 | 1.960% |
| Liaoning | 1,759 | 1,759 | 1.954% |
| Hubei | 1,627 | 1,627 | 1.808% |
| Henan | 1,517 | 1,517 | 1.686% |
| Shanxi | 1,448 | 1,448 | 1.609% |
| Guizhou | 1,411 | 1,411 | 1.568% |
| Jiangxi | 1,364 | 1,364 | 1.516% |
| Zhejiang | 1,259 | 1,259 | 1.399% |
| Anhui | 1,214 | 1,214 | 1.349% |
| Jiangsu | 1,182 | 1,182 | 1.313% |
| Fujian | 1,049 | 1,048 | 1.164% |
| Chongqing | 719 | 719 | 0.799% |
| Ningxia | 522 | 522 | 0.580% |
| Hainan | 375 | 375 | 0.417% |
| Shanghai | 167 | 167 | 0.186% |
| Beijing | 155 | 155 | 0.172% |
| Tianjin | 139 | 139 | 0.154% |

Tibet's 11.456% share reflects its large area under geographic sampling; it was
absent from this table entirely before this release. These rates reflect
geographic sampling and available administrative geometry, not population,
tourism activity, or completeness of lower-level coverage.

---

# 11. Boundary Isolation and Regional Probes

The 10,000 expected-outside samples produced zero validation failures under the
tester's policy rules. A scoped administrative polygon and its 2 km nearby
policy can legitimately produce assignments near the sampling boundary; the
pass rate is not an exhaustive cross-border leakage guarantee.

Independent probes confirm:

| Region or Probe Group | Observed Result |
| --- | --- |
| Fuzhou | `Fujian > Fuzhou City > Taijiang District`; previously failed |
| Xiamen | `Fujian > Xiamen > Siming District`; previously failed |
| Gulangyu | `Fujian > Xiamen > Siming District`; previously failed |
| Quanzhou | `Fujian > Quanzhou > Fengze District`; previously failed |
| Zhangzhou, Putian, Ningde | Resolve through their own prefectures; previously failed |
| Longyan, Sanming, Nanping | Resolve; inland prefectures already assembled |
| Lhasa | `Tibet > Lhasa > Chengguan District`; gains a level-4 parent |
| Shigatse | `Tibet > Shigatse Prefecture > Samzhubzê District`; newly covered |
| Nyingchi | `Tibet > Nyingchi Prefecture > Bayi District`; gains a level-4 parent |
| Haikou | `Hainan > Haikou City > Xiuying District`; unchanged |
| Shanghai Disneyland | `Shanghai > Pudong`; unchanged |
| Jinjiang, Pingtan | Empty administrative hierarchy; known county gaps |
| Sansha / Nansha | Empty administrative hierarchy; declared scope exclusion |

**All 31 provincial-capital and municipality probes now resolve**, against 30 in
v1.0.5 and 29 in v1.0.4; 27 of the 31 return a complete three-level hierarchy.

Xiamen, Gulangyu and Quanzhou resolve even though the coarse world-classification
layer still treats those points as open sea, because the existing 20 km
offshore/nearby policy recovers them now that Fujian carries administrative
geometry. **No `land_overrides` entry was required for this release**, and the
Cadis runtime package is unchanged.

---

# 12. Release Comparison and Remaining Limitations

| Level | v1.0.5 | v1.0.6 | Change |
| --- | --- | --- | --- |
| 4 | 29 | 31 | +2 |
| 5 | 325 | 335 | +10 |
| 6 | 2,775 | 2,847 | +72 |
| Total | 3,129 | 3,213 | +84 |

The additions are Fujian and Tibet at level 4, ten prefectures at level 5, and 72
county-level units. No feature was removed. Of the 3,129 existing records, 23
changed, and in every case the only altered field is `parent_id` moving from
`null` to a real parent: six Tibetan prefectures now attach to Tibet, and 17
Shigatse counties to Shigatse. No existing geometry, name, level, or
representative point changed. Runtime policy is byte-identical to v1.0.4 and
v1.0.5 (`aa9d98a4434813d838e37e0a0e5ce4cb03fcf89d21c46c6005d81df8a4baaf27`).

Derived polygons match independent references for their units' land extent within
the margin introduced by county boundaries that extend slightly offshore. Fujian
derives to approximately 126,000 km² against a roughly 124,000 km² reference.

The material remaining limitations are the ten unassembled Fujian counties and
one Shigatse county, the absent Pingtan prefecture, the declared Sansha
exclusion, remaining roots without explicit parent linkage, and source-dependent
multilingual aliases. Closing the county gaps requires member ways that no
published extract carries. The declared scope and successful mass evaluation do
not eliminate those limitations, and as noted in section 5 the mass test cannot
by construction detect them.

---

# 13. Reproducibility and Release Integrity

- Engine commit: `1eebc099f5915fc8de72f254573cf9b181e31280` (clean working tree).
- Cadis runtime: `0.10.10`, unchanged by this release.
- Build environment: the SOP's pinned Docker image
  `ghcr.io/isemptyc/cadis-dataset-engine@sha256:e5b134214a51368391576e5bbe6b06b403e3acf19d9d91124063caba72f1ae76`.
- Boundary generation and evaluation environment: conda `photo-importer-geo`.
- Source: physical China and Tibet PBF snapshots, both dated 2026-05-11 with
  replication timestamp `2026-05-11T20:20:52Z`, identical to v1.0.4 and v1.0.5.
- Composite source identity: `8a22fbaf3c88741431aea291532d18c180dcfbde61837a24a5319a03902b91bb`.
- Scope-builder configuration SHA-256:
  `cffacb04e6ea49062e16edfb9eea9938670b631ed0220a3b4696a78db313ffd8`.
- Build/evaluation boundary SHA-256:
  `8654ab79428ec045dac7991559626be69051bd3037f1b5b54fd3e0435b15af2f`.
- Release recipe: `scripts/cn_dataset_release.txt` in the build workflow repo.
- Release destination: `releases/CN/cn.admin/v1.0.6`.

The source files and composite source identity are unchanged across v1.0.4,
v1.0.5 and v1.0.6, so all added coverage is attributable to the build-scope and
derivation rules rather than to any source update. Exact reproduction of every
existing artifact record, apart from the 23 parent-link improvements listed
above, establishes that no corresponding drift was observed in this rebuild.

The standard release manifest records payload SHA-256 values and sizes.
The SOP verifies the local-release payload against the evaluated v1.0.6
manifest before preparing the release directory. Package and manifest sidecars
provide distribution integrity. Detailed mass-test JSON, source-completeness
probes, and fixed probe results remain local supporting evidence.

---

# 14. Conclusion

`cn.admin v1.0.6` passes the overall CN evaluation for its declared
administrative scope. It completes province-level coverage: all 31 units carry
materialized level-4 geometry, all 31 provincial capitals resolve, and Tibet no
longer depends on the level-5/6 anchor accommodation. Existing geometry is
preserved and 23 previously orphaned records gain correct parents. The remaining
county-level gaps and the Sansha exclusion are stated explicitly rather than left
implicit in the scope. The release is ready for operator review before Git and R2
distribution.
