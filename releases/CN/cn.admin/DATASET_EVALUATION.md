# CN_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `cn.admin`

Version: `v1.0.5`

Country: `CN`

Policy Version: `1.0`

Evaluation Date: `2026-09-04`

---

# 1. Purpose

This report evaluates the overall administrative coverage, structural integrity,
runtime behavior, and boundary handling of `cn.admin v1.0.5` under Cadis Runtime.

CN remains the public dataset identifier for the combined China and Tibet
source components. The dataset exposes administrative anchors at OSM levels
4, 5, and 6. Tibet is represented through its existing level-5/6 anchors.
Lower locality and detailed administrative levels remain outside the runtime
contract.

Version v1.0.5 preserves that model and restores Hainan and its 26 subordinate
units through a validated build-scope inclusion that derives the province from
its declared OSM subareas. Country-level evaluation and comparison with v1.0.4
confirm that existing feature content is unchanged. The scope is available
administrative coverage; it does not claim complete sovereign CN territory
coverage.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `cn.admin` |
| Dataset Version | `v1.0.5` |
| Country | `CN` |
| Country Name | China |
| Policy Version | `1.0` |
| Cadis Version | `0.10.10` |
| Hierarchy Required | True |
| Repair Required | False |
| Runtime Policy Detected | True |
| Name Schema | `multilingual_v1` |
| Administrative Features | 3,129 |
| Geometry/Metadata/Hierarchy/Policy Payload | 7,333,741 bytes (6.99 MiB) |
| Package Size | Approximately 5.5 MiB compressed; exact integrity values are in the release sidecars |

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

Two inclusion forms exist. The Shanghai inclusion (`913067`, expected level `4`,
expected `ISO3166-2=CN-SH`) admits normally assembled geometry that scope
membership would otherwise reject. The Hainan inclusion (`2128285`, expected
level `4`, expected `ISO3166-2=CN-HI`) is marked `derive_from_subareas`: the unit
contributes the union of the subareas OSM declares for it, because its own ring
cannot be assembled from the source. Both forms validate relation identity
against the configured administrative level and ISO code, and missing or
mismatched configured inclusions fail the build. Snapshot and release information
is historical provenance, not a version applicability limit.

**Why Hainan is derived.** Hainan's own level-4 ring is the South China Sea
maritime territorial boundary. Sixteen of its twenty-five outer member ways lie
between roughly 8.7°N and 11.1°N, in the Spratly area, and fall outside the clip
polygons used to cut the published regional extracts. They are therefore absent
from the source, and the relation cannot close. Hainan island itself is complete
in the source: nineteen of the province's twenty declared subareas assemble
normally, and their union reproduces the province's land extent.

**Declared scope exclusion.** Sansha (`2833102`) is named in
`excluded_subarea_relation_ids` and is outside this dataset's scope by
declaration rather than by source absence. It administers the Paracel, Spratly
and Zhongsha groups roughly 800 to 1,200 km offshore. Probes at Woody Island and
in the Nansha area return no administrative coverage, which is the declared
behavior and not a defect.

This build-scope rule does not add a runtime land override. Administrative
polygons can include water; their area must not be interpreted as a land-area
estimate. The runtime's existing scope flags and nearby/offshore policy remain
unchanged.

**Known exclusions and gaps:** Fujian's province relation and all seven of its
coastal prefectures still do not assemble from these extracts; their missing
member ways are Kinmen/Matsu restricted-water and coastline ways, which are
likewise absent from every published extract. Tibet has no level-4 polygon, for
the same class of reason, and continues through level-5/6 coverage. Other
relations rejected by the normal CN selection rule remain excluded. Detail levels
below the level-6 runtime contract remain excluded by design.

---

# 4. Administrative Model and Structural Integrity

| Level | Runtime Label | Dataset Count | Role |
| --- | --- | --- | --- |
| 4 | `admin_region` | 29 | Primary regional anchors, including Shanghai and Hainan |
| 5 | `admin_district` | 325 | Intermediate administrative units, including Tibet anchors |
| 6 | `admin_municipality` | 2,775 | District/county-level units and Tibet anchors |

All seven non-empty combinations of levels 4, 5, and 6 are allowed by runtime
policy and map to `ok`. A non-empty result need not contain every level.
Nearby fallback is enabled within 2 km; the offshore distance limit is 20 km.
Hierarchy support is required, while geometry repair fallback is not required.

