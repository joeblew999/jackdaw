# Proposal: optional mise-based dev + CI (one command, cross-platform, from scratch)

> Draft for an upstream issue on `jbuehler23/jackdaw`. Framed as a proposal per
> CONTRIBUTING ("chat about overarching changes first"), not a blind PR.

## Motivation

Building Jackdaw from a clean checkout currently trips new contributors on three
things that aren't surfaced in one place:

1. **Nightly toolchain** — already pinned in `rust-toolchain.toml` ✅, so this one
   is handled (as long as nothing overrides it — see the gotcha below).
2. **cmake** — `manifold-csg-sys`'s build script shells out to cmake. Without it,
   `cargo build` fails partway through with a non-obvious error. CI doesn't notice
   because GitHub runners ship cmake.
3. **Bevy's per-OS system deps** — `libasound2-dev libudev-dev libwayland-dev` on
   Linux, MSVC Build Tools on Windows, Xcode CLT on macOS.

There's no single "make me build" command, and CI re-encodes these prerequisites
separately from local setup, so the two can drift.

## Proposal

An **optional, additive** `mise.toml` (+ a small CI workflow) so that:

```sh
mise run build      # works from a clean checkout on Linux, macOS, and Windows
```

…and **CI runs the exact same command**. This doesn't replace `cargo` or the
existing CI — it's purely opt-in tooling for contributors who want a one-command
setup and for keeping local/CI in lockstep.

## How it works

- **Tools** (cmake, nextest, bevy_cli) are declared in `mise.toml` and
  auto-installed/version-pinned by [mise](https://mise.jdx.dev).
- **Toolchain** stays owned by the existing `rust-toolchain.toml` + rustup —
  **not** mise. ⚠️ Heads-up worth documenting: mise selects a Rust toolchain by
  exporting `RUSTUP_TOOLCHAIN`, which is rustup's highest-precedence override and
  silently defeats `rust-toolchain.toml`. So `rust` must **not** be a mise tool;
  rustup auto-installs the pinned nightly on first `cargo` call.
- **Host system deps** are installed by an OS-dispatching `setup:deps` task
  (apt/dnf/pacman · Xcode CLT · MSVC via winget), idempotent with a fast no-sudo
  guard, which the build tasks depend on. That's what makes `mise run build`
  self-contained per OS.
- **Tasks are written in nushell** so they behave identically on every OS (no
  bash-isms like `cat`/`find`/`$VAR`/`/tmp`).

## Local: the task menu

```
$ mise tasks
setup            First-time setup: build editor + SDK dylib + rustc wrapper
build            Build the editor
build:dylib      Build with dylib/extension-loading support
build:release    Build in release mode
open             Launch the editor with full extension support
open:project     Open a project directory directly
run              Launch the editor (quick)
new:extension    Scaffold a new extension
new:game         Scaffold a new game project
test             Run all tests
fmt / clippy     Format / lint
ci               Full local CI (fmt + clippy + test + doc)
dist:install     Install the published release from crates.io
clean            Remove build artifacts (target/); clean:dist, clean:all too
…                (plumbing like setup:deps is hidden; `mise tasks --hidden` shows it)
```

## CI: the same command, three desktops

Standalone, dependency-free workflow (one matrix job, `mise run build`). In our
own fleet we run the equivalent via a shared reusable workflow across repos, but
the version below has no external dependency — drop-in for upstream.

```yaml
name: mise-desktop-build
on: { push: { branches: [main] }, pull_request: { branches: [main] }, workflow_dispatch: {} }
jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    steps:
      - uses: actions/checkout@v4
      - if: runner.os == 'Linux'    # free disk for the large Bevy build
        run: sudo rm -rf /usr/share/dotnet /usr/local/lib/android /opt/ghc /opt/hostedtoolcache/CodeQL
      - uses: jdx/mise-action@v2
        with: { cache: true }
      - uses: Swatinem/rust-cache@v2
        with: { shared-key: mise-desktop-build-${{ matrix.os }} }
      - run: mise run build        # setup:deps installs host deps per OS, then cargo build
```

## Why this is nice

- **One command, from scratch, any desktop** — `mise run build` installs the host
  deps it needs and builds. No README checklist of apt packages.
- **Local ↔ CI parity** — CI literally runs `mise run build`; it can't drift from
  what contributors run locally.
- **Cross-platform by construction** — nushell tasks + OS-dispatching `setup:deps`.
- **Additive** — leaves `cargo` and the current CI untouched.

## Offer

Reference implementation (full files) on our fork branch:
- `mise.toml`: https://github.com/joeblew999/jackdaw/blob/joeblew999/mise.toml
- workflow: https://github.com/joeblew999/jackdaw/blob/joeblew999/.github/workflows/mise-desktop-build.yaml

Happy to open a PR if this is something you'd want upstream. Flagging here first
to check interest and scope.
