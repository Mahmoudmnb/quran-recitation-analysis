# Experiment summary

The following values are reported for specific experiments documented in the project. They are reproduced from the supplied project facts without adding new metrics.

| Experiment | Training accuracy | Validation accuracy | Test accuracy | Epochs |
|---|---:|---:|---:|---|
| MFCC + BiLSTM + CTC | 59.7% | 51.8% | Not reported | 100 |
| Wav2Vec2 + Conformer + CTC | 90.3% | 80.8% | 78.8% | 100 |
| Segmentation-related experiment A | 90.1% | 78.7% | 69.05% | Not specified |
| Segmentation-related experiment B | 90.1% | 77.7% | 70.05% | Not specified |

## What the values mean

These are **experiment-specific reported accuracy values**. The supplied facts do not define each score's unit, denominator, aggregation procedure, or matching tolerance. In particular, the segmentation-related scores must not be relabeled as boundary precision, boundary recall, or duration accuracy.

The values are **not directly comparable to Phoneme Error Rate (PER) or Character Error Rate (CER)**. Neither error rate can be inferred by subtracting these accuracy values from 100%. They are also **not universal Tajweed accuracy or production guarantees**.

## Interpretation and comparability

The Conformer-related run reports higher recognition accuracy than the BiLSTM run. Complete configurations and scoring definitions are not supplied, so the table does not establish a controlled comparison or attribute the difference to a single architectural change.

Training-to-evaluation gaps motivate further study of overfitting, speaker dependence, and recording variation. The design included different reciters across subsets and different verses and voices in the test set; the values do not guarantee performance on arbitrary reciters.

Experiments A and B are identifiers for the two supplied segmentation-related result sets. They do not imply a documented chronological order, known hyperparameter difference, or universal ranking. No epoch counts are supplied for them. Missing values are not zeros.

## Evaluation context and missing evidence

The report describes approximately 100 hours of audio with an 80/10/10-hour training/validation/test split. This showcase excludes the dataset, weights, private implementation, and full evaluation protocol, limiting independent reproduction.

Future reporting should specify metric definitions, split construction, PER, boundary error in milliseconds, duration-error metrics, and uncertainty. Those measurements are proposed work and are not available results in this repository.