The hierarchy contains **3,129 nodes**, **2,972 parent links**, and **157 roots**.
A root can represent a regional anchor or a feature without a materialized
parent link; it does not itself establish a geometry defect. Hainan's eighteen
direct subareas receive explicit parent links to the derived province, because
spatial parent inference only pairs adjacent levels and cannot attach a
county-level unit that sits directly beneath a province.

Canonical names follow the existing engine order: `name:en`, `name`,
`official_name`, `name:zh`, `name:bo`, and `name:ru`. Multilingual aliases are
bounded to `en`, `zh`, `bo`, and `ru`.

| Language Alias | Features Carrying Alias |
| --- | --- |
| `en` | 3,118 |
| `zh` | 3,109 |
| `bo` | 222 |
| `ru` | 1,230 |

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
  locations, the restored Hainan units, the declared Sansha/Nansha exclusion,
  and the remaining Fujian gap.
- All 3,102 existing feature records are compared against v1.0.4, and every
  added record is checked in v1.0.5.

Sampling is geographic rather than population-weighted. The same-scope mass
pass rate describes the declared administrative coverage and cannot discover
all territory omitted from the scope. Because the scope itself grew with this
release, the v1.0.4 and v1.0.5 sample populations are not identical and their
aggregate counts are not a direct longitudinal performance comparison. Fixed
probes and structural comparison provide independent evidence for the restored
coverage and remaining gaps.

---

# 6. Evaluation Results

| Metric | Value |
| --- | --- |
| Overall Pass Rate | 100.00% (100,000/100,000) |
| Inside Coverage Pass Rate | 100.00% (90,000/90,000) |
| Policy Pass Rate | 100.00% |
| Validation Failures | 0 |
| Inside Validation Failures | 0 |
| Throughput | 1183.600 QPS |
| Total Runtime | 84.488 seconds |
| Runtime Lookup Status: ok | 90,017 |
| Runtime Lookup Status: partial | 0 |
| Runtime Lookup Status: failed | 9,983 |
| Empty Administrative Hierarchies | 10,000 |
| Offshore Outcomes | 17 |

Expected outside points may return `failed`; that is distinct from a validation
failure. Offshore outcomes have empty administrative hierarchies and can carry
transport/lookup status `ok`. Such status alone does not imply administrative
coverage. The evaluation used runtime mode, so the tester's `http_200` counter
is not evidence of 100,000 actual network HTTP requests.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Statuses: ok / partial / failed / unknown |
| --- | --- | --- | --- | --- | --- |
| `full_policy` | 100.000% | 100.000% | 0 | 0 | 90,017 / 0 / 9,983 / 0 |
| `no_hierarchy` | 100.000% | 100.000% | 0 | 0 | 90,017 / 0 / 9,983 / 0 |
| `no_repair` | 100.000% | 100.000% | 0 | 0 | 90,017 / 0 / 9,983 / 0 |
| `no_nearby` | 99.998% | 99.998% | 2 | 2 | 89,998 / 0 / 10,002 / 0 |
| `osm_only` | 99.998% | 99.998% | 2 | 2 | 89,998 / 0 / 10,002 / 0 |

Rates above are recomputed from exact counts rather than copied from rounded
summary percentages. Full policy, no hierarchy, and no repair have zero
validation failures. Disabling nearby behavior yields two inside failures,
matching the v1.0.4 result.

---

# 8. Layer Contribution Analysis

| Scenario-Delta Metric | Samples |
| --- | --- |
| Rescued by hierarchy | 0 |
| Rescued by repair | 0 |
| Rescued by nearby | 19 |
| Total versus OSM-only | 19 |

The 19-sample nearby delta measures changes in scenario outcomes, including
offshore handling; it is not the number of inside validation failures prevented.
Only two inside failures occur without nearby behavior. Direct nearby source
usage appears in two full-policy samples, while 17 outcomes are offshore.

The tester reports zero hierarchy-rescue and zero repair-rescue samples. Its
node-source distribution also contains 15 `admin_tree_id` uses. These are
different metrics: source tagging does not establish that hierarchy was needed
to change a failed scenario into a passing one.

---

# 9. Structural Distribution

## 9.1 Administrative Shapes

| Category | Samples |
| --- | --- |
| `[4,5,6]` | 77,254 |
| `[]` | 10,000 |
| `[5,6]` | 8,929 |
| `[4,6]` | 1,838 |
| `[6]` | 1,524 |
| `[4]` | 358 |
| `[4,5]` | 97 |

