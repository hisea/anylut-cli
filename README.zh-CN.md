# anylut

[English](README.md) | **中文**

`anylut` 是一个命令行工具，用来对照片做非破坏性编辑，为本地 agent 和脚本设计。它读取照片（RAW、JPEG 或 HEIC），让你编辑一个小而无损的配方（recipe）文件，再渲染出预览图或最终的 JPEG/PNG。原始文件不会被修改。

本仓库存放 macOS 的**预编译、已签名并已公证的二进制文件**，源代码不在这里公开。

## 安装

```bash
brew install hisea/anylut/anylut
```

需要 Apple Silicon Mac，系统为 macOS 14（Sonoma）或更高版本。

升级：`brew upgrade anylut`。

Homebrew 会要求你信任非官方 tap。上面的命令写出了完整的公式名，只信任这一个公式。如果你先执行 `brew tap hisea/anylut`，又想用短名安装，需要显式信任：

```bash
brew tap hisea/anylut
brew trust --formula hisea/anylut/anylut
brew install anylut
```

## 在 agent 中使用

skill `anylut` 教 agent 如何运行这个工具。它让 agent 先运行 `anylut guide`，所以说明始终与已安装的版本一致。请先安装工具（见上）。

| Agent | 安装 |
| --- | --- |
| Claude Code | `claude plugin marketplace add hisea/anylut-cli`，然后 `claude plugin install anylut@anylut` |
| Codex | `codex plugin marketplace add hisea/anylut-cli`，然后 `codex plugin add anylut@anylut` |
| pi | `pi install git:github.com/hisea/anylut-cli` |
| omp | `omp plugin marketplace add hisea/anylut-cli`，然后 `omp plugin install anylut@anylut` |
| opencode | `git clone https://github.com/hisea/anylut-cli /tmp/anylut-cli && mkdir -p ~/.config/opencode/skills && cp -R /tmp/anylut-cli/skills/anylut ~/.config/opencode/skills/` |

其他能读取 `SKILL.md` 的 agent，可以直接使用 `skills/anylut/` 文件夹。

## 作为 MCP 服务使用

`anylut mcp` 让工具作为 MCP 服务，通过 stdio 运行。它把相同的命令作为工具提供出来：`anylut_guide`、`anylut_capabilities`、`anylut_inspect`、`anylut_recipe_new`、`anylut_recipe_show`、`anylut_patch`、`anylut_lut_set`、`anylut_lut_clear`、`anylut_render_preview`、`anylut_render_export`。预览会作为图片返回，模型可以看到每次编辑的效果。工具的文件路径必须是绝对路径。

从 Dock 启动的应用看不到 shell 的 `PATH`，所以请使用工具的完整路径（`/opt/homebrew/bin/anylut`，可用 `command -v anylut` 确认）。

| 客户端 | 配置 |
| --- | --- |
| Claude Desktop | 在 `claude_desktop_config.json` 中加入：`{"mcpServers":{"anylut":{"command":"/opt/homebrew/bin/anylut","args":["mcp"]}}}` |
| Claude Code | `claude mcp add anylut -- anylut mcp` |
| Cherry Studio | 设置 → MCP 服务器 → 添加。类型选 `STDIO`，命令填 `/opt/homebrew/bin/anylut`，参数填 `mcp`。 |
| Codex（GPT） | `codex mcp add anylut -- anylut mcp` |

ChatGPT（网页版和桌面版）只连接通过 HTTPS 提供的远程 MCP 服务，不能启动本地程序，所以 GPT 模型请使用 Codex。

## 快速开始

```bash
anylut capabilities --json                        # 列出所有可写参数及其范围
anylut inspect IMG.ARW --json
anylut recipe new --source IMG.ARW --output edit.anylut.json
anylut patch edit.anylut.json --patch-file adjust.json --output edit-v1.anylut.json --json
anylut render IMG.ARW --recipe edit-v1.anylut.json --preview --output preview.jpg --json
anylut render IMG.ARW --recipe edit-v1.anylut.json --full-resolution --output final.jpg --json
```

补丁（patch）是一个小的 JSON 文件（也可以用 `--patch-file -` 从 stdin 读取）：

```json
{ "patchVersion": 1, "operations": [
  { "op": "set", "path": "light.exposure", "value": 0.35 },
  { "op": "add", "path": "color.temperature", "value": 6 },
  { "op": "set", "path": "hsl.orange.saturation", "value": -5 } ] }
```

`set` 给出绝对值，`add` 给出相对当前值的变化。超出范围的值会报错，不会被悄悄截断。`anylut --help` 列出所有命令。

## 保证

- 使用 `--json` 时，每个命令在 stdout 上只输出一行、一个 JSON 对象。出错时是一个 `{"ok":false,"error":{...}}` 对象，带有稳定的 `code`。退出码：`0` 成功，`2` 用法错误，`3` 不支持的文件，`4` 校验失败，`5` 解码失败，`6` 渲染失败，`7` 编码或写入失败。
- 输出文件以原子方式写入；除非传入 `--overwrite`，否则不会覆盖已有文件。
- 编辑文件会被严格校验：未知、缺失或重复的键，以及超出范围的值，都会被拒绝，不会被自动修复。
- 渲染完全在你的 Mac 上进行，不会上传任何内容。

## 限制

含有裁剪、旋转、翻转或拉直的配方会被拒绝，而不是忽略这些编辑去渲染。RAW 支持取决于你的 macOS 版本自带的 RAW 解码器；对文件运行 `anylut inspect` 可以查看是否支持。
