# Tact

A lightweight menu bar app that records meetings and transcribes them locally using Whisper. No cloud, no cost, full privacy.

The app does one thing well: turn spoken audio into a transcript file in the folder you choose.

Tact is not a standalone productivity tool. It fits into existing workflows — an Obsidian vault, a project folder, whatever structure you already use. The app's job ends at producing a well-formatted transcript file in the right place.

---

## Design Principles

**Simple and tidy.** Lives in the menu bar. No dock icon, no main window. Day-to-day interaction is: shortcut, record, stop, transcript appears.

**Lightweight.** Comfortable running all day. Idle memory ~30MB. The Whisper model loads only when transcribing and unloads after. No persistent model in memory.

**Local-first.** All processing happens on-device via Metal GPU. No network calls, no accounts, no telemetry. Works on an airplane.

**Output-oriented.** Produces a timestamped Markdown file in a folder. Not a knowledge base, not a meeting assistant. What happens to that file afterward is up to you (or your automation).

---

## Core

**Recording** — Press `⌥⇧R` (global hotkey) or click the menu bar icon to start/stop recording. Audio is saved as AAC m4a. Live duration timer shows in the popover.

**Transcription** — whisper.cpp with Metal GPU acceleration. A 1-hour meeting transcribes in ~4.5 minutes on M1 Pro. Output is a Markdown file with `[HH:MM:SS]` timestamps, written to your chosen destination folder.

**Automation** — The app's primary integration point. Two systems:

- **AI Summary** — After transcription, automatically generates a summary using Claude Code CLI. Configurable destination: same folder as transcript, a fixed folder, or a relative path. Requires Claude Code CLI installed.
- **Hooks** — Run scripts at three points: on-recording-complete, on-transcription-complete, on-error. Managed through the Automation page with enable/disable toggles per hook, or via `~/.config/tact/hooks.json`. Scripts receive environment variables: `$TACT_PROJECT_DIR`, `$TACT_DATE`, `$TACT_TIME`, `$TACT_DURATION`, `$TACT_LANGUAGE`, `$TACT_MODEL`, `$TACT_AUDIO`, `$TACT_ERROR`, and `$1` (transcript path).

**Models** — 4 Whisper model options downloaded from HuggingFace, managed in Settings > Models:

| Model | Size | Speed (1hr) | RAM | Best For |
|-------|------|-------------|-----|----------|
| Turbo v3 f16 | 1.5 GB | ~4.5 min | 2.4 GB | Default |
| Turbo v3 q5 | 547 MB | ~4.9 min | 1.5 GB | 8GB devices |
| Large v3 f16 | 2.9 GB | ~7.6 min | 4.5 GB | Best accuracy (24GB) |
| Large v3 q5 | 1.0 GB | ~6.4 min | 2.4 GB | Best accuracy (16GB) |

Benchmarked on Apple M1 Pro 16GB.

---

## Features

**Transcription Queue** — Three timing modes: Immediately, On Return, and Manual. Queue persists across app restarts. Failed items can be retried. Sequential processing — one at a time.

**On Return** — When you stop recording, the file is queued as "waiting." When your Mac sleeps and wakes (you left a meeting and came back to your desk), Tact detects the wake event and automatically starts transcribing all waiting items. No manual action needed.

**Upload** — Import existing audio files (.m4a, .wav, .mp3) for transcription. Same queue and timing options as recordings.

**Folder Management** — Pick a destination folder for transcripts. Favorite folders for quick access, recent folders remembered (last 5). Persisted with macOS security-scoped bookmarks so sandbox access survives restarts.

**Speaker Diarization** — Optional speaker identification powered by FluidAudio. Runs concurrently on CPU while Whisper uses GPU. Enroll your own voice in Settings so your segments are labeled "Me" instead of "Speaker 1." Four model variants for different environments (general, noisy, in-person, phone).

**Engine Settings** — Tunable whisper.cpp parameters to reduce repetition loops:
- Beam search (1–8) — explores multiple hypotheses instead of greedy decoding
- Max context (0–224) — limits how much previous text feeds into the next chunk
- Entropy threshold (2.4–3.0) — rejects repetitive output
- Temperature fallback — retries with randomness when a loop is detected
- Silero VAD — skips silence segments to prevent hallucination

**Storage** — Configurable recording retention (7 days, 30 days, forever). Optional audio export copies the recording file alongside the transcript in the destination folder.

**Updates** — Manual update check via Sparkle (Settings > General > Check for Updates).

---

## Requirements

- macOS 14.0+
- Apple Silicon (M1/M2/M3/M4)

## Install

Download the latest `.dmg` from [Releases](https://github.com/jayl930/tact-releases/releases).

1. Open the .dmg and drag Tact to Applications
2. Right-click Tact.app → Open → Open (to bypass Gatekeeper on first launch)
3. Go to Settings → Models and download a model
4. Select a destination folder
5. Press `⌥⇧R` to record
