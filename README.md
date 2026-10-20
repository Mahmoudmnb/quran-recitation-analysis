# Quran Recitation Analysis

An **Audio AI system for Quran recitation analysis** using phoneme recognition, boundary detection, duration estimation, CTC, Wav2Vec2, and Conformer.

The project investigates how speech processing and deep learning can analyze Quran recitation at the **phoneme level**, going beyond conventional speech-to-text toward temporal analysis of pronunciation and selected Tajweed-related features.

<p align="center">
  <strong>
    Audio AI • Speech Processing • Phoneme Recognition • Wav2Vec2 • Conformer • CTC • Tajweed Analysis
  </strong>
</p>

> **Public showcase repository**
>
> This repository presents the research methodology, architecture, model evolution, selected experimental results, public data resources, limitations, and illustrative examples of a team graduation project.
>
> Full training and inference code, private preprocessing and experiment code, model checkpoints, and private project assets are intentionally excluded.

---

## 🎯 Project Overview

Correct Quran recitation involves more than recognizing which words were spoken.

Some pronunciation and Tajweed-related characteristics depend on:

- the phoneme that was pronounced
- the order of phonemes
- the temporal location of each phoneme
- its beginning and ending boundaries
- its duration
- differences between expected and observed pronunciation

The project therefore investigates **phoneme-level Quran recitation analysis** with three main technical objectives:

1. **Phoneme Recognition** — identify the spoken phoneme sequence.
2. **Phoneme Boundary Detection** — estimate where individual phonemes begin and end.
3. **Phoneme Duration Estimation** — calculate temporal duration for further analysis.

These outputs provide a foundation for analyzing selected pronunciation and Tajweed-related phenomena, especially duration-sensitive cases such as **Madd** and **Ghunnah**.

The intended role is **assistive education**. The system does not replace a qualified Quran teacher.

---

## 🖼️ System Overview

<p align="center">
  <img
    src="docs/architecture/system-overview.png"
    alt="Quran recitation analysis system overview"
    width="100%">
</p>

At a high level, the project follows this pipeline:

```text
Quran Recitation Audio
        │
        ▼
Audio Processing / Augmentation
        │
        ▼
Acoustic Representation
        │
        ▼
Phoneme Recognition
        │
        ▼
Boundary Detection
        │
        ▼
Duration Estimation
        │
        ▼
Selected Tajweed Analysis
```

The system aims to combine:

```text
What was pronounced?
        +
When was it pronounced?
        +
How long was it pronounced?
```

---

## 🧠 Why Phoneme-Level Analysis?

Ordinary automatic speech recognition primarily answers questions such as:

> **What words were spoken?**

For detailed Quran recitation analysis, that is not enough.

The project also needs to approach questions such as:

> **Which phonemes were spoken?**

> **Where did each phoneme begin and end?**

> **How long did each phoneme last?**

Word-level timestamps do not necessarily provide the temporal precision needed for phoneme-duration analysis.

This distinction became important throughout the project and influenced the progression from general speech recognition toward dedicated phoneme recognition and segmentation.

---

## 🧬 Quran-Aware Phoneme Mapping

A central design decision in this project was to align the audio with **phonemes rather than directly with written Arabic characters**.

The written form of Quranic Arabic does not always correspond one-to-one with the sound actually produced during recitation.

Depending on context, some written elements may:

- not be pronounced independently
- change pronunciation when stopping or continuing
- merge into neighboring sounds
- acquire Ghunnah
- change because of Tajweed rules
- change their acoustic characteristics
- affect phoneme duration
- represent sounds differently from their written form

For this reason, the training target was designed to represent the **actual expected spoken sound sequence**, not simply the orthographic spelling.

The project developed a custom **Qur’an-aware phoneme mapper** that converts Quranic text into a machine-readable phonetic representation.

The representation uses Latin and phonetic symbols as **technical labels for Arabic sounds**.

It is not a translation of the Quranic text into English.

### Why This Matters

Instead of trying to learn:

```text
Audio Frames
     ↕
Written Arabic Characters
```

the acoustic model is trained toward:

