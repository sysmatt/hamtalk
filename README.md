# hamtalk

> ### ⚠️ WARNING: EXPERIMENTAL AND INCOMPLETE ⚠️
>
> **This software is in early, active development and is NOT ready for any serious use.**
> It is experimental, incomplete, and will have bugs, missing features, and breaking changes
> without notice. Use it for tinkering only.

---

Ham radio CQ exchange listener. Captures microphone audio, transcribes speech in near-real-time
using an offline Whisper model, and detects ham radio callsigns inline in the output.

## Features

- Fully offline — no cloud APIs, no data leaves your machine
- VAD (voice activity detection) gating to reduce Whisper hallucinations
- Callsign detection supporting multiple input formats:
  - Direct: `KE2R`
  - Split tokens: `K2 TTA`, `KD2 GI Y`
  - Hyphenated: `K-E-2-R`, `Kilo-2-Tango-Tango-Alpha`
  - NATO phonetics: `Kilo Echo 2 Romeo`
  - Mixed sentence: `CQ CQ, this is Kilo 2 Tango Tango Alpha`
- Detected callsigns highlighted in bold green inline with transcribed text

## Requirements

- Python 3.10+
- PortAudio (required by `sounddevice`)

On Debian/Ubuntu:
```bash
sudo apt install portaudio19-dev
```

## Installation

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

On first run, the Whisper model will be downloaded and cached locally (~150MB for `base`).
Subsequent runs are fully offline.

## Usage

List available audio input devices:
```bash
./hamtalk --list-devices
```

Run with the default microphone:
```bash
./hamtalk
```

Run with a specific device and model:
```bash
./hamtalk --device 2 --model small
```

### Options

| Flag | Default | Description |
|---|---|---|
| `--device`, `-d` | system default | Audio input device index (see `--list-devices`) |
| `--model`, `-m` | `base` | Whisper model size: `tiny`, `base`, `small`, `medium`, `large-v2`, `large-v3` |
| `--compute-type` | `int8` | `int8` (fastest on CPU), `float16`, `float32` |
| `--list-devices` | — | Print available input devices and exit |

## Tuning

If the VAD triggers on background noise, raise `ENERGY_THRESHOLD` in the script (default `0.01`).
If it clips the start of words, lower it.

If utterances are being cut off mid-sentence, raise `SILENCE_FRAMES_TO_FLUSH` (default `15` frames ≈ 450ms).

## Known Limitations

- Callsign detection is heuristic — false positives and missed detections will occur
- Only US callsign format is supported (`[AKNW][A-Z]{0,2}\d[A-Z]{1,4}`)
- Multi-utterance callsigns (split across VAD chunk boundaries) are not detected
- No FCC database lookup — callsigns are pattern-matched only, not validated
- Whisper `base` model accuracy on ham radio jargon is limited; `small` or `medium` may help
