# Quran Recitation Analysis

An **Audio AI system for Quran recitation analysis** focused on phoneme recognition, temporal boundary detection, phoneme-duration estimation, and the analysis of selected pronunciation and Tajweed-related features.

The project explores how modern speech-processing and deep-learning techniques can be used to analyze Quran recitation at a level finer than conventional speech-to-text.

<p align="center">
  <strong>Audio AI • Speech Processing • Phoneme Recognition • Conformer • Wav2Vec2 • CTC • Tajweed Analysis</strong>
</p>

> **Public showcase repository**
>
> This repository presents the architecture, methodology, experiments, selected results, and technical evolution of the project.
>
> The complete training/inference source code, trained model checkpoints, and non-public datasets are intentionally not included.

---

## 🎯 Project Overview

Correct Quran recitation involves more than recognizing which words were spoken.

Some pronunciation and Tajweed-related characteristics depend on:

- the phoneme that was pronounced
- the order of phonemes
- the temporal location of each phoneme
- its beginning and ending boundaries
- its duration
- acoustic differences between expected and observed pronunciation

The goal of this project was therefore to move beyond ordinary speech-to-text and analyze recitation at the **phoneme level**.

The system was designed toward the following workflow:

```text
Quran Recitation
       │
       ▼
Audio Processing
       │
       ▼
Acoustic Representation
       │
       ▼
Phoneme Recognition
       │
       ▼
Phoneme Boundary Detection
       │
       ▼
Duration Estimation
       │
       ▼
Expected vs. Observed Analysis
       │
       ▼
Pronunciation / Tajweed Feedback
```

The project particularly investigates rules and phenomena for which temporal information is important, including **Madd** and **Ghunnah**.

---

## 🧠 Why Phoneme-Level Analysis?

A normal automatic speech recognition system can answer:

> **What was spoken?**

For this project, that is not enough.

The system also needs to approach questions such as:

> **Which phonemes were pronounced?**

> **Where did each phoneme start and end?**

> **How long did each phoneme last?**

These details are important when evaluating recitation phenomena whose correctness depends partly on temporal duration.

For example, identifying the word alone does not tell us whether a duration-sensitive sound was held for an appropriate amount of time.

---

## 🏗️ Final System Architecture

The final direction separates **phoneme recognition** from **phoneme segmentation** because the two tasks have different objectives.

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
              CTC Branch           Segmentation Branch
                   │                       │
                   ▼                       ▼
          Phoneme Recognition      Boundary Detection
                   │                       │
                   ▼                       ▼
            Phoneme Sequence       Start / End Frames
                   │                       │
                   └───────────┬───────────┘
                               ▼
                    Phoneme Duration Estimation
                               │
                               ▼
                Recitation / Tajweed Analysis
```

The architecture uses a shared speech representation and then separates the recognition and timing objectives.

### Recognition Branch

The CTC branch is responsible for predicting the sequence of spoken phonemes.

Typical flow:

```text
Audio
  ↓
Wav2Vec2
  ↓
Conformer
  ↓
Classifier
  ↓
CTC
  ↓
CTC Decoding
  ↓
Phoneme Sequence
```

### Segmentation Branch

The segmentation branch focuses on temporal information:

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
Phoneme Boundaries
       │
       ▼
Duration Estimation
```

Separating these tasks allows the recognition model to focus on **what phoneme was spoken**, while the segmentation model focuses on **when it was spoken**.

---

## 🔬 Project Evolution

An important part of this project was not only the final model, but the sequence of experiments that led to it.

### 1. Whisper Experiment

The project initially explored using **Whisper** for speech recognition and word-level timestamps.

The idea was approximately:

```text
Recitation
   ↓
Whisper
   ↓
Recognized Words
   ↓
Word Timestamps
   ↓
Extract Individual Words
   ↓
Analyze Pronunciation
```

Whisper was useful for general speech recognition, but experiments showed that its timing estimates were not precise enough for the phoneme-level temporal analysis required by this project.

Observed problems included:

- approximate rather than precise acoustic boundaries
- overlap between neighboring word segments
- insufficient temporal precision for short phonemes
- inability to directly measure fine-grained phoneme durations