```text
Audio Frames
     ↕
Expected Spoken Phonemes
```

This reduces the mismatch between Quranic orthography and the sound actually expected during recitation.

Tajweed rules are applied during the text-to-phoneme transformation so that the target sequence better represents the expected acoustic output.

Conceptually:

```text
Quranic Text
      │
      ▼
Quran-Aware Text-to-Phoneme Mapping
      │
      ▼
Tajweed-Aware Phoneme Sequence
      │
      ▼
Phoneme Vocabulary IDs
      │
      ▼
Target Sequence for Model Training
```

---

## 🔤 Basic Phoneme Examples

The project defines symbols for:

- consonants
- short vowels
- long vowels
- tanween
- shaddah-related behavior
- Tajweed-dependent pronunciation features

Examples documented during the project include:

### Tanween

```text
كِتَابٌ
→ ['ki', 'taa', 'bun']
```

### Madd / Long Vowel

```text
قَالَ
→ ['qaa', 'la']
```

### Shaddah

```text
عَلَّمَ
→ ['ʕa', 'l', 'la', 'ma']
```

These examples show that the representation describes the **pronounced sequence**, rather than simply assigning one model label to every written character.

---

## ☀️ Sun Letters vs. Moon Letters

The Arabic definite article `الـ` changes acoustically depending on the following consonant.

### Sun Letter

With a sun letter, the sound of `ل` is not pronounced independently and the following consonant is assimilated.

Example:

```text
الشَّمْس
→ ['<wasl>', 'ʃ', 'ʃa', 'm', 'sa']
```

The written `ل` exists orthographically, but its independent sound is absent from the expected recitation.

### Moon Letter

With a moon letter, the `ل` remains pronounced.

Example:

```text
القَمَر
→ ['<wasl>', 'l', 'qa', 'ma', 'r']
```

This illustrates why direct character-level alignment can create a mismatch between text and the acoustic signal.

---

## 🔄 Taa Marbuta Depends on Context

The written character `ة` can correspond to different sounds depending on whether the reader continues or stops.

### In Connected Speech

```text
رَحْمَةُ اللَّهِ
→ ['ra', 'ħ', 'ma', 'tu', 'a', 'l', 'laa', 'hi']
```

### When Stopping

```text
رَحْمَةْ
→ ['ra', 'ħ', 'ma', 'h']
```

The same written character therefore cannot always be assigned one fixed acoustic target.

---

## 🔗 Hamzat al-Wasl

Hamzat al-Wasl may be pronounced when starting from a word but omitted when the word is connected to preceding speech.

### Beginning from the Word

```text
ٱلْقَمَرُ
→ ['ʔa', 'l', 'qa', 'ma', 'ru']
```

### Connected Speech

```text
وَٱلْقَمَرُ
→ ['wa', 'l', 'qa', 'ma', 'ru']
```

The written form remains present, but the expected acoustic sequence changes.

---

## 🎵 Ghunnah

Ghunnah is represented in the phoneme mapping by extending or doubling the relevant nasal sound.

Conceptually:

```text
ن → nn
م → mm
```

This allows information related to nasal continuation to be retained in the training labels.

---

## 🔀 Noon Sakinah & Tanween

The expected phonetic representation changes depending on the following sound.

### Idgham with Ghunnah

Example:

```text
مِن نُورٍ
→ ['mi', 'nn', 'nuu', 'rin']
```

The noon is not represented as an ordinary independent `n`; the expected nasalized merged sound is encoded instead.

### Idgham without Ghunnah

Example:

```text
مِن رَبِّهِم
→ ['mi', 'r', 'ra', 'b', 'bi', 'h', 'm']
```

The noon is removed from the expected acoustic sequence and the following consonant reflects the assimilation.

### Iqlab

When noon sakinah or tanween occurs before `ب`, the expected sound changes toward a meem-like nasal realization.

Example:

```text
مِن بَعْدِ
→ ['mi', 'mm', 'ba', 'ʕ', 'di']
```

Conceptually:

```text
n → mm
```

under the corresponding Iqlab condition.

### Ikhfa