## 9.2 Node Sources

| Category | Samples |
| --- | --- |
| `polygon` | 89,998 |
| `admin_tree_id` | 15 |
| `nearby` | 2 |

## 9.3 Source Mix

| Category | Samples |
| --- | --- |
| `polygon` | 89,983 |
| `__none__` | 10,000 |
| `admin_tree_id / polygon` | 15 |
| `nearby` | 2 |

## 9.4 Policy Reasons

| Category | Samples |
| --- | --- |
| `shape_status_map` | 90,000 |
| `empty_shape` | 9,983 |
| `offshore` | 17 |

These distributions describe sampled runtime outcomes, not the stored feature
count. Empty hierarchies are expected for outside-scope and offshore outcomes.

---

# 10. Level-4 Coverage

All **29** stored level-4 units were hit by the evaluation. There were
**79,547** total level-4 hits, all of them inside-scope hits. Tibet can
resolve through level 5 and level 6 without a level-4 result.

| Level-4 Unit | All Hits | Inside Hits | Inside-Sample Hit Rate |
| --- | --- | --- | --- |
| Xinjiang | 16,255 | 16,255 | 18.061% |
| Inner Mongolia | 11,732 | 11,732 | 13.036% |
| Qinghai | 5,844 | 5,844 | 6.493% |
| Heilongjiang | 5,260 | 5,260 | 5.844% |
| Sichuan | 4,389 | 4,389 | 4.877% |
| Gansu | 3,988 | 3,988 | 4.431% |
| Yunnan | 3,366 | 3,366 | 3.740% |
| Guangxi | 2,144 | 2,144 | 2.382% |
| Jilin | 2,011 | 2,011 | 2.234% |
| Guangdong | 2,008 | 2,008 | 2.231% |
| Shaanxi | 1,952 | 1,952 | 2.169% |
| Hebei | 1,951 | 1,951 | 2.168% |
| Shandong | 1,837 | 1,837 | 2.041% |
| Hunan | 1,786 | 1,786 | 1.984% |
| Liaoning | 1,780 | 1,780 | 1.978% |
| Hubei | 1,640 | 1,640 | 1.822% |
| Henan | 1,537 | 1,537 | 1.708% |
| Shanxi | 1,458 | 1,458 | 1.620% |
| Guizhou | 1,425 | 1,425 | 1.583% |
| Jiangxi | 1,381 | 1,381 | 1.534% |
| Zhejiang | 1,278 | 1,278 | 1.420% |
| Anhui | 1,225 | 1,225 | 1.361% |
| Jiangsu | 1,192 | 1,192 | 1.324% |
| Chongqing | 729 | 729 | 0.810% |
| Ningxia | 525 | 525 | 0.583% |
| Hainan | 393 | 393 | 0.437% |
| Shanghai | 167 | 167 | 0.186% |
| Beijing | 155 | 155 | 0.172% |
| Tianjin | 139 | 139 | 0.154% |

These rates reflect geographic sampling and available administrative geometry,
not population, tourism activity, or completeness of lower-level coverage.
The number 29 describes materialized level-4 coverage in this artifact and must
not be read as a claim that all province-level territories are represented.

---

# 11. Boundary Isolation and Regional Probes

The 10,000 expected-outside samples produced zero validation failures under the
tester's policy rules. A scoped administrative polygon and its 2 km nearby
policy can legitimately produce assignments near the sampling boundary; the
pass rate is not an exhaustive cross-border leakage guarantee.

Independent probes confirm:

| Region or Probe Group | Observed Result |
| --- | --- |
| Haikou | `Hainan > Haikou City > Xiuying District`; previously offshore with an empty hierarchy |
| Sanya | `Hainan > Sanya City > Jiyang District`; newly covered |
| Hainan inland | `Hainan > Qiongzhong Li and Miao Autonomous County`; previously failed |
| Danzhou, Wenchang, Qionghai, Wuzhishan, Dongfang, Lingshui | All resolve through Hainan; newly covered |
| Sansha / Woody Island | Failed with empty hierarchy; declared scope exclusion |
| Nansha offshore control | Failed with empty hierarchy; unchanged |
| Shanghai centre | `Shanghai > Huangpu District`; unchanged |
| Shanghai Disneyland | `Shanghai > Pudong`; unchanged |
| Universal Beijing | `Beijing > Tongzhou District`; unchanged |
| Lhasa | `Lhasa > Chengguan District` through level-5/6 anchors; unchanged |
| Fuzhou and Xiamen | Failed with empty administrative hierarchies; unchanged Fujian gap |

