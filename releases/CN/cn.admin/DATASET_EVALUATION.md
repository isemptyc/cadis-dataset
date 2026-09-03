# CN_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `cn.admin`

Version: `v1.0.4`

Country: `CN`

Policy Version: `1.0`

Evaluation Date: `2026-09-03`

---

# 1. Purpose

This report evaluates the overall administrative coverage, structural integrity,
runtime behavior, and boundary handling of `cn.admin v1.0.4` under Cadis Runtime.

CN remains the public dataset identifier for the combined China and Tibet
source components. The dataset exposes administrative anchors at OSM levels
4, 5, and 6. Tibet is represented through its existing level-5/6 anchors.
Lower locality and detailed administrative levels remain outside the runtime
contract.

Version v1.0.4 preserves that model and restores Shanghai and its 16 districts
through a validated build-scope inclusion. Country-level evaluation and
comparison with v1.0.3 confirm that existing feature content is unchanged.
The scope is available administrative coverage; it does not claim complete
sovereign CN territory coverage.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `cn.admin` |
| Dataset Version | `v1.0.4` |
| Country | `CN` |
| Country Name | China |
| Policy Version | `1.0` |
| Cadis Version | `0.10.10` |
| Hierarchy Required | True |
| Repair Required | False |
| Runtime Policy Detected | True |
| Name Schema | `multilingual_v1` |
| Administrative Features | 3,102 |
| Geometry/Metadata/Hierarchy/Policy Payload | 7,290,472 bytes (6.95 MiB) |
| Package Size | Approximately 5.44 MiB compressed; exact integrity values are in the release sidecars |

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
| Boundary BBox | `[73.50254522441283, 20.12183040413475, 134.77345103091676, 53.55881051212936]` |
| Build and Evaluation Scope | The same generated boundary is used for both |

Ordinary China level-4 administrative areas are selected when their
representative point falls within the Natural Earth CN boundary, or the existing
`ISO3166-1:alpha2=CN` condition applies. Explicitly configured and validated scope
inclusions are then admitted. Tibet contributes materialized level-5/6
administrative areas. The combined geometry is unioned, repaired if necessary,
and given the existing 0.002-degree inward buffer.

The Shanghai inclusion is restricted to OSM relation `913067`, expected
administrative level `4`, and expected `ISO3166-2=CN-SH`, with usable extracted
polygon geometry. It bypasses scope-membership rejection only; source geometry
is retained through the ordinary extraction/repair path. Missing or mismatched
configured inclusions fail the build. Snapshot and release information is
historical provenance, not a version applicability limit.

This build-scope rule does not add a runtime land override. Administrative
polygons can include water; their area must not be interpreted as a land-area
estimate. The runtime's existing scope flags and nearby/offshore policy remain
unchanged.

**Known exclusions and gaps:** Fujian and Hainan province geometries do not
assemble from the retained extracts, leaving the associated regional coverage
gaps. Tibet has no newly added level-4 polygon and continues through level-5/6
coverage. Other relations rejected by the normal CN selection rule remain
excluded. Detail levels below the level-6 runtime contract remain excluded by
design.

---

# 4. Administrative Model and Structural Integrity

| Level | Runtime Label | Dataset Count | Role |
| --- | --- | --- | --- |
| 4 | `admin_region` | 28 | Primary regional anchors, including Shanghai |
| 5 | `admin_district` | 322 | Intermediate administrative units, including Tibet anchors |
| 6 | `admin_municipality` | 2,752 | District/county-level units and Tibet anchors |

All seven non-empty combinations of levels 4, 5, and 6 are allowed by runtime
policy and map to `ok`. A non-empty result need not contain every level.
Nearby fallback is enabled within 2 km; the offshore distance limit is 20 km.
Hierarchy support is required, while geometry repair fallback is not required.

The hierarchy contains **3,102 nodes**, **2,946 parent links**, and **156 roots**.
A root can represent a regional anchor or a feature without a materialized
parent link; it does not itself establish a geometry defect. The 17 newly
retained Shanghai records have no explicit parent links under the existing
strict hierarchy construction. Their validated lookups resolve through polygon
containment.

Canonical names follow the existing engine order: `name:en`, `name`,
`official_name`, `name:zh`, `name:bo`, and `name:ru`. Multilingual aliases are
bounded to `en`, `zh`, `bo`, and `ru`.