The representation can also encode the altered articulation produced by Ikhfa rather than retaining an ordinary `n`.

Examples documented in the project include:

```text
أَنْتُمْ
→ ['ʔa', 'nt', 'tu', 'm']
```

and:

```text
يَنْطِقُونَ
→ ['ya', 'ntˤ', 'tˤi', 'quu', 'na']
```

---

## 🔁 Idgham Between Consonants

The mapper also models cases where consonants merge during recitation.

### Similar Consonants

```text
قَد دَخَلُوا
→ ['qa', 'd', 'da', 'xa', 'luu']
```

### Assimilation into a Following Consonant

```text
قَد تَبَيَّنَ
→ ['qa', 't', 'ta', 'ba', 'y', 'ya', 'na']
```

Instead of preserving the untouched written sequence, the phoneme target represents the expected result of assimilation.

---

## 🔊 Emphatic Raa

The project distinguishes between a normal and emphatic realization of `ر`.

Conceptually:

```text
Normal:
رَ → ['ra']

Emphatic:
رَ → ['rMa']
```

A dedicated marker allows the model target to preserve a pronunciation characteristic not represented by a basic character label alone.

---

## ⚡ Qalqalah

Qalqalah is represented by attaching an additional marker to the affected consonant.

Examples:

```text
بْ → ['bK']
دْ → ['dK']
قْ → ['qK']
```

The `K` marker indicates Qalqalah behavior associated with the consonant rather than introducing an unrelated standalone sound.

---

## 🔢 Phoneme Vocabulary

After creating Quran-aware phoneme sequences, all possible labels are collected into a **Phoneme Vocabulary**.

Each label receives a numerical ID that can be used by the neural network.

The documented project vocabulary contained approximately:

```text
379 labels
```

The model therefore operates conceptually on:

```text
Audio
   │
   ▼
Acoustic Encoder
   │
   ▼
Classifier
   │
   ▼
Probability Distribution over 379 Labels
   │
   ▼
CTC Decoding
   │
   ▼
Predicted Phoneme Sequence
```

This Quran-aware target representation is one of the key components of the project because it reduces the gap between **how Quranic text is written** and **how it is expected to sound during recitation**.

---

## 🏗️ Final Model Architecture

<p align="center">
  <img
    src="docs/architecture/final-model.png"
    alt="Final Quran recitation analysis model architecture"
    width="100%">
</p>

The final research direction separates **phoneme recognition** from **temporal segmentation**.

```text
                    Quran Recitation Audio
                             │
                             ▼
                Audio Processing / Augmentation
                             │
                             ▼
                     Wav2Vec2 Features
                             │
                             ▼
                      Conformer Encoder
                             │
                 ┌───────────┴───────────┐
                 │                       │
                 ▼                       ▼
          Recognition Branch      Segmentation Branch
                 │                       │
                 ▼                       ▼
            Classifier              Count Head
                 │                 Segmentation Head
                 ▼                       │
                CTC                      ▼
                 │              Segmentation Classifier
                 ▼                       │
           CTC Decoding                  ▼
                 │                Phoneme Boundaries
                 ▼                       │
          Phoneme Sequence               │
                 └───────────┬───────────┘
                             ▼
                    Duration Estimation
                             │
                             ▼
               Selected Tajweed Analysis
```

A documented architecture used **4 consecutive Conformer blocks**.

---

## Recognition Branch

The recognition branch focuses on:

> **What phoneme was spoken?**

Its conceptual flow is:

```text
Wav2Vec2 Features
        │
        ▼
Conformer Encoder
        │
        ▼
Classifier
        │
        ▼
CTC
        │
        ▼
CTC Decoding
        │
        ▼
Phoneme Sequence
```

---

## Segmentation Branch

The segmentation branch focuses on:

> **When does each phoneme begin and end?**

The Segmentation Head learns a task-specific representation by reducing the encoded feature size approximately as:

```text
512 → 256 → 64
```

Conceptually:

```text
Conformer Features
        │
        ▼
Segmentation Head
        │
        ▼
Segmentation Classifier
        │
        ▼
Boundary Detection
        │
        ▼
Start / End Frames
        │
        ▼
Phoneme Duration
```