This led to the conclusion that **speech recognition and precise forced alignment are different problems**.

---

### 2. Traditional Audio Processing & HMM Exploration

A more controlled audio-processing pipeline was then explored.

The preprocessing experiments included techniques such as:

- High-Pass Filtering
- Band-Pass Filtering
- Noise-profile estimation
- Noise reduction
- Energy-based silence removal
- MFCC extraction
- Delta MFCC
- Delta-Delta MFCC
- CMVN

A traditional HMM-based forced-alignment direction was also investigated.

The purpose was not simply to determine *what* had been recited, but to determine *when* known textual/phonetic units occurred in the signal.

---

### 3. MFCC + BiLSTM + CTC

The deep-learning experiments then moved toward phoneme recognition using:

```text
Audio
   ↓
Audio Augmentation
   ↓
MFCC / CMVN
   ↓
BiLSTM
   ↓
Neural Classifier
   ↓
CTC
   ↓
CTC Decoding
   ↓
Phoneme Sequence
```

CTC was useful because it allows sequence models to learn audio-to-phoneme mappings without requiring an exact frame-level alignment for every target phoneme.

One documented BiLSTM experiment after 100 epochs reported:

| Metric | Accuracy |
|---|---:|
| Training | 59.7% |
| Validation | 51.8% |

These values represent the reported accuracy for that specific experiment and should not be interpreted as universal Quran-recitation or Tajweed accuracy.

---

### 4. Conformer-Based Recognition

The BiLSTM sequence model was later replaced with a **Conformer encoder**.

The motivation was to combine:

- **self-attention** for long-range temporal context
- **convolution** for local acoustic patterns

The implemented architecture used multiple consecutive Conformer blocks.

The resulting pipeline became:

```text
Audio
   ↓
Wav2Vec2
   ↓
Conformer
   ↓
Classifier
   ↓
CTC Decoding
   ↓
Phoneme Sequence
```

A documented 100-epoch experiment reported:

| Split | Accuracy |
|---|---:|
| Training | 90.3% |
| Validation | 80.8% |
| Test | 78.8% |

These are experiment-specific reported values and are not presented as a general benchmark for all reciters or all Tajweed rules.

---

### 5. Phoneme Count & Segmentation Heads

Recognition alone still did not provide sufficiently precise phoneme timing.

Additional heads were therefore investigated.

#### Count Head

The Count Head was designed to estimate how many phonemes are present in an audio segment.

It uses information derived from the encoded sequence, including:

- mean representation
- standard deviation
- encoded sequence length

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

---

#### Segmentation Head

The Segmentation Head was designed specifically to learn transitions between phoneme regions.

Instead of using the full feature representation directly, a lower-dimensional segmentation-specific embedding is learned.

Conceptually:

```text
Conformer Features
     (B, T, 512)
          │
          ▼
       512 → 256
          │
          ▼
       256 → 64
          │
          ▼
Segmentation Representation
          │
          ▼
Boundary Classification
```

The objective is to determine transitions such as:

```text
Frames 1–20   → phoneme A
Frames 21–45  → phoneme B
Frames 46–65  → phoneme C
Frames 66–80  → phoneme D
```

From those boundaries:

```text
Start Frame + End Frame
           ↓
     Phoneme Duration
```

This temporal information is especially useful for duration-sensitive Tajweed analysis.

---

## 🔤 Phoneme Representation

A major part of the project involved converting Quranic text into a representation suitable for speech modeling.

The phoneme representation was designed to account for pronunciation behavior rather than treating Arabic text as a simple character sequence.

The project explored phonetic handling for areas including:

- الشمسية والقمرية
- التاء المربوطة
- الوقف
- همزة الوصل
- الغنة
- أحكام النون الساكنة والتنوين
- أحكام الميم الساكنة
- الإدغام
- الراء المفخمة
- القلقلة

The objective is to generate a phoneme sequence that better represents the expected recitation than ordinary text characters alone.

---

## 🔊 Audio Processing

Audio preprocessing was explored extensively during the project.

### Frequency Filtering

Experiments used High-Pass and Band-Pass filtering to reduce unwanted frequency components and emphasize useful speech information.

An experimentally selected frequency region of approximately:

