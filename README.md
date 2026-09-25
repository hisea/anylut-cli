# anylut

`anylut` is a command-line tool for editing photos non-destructively, built for
local agents and scripts. It reads a photo (RAW, JPEG or HEIC), lets you edit a
small, lossless recipe file, and renders previews or a final JPEG/PNG. The
original file is never modified.

This repository hosts the **prebuilt, signed and notarized binaries** for macOS.
The source is not published here.

## Install

```bash
brew install hisea/anylut/anylut
```

Requires an Apple Silicon Mac running macOS 14 (Sonoma) or later.

Update with `brew upgrade anylut`.

## Quick start

```bash
anylut capabilities --json                        # every parameter you can write, with ranges
anylut inspect IMG.ARW --json
anylut recipe new --source IMG.ARW --output edit.anylut.json
anylut patch edit.anylut.json --patch-file adjust.json --output edit-v1.anylut.json --json
anylut render IMG.ARW --recipe edit-v1.anylut.json --preview --output preview.jpg --json
anylut render IMG.ARW --recipe edit-v1.anylut.json --full-resolution --output final.jpg --json
```

A patch is a small JSON file (or `--patch-file -` to read it from stdin):

```json
{ "patchVersion": 1, "operations": [
  { "op": "set", "path": "light.exposure", "value": 0.35 },
  { "op": "add", "path": "color.temperature", "value": 6 },
  { "op": "set", "path": "hsl.orange.saturation", "value": -5 } ] }
```

`set` assigns a value, `add` is relative. Out-of-range values are an error and are
never silently clamped. `anylut --help` lists every command.

## Guarantees

- With `--json`, every command prints exactly one JSON object on one line to
  stdout; errors are a single `{"ok":false,"error":{...}}` object with a stable
  `code`. Exit codes: `0` ok, `2` usage, `3` unsupported file, `4` validation,
  `5` decode, `6` render, `7` encode/write.
- Output files are written atomically and never overwrite an existing file unless
  you pass `--overwrite`.
- Edit files are validated strictly - unknown, missing or duplicate keys and
  out-of-range values are rejected rather than repaired.
- Rendering runs entirely on your Mac; nothing is uploaded.

## Limits (v0.1)

Recipes containing crop, rotation, flip or straighten are refused rather than
rendered without them. RAW support is whatever your macOS version's RAW decoder
supports; run `anylut inspect` on a file to find out.