Of the 31 provincial-capital and municipality probes, **30 now resolve**; only
Fuzhou fails, against 29 in v1.0.4. Lingshui resolves through the existing
offshore/nearby policy despite the coarse world-classification layer treating
that coastal point as open sea, so no land override was required for this
release. These samples do not establish complete coverage of all provinces.

---

# 12. Release Comparison and Remaining Limitations

| Level | v1.0.4 | v1.0.5 | Change |
| --- | --- | --- | --- |
| 4 | 28 | 29 | +1 |
| 5 | 322 | 325 | +3 |
| 6 | 2,752 | 2,775 | +23 |
| Total | 3,102 | 3,129 | +27 |

The additions are Hainan, its three level-5 cities (Haikou, Sanya, Danzhou), and
23 level-6 units: Baisha, Baoting, Changjiang, Chengmai, Ding'an, Dongfang,
Haitang, Jiyang, Ledong, Lingao, Lingshui, Longhua, Meilan, Qionghai,
Qiongshan, Qiongzhong, Tianya, Tunchang, Wanning, Wenchang, Wuzhishan, Xiuying,
and Yazhou. No other feature was added or removed. All 3,102 existing features
retain identical geometry content, metadata, and hierarchy records. Runtime
policy is byte-identical to v1.0.4
(`aa9d98a4434813d838e37e0a0e5ce4cb03fcf89d21c46c6005d81df8a4baaf27`).

The derived Hainan polygon is the union of the eighteen declared subareas that
materialize at levels 4 through 6, simplified under the existing level-4 policy.
Its extent matches independent references for the province's land area within
the margin introduced by county boundaries that extend slightly offshore.

The material remaining limitations are the Fujian source-assembly gap, Tibet's
level-5/6 anchoring, the declared Sansha exclusion, incomplete explicit parent
linkage for some records, and source-dependent multilingual aliases. The
declared scope and successful mass evaluation do not eliminate those
limitations.

---

# 13. Reproducibility and Release Integrity

- Engine commit: `ea98134edd83239227e4c7a1513fcd2cfebf16ab` (clean working tree).
- Cadis runtime: `0.10.10`.
- Build environment: the SOP's pinned Docker image
  `ghcr.io/isemptyc/cadis-dataset-engine@sha256:e5b134214a51368391576e5bbe6b06b403e3acf19d9d91124063caba72f1ae76`.
- Boundary generation and evaluation environment: conda `photo-importer-geo`.
- Source: physical China and Tibet PBF snapshots, both dated 2026-05-11 with
  replication timestamp `2026-05-11T20:20:52Z`, identical to v1.0.4.
- Composite source identity: `8a22fbaf3c88741431aea291532d18c180dcfbde61837a24a5319a03902b91bb`.
- Scope-builder configuration SHA-256:
  `6ae175864f1e8c2ebfdd853cc932c176293e206c647a92397c018d25909acf3c`.
- Build/evaluation boundary SHA-256:
  `189244b88abfa9cd987b0259b7c8e89f062d2c461b6d73c83b2bd93967c49d83`.
- Release recipe: `scripts/cn_dataset_release.txt` in the build workflow repo.
- Release destination: `releases/CN/cn.admin/v1.0.5`.

The boundary environment differs from the pinned engine container. Exact
reproduction of every existing artifact record establishes that no corresponding
drift was observed in this rebuild. The source files and composite source
identity are unchanged from v1.0.4, so the added coverage is attributable to the
build-scope and derivation rules rather than to a source update.

The standard release manifest records payload SHA-256 values and sizes.
The SOP verifies the local-release payload against the evaluated v1.0.5
manifest before preparing the release directory. Package and manifest sidecars
provide distribution integrity. Detailed mass-test JSON, source-completeness
probes, and fixed probe results remain local supporting evidence.

---

# 14. Conclusion

`cn.admin v1.0.5` passes the overall CN evaluation for its declared
administrative scope. It retains the China/Tibet level-4/5/6 runtime contract,
restores Hainan and its 26 subordinate units through a declared subarea
derivation, and preserves every existing feature. The Sansha exclusion is
explicit rather than incidental, and the known Fujian and Tibet limitations
remain stated. The release is ready for operator review before Git and R2
distribution.