Separating the two tasks allows recognition and temporal analysis to focus on different objectives.

---

## 🔬 Model Evolution

<p align="center">
  <img
    src="docs/architecture/model-evolution.png"
    alt="Quran recitation analysis model evolution"
    width="100%">
</p>

The final architecture was reached through several experimental stages.

| Stage | Approach | Purpose / Observation |
|---|---|---|
| 1 | Whisper | Explored general ASR and word timestamps |
| 2 | Signal Processing + HMM | Explored preprocessing and forced alignment |
| 3 | MFCC + BiLSTM + CTC | Learned phoneme sequence recognition |
| 4 | Wav2Vec2 + Conformer + CTC | Improved acoustic and contextual modeling |
| 5 | Count Head + Segmentation Head | Separated recognition from temporal boundary estimation |

---

## 1️⃣ Whisper Experiment

The project initially explored **Whisper** for speech recognition and word-level timestamps.

The intended workflow was approximately:

```text
Recitation
   │
   ▼
Whisper
   │
   ▼
Recognized Words
   │
   ▼
Word Timestamps
   │
   ▼
Extract Audio Segments
   │
   ▼
Pronunciation Analysis
```

Whisper was useful for general speech recognition, but experiments showed that its temporal estimates were insufficient for the precision required by phoneme-level analysis.

Observed issues included:

- approximate timestamps
- overlap between neighboring word segments
- insufficient precision for short phonemes
- difficulty obtaining reliable phoneme durations

This demonstrated an important distinction:

> **Speech recognition and precise temporal alignment are different tasks.**

---

## 2️⃣ Traditional Signal Processing & HMM Exploration

The project then explored a more controlled audio-analysis pipeline.

Techniques investigated included:

- High-Pass Filtering
- Band-Pass Filtering
- Noise-profile estimation
- Noise reduction
- Energy-based silence removal
- MFCC
- Delta MFCC
- Delta-Delta MFCC
- CMVN
- Hidden Markov Models
- Forced Alignment

The HMM direction aimed to align known Quranic content with the audio signal so temporal locations could be estimated more directly.

The emphasis shifted from:

```text
What was spoken?
```

toward:

```text
When was each known phonetic unit spoken?
```

---

## 3️⃣ MFCC + BiLSTM + CTC

The project later moved toward deep learning for phoneme recognition.

```text
Audio
   │
   ▼
Audio Augmentation
   │
   ▼
MFCC / CMVN
   │
   ▼
BiLSTM
   │
   ▼
Neural Classifier
   │
   ▼
CTC
   │
   ▼
CTC Decoding
   │
   ▼
Phoneme Sequence
```

A documented experiment trained for **100 epochs** and reported:

| Metric | Accuracy |
|---|---:|
| Training | 59.7% |
| Validation | 51.8% |

These values are specific to that experiment and are not universal measures of Quran-recitation quality.

---

## 4️⃣ Wav2Vec2 + Conformer + CTC

The BiLSTM-based sequence model was later replaced with a **Conformer encoder**.

The Conformer combines:

- self-attention for long-range context
- convolution for local acoustic patterns

The documented architecture used:

```text
Conformer Block 1
      ↓
Conformer Block 2
      ↓
Conformer Block 3
      ↓
Conformer Block 4
```

The recognition pipeline became:

```text
Audio
   │
   ▼
Wav2Vec2
   │
   ▼
Conformer
   │
   ▼
Classifier
   │
   ▼
CTC
   │
   ▼
CTC Decoding
   │
   ▼
Phoneme Sequence
```

A documented experiment trained for **100 epochs** and reported:

| Split | Accuracy |
|---|---:|
| Training | 90.3% |
| Validation | 80.8% |
| Test | 78.8% |

---

## 5️⃣ Count Head & Segmentation Head

CTC was effective for predicting phoneme sequences, but sequence recognition alone did not reliably determine exact temporal boundaries.

This motivated dedicated temporal-analysis components.

### Count Head

