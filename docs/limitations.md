# Limitations

This public repository documents a graduation research project. It provides neither a runnable model nor evidence of production readiness.

## Speaker dependence and generalization

The evaluation design included different reciters across subsets and different voices and verses in testing. Generalization to unseen speakers remains a challenge. Approximately 100 hours of audio does not establish coverage of all voices, recitation styles, microphones, or environments; detailed diversity statistics are not supplied.

## Overfitting and recording conditions

Reported training values exceed validation and test values where available. This gap warrants investigating overfitting and speaker dependence, but the supplied results cannot isolate their causes. Recording conditions and preprocessing can affect acoustic features and model behavior; robustness across arbitrary conditions is not established.

## Limited public benchmark and reproducibility

The dataset, private code, model checkpoints, and full experiment configurations are excluded. Readers cannot reproduce the reported runs from this showcase. The exact accuracy definitions, uncertainty estimates, and complete split details are not supplied. The reported values are not directly comparable to PER or CER and do not constitute a public standardized benchmark.

## Boundary estimation

CTC sequence recognition does not automatically deliver accurate phoneme boundaries. Acoustic transitions can be gradual, and timing errors affect duration estimates. Dedicated segmentation addresses this objective, but the reported accuracy values do not quantify boundary error in milliseconds or duration error.

## Temporal Tajweed complexity

Madd and Ghunnah motivate duration-sensitive analysis, yet duration alone does not establish complete recitation correctness. Interpretation depends on phonetic context and expert judgment. This showcase supplies no universal timing thresholds or validated coverage of every rule.

## Selected rules only

The phonetic representation explored selected categories, including definite article behavior, stopping, hamzat al-wasl, Ghunnah, noon and meem sakinah rules, idgham-related cases, emphatic raa, and qalqalah. This is not a claim of complete end-to-end modeling. **No claim of full automatic Tajweed correction is made.**

## Dataset licensing and redistribution

The dataset is not distributed due to ownership, licensing, and redistribution considerations. The repository license grants no rights to private data, third-party audio, Quran recitation recordings, or other excluded material. Future public assets require verification of the relevant rights.

## Educational interpretation

The system is intended as an assistive educational tool. Predictions and timing estimates may be inaccurate. It is **not a replacement for a qualified Quran teacher** or expert instruction and assessment.