| Language Alias | Features Carrying Alias |
| --- | --- |
| `en` | 3,091 |
| `zh` | 3,082 |
| `bo` | 219 |
| `ru` | 1,223 |

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
  locations, additional city/attraction/control points, and known gaps.
- All 3,085 existing feature representative points are compared against v1.0.3;
  all 17 added feature representative points are checked in v1.0.4.
- Payload checksums, per-feature geometry, metadata, and hierarchy are compared
  with the verified published v1.0.3 archive.

Sampling is geographic rather than population-weighted. The same-scope mass
pass rate describes the declared administrative coverage and cannot discover
all territory omitted from the scope. Fixed probes and structural comparison
provide independent evidence for the restored coverage and remaining gaps.
The previous v1.0.3 report used an 85%/15% inside/outside split; aggregate
sample counts are therefore not a direct longitudinal performance comparison.

---

# 6. Evaluation Results

| Metric | Value |
| --- | --- |
| Overall Pass Rate | 100.00% (100,000/100,000) |
| Inside Coverage Pass Rate | 100.00% (90,000/90,000) |
| Policy Pass Rate | 100.00% |
| Validation Failures | 0 |
| Inside Validation Failures | 0 |
| Throughput | 1167.940 QPS |
| Total Runtime | 85.621 seconds |
| Runtime Lookup Status: ok | 90,021 |
| Runtime Lookup Status: partial | 0 |
| Runtime Lookup Status: failed | 9,979 |
| Empty Administrative Hierarchies | 9,999 |
| Offshore Outcomes | 20 |

Expected outside points may return `failed`; that is distinct from a validation
failure. Offshore outcomes have empty administrative hierarchies and can carry
transport/lookup status `ok`. Such status alone does not imply administrative
coverage. The evaluation used runtime mode, so the tester's `http_200` counter
is not evidence of 100,000 actual network HTTP requests.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Statuses: ok / partial / failed / unknown |
| --- | --- | --- | --- | --- | --- |
| `full_policy` | 100.000% | 100.000% | 0 | 0 | 90,021 / 0 / 9,979 / 0 |
| `no_hierarchy` | 100.000% | 100.000% | 0 | 0 | 90,021 / 0 / 9,979 / 0 |
| `no_repair` | 100.000% | 100.000% | 0 | 0 | 90,021 / 0 / 9,979 / 0 |
| `no_nearby` | 99.998% | 99.998% | 2 | 2 | 89,998 / 0 / 10,002 / 0 |
| `osm_only` | 99.998% | 99.998% | 2 | 2 | 89,998 / 0 / 10,002 / 0 |

Rates above are recomputed from exact counts rather than copied from rounded
summary percentages. Full policy, no hierarchy, and no repair have zero
validation failures. Disabling nearby behavior yields two inside failures.

---

# 8. Layer Contribution Analysis

| Scenario-Delta Metric | Samples |
| --- | --- |
| Rescued by hierarchy | 0 |
| Rescued by repair | 0 |
| Rescued by nearby | 23 |
| Total versus OSM-only | 23 |

The 23-sample nearby delta measures changes in scenario outcomes, including
offshore handling; it is not the number of inside validation failures prevented.
Only two inside failures occur without nearby behavior. Direct nearby source
usage appears in three full-policy samples, while 20 outcomes are offshore.

The tester reports zero hierarchy-rescue and zero repair-rescue samples. Its
node-source distribution also contains 16 `admin_tree_id` uses. These are
different metrics: source tagging does not establish that hierarchy was needed
to change a failed scenario into a passing one.

---

# 9. Structural Distribution

## 9.1 Administrative Shapes

| Category | Samples |
| --- | --- |
| `[4,6]` | 1,531 |
| `[4,5,6]` | 77,555 |
| `[]` | 9,999 |
| `[5,6]` | 8,961 |
| `[6]` | 1,530 |
| `[4]` | 359 |
| `[4,5]` | 65 |

## 9.2 Node Sources

| Category | Samples |
| --- | --- |
| `polygon` | 89,998 |
| `admin_tree_id` | 16 |
| `nearby` | 3 |

## 9.3 Source Mix

| Category | Samples |
| --- | --- |
| `polygon` | 89,982 |
| `__none__` | 9,999 |
| `admin_tree_id / polygon` | 16 |
| `nearby` | 3 |

