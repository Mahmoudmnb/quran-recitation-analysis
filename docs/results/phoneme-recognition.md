# Phoneme recognition with CTC

The recognition direction maps audio representations through a sequence encoder and classifier to phoneme-label scores, including a CTC blank symbol. CTC supports learning an output sequence without requiring an exact frame-level alignment for every target label.

## Conceptual decoding

The following is a generic illustrative path, not a recorded prediction or a Quranic phoneme sequence:

```text
Frame-level path:           blank A A blank A B B blank
Merge consecutive repeats: blank A   blank A B   blank
Remove blanks:                   A         A B
Decoded sequence:          A A B
```

The operation order matters: merge consecutive repeated labels before removing blanks. A blank separating two A labels preserves two occurrences. A blank is an alignment symbol; it does not necessarily indicate acoustic silence.

This describes the basic collapse operation on a path. It does not claim that the private project used a particular greedy decoder, beam search, language model, or public API.

## Why recognition is insufficient

The decoded sequence identifies phoneme order but loses the detailed path structure. Frame-level label emissions also do not establish precise acoustic starts and ends. Correct labels can therefore coexist with inaccurate timing.

The final project direction separates CTC recognition from segmentation-based boundary estimation. Boundaries support duration estimation and selected duration-sensitive analysis, especially for Madd and Ghunnah. Neither a recognized sequence nor the reported recognition accuracy establishes full Tajweed correctness.

See the [experiment summary](experiment-summary.md) for supplied scores and the [segmentation example](segmentation-example.md) for a conceptual timing demonstration. The [JSON sequence example](../../examples/sample-phoneme-sequence.json) uses generic labels with unspecified confidence.
