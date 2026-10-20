# Methodology

This document summarizes the documented research direction. It does not distribute the private implementation or fill in unspecified training details. Selected values are recorded in the [experiment summary](results/experiment-summary.md).

## Preprocessing and acoustic representations

Traditional processing experiments explored High-Pass and Band-Pass filters, noise-profile estimation, noise reduction, and energy-based silence removal. Earlier feature pipelines used mel-frequency cepstral coefficients (MFCC), their first-order differences (Delta MFCC), second-order differences (Delta-Delta MFCC), and cepstral mean and variance normalization (CMVN).

These methods address frequency content, noise, low-energy regions, spectral structure, and feature normalization. They are explored techniques rather than a single verified recipe applied to every run. Filter cutoffs, noise-estimation settings, augmentation recipes, and feature-extraction parameters are not specified in the supplied public facts.

Later experiments used Wav2Vec2-style learned speech representations as input to the temporal encoder. The particular checkpoint, fine-tuning policy, and feature stride are not specified here.

## Pronunciation-aware phoneme representation

The project explored phonetic handling of definite articles with sun/moon letters, taa marbuta, stopping behavior, hamzat al-wasl, Ghunnah, noon sakinah and tanween, meem sakinah, idgham-related cases, emphatic raa, and qalqalah.

This direction addresses the distinction between written text and expected spoken units. The public examples use generic labels, not an asserted Quranic phoneme inventory. The private conversion rules are excluded, and exploration of these categories does not establish complete rule coverage.

## Early experiments

Whisper experiments explored ASR and word timestamps. Timing estimates could overlap and did not provide sufficient precision for phoneme-level duration analysis.

Traditional signal-processing and HMM / forced-alignment exploration followed. Forced alignment addresses where expected units occur in audio, while recognition estimates the unit sequence itself. No HMM performance metric is supplied in the public facts.

## MFCC + BiLSTM + CTC

The earlier neural recognition approach combined MFCC features, bidirectional LSTM sequence modeling, a neural classifier, and Connectionist Temporal Classification (CTC). CTC permits sequence supervision without assigning an exact frame interval to every target phoneme.

The documented 100-epoch experiment reported 59.7% training accuracy and 51.8% validation accuracy. A test value and exact accuracy definition are not supplied.

## Wav2Vec2 + Conformer + CTC

The later recognition direction used Wav2Vec2-style features followed by a Conformer encoder, classifier, and CTC. A documented architecture used **4 Conformer blocks**. Conformer combines attention-based context modeling with convolutional modeling of local structure.

The documented 100-epoch experiment reported 90.3% training, 80.8% validation, and 78.8% test accuracy. These are experiment-specific reported values; they do not establish the isolated effect of replacing any single component.

## CTC decoding

CTC models paths containing phoneme labels and a blank symbol. A conceptual decoding procedure merges consecutive repeated labels in a path and then removes blanks to form a phoneme sequence. Blanks can separate two occurrences of the same label; they are not themselves phonemes or guaranteed silence regions.

Sequence decoding does not by itself establish reliable acoustic start and end times. No beam-search configuration, language model, or production decoding API is specified here. See [phoneme recognition](results/phoneme-recognition.md).

## Count Head

The Count Head estimates approximate phoneme count. Count information concerns sequence length and cannot determine phoneme identities or exact boundaries on its own. Its private implementation, loss formulation, and quantitative count-error evaluation are not included.

## Segmentation Head

The Segmentation Head learns phoneme temporal boundaries. The documented direction uses a segmentation-specific embedding with approximate dimensional reduction **512 → 256 → 64**.

This representation supports boundary estimation separately from the CTC recognition objective. Exact boundary-label construction, loss weights, frame rate, and decoding details are not specified in the public facts.

## Separating recognition from segmentation

Recognition estimates which phonemes were spoken and in what order. Segmentation estimates their temporal regions. The final direction treats these as distinct objectives over encoded speech features, allowing phoneme identities and boundary estimates to inform duration estimation.

The public materials do not establish how every predicted label is paired with every predicted region or demonstrate a fully validated end-to-end Tajweed correction system.

## Duration estimation

Given a phoneme start time and end time in a common coordinate system:

```text
duration_seconds = end_seconds - start_seconds
```

Converting frame boundaries into time requires the actual frame stride, indexing convention, and any preprocessing offsets. These are not supplied, so no project-specific conversion constant is asserted. Boundary uncertainty affects duration uncertainty; phoneme identity alone cannot establish duration correctness.

Madd and Ghunnah motivate duration-sensitive analysis, but no universal duration threshold or complete rule-specific decision procedure is provided. See the [illustrative segmentation example](results/segmentation-example.md).

## Experiment methodology and reporting

The report documents approximately 100 hours of audio divided into 80 hours for training, 10 for validation, and 10 for testing. Different reciters were included across subsets, and the test set included different verses and different voices. Exact speaker-overlap counts and corpus manifests are not supplied.

The two segmentation-related experiments report train/validation/test accuracy of 90.1% / 78.7% / 69.05% and 90.1% / 77.7% / 70.05%, respectively. Their exact metric units, configurations, and epoch counts are not specified here.

All supplied numbers must remain labeled as experiment-specific reported accuracy. They are not PER, CER, boundary error, duration error, or universal Tajweed accuracy. A stronger future evaluation would report PER, boundary error in milliseconds, duration-error metrics, confidence estimates, and detailed split and scoring definitions. The excluded data and implementation limit independent reproduction from this repository.
