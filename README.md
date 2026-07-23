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
   Folders are read recursively.
2. **Output** — either:
   * **Download as .zip** — works in every browser.
   * **Save to a folder I choose** (Chrome / Edge) — writes into a working
     folder of your choosing, with no archive to unpack.
3. **Convert** — progress is shown per file, with a result table.
4. **Copy the files into place** — see *Installing the files* below.

### What you get out

Always the same shape, whatever you put in:

```
Samples/    .raw12b files, flat
WT/         .raw files, flat
```

Each type has its own destination in MiniFreak V, so the two are separated for
you. Input sub-folders are **flattened away** — MiniFreak V reads files only
and ignores anything nested — so `drums/kick.wav` and `bass/kick.wav` both land
in `Samples/`. When flattening collides like that, the later file gets `_2`,
`_3` and so on rather than overwriting the earlier one.

### A non-standard install

If your Arturia content isn't in the stock location — another drive, a moved
folder — edit the **Factory folder** field under the paths. Every path shown
and copied follows it, and it's remembered for next time; **Reset** restores
the OS default.

## Install locations

The page detects your operating system and shows the matching paths, with
a **Copy** button for each; the *MiniFreak V is installed on* selector
overrides the detection when you're converting on one machine for another,
and the *Factory folder* field overrides the paths themselves.

**Windows**

```
C:\ProgramData\Arturia\Samples\MiniFreak V\Factory\Samples\User
C:\ProgramData\Arturia\Samples\MiniFreak V\Factory\WT\User
```

The last segment is a folder *you* create and can name anything; `User` is the
name used throughout these docs.

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

### Plugin only, or on the synth too?

Where you put the content decides whether it reaches the hardware:

* **Inside `Factory`'s own sample folders** — included when MiniFreak V syncs
  to the synth, so it ends up on the instrument itself.
* **In a folder of your own** (the `User` folder in these steps) — available
  in the plugin, but left out of the sync.

Neither choice is permanent — see *Removing samples from the hardware* below.

## Installing the files

The page shows these steps for your detected OS, with a copy button for each
path. Both are listed here too.

### Windows

1. Convert, and unpack the result — right-click the .zip → **Extract All…**
   You'll have a `Samples\` folder and a `WT\` folder, already flattened.
2. Open **File Explorer**, click the address bar (or press
   <kbd>Ctrl</kbd>+<kbd>L</kbd>), and paste:
   ```
   C:\ProgramData\Arturia\Samples\MiniFreak V\Factory
   ```
3. Make a folder inside each of `Samples\` and `WT\` to hold your content.
   **Name it whatever you like** — MiniFreak V doesn't care; `User` is just a
   tidy convention. The one rule is that it must contain **files only**:
   sub-folders nested inside it are not read.
4. Copy the *contents* of the converter's `Samples\` folder into the folder
   you just made under `Samples\`, and likewise for `WT\`.
5. **That's it.** MiniFreak V picks up new content as soon as it lands; no
   restart or rescan is needed.

### macOS

1. Convert, and unpack the result — double-click the .zip. Safari may have
   unzipped it already, so look for a folder rather than an archive. You'll
   have a `Samples/` folder and a `WT/` folder, already flattened.
2. In **Finder** press <kbd>⇧⌘G</kbd> and paste:
   ```
   /Library/Arturia/Samples/MiniFreak V/Factory
   ```
3. Make a folder inside each of `Samples/` and `WT/` to hold your content.
   **Name it whatever you like** — `User` is just a tidy convention. The one
   rule is that it must contain **files only**: sub-folders nested inside it
   are not read.
4. Copy the *contents* of the converter's `Samples/` folder into the folder
   you just made under `Samples/`, and likewise for `WT/`.
5. **That's it.** MiniFreak V picks up new content as soon as it lands; no
   restart or rescan is needed.

Your content then appears under the folder name you chose — samples for the
Sampler and Grain engines, wavetables for the Wavetable engine.

## Removing samples from the hardware

If you've synced content to the synth and want it gone, the instrument's debug
menu can reset its storage:

1. Hold the **oscillator select** and **filter type** buttons together for a
   few seconds, then release to open the **debug menu**.
2. Choose **Format Filesystem**.
3. Sync with MiniFreak V again to reload the factory tables and samples.

> **This wipes _all_ wavetables and samples on the unit**, factory content
> included — it is not a way to remove one sample. Step 3 puts the factory
> content back.

Delete your custom files from the computer *before* syncing, or they'll simply
be copied onto the unit again.

*Procedure via u/zeknife on r/MiniFreak.*

## Implementation notes

Everything is inline in `index.html` — no external scripts or CDNs:

* RIFF/WAV parser (float32, and 8/16/24/32-bit PCM), channel downmix.
* Iterative radix-2 FFT for band-limited single-cycle resampling.
  Non-power-of-two frame sizes fall back to linear interpolation (the
  Python version uses numpy's arbitrary-size FFT there, so results can
  differ for such files — Serum tables are always powers of two).
* A minimal stored-mode ZIP writer with CRC-32.
* File System Access API for direct folder writing, feature-detected with
  an automatic fall back to ZIP (the folder option is disabled outright when
  the API is missing).
* The custom Factory-folder string lives in `localStorage`.

## Limitations

* Direct folder writing needs a Chromium browser; other browsers get the
  ZIP path.
* The folder picker cannot open Arturia's own directories — Chrome and Edge
  block protected system locations, including `C:\ProgramData` and
  `/Library` ("can't open this folder because it contains system files").
  Converted files always have to be copied into place by hand.
* Wavetable import expects **mono** frame-structured WAVs.
* Large batches are converted in memory; a few hundred files is fine,
  but the ZIP is built in RAM before download.