The Count Head estimates the approximate number of phonemes in an audio segment.

It uses encoded information such as:

- mean representation
- standard deviation
- sequence length

Conceptually:

```text
Encoded Audio Features
        │
        ├── Mean
        ├── Standard Deviation
        └── Sequence Length
                │
                ▼
            Count Head
                │
                ▼
      Estimated Phoneme Count
```

### Segmentation Head

The Segmentation Head focuses specifically on temporal boundary estimation.

The encoded features are transformed approximately as:

```text
512 → 256 → 64
```

An illustrative segmentation might look like:

```text
Frames 1–20   → phoneme A
Frames 21–45  → phoneme B
Frames 46–65  → phoneme C
Frames 66–80  → phoneme D
```

From these predicted boundaries:

```text
Start Frame + End Frame
           │
           ▼
     Phoneme Duration
```

---

## 🔊 Audio Processing

Audio preprocessing was explored extensively during the project.

### Frequency Filtering

Experiments included:

- High-Pass Filtering
- Band-Pass Filtering

One preprocessing experiment investigated an approximate frequency range of:

```text
300 Hz – 4500 Hz
```

to emphasize useful speech information while reducing unwanted low- and high-frequency components.

### Noise Reduction

A noise profile was estimated from an initial quiet portion of recordings and used for background-noise reduction.

Experiments showed that overly aggressive noise reduction could affect useful acoustic characteristics, so preserving the speech signal remained important.

### Silence Removal

Energy-based silence detection was explored for removing low-energy regions from the beginning and end of recordings.

### Acoustic Features

Earlier experiments used:

- MFCC
- Delta MFCC
- Delta-Delta MFCC
- CMVN

Later experiments moved toward learned Wav2Vec2-style speech representations.

---

# 📚 Datasets

The project used approximately **100 hours of Quran recitation audio**.

| Subset | Approximate Duration |
|---|---:|
| Training | 80 hours |
| Validation | 10 hours |
| Test | 10 hours |

The experimental design included different reciters across subsets.

The test configuration also included different verses and different voices, helping investigate generalization beyond the training speakers.

---

## 🎙️ Audio Dataset

The Quran recitation audio used by the project was sourced from the public Kaggle dataset:

**Quran Reciters**

Kaggle dataset identifier:

```text
omartariq612/quran-reciters
```

It can be downloaded using `kagglehub`:

```python
import kagglehub

path = kagglehub.dataset_download(
    "omartariq612/quran-reciters"
)

print("Path to dataset files:", path)
```

The source audio dataset is maintained by its respective publisher.

This repository does not claim ownership of the source recordings.

Users should review the corresponding Kaggle dataset page, license, and redistribution terms before using or redistributing the audio.

---

## 📑 Quran Phoneme / Metadata CSV Dataset

As part of the project, a derived CSV dataset was created to connect the audio samples with the structured information required by the training pipeline.

The dataset includes the **Quran-aware phoneme representation** associated with the Arabic text used by the project.

Kaggle dataset identifier:

```text
mohammeddeebjalab1/quran-ds-csv-files
```

It can be downloaded using:

```python
import kagglehub

path = kagglehub.dataset_download(
    "mohammeddeebjalab1/quran-ds-csv-files"
)

print("Path to dataset files:", path)
```

This dataset is intended to connect:

```text
Audio Sample
      │
      ▼
Quranic Text / Metadata
      │
      ▼
Quran-Aware Phoneme Representation
      │
      ▼
Model Target Sequence
```

The CSV dataset and source audio should be used according to the terms and licenses applicable to both resources.

---

## 📈 Experimental Results

The following values are selected reported results from specific project experiments.

| Experiment | Train | Validation | Test | Epochs |
|---|---:|---:|---:|---:|
| MFCC + BiLSTM + CTC | 59.7% | 51.8% | Not reported | 100 |
| Wav2Vec2 + Conformer + CTC | 90.3% | 80.8% | 78.8% | 100 |
| Segmentation-related experiment A | 90.1% | 78.7% | 69.05% | 100 |
| Segmentation-related experiment B | 90.1% | 77.7% | 70.05% | 100 |

