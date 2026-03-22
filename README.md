# cadis-dataset

Cadis dataset release repository (CDN surface).

This repository contains immutable, versioned country dataset releases
consumed by the Cadis runtime.

It serves as the public distribution surface for released datasets only.

## Scope

This repository contains release artifacts, including:

- `releases/dataset_manifest.json` (repository-level dataset index)
- dataset-level `DATASET_EVALUATION.md` (evaluation + reproducibility report)
- versioned `dataset_release_manifest.json` and `dataset_release_manifest.sha256`
- versioned `dataset_package.tar.gz` and `dataset_package.tar.gz.sha256`

It does **not** contain:

- dataset transformation engine code
- internal build/governance tooling
- OpenStreetMap extract files

The separate `cadis-dataset-engine` repository contains reproducibility/build
code used to generate published dataset artifacts. That repository is licensed
under Apache License 2.0. Its code license does not apply to dataset artifacts
distributed from this repository.

## Release Contract

1. Releases are immutable once published.
2. Existing versions are never modified in place.
3. Any dataset change must be published as a new version.
4. Cadis runtime treats this repository as release truth for integrity and policy-layer validation.
5. Dataset-level evaluation reports are released with datasets and document structural integrity, boundary behavior, and reproducibility metadata.

## Layout

```text
cadis-dataset/
└── releases/
    ├── dataset_manifest.json
    └── <ISO2>/
        └── <dataset_id>/
            ├── DATASET_EVALUATION.md
            └── <version>/
                ├── dataset_release_manifest.json
                ├── dataset_release_manifest.sha256
                ├── dataset_package.tar.gz
                └── dataset_package.tar.gz.sha256
```

## Manifest Profile

Release manifests use:

- `profile = "cadis.dataset.release"`
- `schema_version = 2`

Dataset index manifest uses:

- `path = "releases/dataset_manifest.json"`
- `schema_version = 1`

## Licensing and Attribution

Datasets in this repository are derived from OpenStreetMap data.

© OpenStreetMap contributors  
Licensed under the Open Database License (ODbL) 1.0  
[https://www.openstreetmap.org/copyright](https://www.openstreetmap.org/copyright)

Published dataset artifacts in this repository remain separately licensed under
ODbL as applicable. The Apache 2.0 license of `cadis-dataset-engine` applies to
that repository's code only.

This repository does not distribute raw OpenStreetMap extracts.
