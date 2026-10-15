# GenMusic

GenMusic is a dataset for AI-generated music detection. It contains **1,132 audio recordings**, comprising **345 real recordings** and **787 generated recordings** produced by **ACE-Step 1.5, YuE, and LeVo**.

The dataset provides predefined training, validation, and test partitions. Each real recording and all its associated generated samples remain in the same partition.

## Dataset Overview

| Source | Recordings | Format |
|---|---:|---|
| Real music | 345 | MP3 |
| ACE-Step 1.5 | 345 | WAV |
| YuE | 221 | WAV |
| LeVo | 221 | FLAC |
| **Total** | **1,132** | |

The audio files occupy approximately **11.63 GB** before compression. The compressed archive is approximately **7.69 GiB**.

## Dataset Construction

GenMusic is constructed from real music recordings and their associated textual descriptions. The selected source collection contains **221 vocal recordings** and **124 instrumental recordings**, according to the available annotations.

For each vocal source, a shared music description and a shared set of lyrics condition all three generators. Lyrics are prepared using the ACE-Step language model and organized into verse and chorus sections. Each vocal source therefore corresponds to one real recording and three generated recordings.

For each instrumental source, ACE-Step 1.5 generates one instrumental recording. YuE and LeVo are used only for the vocal subset.

All generation uses **text-only conditioning**. Source recordings are not provided as reference audio. Generated recordings share descriptive conditions with their corresponding real recordings, rather than being audio-conditioned reconstructions.

### Generation Settings

| Generator | Subset | Duration setting |
|---|---|---|
| ACE-Step 1.5 | Vocal and instrumental | 60-second target |
| YuE | Vocal | Two lyric sections; variable output duration |
| LeVo | Vocal with accompaniment | 60-second duration cap |

ACE-Step 1.5 uses the `acestep-v15-xl-sft` checkpoint with 50 inference steps and a guidance scale of 7. LeVo uses the local `songgeneration_base_new` configuration in mixed-audio mode.

Original audio formats and output durations are retained. No common resampling, cropping, or loudness-normalization procedure is applied across all recordings during dataset packaging.

## Dataset Splits

| Split | Real | ACE-Step 1.5 | YuE | LeVo | Total |
|---|---:|---:|---:|---:|---:|
| Train | 214 | 214 | 148 | 148 | 724 |
| Validation | 63 | 63 | 40 | 40 | 206 |
| Test | 68 | 68 | 33 | 33 | 202 |
| **Total** | **345** | **345** | **221** | **221** | **1,132** |

The dataset follows a **source-disjoint** partitioning protocol:

- A real recording and all its generated variants belong to the same partition.
- Available artist, album, and exact decoded-audio identity information is used to group related sources.
- No source group appears in multiple partitions.
- All three generators are represented in every partition.

## Evaluation Protocols

### Within-Dataset Evaluation

Train detectors on the training partition, select models and thresholds using the validation partition, and report final results on the test partition.

This protocol evaluates detection on unseen source recordings from the same three generators. It is a closed-set protocol and does not hold out a generator.

### External Evaluation

The entire dataset can serve as an external test set for detectors trained on other datasets.

Under this protocol, GenMusic must not be used for training, fine-tuning, threshold selection, or model selection. Any known overlap with a detector’s training data should be checked and reported.

## Download

**Download:** [GenMusic Dataset v1 (Quark Cloud)](https://pan.quark.cn/s/f4fb2d316d90?pwd=kuXe)

Archive filename:

```text
GenMusic_dataset_v1_source_disjoint.tar.zst
```

The archive includes the selected audio recordings, metadata, and predefined split manifests. Model weights and generation intermediate files are excluded.

Extract the archive with `zstd` installed:

```bash
zstd -dc GenMusic_dataset_v1_source_disjoint.tar.zst | tar -xf -
```

## Directory Structure

The main dataset files are organized as follows:

```text
dataset_release_v1/
├── README.md
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

## Metadata

Each manifest uses JSON Lines format, with one recording per line.

| Field | Description |
|---|---|
| `sample_id` | Unique sample identifier |
| `path` | Audio path relative to the dataset root |
| `label` | Binary class: `real` or `ai` |
| `generator` | Recording source: `real`, `ace`, `yue`, or `levo` |
| `split` | Partition: `train`, `validation`, or `test` |
| `source_group_id` | Links a real recording with its generated variants |
| `partition_group_id` | Identifies related sources grouped during partitioning |
| `caption` | Source-associated music description |
| `vocal_label` | Vocal or instrumental annotation |
| `protocol` | Partitioning protocol: `source_disjoint` |

Use `path` to locate audio after extraction. Any retained absolute source paths are provenance fields and are not portable.

## Usage

Load the training manifest:

```python
import json
from pathlib import Path

dataset_root = Path("dataset_release_v1")
manifest_path = dataset_root / "metadata" / "train.jsonl"

with manifest_path.open(encoding="utf-8") as f:
    samples = [
        json.loads(line)
        for line in f
        if line.strip()
    ]

for sample in samples:
    audio_path = dataset_root / sample["path"]
    label = 0 if sample["label"] == "real" else 1

    print(audio_path, label, sample["generator"])
```

For validation or testing, replace `train.jsonl` with `validation.jsonl` or `test.jsonl`.

## Experimental Considerations

- **Class imbalance:** Generated recordings outnumber real recordings. Document class weighting or sampling strategies and report class-sensitive metrics.
- **Audio differences:** Codecs, sample rates, channel counts, and durations vary across sources. Use a consistent input-processing protocol to reduce reliance on these differences.
- **Segment-level leakage:** Extract segments after assigning recording-level partitions. All segments from one recording must remain in its assigned partition.
- **Annotation uncertainty:** Some vocal and language annotations are inferred from descriptions and have not been uniformly verified through listening.
- **Lyric preparation:** Shared lyrics are prepared using the ACE-Step language model, which may introduce conditioning preferences.
- **Duplicate detection:** Exact audio identity checks do not rule out covers or near-duplicate recordings.
- **LeVo configuration:** The dataset uses `songgeneration_base_new`; this checkpoint has not been verified as LeVo 2.

## Source Attribution and License

Original music collection attribution and release licensing will be provided before public release.

Rights and attribution requirements for source recordings must be documented separately from those for generated recordings and newly created metadata.
