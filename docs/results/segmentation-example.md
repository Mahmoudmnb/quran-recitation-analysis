# Illustrative segmentation example

**Conceptual example only.** Every label, frame interval, and timestamp on this page is illustrative. These are not measured model predictions, Quranic phonemes, or experiment results.

| Illustrative frame region | Generic label |
|---|---|
| Frames 1–20 | Phoneme A |
| Frames 21–45 | Phoneme B |
| Frames 46–65 | Phoneme C |

The regions show the idea of assigning a temporal span to each unit. They do not specify the project's frame rate, frame indexing convention, or actual boundary precision.

## From boundaries to duration

Once start and end times share the same time coordinate system:

```text
duration_seconds = end_seconds - start_seconds
```

The separate [illustrative JSON example](../../examples/sample-analysis.json) gives a generic unit a start of 0.0 seconds and an end of 0.2 seconds, producing a duration of 0.2 seconds. This synthetic format example is not a conversion of the frame table above.

Actual conversion from frame regions to seconds requires the encoder's temporal stride, a consistent boundary convention, and any preprocessing offsets. These details are not supplied here. Accurate phoneme identity does not guarantee accurate timing, and boundary errors propagate into durations.

Duration estimates can support selected Madd and Ghunnah analysis, but this example establishes no recitation assessment or rule-specific duration threshold.
