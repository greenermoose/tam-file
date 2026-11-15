# Ideas from Yazi

## Overview

- **What it is:** A terminal file manager based on non-blocking async I/O,
  featuring a three-pane Miller-column layout, task scheduler, previews,
  multi-tab workspace, and external tool integrations.
- **Developers:** Created and maintained by `sxyazi` and contributors.
- **License:** MIT License.
- **Language:** Rust (with an embedded Lua plugin runtime).
- **Source Code:** <https://github.com/sxyazi/yazi>.

---

## Ideas We Borrow

### 1. Three-Pane Miller-Column UX & Vim Ergonomics
- Parent / Current / Preview three-pane layout with clean visual hierarchy.
- Intuitive Vim-like bindings (`h`/`j`/`k`/`l`, `gg`/`G`, `Space` select,
  `v` visual mode, `y`/`x`/`p` yank/cut/paste, `/` find, `f` filter) plus
  `which-key` prefix hints (`g`, `c`, `d`, `z`).

### 2. Non-Blocking Async Task Scheduler
- Never blocking the UI on slow directory reads, large file copies/deletes,
  or preview generation.
- Dedicated task manager overlay (`w`) showing active/queued tasks, progress,
  and cancellation.

### 3. Pre-Loading & Cancellable Previews
- Cancelling stale preview jobs immediately when the user scrolls rapidly
  through a directory.
- Scrollable preview pane (`J`/`K`) supporting directories, text, archives,
  and external renderers.

### 4. Bulk Rename via `$EDITOR` & Shell Integration
- Selecting multiple files and editing their names in `$EDITOR` (`tam-edit`),
  applying validated renames cleanly.
- `--cwd-file` for `cd`-on-exit and `--chooser-file` for file-picker mode.

### 5. External Tool Composition
- Delegating specialized search/jump tasks to `fd`, `ripgrep`, `fzf`, and
  `zoxide` rather than reimplementing them poorly.

---

## Ideas We Adapt

### 1. Implementation Stack & Concurrency
- **Yazi:** Multi-crate Rust workspace (`yazi-fm`, `yazi-core`,
  `yazi-scheduler`, `yazi-vfs`, etc.) using Tokio async runtime.
- **tam-file:** Pure C11 + `libtam.a` (`tam_tty`, `tam_key`, `tam_uni`,
  `tam_fs_tree`, `tam_exec`, `tam_ini`) with a `poll()` event loop and
  lightweight worker pipes/threads.

### 2. Previews & Code Highlighting
- **Yazi:** Links Rust image decoders (`image`), syntax highlighting crates,
  and multiple image protocols directly into the binary.
- **tam-file:** Keeps the binary small and `libc` + `libtam`-only by
  rendering text/directory previews natively and delegating syntax/PDF/image
  previews via `tam_exec` to optional external tools (`chafa`, `pdftotext`,
  etc.), with clean fallback on `TERM=linux`.

### 3. Configuration
- **Yazi:** Uses TOML (`yazi.toml`, `keymap.toml`, `theme.toml`).
- **tam-file:** Uses `tam_ini` (`tam-file.ini`, `keymap.ini`, `theme.ini`)
  for consistency across all Tamlinux tools.

---

## Ideas We Avoid

- **Embedded Lua Plugin System & Package Manager (`ya`):** Adds a scripting
  runtime, FFI surface (`yazi-binding`), and package-management complexity
  that conflict with Tamlinux's zero-bloat C11 philosophy. Useful features
  (like git status or mount info) will be built cleanly in C or via `tam_exec`.
- **DDS (Data Distribution Service) Pub-Sub Architecture:** Cross-instance
  Lua pub-sub state syncing (`yazi-dds`) adds complexity not needed for a
  lean single-instance file manager.
- **Built-In SFTP / VFS Stack (`yazi-sftp`, `yazi-vfs`):** Remote filesystems
  are better handled at the OS/VFS layer (e.g., FUSE/sshfs) or external CLI
  tools than by embedding an SSH/SFTP library in the file manager.
- **Cross-Platform Windows Support:** Unnecessary complexity; `tam-file`
  targets Linux/POSIX only, with first-class `TERM=linux` VT support.
