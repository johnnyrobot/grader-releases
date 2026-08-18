# Grader Releases

Official signed and notarized macOS public-beta artifacts for Grader.

**[Download the latest release →](https://github.com/johnnyrobot/grader-releases/releases/latest)**

Grader drafts rubric-based grades on your own Mac. Student work stays on the instructor
device, and posting anything back to Canvas always requires explicit teacher approval.

## Requirements

| | |
|---|---|
| Processor | **Apple Silicon (M1 or later)** |
| macOS | 13.0 Ventura or later |
| Memory | 16 GB minimum |
| Storage | ~10 GB free (the app is ~1.2 GB; the models are the rest) |
| [Ollama](https://ollama.com) | 0.4 or later |

Grader is an Apple Silicon build and does not run on Intel Macs. The in-app updater will
not offer an update to an Intel Mac rather than install one that cannot launch.

You do **not** need Python, `uv`, Homebrew, or any developer tooling. Grader carries its
own engine inside the app.

## Install

1. Download `Grader.dmg` from the [latest release](https://github.com/johnnyrobot/grader-releases/releases/latest).
2. Open the DMG and drag **Grader** to your Applications folder.
3. Launch Grader from Finder. It is signed and notarized, so there is no Gatekeeper warning.

### Verify the download (optional)

Each release publishes `SHA256SUMS`. From the folder holding the downloaded files:

```bash
shasum -a 256 -c SHA256SUMS --ignore-missing
```

## Set up the grading models

Grader runs its models locally through Ollama. Install Ollama first, then pull the base
models and build Grader's capped variants.

```bash
brew install ollama
ollama serve &                     # if it is not already running

ollama pull qwen3:8b
ollama pull minicpm-v4.6:latest
ollama pull nomic-embed-text:latest

# Build the capped grader-* tags that Grader grades with:
curl -fsSLO https://raw.githubusercontent.com/johnnyrobot/grader-releases/main/scripts/make_models.sh
chmod +x make_models.sh
./make_models.sh
```

`scripts/make_models.sh` lives in this repository. It is required, not an optimization. Ollama
loads a model at its full default context window, and those defaults are enormous — a
4-billion-parameter model was measured holding **41 GB** of memory, almost all of it KV
cache. Grading never needs more than a few thousand tokens, so the script bakes variants
capped at 16,384. Without it, Grader has no `grader-text:8b` tag to grade with, and the
uncapped base models will not load on a 16 GB Mac.

After it finishes, `ollama list` should show `grader-text:8b` and `grader-vision:1.3b`.

## Updates

Grader updates itself. When an update is available the app offers it; choose **Install
update and restart**, then confirm the relaunched app reports the new version. The updater
reads `latest.json` from the most recent release in this repository.

## Notes for beta testers

- Student work remains on the instructor device.
- Canvas posting requires explicit teacher approval.
- Citation verification is experimental during the beta.

The application source repository remains private during the supervised beta. This
repository contains release assets, the model setup script, and source-build provenance only.
