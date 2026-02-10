# CodecPrint

**Identify which neural audio codec processed any audio file.**

[![License: Non-Commercial](https://img.shields.io/badge/License-Non--Commercial-green.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)

---

Every AI music generator encodes audio through a neural codec — EnCodec, DAC, SoundStream, Oobleck — and each one leaves a measurable spectral fingerprint. CodecPrint detects and classifies these fingerprints.

## Quick Start

```bash
pip install codecprint
```

```python
from codecprint import identify

result = identify("track.mp3")

print(result.codec)          # "encodec_48khz"
print(result.confidence)     # 0.91
print(result.top_codecs)     # [("encodec_48khz", 0.91), ("dac_44khz", 0.12), ...]
print(result.evidence)       # detailed spectral evidence
```

### CLI

```bash
# Single file
codecprint track.mp3

# Directory scan
codecprint --dir uploads/ --format json

# Detailed evidence report
codecprint track.mp3 --verbose
```

```
$ codecprint track.mp3
track.mp3 → encodec_48khz (confidence: 0.91)
```

```
$ codecprint track.mp3 --verbose
track.mp3
  Detected:  encodec_48khz (0.91)
  Runners-up: dac_44khz (0.12), none (0.04)
  Evidence:
    spectral_peak_regularity  0.93  peaks at 4000, 8000, 12000 Hz (EnCodec bandwidths)
    cepstral_periodicity      0.87  quefrency peaks consistent with EnCodec RVQ
    reconstruction_residual   0.84  noise floor pattern matches EnCodec 48kHz
    temporal_frame_pattern    0.79  6.25ms frame boundaries detected
```

## Supported Codecs

| Codec | Source | Used By | Sample Rates |
|-------|--------|---------|--------------|
| **EnCodec** | Meta | Suno, MusicGen | 24kHz, 48kHz |
| **DAC** | Descript | Stable Audio | 44.1kHz |
| **SoundStream** | Google | MusicLM, AudioPaLM | 16kHz, 24kHz |
| **Oobleck** | Stability AI | Stable Audio Open | 44.1kHz |
| **Vocos** | Various | ElevenLabs | 24kHz |

Adding a new codec? See [Contributing](#contributing).

## How It Works

Neural audio codecs compress audio into a quantized latent space and reconstruct it through a learned decoder. This reconstruction introduces characteristic artifacts that differ by codec architecture:

### Feature Extraction

CodecPrint extracts four categories of spectral evidence:

**1. Spectral Band Energy**
Each codec's decoder produces energy at specific frequency intervals determined by its upsampling architecture. EnCodec's HiFi-GAN decoder with 2x, 4x, 5x, 8x upsampling creates peaks at different intervals than DAC's 2x, 4x, 8x, 8x pattern.

**2. Cepstral Regularity**
The cepstral domain (inverse FFT of the log spectrum) reveals periodic spectral structure. Codecs with regular quantization patterns produce distinctive quefrency peaks.

**3. Temporal Micro-Structure**
Frame-level reconstruction boundaries vary by codec. EnCodec at 48kHz uses 320-sample frames (6.67ms), while DAC uses 512-sample frames (11.6ms). These boundaries leave detectable discontinuities.

**4. Quantization Residuals**
Residual Vector Quantization (RVQ) — used by all major codecs — adds codec-specific noise patterns in the high-frequency tail of the spectrum.

### Classification

Features are classified using a lightweight ensemble:
- Per-codec logistic regression models (fast, interpretable)
- Confidence calibrated via isotonic regression on held-out data
- Multi-label output (audio can show traces of multiple processing steps)

Total inference time: **< 1 second** per track on CPU.

## Python API

```python
from codecprint import identify, extract_features, compare

# Basic identification
result = identify("track.mp3")
result.codec          # str: best matching codec or "none"
result.confidence     # float: calibrated confidence [0, 1]
result.top_codecs     # list of (codec, confidence) tuples
result.evidence       # dict of feature_name → score

# Raw feature extraction (for research)
features = extract_features("track.mp3")
features.spectral_peaks       # array of detected peak frequencies
features.cepstral_vector      # cepstral feature vector
features.frame_boundaries     # detected frame boundary positions
features.quantization_noise   # high-freq noise profile

# Compare two files
diff = compare("original.wav", "processed.wav")
diff.codec_detected     # codec found in processed but not original
diff.artifact_delta     # per-feature difference map
```

## Batch Processing

```python
from codecprint import scan_directory

results = scan_directory("uploads/", workers=4)
for path, result in results.items():
    if result.confidence > 0.8:
        print(f"{path}: {result.codec} ({result.confidence:.2f})")
```

## Evaluation

We benchmark against a held-out test set of 500 tracks (100 per codec) re-encoded at various bitrates and with common post-processing (MP3, normalization, EQ).

| Codec | Precision | Recall | F1 |
|-------|-----------|--------|----|
| EnCodec 24kHz | — | — | — |
| EnCodec 48kHz | — | — | — |
| DAC 44.1kHz | — | — | — |
| SoundStream 24kHz | — | — | — |
| Oobleck 44.1kHz | — | — | — |
| None (human) | — | — | — |

*Benchmarks will be populated after initial training.*

## Installation

```bash
# From PyPI
pip install codecprint

# From source
git clone https://github.com/hsverify/codecprint.git
cd codecprint
pip install -e .
```

Requirements: Python 3.9+, numpy, scipy, librosa. No GPU needed.

## Contributing

We welcome contributions, especially:

- **New codec profiles** — run a codec on reference audio and submit the extracted fingerprint
- **Robustness testing** — adversarial post-processing that defeats detection
- **Research** — novel features or classification approaches

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## Limitations

- Post-processing (heavy MP3 compression, resampling, EQ) can degrade codec fingerprints
- Multiple sequential codecs (e.g., EnCodec → MP3 → AAC) may produce ambiguous results
- Detection confidence decreases with very short audio (< 5 seconds)
- This tool identifies codec artifacts, not whether content is AI-generated — a human recording processed through EnCodec will be identified as EnCodec

## License

CodecPrint is available under a **non-commercial license**. Free for academic research, independent artists, non-profit organizations, and personal use.

Commercial use (including integration into paid products or services) requires a license from HumanStandard. Contact **enterprise@jobsbyhumans.com**.

See [LICENSE](LICENSE) for full terms.

## Enterprise

CodecPrint identifies codec artifacts — one forensic signal among many. If you need production-grade AI music detection, HumanStandard offers:

- **AI Detection API** — real-time classification powered by a multi-model ensemble
- **Forensic Stem Analysis** — per-stem human vs. AI involvement scoring
- **Platform Integration** — high-throughput streaming API for distributors, DSPs, and labels

Learn more at [hsverify.com](https://hsverify.com) or contact **enterprise@jobsbyhumans.com**.

## Citation

```bibtex
@software{codecprint2026,
  title={CodecPrint: Neural Audio Codec Forensics},
  author={HumanStandard},
  year={2026},
  url={https://github.com/hsverify/codecprint}
}
```

## About

Built by [HumanStandard](https://hsverify.com). We build audio forensics and detection technology to protect human creativity at scale. Our tools analyze audio — they never generate it.
