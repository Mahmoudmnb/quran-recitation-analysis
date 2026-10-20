# Maintainer notes

This file is a publication checklist for the public showcase. The supplied project facts control the documentation; unspecified implementation details should remain unspecified until verified against an approved source.

## Public scope

Publish project motivation, architecture summaries, methodology, selected reported results, technical limitations, academic credits, responsible-use notes, and clearly illustrative examples. Original public assets may be included only after rights and content review.

## Keep private

Do not commit model weights or checkpoints, datasets, private audio, full training or inference code, private preprocessing code, or proprietary/private experiment code. Do not copy implementation details from the private repository to fill gaps in the showcase.

Ignore rules are a convenience, not a publication review; they do not remove files already tracked by Git.

## Before adding new assets or publishing changes

- [ ] Verify that the material belongs in the public showcase.
- [ ] Verify contributor approval and the relevant ownership, licensing, and redistribution rights.
- [ ] Verify rights before uploading audio; keep restricted recordings and dataset excerpts private.
- [ ] Confirm that no model weights, datasets, or private code are included.
- [ ] Remove credentials, secrets, identifying local paths, and private configuration.
- [ ] Check figures for private implementation details and verify them against documented architecture.
- [ ] Keep synthetic labels and timing examples clearly marked as illustrative.
- [ ] Preserve experiment-specific metric labels; do not relabel accuracy as PER, CER, or universal Tajweed accuracy.
- [ ] Review attribution, the custom LICENSE notice, and permissions from relevant rights holders.
- [ ] Check links, JSON examples, repository diff, and staged file list before committing.

## Pending manual materials

The following PNGs await real, reviewed assets; no placeholder image files exist:

- `docs/architecture/system-overview.png`
- `docs/architecture/model-evolution.png`
- `docs/architecture/final-model.png`

Add exact research citations from the graduation report to `references/README.md`. Confirm publication permission before distributing any team report; no report is included here. Metric definitions and additional protocol details require an approved source before documentation can be expanded.

## Suggested GitHub metadata

**Description:** Audio AI system for Quran recitation analysis using phoneme recognition, boundary detection, duration estimation, CTC, Wav2Vec2, and Conformer.

**Topics:**

```text
audio-ai
speech-processing
quran
tajweed
phoneme-recognition
phoneme-alignment
phoneme-segmentation
speech-recognition
wav2vec2
conformer
ctc
pytorch
deep-learning
audio-processing
```

These are suggestions for manual repository settings. They do not assert that source code, dependencies, a runnable model, or datasets are distributed here.
