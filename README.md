# MiniFreak Sample &amp; Wavetable Converter — static web version

A **single self-contained `index.html`** that converts audio into Arturia
**MiniFreak V** user content, entirely in the browser. No install, no
Python, no server, no dependencies — and nothing is uploaded anywhere.

* **Samples** — any WAV → `.raw12b` (headerless signed 8-bit mono PCM at
  48 kHz), for the Sample / Grain engines.
* **Wavetables** — Serum-style frame-structured WAV → `.raw` (512-sample
  24-bit frames), for the Wavetable engine, each source cycle
  band-limit-resampled via FFT and the table evenly reduced to the
  engine's 189-frame limit.

## Use

Open `index.html` in a browser (double-click it, or host it anywhere).

1. **Input** — pick a folder or files, or drag WAVs onto the drop zone.
   Auto-detect treats a WAV carrying a Serum `clm ` chunk as a wavetable
   and everything else as a one-shot sample; you can also force a type.
   Folders are read recursively and their structure is preserved.
2. **Output** — either:
   * **Write to a folder** (Chrome / Edge): pick a destination once and
     files are written directly — point it at
     `…\MiniFreak V\Factory\WT\User` or `…\Factory\Samples\User` to
     install in place.
   * **Download as .zip** (any browser): unzip into those folders.

   If a batch contains *both* types they're split into `Samples\` and
   `WT\` sub-folders, since each belongs in a different plugin folder.
3. **Convert** — progress is shown per file, with a result table.

## Verified against the Python tool

The conversion output is **byte-identical** to the
[MinifreakConverter](../../Python%20Projects/MinifreakConverter) Python
implementation — verified by SHA-256 on both a wavetable and a
resampled stereo sample, and by extracting a generated `.zip` with
Python's `zipfile` (all CRCs valid). The JS mirrors the Python exactly:
the same `round(x * 127)` sample rule, the same `np.interp`-equivalent
rate conversion, and the same FFT spectral truncation with `m/n` scaling
for wavetable cycles.

## Implementation notes

Everything is inline in `index.html` — no external scripts or CDNs:

* RIFF/WAV parser (float32, and 8/16/24/32-bit PCM), channel downmix.
* Iterative radix-2 FFT for band-limited single-cycle resampling.
  Non-power-of-two frame sizes fall back to linear interpolation (the
  Python version uses numpy's arbitrary-size FFT there, so results can
  differ for such files — Serum tables are always powers of two).
* A minimal stored-mode ZIP writer with CRC-32.
* File System Access API for direct folder writing, feature-detected
  with an automatic fall back to ZIP.

## Limitations

* Direct folder writing needs a Chromium browser; other browsers get the
  ZIP path. Browsers also block some protected system folders from the
  directory picker.
* Wavetable import expects **mono** frame-structured WAVs.
* Large batches are converted in memory; a few hundred files is fine,
  but the ZIP is built in RAM before download.
