# tam-file Roadmap

This document outlines the architectural plan and phased roadmap for `tam-file`:
a pure-C11 terminal file manager built on [`libtam`](https://github.com/greenermoose/libtam),
starting as a ground-up rewrite of [Yazi](https://github.com/sxyazi/yazi) and
then diverging to fit Tamlinux workflows.

**Current status:** Initial repository setup. No implementation exists yet.

---

## Architectural Principles

1. **Pure C11 & `libtam` Only:**
   - Static link against `libtam.a` (`tam_str`, `tam_fs`, `tam_fs_tree`,
     `tam_path`, `tam_exec`, `tam_ini`, `tam_tty`, `tam_key`, `tam_uni`).
   - Zero external library dependencies beyond POSIX libc.
2. **Non-Blocking UI & Worker Pool:**
   - Adopt Yazi's core strength: directory reading, file operations, and
     preview generation run off the main `tam_tty` event loop via `poll()`,
     pipes, and/or POSIX worker threads so navigation is always fast.
3. **Filesystem Safety via `tam_fs_tree`:**
   - Use `tam_fs_walk`, `tam_fs_clone_or_copy` (reflink/extent sharing where
     supported), `tam_fs_remove_tree` (never following symlinks or crossing
     filesystems), and `tam_fs_write_atomic` for all mutations.
4. **First-Class Linux VT & Modern Terminal Support:**
   - Full usability on `TERM=linux` (8/16 colors, ASCII/CP437 fallback) as
     well as modern graphical terminals (`foot`, synchronized frames,
     Sixel/Kitty image preview passthrough when available).

---

## Release Versioning Plan

`tam-file` follows [Semantic Versioning (SemVer 2.0.0)](https://semver.org/):

| Version | Phase | Milestone Scope | Status |
| :--- | :--- | :--- | :--- |
| **`v0.1.0`** | **Phase 0** | Repo skeleton, Makefile, ASan/UBSan tests, `tam_tty` 3-pane Miller-column shell | Planned |
| **`v0.2.0`** | **Phase 1** | Directory scanner, sorting/filtering, Vim navigation, opener & `--cwd-file` (**MVP**) | Planned |
| **`v0.3.0`** | **Phase 2** | Multi-select, yank/cut/paste, async task scheduler, `tam_fs_tree` copy/move/delete | Planned |
| **`v0.4.0`** | **Phase 3** | Previewer pipeline (dir/text/archive/image), `fd`/`rg`/`fzf`/`zoxide`, bulk rename | Planned |
| **`v0.5.0`** | **Phase 4** | Multi-tab workspace, visual mode, trash, `--chooser-file`, `keymap.ini` & `theme.ini` | Planned |
| **`v1.0.0`** | **Phase 5** | Tamlinux divergence: `tam-edit`/`tam-data` hooks, git/mount status, hardening | Planned |

---

## Phase 0: Foundation & Three-Pane Compositor (v0.1.0)

**Goal:** Establish the build system, sanitizer harness, INI configuration,
and flicker-free three-pane terminal layout.

- `Makefile` with strict C11 flags (`-std=c11 -Wall -Wextra -Werror`) and
  `make test` under ASan/UBSan.
- `tam_ini` configuration loading (`tam-file.ini`, `keymap.ini`, `theme.ini`)
  via `tam_xdg_path`.
- Three-pane Miller-column compositor built on `tam_tty`, `tam_key`, and
  `tam_uni` (parent pane, current directory pane, preview pane, plus top tab
  bar and bottom status/prompt bar).

## Phase 1: Directory Browser & Navigation MVP (v0.2.0)

**Goal:** Fast, daily-usable directory browsing and file opening.

- Incremental/streaming directory loading with `stat`/`lstat` metadata (size,
  permissions, mtime, symlink targets) and sorting (natural, alphabetical,
  size, mtime, extension, dirs-first).
- Vim navigation (`h`/`j`/`k`/`l`, `gg`/`G`, `Ctrl+u`/`Ctrl+d`, `~` home,
  `-` previous dir), incremental filter (`f`), and find (`/`, `?`, `n`, `N`).
- File opener rules via `tam_exec` (MIME/extension dispatch) and
  `--cwd-file` for seamless shell `cd`-on-exit integration.

## Phase 2: Async File Operations & Task Scheduler (v0.3.0)

**Goal:** Safe, non-blocking file manipulation with live progress tracking.

- Selection model (`Space` toggle, `v` visual range, `Ctrl+a` select all,
  `Ctrl+r` invert) and cross-directory yank/cut registers (`y`/`x`/`p`/`P`).
- Background task scheduler (worker queue + `poll()` pipe notifications)
  with a task manager overlay (`w`) showing progress, cancellation, and errors.
- File mutations powered by `tam_fs_tree`: `tam_fs_clone_or_copy`, `rename`,
  `tam_fs_remove_tree`, `mkdir` (`a`), `chmod`, and symlink creation.

## Phase 3: Previews, External Search & Bulk Rename (v0.4.0)

**Goal:** Rich file inspection and power-user search/rename workflows.

- Scrollable preview pane (`J`/`K`) with caching and cancellation on rapid
  cursor movement:
  - Directory listing preview
  - UTF-8 / binary-safe text preview (with optional external highlighter via
    `tam_exec`)
  - Archive/PDF/media metadata & text previews via external CLI tools
  - Optional Sixel/Kitty/Chafa image preview when supported by the terminal
- External tool integration via `tam_exec`: `fzf` jump, `zoxide` jump,
  `fd` file search, `rg` content search.
- Bulk rename (`r` on multiple selected files) opening paths in `$EDITOR`
  (`tam-edit`) and validating/applying renames atomically.

## Phase 4: Multi-Tab, Trash, Chooser & Customization (v0.5.0)

**Goal:** Complete core Yazi feature parity for daily use.

- Multi-tab workspace (`t`, `1`-`9`, `[`/`]`, `Ctrl+c` close tab).
- FreeDesktop Trash support (`~/.local/share/Trash`) alongside permanent
  delete (`D`).
- `--chooser-file` mode for `xdg-desktop-portal` / `tam_portal` / CLI picker
  integration.
- `which-key` style popup hint overlay on multi-key prefixes (`g`, `c`, `d`,
  `z`).

## Phase 5: Tamlinux Divergence & v1.0.0 Hardening (v1.0.0)

**Goal:** Move beyond a pure Yazi clone to tailor `tam-file` for Tamlinux.

- First-class Tamlinux ecosystem integration:
  - Seamless handoff to `tam-edit` (text/config), `tam-data` (CSV/TSV/JSON),
    and `tam-mail` (attachments/Maildir).
  - Lightweight built-in git status indicator and mount/unmount view without
    requiring a Lua plugin runtime.
- Deliberate omissions from Yazi (see [ideas/yazi.md](ideas/yazi.md)): no Lua
  plugin engine/package manager, no DDS pub-sub server, and no Windows code.
- Stress testing on 100k-entry directories, ASan/UBSan fuzzing, and published
  cold-start / RSS benchmarks.
