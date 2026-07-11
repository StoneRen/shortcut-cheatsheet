# AGENTS.md — 快捷键速查维护规范

本项目是一个静态快捷键速查站点（`index.html`），聚合 **WezTerm / tmux / Neovim** 三类终端快捷键，通过 GitHub Pages 直接网页查看。

## 核心原则（强制）

> **所有快捷键数据必须通过读取本地配置文件生成，禁止手工编辑 `index.html` 内的 `DATA` 数据。**

`index.html` 中的 `DATA` 对象是「生成产物」，不是手工维护的源。任何键位变更都必须从本地配置重新导出后再同步到 `DATA`，而不是直接改 HTML 里的条目。手改 `DATA` 会被下次重新生成覆盖。

## 数据来源与生成方式

### WezTerm
- 配置文件：`~/.config/wezterm/wezterm.lua`（软链到 `~/workspace/github/MyDevSettings/wezterm.lua`）
- 自定义键：解析 `wezterm.lua` 中的 `keys = { ... }` / `key_tables`，标注 `src: "custom"`
- 默认键：执行 `wezterm show-keys --lua`（macOS）导出实际生效按键，标注 `src: "def"`
- 修饰键记法：`Cmd+Shift+X`（`Cmd`=⌘ `Shift`=⇧ `Ctrl`=⌃ `Alt`=⌥）

### tmux
- 配置：已回归默认（**无自定义 `tmux.conf`**，仅保留 `set -g mouse on` 与 `default-terminal`）
- 前缀：`C-b`（tmux 原生默认）
- 生成命令：
  - `tmux -f /dev/null list-keys -T prefix`
  - 复制模式键：`tmux -f /dev/null list-keys -T copy-mode`
- 记法：`C-b d`（前缀 + 功能键）；复制模式内 `C-b` 表示 `Ctrl+b` 而非前缀

### Neovim (LazyVim)
- 配置目录：`~/.config/nvim`（LazyVim，含 `lua/`、`lazy-lock.json`、`lazyvim.json`）
- 默认键来源：已安装 LazyVim 源码 `LazyVim/lua/lazyvim/**` 的 keymap 定义
- 插件键来源：`lazy-lock.json` 中锁定的插件各自定义的 keymap（snacks.nvim、blink.cmp、claudecode.nvim、gitsigns、flash、which-key 等）
- 内置命令来源：本机 Neovim runtime help（如 `:help diff-mode` / `:help copy-diffs`），仅用于记录 `git difftool` 进入两栏 diff 后的 Neovim 内置 diff 操作（如 `]c` / `[c` / `do` / `dp`）
- 生成方式：读取上述源码中的 `vim.keymap.set` 与 LazyVim/插件的 `keys` spec，并读取 Neovim 内置 help 中的 diff-mode 操作，按分类归整；每条标注 `mode`（n / x / v / i / o / c / t / s，可多值空格分隔）
- `leader` = `<Space>`，`localleader` = `\`

## 数据结构约定

`index.html` 的 `DATA = { wezterm:[...], tmux:[...], nvim:[...] }`：

| 工具     | 条目字段                         | 徽章列         |
| -------- | -------------------------------- | -------------- |
| WezTerm  | `{ src, key, desc, note }`       | 来源（自定义/默认） |
| tmux     | `{ src, key, desc, note }`       | 来源（默认）   |
| Neovim   | `{ mode, key, desc, note }`      | 模式（n/x/...）|

- `src`：`"custom"`（来自本地配置）/ `"def"`（默认）
- `mode`：vim 模式字符，可多值空格分隔，如 `"n x o"`

更新数据时保持字段与现有格式一致。渲染按字段**自适应分发**：条目有 `mode` 走 nvim 渲染器（解析 `<leader>`/`<C-/>`/`a / b` 备选键）+ 模式徽章；有 `src` 走修饰键渲染器（解析 `Cmd+Shift+X`）+ 来源徽章。两套渲染器互不影响。

## 本地预览

```sh
open index.html
```

## 部署

GitHub Pages，仓库 `StoneRen/shortcut-cheatsheet`，访问 <https://stoneren.github.io/shortcut-cheatsheet>。
推送 `main` 分支即自动重新构建并更新线上页面。
