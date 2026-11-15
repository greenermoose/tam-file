# Publication preparation, 2026-11-15

- **Tool**: Codex CLI 0.169.0, checked with the runtime detector.
- **Model**: GPT-6.1 Sol (`gpt-6.1-sol`), detected from the current runtime.
- **Transcript**: Retained privately by the author.
- **Prepared source commit**: `c36fd7e5dd7068272ed1fb4fbbb8281f2dca0e07`.
- **Authorship boundary**: This session removed local reference paths from
  unpublished documentation history and prepared publication. Existing
  implementation authorship is unchanged and is not inferred from Git names.

## Guiding prompts

> Push tam-data, tam-edit, tam-file, tam-mail, and tam-ring to their public github repos.

> Remove paths before publishing

## Changes and verification

Removed a workstation checkout path from the upstream ideas note, retaining
its public source URL. Rewrote only unpublished history; no force push is
needed. Original authors and messages are preserved, with Codex trailers
added to commits whose documentation changed. The original history remains
in a local archive ref and is excluded from publication.

Verified the documentation diff and checked unpublished commits for local
paths. Untracked packaging files were left untouched. No application source
changed; no build or runtime tests were needed for this documentation edit.