```text
300 Hz – 4500 Hz
```

was investigated during one preprocessing stage.

### Noise Reduction

A noise profile was estimated from an initial silent portion of recordings and used for background-noise reduction.

The project found that overly aggressive noise removal could also affect useful recitation characteristics, so preserving the original acoustic structure remained important.

### Silence Removal

Energy-based silence detection was used to remove low-energy regions at the beginning and end of recordings while attempting to preserve internal recitation characteristics.

### Feature Extraction

Earlier experiments used:

- MFCC
- Delta MFCC
- Delta-Delta MFCC
- CMVN

Later architectures moved toward learned Wav2Vec2 representations.

---

## 📚 Dataset

The documented experiments used approximately **100 hours of Quran recitation audio**.

The project report describes the following experimental split:

| Split | Approx. Duration |
|---|---:|
| Training | 80 hours |
| Validation | 10 hours |
| Test | 10 hours |

The experimental design used different reciters across subsets, with the test configuration intended to evaluate performance on both different voices and different verses.

This was important because speaker generalization is a major challenge in speech recognition.

### Dataset Availability

The audio dataset is **not distributed through this repository**.

Reasons include:

- dataset ownership and licensing considerations
- audio redistribution rights
- repository size
- privacy / usage restrictions where applicable

This repository therefore documents the methodology without publishing the underlying training corpus.

---

## 📈 Selected Experimental Results

The following values come from specific experiments documented during project development.

| Experiment | Train | Validation | Test |
|---|---:|---:|---:|
| BiLSTM-based recognition | 59.7% | 51.8% | — |
| Conformer-based recognition | 90.3% | 80.8% | 78.8% |
| Segmentation-related experiment A | 90.1% | 78.7% | 69.05% |
| Segmentation-related experiment B | 90.1% | 77.7% | 70.05% |

> **Important:** These numbers are experiment-specific accuracy values from the project report. They should not be interpreted as a universal Tajweed-correction accuracy, phoneme-error-rate benchmark, or performance guarantee on arbitrary reciters.

The experiments also highlighted a meaningful generalization gap between training and unseen-speaker evaluation, indicating **overfitting / speaker-dependence** as an important limitation.

---

## ⚠️ Challenges & Lessons Learned

### Precise Timing Is Different from Speech Recognition

One of the project's most important findings was that recognizing the correct linguistic content does not guarantee accurate temporal boundaries.

For duration-sensitive recitation analysis, the system needs both.

---

### CTC Is Strong for Sequence Recognition, Not Exact Boundaries

CTC was useful for learning phoneme sequences without frame-level labels.

However, CTC output alone did not provide sufficiently reliable phoneme start/end boundaries for the intended temporal analysis.

This motivated the dedicated segmentation branch.

---

### Speaker Generalization Is Difficult

Speech models can learn characteristics associated with particular speakers, recording conditions, and recitation styles.

The gap between training and test performance showed the need for:

- greater speaker diversity
- stronger augmentation
- more varied recording conditions
- improved regularization
- larger evaluation sets

---

### Temporal Tajweed Analysis Requires Dedicated Modeling

Rules involving duration cannot be evaluated from text recognition alone.

A useful system must model:

```text
phoneme identity
        +
phoneme position
        +
phoneme duration
```

---

## 🎧 Example Analysis Format

A future public inference interface could expose results in a structured form such as:

```json
{
  "phonemes": [
    {
      "phoneme": "example_phoneme_1",
      "start": 0.42,
      "end": 0.63,
      "duration": 0.21
    },
    {
      "phoneme": "example_phoneme_2",
      "start": 0.63,
      "end": 0.97,
      "duration": 0.34
    }
  ]
}
```

This file is only an example of the intended public result representation. It is not a claim that the private production implementation exposes exactly this API.

---

## 📂 Public Repository Structure

```text
quran-recitation-analysis/
├── README.md
├── LICENSE
├── CITATION.cff
├── .gitignore
│
├── docs/
│   ├── architecture/
│   │   ├── system-overview.png
│   │   ├── model-evolution.png
│   │   └── final-model.png
│   │
│   ├── results/
│   │   ├── phoneme-recognition.png
│   │   ├── segmentation-example.png
│   │   └── experiment-summary.png
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
└── references/
    └── README.md
```

