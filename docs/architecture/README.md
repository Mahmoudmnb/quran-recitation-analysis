# Architecture guide

The [main README](../../README.md#system-architecture) contains a text overview of the documented project direction. No PNG files are included yet. Add the following visuals manually only after checking their technical accuracy and publication rights.

| Intended filename | Intended content |
|---|---|
| `system-overview.png` | Audio processing / augmentation, Wav2Vec2 features, Conformer encoding, separate CTC and segmentation branches, duration estimation, and selected Tajweed analysis. |
| `model-evolution.png` | Whisper exploration; traditional signal processing and HMM / forced alignment; MFCC + BiLSTM + CTC; Wav2Vec2 + Conformer + CTC; Count Head and Segmentation Head. |
| `final-model.png` | The documented 4-block Conformer architecture, CTC recognition, approximate phoneme Count Head, and segmentation embedding reduction of approximately 512 → 256 → 64 for boundary estimation. |

The reserved repository paths are:

```text
docs/architecture/system-overview.png
docs/architecture/model-evolution.png
docs/architecture/final-model.png
```

All eventual visuals must be derived from the documented project architecture. Do not imply undocumented layer configurations, head connections, loss weights, or additional measured results. The Count Head's placement and detailed connections should be verified against the graduation report before drawing them.

No empty or fabricated PNG placeholders have been created. Once real, approved images are added at these exact paths, add working Markdown image references to the main README. Remove private implementation details, local paths, and restricted material before publication.