> **Metric interpretation**
>
> These are experiment-specific reported accuracy values.
>
> They are **not**:
>
> - universal Tajweed accuracy
> - Phoneme Error Rate (PER)
> - Character Error Rate (CER)
> - production guarantees
>
> Exact scoring definitions and full experiment protocols are not distributed through this public showcase, so these experiments should not be treated as a controlled public benchmark.

Additional discussion is available under:

```text
docs/results/
```

---

## 🧩 Recognition vs. Segmentation

One of the project's most important conclusions was that **recognition and alignment are different problems**.

### Recognition

CTC attempts to determine:

```text
What sequence of phonemes was spoken?
```

### Segmentation

The segmentation branch attempts to determine:

```text
When does each phoneme begin and end?
```

A correct phoneme sequence does not guarantee a correct temporal distribution across audio frames.

For duration-sensitive analysis, both outputs are needed.

---

## ⚠️ Key Challenges

### ASR vs. Alignment

Correctly recognizing linguistic content does not guarantee precise acoustic boundaries.

### CTC vs. Exact Timing

CTC is useful for sequence recognition without requiring exact frame-level labels.

However, the resulting sequence alone does not reliably provide true phoneme start/end boundaries.

### Speaker Generalization

Different speakers vary in:

- voice characteristics
- pronunciation
- speed
- pitch
- recording environment
- recitation style

Generalizing across unseen reciters therefore remains challenging.

### Overfitting

Some experiments showed a noticeable gap between training and unseen-speaker evaluation.

This motivates further work on:

- speaker diversity
- augmentation
- regularization
- larger evaluation datasets

### Temporal Modeling

Errors in boundary prediction directly affect duration estimation.

This is especially important for analyses involving duration-sensitive phenomena such as Madd and Ghunnah.

---

## 🎧 Illustrative Output Format

The public repository contains generic examples showing how future analysis results could be represented.

For example:

```json
{
  "source": "illustrative_example",
  "phonemes": [
    {
      "label": "phoneme_1",
      "start_seconds": 0.0,
      "end_seconds": 0.2,
      "duration_seconds": 0.2
    },
    {
      "label": "phoneme_2",
      "start_seconds": 0.2,
      "end_seconds": 0.48,
      "duration_seconds": 0.28
    }
  ],
  "tajweed_analysis": {
    "status": "illustrative_only"
  }
}
```

These examples demonstrate data structure only.

They are **not real model predictions**.

See:

```text
examples/
```

---

## 📂 Repository Structure

```text
quran-recitation-analysis/
├── README.md
├── LICENSE
├── CITATION.cff
├── .gitignore
│
├── docs/
│   ├── architecture/
│   │   ├── README.md
│   │   ├── system-overview.png
│   │   ├── model-evolution.png
│   │   └── final-model.png
│   │
│   ├── results/
│   │   ├── README.md
│   │   ├── experiment-summary.md
│   │   ├── phoneme-recognition.md
│   │   └── segmentation-example.md
│   │
│   ├── methodology.md
│   └── limitations.md
│
├── examples/
│   ├── README.md
│   ├── sample-phoneme-sequence.json
│   └── sample-analysis.json
│
├── assets/
│   └── audio/
│       └── README.md
│
├── references/
│   └── README.md
│
└── PROJECT_NOTES.md
```

---

## 🔒 Repository Scope

This repository is intentionally a **technical showcase**, not the complete original project repository.

### Publicly Included

- project methodology
- system architecture
- model evolution
- selected experimental results
- architecture diagrams
- generic illustrative examples
- technical limitations
- academic attribution
- links to public datasets

### Publicly Available Externally

Large datasets are not duplicated inside this Git repository.

Public Kaggle resources used by the project include:

```text
Audio:
omartariq612/quran-reciters
```

```text
Derived phoneme / metadata CSV files:
mohammeddeebjalab1/quran-ds-csv-files
```

Keeping these resources on Kaggle avoids placing large datasets directly in Git while preserving a clearer path toward reproducing the data workflow.

### Intentionally Not Included

