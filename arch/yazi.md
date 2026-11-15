# Yazi Architecture & `tam-file` Adaptation Notes

`tam-file` is currently in pre-`v0.1.0` planning (`ROADMAP.md`), with no C source files yet. This document records the verified data structures and algorithms of **Yazi** (`/home/fred/Code/yazi`, tracked in `ideas/yazi.md`) and how `tam-file` plans to adapt them to pure C11 + `libtam.a`.

---

## 1. Async Task Scheduler

### Yazi (`yazi-scheduler`)
- **`Scheduler`** (`yazi-scheduler/src/scheduler.rs`) wraps **`Worker`** (`yazi-scheduler/src/worker.rs`) and `Behavior`, running a pool of Tokio `Vec<JoinHandle<()>>` tasks.
- **`Worker::make()`** creates separate `async_priority_channel::unbounded()` priority queues (`LOW`, `NORMAL`, `HIGH`) per domain (`file`, `plugin`, `fetch`, `preload`, `size`, `process`, `hook`), plus an `mpsc::unbounded_channel()` (`op_tx`/`op_rx`) for `TaskOp` progress updates.
- **`Ongoing`** (`yazi-scheduler/src/ongoing.rs`) indexes active jobs in `hashbrown::HashMap<Id, Task>` keyed by monotonically generated `Id`s.
- **`Task`** (`yazi-scheduler/src/task.rs`) holds `TaskHandle`, a hierarchical cancellation **`Scope`**, `TaskProg`, optional `HookIn`, and a log channel. `Worker::work()` uses `tokio::select! { biased; ... }` to race completion, cancellation, and execution.

### Planned `tam-file` Adaptation (`ROADMAP.md`)
- Replace Tokio + multi-channel async runtime with a **non-blocking `tam_tty` `poll()` event loop**, worker threads/processes, and **pipe notifications** (`v0.3.0`), keeping a `w` task-overlay queue for copy/move/delete/chmod and `tam_fs_tree` (`tam_fs_clone_or_copy`, `tam_fs_remove_tree`).

---

## 2. Directory Entries, Sorting, Tabs & Selection

### Yazi (`yazi-fs`, `yazi-core`)
- **`File`** (`yazi-fs/src/file/file.rs`) pairs `url: UrlBuf`, `stat: Stat` (`StatKind` bitflags such as `DUMMY`/`HIDDEN`, `StatMode`, `len`, timestamps, `dev`/`uid`/`gid`/`nlink` in `yazi-fs/src/stat/stat.rs`), and `extra: FileExtra` (e.g., `lstat` for symlinks).
- **`Files`** (`yazi-fs/src/file/files.rs`) is `Vec<File>`. **`Entries`** (`yazi-fs/src/entries.rs`) partitions `hidden: Files` and `items: Files`, tracks `ticket: Id` and `revision: u64` for incremental/streaming updates (`update_full`, `update_part`, `update_size`, `update_rank`, `update_fail`), and caches directory `sizes` and `ranks` in `HashMap`s.
- **`FilesSorter`** (`yazi-fs/src/sorter.rs`) uses `slice::sort_unstable_by` with optional `dir_first` promotion across `SortBy` modes (`None`, `Mtime`, `Btime`, `Extension`, `Alphabetical`, `Natural` via `yazi_shared::natsort`, `Size`, `Random` via `SmallRng`, `Custom`).
- **`Folder`** (`yazi-core/src/tab/folder.rs`) wraps `file`, `entries`, `stage: FolderStage`, `offset`, `cursor`, `page`, and `trace`.
- **`Tab`** (`yazi-core/src/tab/tab.rs`) holds `current`, `parent: Option<Folder>`, **`Backstack`** (`backstack.rs`, capped `Vec<UrlBuf>`), **`History`** (`history.rs`, `IndexMap<UrlBuf, Folder>` LRU-evicted above 100 entries with a parent-child `indices` map), and **`Selected`** (`selected.rs`, `IndexMap<FileCov, u64>` + `parents: HashMap<UrlBufCov, usize>` to prevent selecting both a parent and its child).

### Planned `tam-file` Adaptation (`ROADMAP.md`)
- Incremental/streaming directory scanner with `stat`/`lstat` (`v0.2.0`), natural/alpha/size/mtime/extension/dirs-first sorting, 3-pane Miller-column compositor (`tam_tty`, `tam_key`, `tam_uni`), multi-tab workspace (`v0.5.0`), and cross-directory selection/yank/cut/paste (`v0.3.0`).

---

## 3. Preview Cache & Cancellation

### Yazi (`yazi-core`)
- **`Preview`** (`yazi-core/src/tab/preview.rs`) tracks `lock: Option<PreviewLock>`, `skip: usize`, `folder_lock: Option<UrlBuf>`, and a child **`Scope`** that cancels in-flight preview jobs when the cursor moves.
- **`PreviewLock`** (`yazi-core/src/tab/preview_lock.rs`) caches rendered `data: Vec<Renderable>` with `url`, `sig: Id`, `scope`, `skip`, and `area: Rect`.
- **`PreviewSig`** (`yazi-core/src/tab/preview_sig.rs`) hashes `(FileSig(file), mime, area)` via `FsHash64` (`Preview::same_file` / `same_lock`) for O(1) cache hit/invalidation checks.

### Planned `tam-file` Adaptation (`ROADMAP.md`)
- Scrollable preview pane (`v0.4.0`) with caching and cancellation on rapid cursor movement, native directory/UTF-8/binary-safe text previews, and external previewers (`chafa`, `pdftotext`, etc.) via `tam_exec`.

---

## 4. Virtual Filesystem, MIME & Key Routing

### Yazi (`yazi-vfs`, `yazi-core`, `yazi-config`, `yazi-fm`)
- **`yazi-vfs`** (`yazi-vfs/src/engine/engines.rs`) dispatches VFS traits across `enum Engines<'a>` (`Local`, `Lua`, `Sftp`). (`tam-file` intentionally **avoids** SFTP/Lua VFS in favor of local POSIX/`libtam` `tam_fs_tree`.)
- **`Mimetype`** (`yazi-core/src/mgr/mimetype.rs`) caches MIME types in `hashbrown::HashMap<UrlBufCov, Symbol<str>>`, populated asynchronously by `Scheduler::fetch_mimetype()`.
- **`Keymap`** (`yazi-config/src/keymap/keymap.rs`) stores per-layer `KeymapSection`s (`mgr`, `tasks`, `spot`, `pick`, `input`, `confirm`, `help`, `cmp`). **`Router::route()`** (`yazi-fm/src/router.rs`) intercepts modal layers first, then linearly scans `KEYMAP.chords(src)`: single-key chords dispatch immediately, while multi-key chords activate the `which` (`WhichOpt`) hint overlay.
- `tam-file` plans `tam_ini`-based `keymap.ini` modal bindings and `which-key` prefix hints (`g`, `c`, `d`, `z` in `v0.5.0`).
