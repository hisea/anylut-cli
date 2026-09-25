---
name: anylut
description: Edit and color-grade photos (RAW, JPEG, HEIC) with the anylut command-line tool on an Apple Silicon Mac. Use this skill when the user asks to adjust exposure, contrast, white balance, HSL, curves, or a LUT look on a photo, to make a preview, or to export a graded JPEG or PNG. Do not use it for video, for crop or rotation edits, or on a computer that is not a Mac.
license: Proprietary
compatibility: Apple Silicon Mac with macOS 14 or later. Needs the anylut tool from Homebrew.
metadata:
  homepage: https://github.com/hisea/anylut-cli
---

# anylut

`anylut` changes the look of a photo. It does not change the original file. Each edit is a small JSON recipe file.

## Check the tool

1. Run `command -v anylut`.
2. If the command finds nothing, tell the user to run `brew install hisea/anylut/anylut`. Wait for the user to confirm. Do not install the tool without the user's approval.
3. Run `anylut --version`.

## Read the guide

1. Run `anylut guide`.
2. Follow the text that the command prints. The text is the full instruction for the installed version.
3. Do not use a remembered command or parameter. The guide and `anylut capabilities --json` are the only sources.

## Rules

- Add `--json` to each command.
- Write each version of the recipe to a new file.
- Look at each preview before you tell the user that the edit is correct.
- Do not use `--overwrite` unless the user asks.
