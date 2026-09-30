# CLAUDE.md — working in this repo

This is a **joeblew999 fleet repo**. Before touching mise tasks, CI, releases, or
skills, **read `joeblew999/.github`'s `AGENTS.md`** — it defines the flows. Work
*with* them; do not reinvent.

## The flows (from .github — distribution is BY REFERENCE, never file-copy)

- **mise tasks** ← `[task_config].includes = ["git::…/tasks/<ns>.toml?ref=vX"]` in `mise.toml`
- **CI** ← `.github/workflows/mise.yaml` → `uses: …/reusable-mise-ci.yml@vX` (runs one `mise run <task>`)
- **global tools** ← `mise run mise:global`
- **claude skills** ← Claude Code plugin marketplace (not per-repo files)

If you're about to clone/copy `.github` content into this repo, STOP — you're
reinventing; the mechanism already exists above.

## This repo (jackdaw fork)

- Fork of `jbuehler23/jackdaw` (Bevy 3D editor). `main` = **pristine upstream
  mirror**; our work lives on the **`joeblew999`** branch as an **overlay**:
  `mise.toml` + `docs/ifc-suite/` + `.github/workflows/*`. **Zero core patches** —
  keep it that way (upstreamable fixes go as PRs, not fork diffs).
- **Toolchain:** nightly via `rust-toolchain.toml` (rustup), **not** mise.
  `[tools]` = `cmake` only; dev-only `cargo:` tools are pinned **per-task**
  (installing them in CI breaks Windows). Host deps via `setup:deps`.
- **Build:** `mise run build` (self-contained via `setup:deps`). CI = `mise.yaml`
  → reusable-mise-ci across ubuntu/macos/windows.
- **Release:** tag-dispatch `release.yaml` → `release:stage` (build + `release:pack`)
  per OS → `release:github`. Binaries: `jackdaw` + `jackdaw-rustc-wrapper`,
  archives `jackdaw_<ver>_<os>_<arch>.tar.gz`. Version scheme `vX.Y.Z-jb.N`.

## House rules

- **RUN tasks, don't just parse** — verify locally before pushing (esp. before any release).
- Never `git push`/commit without explicit approval.
- `.github` is version-protected: refactor it deeply, don't surface-patch.