## 9.4 Policy Reasons

| Category | Samples |
| --- | --- |
| `shape_status_map` | 90,001 |
| `empty_shape` | 9,979 |
| `offshore` | 20 |

These distributions describe sampled runtime outcomes, not the stored feature
count. Empty hierarchies are expected for outside-scope and offshore outcomes.

---

# 10. Level-4 Coverage

All **28** stored level-4 units were hit by the evaluation. There were
**79,510** total level-4 hits, including **79,509** inside-scope hits. Tibet can
resolve through level 5 and level 6 without a level-4 result.

| Level-4 Unit | All Hits | Inside Hits | Inside-Sample Hit Rate |
| --- | --- | --- | --- |
| Xinjiang | 16,334 | 16,334 | 18.149% |
| Inner Mongolia | 11,776 | 11,776 | 13.084% |
| Qinghai | 5,878 | 5,878 | 6.531% |
| Heilongjiang | 5,284 | 5,284 | 5.871% |
| Sichuan | 4,412 | 4,412 | 4.902% |
| Gansu | 4,004 | 4,004 | 4.449% |
| Yunnan | 3,384 | 3,384 | 3.760% |
| Guangxi | 2,154 | 2,154 | 2.393% |
| Jilin | 2,023 | 2,022 | 2.247% |
| Guangdong | 2,013 | 2,013 | 2.237% |
| Shaanxi | 1,960 | 1,960 | 2.178% |
| Hebei | 1,956 | 1,956 | 2.173% |
| Shandong | 1,847 | 1,847 | 2.052% |
| Liaoning | 1,793 | 1,793 | 1.992% |
| Hunan | 1,790 | 1,790 | 1.989% |
| Hubei | 1,646 | 1,646 | 1.829% |
| Henan | 1,545 | 1,545 | 1.717% |
| Shanxi | 1,464 | 1,464 | 1.627% |
| Guizhou | 1,432 | 1,432 | 1.591% |
| Jiangxi | 1,385 | 1,385 | 1.539% |
| Zhejiang | 1,281 | 1,281 | 1.423% |
| Anhui | 1,227 | 1,227 | 1.363% |
| Jiangsu | 1,199 | 1,199 | 1.332% |
| Chongqing | 731 | 731 | 0.812% |
| Ningxia | 528 | 528 | 0.587% |
| Shanghai | 168 | 168 | 0.187% |
| Beijing | 156 | 156 | 0.173% |
| Tianjin | 140 | 140 | 0.156% |

These rates reflect geographic sampling and available administrative geometry,
not population, tourism activity, or completeness of lower-level coverage.
The number 28 describes materialized level-4 coverage in this artifact and must
not be read as a claim that all province-level territories are represented.

---

# 11. Boundary Isolation and Regional Probes

The 10,000 expected-outside samples produced zero validation failures under the
tester's policy rules. A scoped administrative polygon and its 2 km nearby
policy can legitimately produce assignments near the sampling boundary; the
pass rate is not an exhaustive cross-border leakage guarantee. One level-4
hit occurred at an outside-labeled sample, as the totals above show.

Independent probes confirm:

| Region or Probe Group | Observed Result |
| --- | --- |
| Beijing, Tianjin, Chongqing and the other 24 previously passing provincial capitals | All remain ok with unchanged administrative results |
| Lhasa | Lhasa and Chengguan resolve through level-5/6 anchors; unchanged |
| Shanghai center | Shanghai and Huangpu resolve through polygons |
| Shanghai Disneyland | Shanghai and Pudong resolve through polygons |
| Shanghai and Pudong source representative points | Both resolve successfully |
| Universal Beijing | Beijing and Tongzhou resolve; unchanged |
| Fuzhou and Xiamen | Failed with empty administrative hierarchies; unchanged gap |
| Haikou | Offshore classification and empty hierarchy; unchanged gap |
| Hainan inland | Failed with empty hierarchy; unchanged gap |
| Nansha offshore control | Failed with empty hierarchy; unchanged |

Overall, the 28 previously successful provincial-capital/municipality probes
remain successful, Shanghai becomes successful, and Fuzhou/Haikou still lack
administrative coverage. These samples do not establish complete coverage of
all other provinces.

---

