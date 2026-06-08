# Tracking — Jackdaw IFC/CAD integration

Live status board. Update the date when you touch it. Last updated: **2026-06-08**.

## Our action items

| # | Item | Status | Notes |
|---|---|---|---|
| 1 | Update fork to upstream, clean dirty merge | ✅ done | `main` = upstream `2a8ee17` (v0.4.1); fast-forward, 0 divergence |
| 2 | Fix build toolchain (cmake + nightly) | ✅ done | `cmake` in mise `[tools]`; nightly via `rust-toolchain.toml` (rustup-owned) |
| 3 | Reduce fork to zero core patches | ✅ done | `joeblew999` = upstream + `mise.toml` + these docs only |
| 3b | Per-desktop CI build (mise) | ✅ done | `.github/workflows/mise.yaml` calls the shared `joeblew999/.github` **reusable-mise-ci.yml@v0.35.1** (fleet convention) → `mise run build` on ubuntu/macos/windows. setup:deps handles host deps; nightly via rust-toolchain.toml. mise.toml `[task_config].includes` pulls the shared mise:* tasks. |
| 4 | Prove glTF import seam end-to-end | ☐ todo | Load an `ifc-ubuntu` `.glb` (cadrum STEP + IFC) into running Jackdaw |
| 5 | Scaffold `jackdaw_ifc` extension | ☐ todo | import operator + IFC inspector panel (reads baked `source`/`GlobalId`) |
| 6 | Scaffold `jackdaw_host` binary | ☐ todo | `EditorPlugin + JackdawIfc`; later `bevy_pmetra` |
| 7 | Onboard Jackdaw as `ifc-ubuntu` module #10 | ☐ todo | overlay `repos/jackdaw/` + `tasks/jackdaw/` referencing this fork |

## Upstream issues we watch (jbuehler23/jackdaw)

| Issue/PR | Why it matters to us | Status |
|---|---|---|
| BSN migration (`bsn-editor` branch; Bevy PR [#23413](https://github.com/bevyengine/bevy/pull/23413)) | The `.jsn`→BSN scene-format swap is our biggest churn risk. Branch parked since 2026-03-30, so `.jsn` is stable near-term. | watch |
| [#194](https://github.com/jbuehler23/jackdaw/issues/194) operatorify (merged) | Upstream is consolidating *all* editor actions onto the operator API our extension uses — good, we're building on the canonical seam. | context |
| [#38](https://github.com/jbuehler23/jackdaw/issues/38) RFC: Dynamic Wing System (`A-Extension`, `C-Design-Doc`) | Shows the RFC/design-doc process — the path if we ever propose a scene-registration API upstream. | reference |
| [#287](https://github.com/jbuehler23/jackdaw/issues/287) viewport nav broken / [#271](https://github.com/jbuehler23/jackdaw/issues/271) can't close scenes / [#248](https://github.com/jbuehler23/jackdaw/issues/248) crash | Active usability bugs — "early, expect bugs". May affect our testing. | watch |
| [#183](https://github.com/jbuehler23/jackdaw/issues/183) Windows dylib PE export cap | Reason we chose host-binary over cdylib. | resolved-for-us |

## Our upstream PRs (candidates / open)

| PR | Scope | Status |
|---|---|---|
| cmake build prerequisite doc | Add "requires `cmake`" to CONTRIBUTING/book build section (universal — GitHub CI has it, local devs hit it). Provider-agnostic, blind-PR-able. | candidate, not filed |
| mise dev+CI proposal | Optional mise tooling: one-command cross-platform local build + matching CI. Draft at `docs/upstream/issue-mise-local-ci.md`. Issue/RFC first per CONTRIBUTING (not a blind PR — maintainer doesn't use mise). | drafted, not filed |

Note: the two earlier core patches (`MenuAction` reflectable, `bevy_cli` doc tag pin) were
**dropped** on 2026-06-08 — verified unneeded (MenuAction is an observer event, no reflection
used; bevy_cli already pinned in our `mise.toml`). So nothing code-level to upstream.

## Decisions log

- **2026-06-08** — Host-binary delivery (not cdylib): persistence + cross-OS + enables bevy_pmetra. See PLAN §2.
- **2026-06-08** — CAD/IFC stays downstream forever; only generic build-prereq docs are upstreamable. See PLAN §1.
- **2026-06-08** — Keep authoritative model in our format, project to Jackdaw for editing only. See PLAN §4.
- **2026-06-08** — Fork reduced to zero code patches.
