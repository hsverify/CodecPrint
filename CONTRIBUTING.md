# Contributing to CodecPrint

Thanks for your interest in contributing. CodecPrint is maintained by HumanStandard and we welcome contributions from the research and audio forensics community.

## Ways to Contribute

### New Codec Profiles

The most valuable contribution is adding support for a new neural audio codec. To do this:

1. Process a set of reference audio files through the codec at various bitrates
2. Run the feature extraction pipeline on both the original and processed files
3. Submit the extracted fingerprint profile as a PR

We'll validate against our internal test set before merging.

### Robustness Testing

We want CodecPrint to be honest about its limitations. If you find post-processing chains that defeat detection (e.g., specific EQ + resampling + MP3 combinations), please report them. This helps us improve and helps users understand when to trust results.

### Bug Reports

Open an issue with:
- Python version and OS
- Input file format and duration
- Expected vs actual output
- Minimal reproduction steps

### Research Contributions

If you develop novel features or classification approaches for codec identification, we'd love to see them. Open an issue describing the approach before submitting a PR so we can discuss fit.

## Development Setup

```bash
git clone https://github.com/hsverify/codecprint.git
cd codecprint
python -m venv venv
source venv/bin/activate
pip install -e ".[dev]"
```

## Running Tests

```bash
pytest tests/
```

## Code Style

- Python 3.9+
- No strict formatter enforced, but keep it readable
- Type hints appreciated but not required
- Docstrings on public functions

## Pull Request Process

1. Fork the repo and create a branch from `main`
2. Add tests for new functionality
3. Make sure `pytest` passes
4. Open a PR with a clear description of what changed and why

## Contributor License Agreement

By submitting a PR, you agree that your contribution is licensed under the same CC BY-NC 4.0 license as the rest of the project. If your employer may claim ownership of your work, please confirm you have permission before contributing.

## Code of Conduct

Be respectful. We're building tools to protect human creativity — bring that same energy to how you interact with other contributors.

## Questions

Open an issue or email rasha@jobsbyhumans.com.