# 12. Release Comparison and Remaining Limitations

| Level | v1.0.3 | v1.0.4 | Change |
| --- | --- | --- | --- |
| 4 | 27 | 28 | +1 |
| 5 | 322 | 322 | 0 |
| 6 | 2,736 | 2,752 | +16 |
| Total | 3,085 | 3,102 | +17 |

The additions are Shanghai and its 16 districts: Baoshan, Changning, Chongming,
Fengxian, Hongkou, Huangpu, Jiading, Jing'an, Jinshan, Minhang, Pudong, Putuo,
Qingpu, Songjiang, Xuhui, and Yangpu. No other feature was added or removed.
All 3,085 existing features retain identical geometry content, metadata,
hierarchy records, and representative-point lookup results. Runtime policy is
byte-identical to v1.0.3.

Full-source scope regression checks establish that ordinary accepted geometry
is unchanged and the new unbuffered union is exactly the old union plus the
normal extracted/repaired Shanghai geometry. The final scope matches the
existing union/repair/inward-buffer operation. It restores approximately
189.68 km² of adjoining border seams outside Shanghai's source polygon,
all within 0.002 degrees of Shanghai. No additional administrative features
are admitted by those restored seams, and the overall scope bbox is unchanged.
No scope area is removed.

Eleven focused tests cover geometry preservation, bounded inclusion, final
buffer ordering, fail-closed validation, historical provenance surviving source
updates, and real Shanghai/Pudong/Disneyland lookup regressions.

The material remaining limitations are the Fujian/Hainan source-assembly gaps,
Tibet's level-5/6 anchoring, incomplete explicit parent linkage for some records,
and source-dependent multilingual aliases. The declared scope and successful
mass evaluation do not eliminate those limitations.

---

# 13. Reproducibility and Release Integrity

- Engine commit: `7746762fa3aa2877a70436a0e40d0623689c7938` (clean working tree).
- Cadis runtime: `0.10.10`.
- Build environment: the SOP's pinned Docker image
  `ghcr.io/isemptyc/cadis-dataset-engine@sha256:e5b134214a51368391576e5bbe6b06b403e3acf19d9d91124063caba72f1ae76`.
- Docker allocation: 4 CPUs and 8 GB RAM.
- Boundary generation and evaluation environment: conda `photo-importer-geo`.
- Source: physical China and Tibet PBF snapshots, both dated 2026-05-11 with
  replication timestamp `2026-05-11T20:20:52Z`.
- Source component SHA-256 values:
  - China: `12973f26feaad974cc6ca6053c049859cd6d05a7b070df0a04325457fc94cd6c`
  - Tibet: `2a71933f8cb55fa7b346a0fa03c4c9439f688b9fbb7881f9548f2355a2973291`
- Composite source identity: `8a22fbaf3c88741431aea291532d18c180dcfbde61837a24a5319a03902b91bb`.
- Scope-builder configuration SHA-256:
  `acb562a2206277b51d72baedb45e5dab4df6d5217f0e0671da0ca7ab0b1b2eb7`.
- Build/evaluation boundary SHA-256:
  `36054a30f5f8c729411593dd1784be2eba89a79ea33f96cd37198cbfbe8a1fa8`.
- Release recipe: `scripts/cn_dataset_release.txt` in the build workflow repo.
- Release destination: `releases/CN/cn.admin/v1.0.4`.

The boundary environment differs from the pinned engine container. Exact
reproduction of the old scope and identical existing artifact geometries
establish that no corresponding drift was observed in this rebuild.
The new composite source identity hashes exactly the two consumed files;
its difference from v1.0.3's historical directory hash is not evidence of a
geographic source update.

The standard release manifest records payload SHA-256 values and sizes.
The SOP verifies the local-release payload against the evaluated v1.0.4
manifest before preparing the release directory. Package and manifest sidecars
provide distribution integrity. Detailed mass-test JSON, scope audit, fixed
probe results, and the Shanghai investigation remain local supporting evidence.

---

# 14. Conclusion

`cn.admin v1.0.4` passes the overall CN evaluation for its declared
administrative scope. It retains the China/Tibet level-4/5/6 runtime contract,
restores Shanghai and its districts, and preserves every existing feature.
The known regional coverage and structural limitations above remain explicit.
The release is ready for operator review before Git and R2 distribution.
