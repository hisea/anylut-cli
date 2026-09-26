# anylut

**English** | [中文文档](README.zh-CN.md)

`anylut` is a command-line tool for editing photos non-destructively, built for
local agents and scripts. It reads a photo (RAW, JPEG or HEIC), lets you edit a
small, lossless recipe file, and renders previews or a final JPEG/PNG. The
original file is never modified.

This repository hosts the **prebuilt, signed and notarized binaries** for macOS.
The source is not published here.

[<img src="https://anylut.com/assets/app-store-badge.svg" height="40" alt="Download AnyLUT on the App Store">](https://apps.apple.com/us/app/anylut/id6814290745)&nbsp;&nbsp;[**Learn about the macOS CLI →**](https://anylut.com/cli/)

## Install

```bash
brew install hisea/anylut/anylut
```

Requires an Apple Silicon Mac running macOS 14 (Sonoma) or later.

Update with `brew upgrade anylut`.

Homebrew asks you to trust non-official taps. The command above names the formula
in full, which trusts just that one formula. If you `brew tap hisea/anylut` first
and want to install by the short name, trust it explicitly:

```bash
brew tap hisea/anylut
brew trust --formula hisea/anylut/anylut
brew install anylut
```

## Use with an agent

The skill `anylut` teaches an agent to run the tool. It tells the agent to run `anylut guide`, so the
instructions always match the installed version. Install the tool first (see above).

| Agent | Install |
| --- | --- |
| Claude Code | `claude plugin marketplace add hisea/anylut-cli` then `claude plugin install anylut@anylut` |
| Codex | `codex plugin marketplace add hisea/anylut-cli` then `codex plugin add anylut@anylut` |
| pi | `pi install git:github.com/hisea/anylut-cli` |
| omp | `omp plugin marketplace add hisea/anylut-cli` then `omp plugin install anylut@anylut` |
| opencode | `git clone https://github.com/hisea/anylut-cli /tmp/anylut-cli && mkdir -p ~/.config/opencode/skills && cp -R /tmp/anylut-cli/skills/anylut ~/.config/opencode/skills/` |

Any other agent that reads `SKILL.md` can use the folder `skills/anylut/`.

## Use as an MCP server

`anylut mcp` runs the tool as an MCP server on stdio. It offers the same commands as tools
(`anylut_guide`, `anylut_capabilities`, `anylut_inspect`, `anylut_recipe_new`, `anylut_recipe_show`,
`anylut_patch`, `anylut_lut_set`, `anylut_lut_clear`, `anylut_render_preview`, `anylut_render_export`).
A preview comes back as an image, so the model can look at each edit. Tools need absolute file paths.

Apps that you start from the Dock do not see your shell `PATH`. Use the full path of the tool
(`/opt/homebrew/bin/anylut`; run `command -v anylut` to check).

| Client | Setup |
| --- | --- |
| Claude Desktop | Add to `claude_desktop_config.json`: `{"mcpServers":{"anylut":{"command":"/opt/homebrew/bin/anylut","args":["mcp"]}}}` |
| Claude Code | `claude mcp add anylut -- anylut mcp` |
| Cherry Studio | Settings, MCP Servers, Add. Type `STDIO`. Command `/opt/homebrew/bin/anylut`. Arguments `mcp`. |
| Codex (GPT) | `codex mcp add anylut -- anylut mcp` |

ChatGPT (web and desktop) connects only to remote MCP servers over HTTPS. It cannot start a local
program, so use Codex for GPT models.

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

## Limits

Recipes containing crop, rotation, flip or straighten are rendered as framed, but `patch` cannot set them.
RAW support is whatever your macOS version's RAW decoder
supports; run `anylut inspect` on a file to find out.
