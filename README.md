# MiniFreak Sample &amp; Wavetable Converter — static web version

Inspired by https://github.com/mtizim/minifreak_sample_conv/

**USE AT YOUR OWN RISK** - while I have not encountered any issues while 
using this tool, it is not an official Arturia tool and can result in harm 
to your device. 

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
     files are written directly — point it at the wavetable or sample
     folder below to install in place.
   * **Download as .zip** (any browser): unzip into those folders.

   If a batch contains *both* types they're split into `Samples/` and
   `WT/` sub-folders, since each belongs in a different plugin folder.
3. **Convert** — progress is shown per file, with a result table.

## Install locations

The page detects your operating system and shows the matching paths, with
a **Copy** button for each; the *Install paths for* selector overrides the
detection when you're converting on one machine for another.

**Note:** The folders paths below user a "User" directory instead of the 
"Factory" directory that the plugin ships the factory samples in. You can 
name the folder anything you like, but I reccomend keeping it separate or 
else it will try to upload the samples to your hardware MiniFreak when you
link it to the VST. It **is possible** to transfer these files to your 
hardware, but I would advise **CAUTION** when doing so... at this time there
is no known way to remove the files that were sent to the hardware synth.

**Windows**

```
C:\ProgramData\Arturia\Samples\MiniFreak V\Factory\Samples\User
C:\ProgramData\Arturia\Samples\MiniFreak V\Factory\WT\User
```

`C:\ProgramData` is hidden in Explorer by default — paste the path into the
address bar to open it.

**macOS**

```
/Library/Arturia/Samples/MiniFreak V/Factory/Samples/User
/Library/Arturia/Samples/MiniFreak V/Factory/WT/User
```

This is the machine-wide `/Library` at the root of your startup disk, **not**
your home `~/Library` — Finder hides both by default. Press <kbd>⇧⌘G</kbd> in
Finder and paste a path to jump straight there.

### macOS notes

* **Use Safari or Firefox? You'll get the .zip.** Direct folder writing needs
  the File System Access API, which on macOS means Chrome or Edge. The page
  falls back to the ZIP download automatically.
* **Even in Chrome, the picker may refuse `/Library`.** Browsers block a set of
  protected system folders in the directory picker. If the folder can't be
  selected, take the .zip and copy the files across in Finder instead.
* **Writing into `/Library` needs an admin password.** Finder prompts for it on
  the first copy; that's expected for a machine-wide folder.
* **Safari unzips downloads for you.** With *Open "safe" files after
  downloading* enabled you'll find an already-extracted folder in `~/Downloads`
  rather than the `.zip` — copy its *contents* into the folders above, not the
  folder itself.
* **Ignore any `__MACOSX` folder or `._` files** that appear if the archive is
  unzipped by a third-party tool. They're macOS metadata, not samples, and
  MiniFreak V will not read them.

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
  directory picker — on macOS this often includes `/Library`, so the ZIP
  route plus a Finder copy is the reliable path there.
* Wavetable import expects **mono** frame-structured WAVs.
* Large batches are converted in memory; a few hundred files is fine,
  but the ZIP is built in RAM before download.
