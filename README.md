# GenMusic

GenMusic is a dataset for AI-generated music detection. It contains **1,132 audio recordings**, including **345 real recordings** and **787 AI-generated recordings** produced by **ACE-Step 1.5, YuE, and LeVo**.

The dataset includes predefined training, validation, and test partitions. Each real recording and its associated generated samples belong to the same partition.

## Dataset Overview

| Source | Recordings | Format |
|---|---:|---|
| Real music | 345 | MP3 |
| ACE-Step 1.5 | 345 | WAV |
| YuE | 221 | WAV |
| LeVo | 221 | FLAC |
| **Total** | **1,132** | |

The audio files occupy approximately **11.63 GB** before compression. The downloadable archive is approximately **7.69 GiB**.

## Dataset Construction

GenMusic is constructed from real music recordings and their associated textual descriptions. The selected source collection contains **221 vocal recordings** and **124 instrumental recordings**, based on the available annotations.

For each vocal source, a shared music description and a shared set of lyrics are used to condition ACE-Step 1.5, YuE, and LeVo. Lyrics are prepared using the ACE-Step language model and organized into verse and chorus sections. This provides matched textual conditions across the three generators.

For each instrumental source, ACE-Step 1.5 generates one instrumental sample. YuE and LeVo are used only for the vocal subset.

All three generators use **text-only conditioning**. The source recordings are not supplied as reference audio during generation. Generated samples are therefore associated with the source descriptions, rather than being audio-conditioned reconstructions of the real recordings.

### Generation Settings

| Generator | Scope | Duration setting |
|---|---|---|
| ACE-Step 1.5 | Vocal and instrumental music | 60-second target |
| YuE | Vocal music | Two lyric sections; duration emerges from generation |
| LeVo | Vocal music with accompaniment | 60-second duration cap |

ACE-Step 1.5 uses the local `acestep-v15-xl-sft` checkpoint with 50 inference steps and a guidance scale of 7. LeVo uses the local `songgeneration_base_new` configuration in mixed-audio mode.

Original output formats and durations are retained. The release does not apply a common resampling, cropping, or loudness-normalization step to all recordings.

## Dataset Splits

| Split | Real | ACE-Step 1.5 | YuE | LeVo | Total |
|---|---:|---:|---:|---:|---:|
| Train | 214 | 214 | 148 | 148 | 724 |
| Validation | 63 | 63 | 40 | 40 | 206 |
| Test | 68 | 68 | 33 | 33 | 202 |
| **Total** | **345** | **345** | **221** | **221** | **1,132** |

The partitions follow a **source-disjoint** protocol:

- A real recording and all of its generated variants remain in one partition.
- Available artist, album, and exact decoded-audio identity information is used to construct partition groups.
- No source group appears in more than one partition.
- All three generators are represented in training, validation, and testing.

These partitions support **closed-set detection with unseen source recordings**. They do not implement a leave-one-generator-out evaluation.

The full dataset can also serve as an external test set for detectors trained on other datasets. In that setting, no GenMusic samples should be used for training, fine-tuning, threshold selection, or other model-selection decisions.

## Download

<!-- Add the dataset hosting URL here. -->
**Download link:** To be added.

Archive:

```text
GenMusic_dataset_v1_source_disjoint.tar.zst
```

SHA256:

```text
01de5868383c8fd749e4f41ba1e3f8827945158c30863c35705e6878b80d1d0c
```

The archive contains the selected audio files, metadata, split manifests, and per-file checksums. Model weights and generation intermediate files are not included.

## Directory Structure

```text
dataset_release_v1/
├── README.md
├── checksums.sha256
├── audio/
│   ├── real/
│   ├── ace/
│   ├── yue/
│   └── levo/
└── metadata/
    ├── manifest.jsonl
    ├── train.jsonl
    ├── validation.jsonl
    ├── test.jsonl
    └── report.json
```

## Verification and Extraction

On Linux, verify the downloaded archive using its accompanying checksum file:

```bash
sha256sum -c GenMusic_dataset_v1_source_disjoint.tar.zst.sha256
```

On macOS:

```bash
shasum -a 256 -c GenMusic_dataset_v1_source_disjoint.tar.zst.sha256
```

Extract with a Zstandard-capable tool. For example, with `zstd` installed:

```bash
zstd -dc GenMusic_dataset_v1_source_disjoint.tar.zst | tar -xf -
```

Verify the extracted audio files:

```bash
cd dataset_release_v1
sha256sum -c checksums.sha256
```

On macOS, replace the last command with:

```bash
shasum -a 256 -c checksums.sha256
```

## Metadata

Each manifest is a JSON Lines file, with one audio recording per line.

| Field | Description |
|---|---|
| `sample_id` | Unique sample identifier |
| `path` | Audio path relative to `dataset_release_v1/` |
| `label` | `real` or `ai` |
| `generator` | `real`, `ace`, `yue`, or `levo` |
| `split` | `train`, `validation`, or `test` |
| `source_group_id` | Identifier linking a real recording to its generated variants |
| `partition_group_id` | Group used to keep related sources in the same partition |
| `caption` | Source-associated music description |
| `vocal_label` | Vocal or instrumental annotation |
| `protocol` | `source_disjoint` |
| `audio_sha256` | File checksum for generated audio, when present |
| `real_sha256` | File checksum for real audio, when present |

The `path` field is the portable audio location. Any retained absolute source paths are provenance fields and should not be used to locate files after downloading.

### Loading a Split

```python
import json
from pathlib import Path

root = Path("dataset_release_v1")

with (root / "metadata/train.jsonl").open(encoding="utf-8") as f:
    samples = [json.loads(line) for line in f if line.strip()]

for sample in samples:
    audio_path = root / sample["path"]
    label = 0 if sample["label"] == "real" else 1
    generator = sample["generator"]
```

## Experimental Considerations

- **Class imbalance:** The dataset contains more generated recordings than real recordings. Report class-sensitive metrics and document any class weighting or sampling strategy.
- **Audio-format differences:** Real and generated recordings differ in codec, sample rate, channel count, and duration. Apply a consistent input-processing protocol across classes and report it.
- **Source grouping:** Split recordings before extracting segments. Every segment from a recording must inherit its recording-level partition.
- **Annotation uncertainty:** Vocal and language annotations are partly derived from descriptions and are not uniformly verified through listening.
- **Shared lyric preparation:** Lyrics are prepared using the ACE-Step language model, which may introduce conditioning preferences.
- **Duplicate detection scope:** Exact audio identity checks do not establish the absence of covers or near-duplicate recordings.
- **Generator identity:** The LeVo release uses the local `songgeneration_base_new` configuration; it should not be described as a verified LeVo 2 checkpoint.

## Source Attribution and License

<!-- Complete before public release:
     1. Identify the original music collection and provide its official URL.
     2. Document source recording IDs, attribution requirements, and licenses.
     3. Specify the license for newly released metadata and generated audio.
     4. Confirm which source recordings can be redistributed.
-->

Source collection attribution and release licensing are pending completion. This README does not grant redistribution rights for the source recordings.