- full training source code
- full inference source code
- private preprocessing code
- private experiment code
- trained model checkpoints
- dataset copies inside this repository
- restricted or non-public audio
- private configuration
- production assets

The public/private separation is intentional.

---

## 🛠️ Technologies

### Deep Learning

- Python
- PyTorch
- Wav2Vec2
- Conformer
- BiLSTM
- CTC Loss
- CTC Decoding

### Speech & Audio Processing

- Phoneme Recognition
- Phoneme Boundary Detection
- Phoneme Duration Estimation
- Quran-Aware Phoneme Mapping
- Speech Recognition
- Audio Augmentation
- MFCC
- Delta MFCC
- Delta-Delta MFCC
- CMVN
- Noise Reduction
- Energy-Based Silence Detection

### Experimental Methods

- Whisper
- Hidden Markov Models
- Forced Alignment
- Neural Sequence Modeling
- Count Head
- Segmentation Head

---

## 🚧 Limitations

This project should be understood as a **research and graduation-project prototype**.

Current limitations include:

- generalization to unseen speakers
- limited dataset diversity
- sensitivity to recording conditions
- phoneme-boundary estimation difficulty
- limited public evaluation data
- no fully reproducible public training pipeline
- not every Tajweed rule is modeled end-to-end
- experimental rather than authoritative educational feedback

No claim of complete automatic Tajweed correction is made.

The system should **not** be considered a replacement for a qualified Quran teacher.

See:

```text
docs/limitations.md
```

for additional details.

---

## 🔮 Future Work

Potential research directions include:

- larger and more diverse speaker datasets
- stronger speaker-independent evaluation
- improved phoneme-boundary supervision
- better duration modeling
- improved forced alignment
- stronger audio augmentation
- speaker normalization
- Phoneme Error Rate (PER) evaluation
- boundary-error metrics in milliseconds
- duration-error metrics
- confidence estimation
- rule-specific Tajweed analysis
- interactive visual phoneme alignment
- real-time inference
- mobile integration
- personalized learner feedback
- further development of the Quran-aware phoneme representation

These are research directions and are not features currently delivered by this repository.

---

## 🎓 Academic Context

This work was developed as a **fifth-year graduation project** at:

**Aleppo University**  
Faculty of Informatics Engineering  
Department of Artificial Intelligence and Natural Languages

### Project Team

- Mahmoud Bannan
- Mohammed Deeb Jalab
- Mohanad Al-Abd Allah
- Amjad Haj Hammada
- Mahmoud Abd Al-Rhim
- Hayder Qasem

### Supervisor

**Dr. Mohammed Fadel Sukkar**

The complete team is also credited in [`CITATION.cff`](CITATION.cff).

This public showcase repository is maintained by **Mahmoud Bannan**.

---

## ⚖️ Responsible Use

Quran recitation assessment is a specialized domain involving linguistic, phonetic, pedagogical, and religious expertise.

AI predictions can be inaccurate.

This project explores technology intended to **assist** recitation learning and analysis.

It should not be used as the sole authority for Quran recitation correction.

For authoritative instruction and correction, users should consult qualified Quran teachers and specialists.

Users of the public datasets should also respect the applicable licenses, attribution requirements, and redistribution terms of the original dataset publishers.

---

## 👨‍💻 Maintainer

**Mahmoud Bannan**

Flutter Developer & Audio AI Engineer

- GitHub: [@Mahmoudmnb](https://github.com/Mahmoudmnb)
- Portfolio: [mahmoudbannan.com](https://mahmoudbannan.com)
- LinkedIn: [Mahmoud Bannan](https://www.linkedin.com/in/mahmoudbannan)

---

## 📄 License

See [`LICENSE`](LICENSE) for the repository's documentation-focused license notice.

The repository license does **not** automatically grant rights to:

- private source code
- private experiment code
- model checkpoints
- third-party audio
- Quran recitation recordings
- third-party copyrighted material
- materials not contained in this repository

Public datasets linked from this repository remain subject to their own licenses and terms.

Any reuse must also respect the rights of the original project contributors and relevant third-party rights holders.