---

## 🔒 Repository Scope

This repository is intentionally a **technical showcase**, not the complete project source repository.

### Publicly Included

- Project methodology
- System architecture
- Model evolution
- Selected experimental results
- Architecture diagrams
- Non-sensitive examples
- Technical documentation
- Project limitations and lessons learned

### Intentionally Not Included

- Full training source code
- Full inference source code
- Private preprocessing code
- Trained model checkpoints
- Training datasets
- Restricted audio recordings
- Private experiment configuration
- Production assets

This separation allows the technical ideas and research work to be presented publicly without publishing private project assets.

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
- Audio Augmentation
- MFCC
- Delta / Delta-Delta Features
- CMVN
- Noise Reduction
- Energy-Based Silence Detection

### Experimental Methods

- Whisper
- Hidden Markov Models
- Forced Alignment
- Neural Sequence Modeling
- Segmentation Heads
- Count Heads

---

## 🚧 Limitations

This project should be understood as a research and graduation-project prototype.

Current limitations include:

- generalization to unseen speakers
- dependence on available training-data diversity
- sensitivity to recording conditions
- limited public evaluation data
- experimental rather than clinically or pedagogically validated feedback
- not every Tajweed rule is modeled end-to-end
- phoneme-boundary estimation remains a difficult research problem

The system should **not** be considered a replacement for a qualified Quran teacher.

Its intended role is as a technological aid for recitation analysis and future educational tools.

---

## 🔮 Future Work

Potential directions include:

- Larger and more diverse reciter datasets
- Stronger speaker-independent evaluation
- Improved phoneme-boundary supervision
- Better duration modeling
- Dedicated forced-alignment experiments
- Improved augmentation
- Speaker normalization
- Phoneme Error Rate (PER) evaluation
- Boundary-error metrics in milliseconds
- Duration-error metrics
- Confidence estimation
- Rule-specific Tajweed classifiers
- Interactive visual phoneme alignment
- Real-time inference
- Mobile integration
- Personalized learner feedback

---

## 🎓 Academic Context

This work was developed as a **fifth-year graduation project** at:

**Aleppo University**  
Faculty of Informatics Engineering  
Department of Artificial Intelligence and Natural Languages

The project focused on building an intelligent language-processing system for detecting selected pronunciation and Tajweed-related errors in Quran recitation.

### Project Team

- Mahmoud Bannan
- Mohammed Deeb Jalab
- Mohanad Al-Abd Allah
- Amjad Haj Hammada
- Mahmoud Abd Al-Rhim
- Hayder Qasem

### Supervisor

**Dr. Mohammed Fadel Sukkar**

This public repository is maintained by **Mahmoud Bannan** as a technical showcase of the Audio AI work and project methodology.

---

## 📖 Academic Report

The complete graduation report documents:

- the motivation and problem definition
- traditional audio-processing experiments
- Whisper experiments
- HMM / forced-alignment exploration
- Quran text-to-phoneme representation
- BiLSTM + CTC experiments
- Conformer development
- CTC modifications
- segmentation and count heads
- audio augmentation
- project limitations and experimental results

The report is not necessarily distributed publicly through this repository unless publication rights for the complete team document have been confirmed.

---

## ⚖️ Responsible Use

Quran recitation assessment is a specialized domain that includes linguistic, phonetic, and religious expertise.

AI-generated feedback can be useful as an educational aid, but model predictions may be inaccurate.

For authoritative correction and learning, users should consult qualified teachers and specialists.

---

## 👨‍💻 Repository Maintainer

**Mahmoud Bannan**

Flutter Developer & Audio AI Engineer

- GitHub: [@Mahmoudmnb](https://github.com/Mahmoudmnb)
- Portfolio: [mahmoudbannan.com](https://mahmoudbannan.com)
- LinkedIn: [Mahmoud Bannan](https://www.linkedin.com/in/mahmoudbannan)

---

## 📄 License

Documentation and assets in this public showcase repository are provided under the license included in [`LICENSE`](LICENSE).

The license does **not** imply that private source code, datasets, model checkpoints, third-party audio, or other materials not contained in this repository are publicly licensed.
