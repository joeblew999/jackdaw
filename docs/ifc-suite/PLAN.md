# Plan — Jackdaw as the IFC/CAD suite editor

Status legend: ✅ verified · 🔬 hypothesis (not yet proven) · ⚠️ risk

## Goal

Use Jackdaw as the **desktop authoring editor** for the `ifc-ubuntu` CAD+BIM suite.
Jackdaw is complementary to the suite's web viewers (three.js / web-ifc / occt-wasm):
those are the **share/embed** surface; Jackdaw is the **desktop edit** surface. They
meet at glTF (geometry) and, later, our own model format (semantics).

## Architecture decisions

### 1. Downstream layer, not an upstream feature ✅
CAD/IFC is **out of scope** for upstream Jackdaw (a TrenchBroom/Hammer-lineage *game*
level editor, aligned to the official Bevy Editor). Confirmed: zero issues/PRs mention
glTF/import/CAD/IFC; the maintainer's own "open challenges" lists only *game* asset
formats (FBX/USD), "not started". So our layer lives downstream **permanently**.

### 2. Host binary, not a cdylib extension ✅
Two delivery models for a Jackdaw extension:
- **cdylib** (depends on `jackdaw_api` only) — hot-reload, but `dylib` feature is off by
  default and **not green on Windows** (PE export cap, upstream #183). Worse: it **cannot**
  persist newly-imported geometry to the scene (the registration fn isn't in `jackdaw_api`).
- **static plugin in our own host binary** (depends on the full `jackdaw` crate) — upstream's
  "current stable path", cross-OS clean, and **can** call `jackdaw::scene_io::register_entity_in_ast`
  (it's `pub fn`, `pub mod scene_io` in lib.rs:71) to persist imported entities. Also the only
  model that can also add the `bevy_pmetra` plugin.

→ **We build a host binary.** `App::new().add_plugins((DefaultPlugins, EditorPlugin, JackdawIfc, /* later */ PmetraPlugin))`.

### 3. glTF is the import bridge ✅
`ifc-ubuntu` already emits `.glb` from both sides with identity baked in:
- `cadrum-gltf` (pure Rust): STEP → glTF, `extras{source}`
- `IfcConvert`: IFC → glTF, `GlobalId` in node names
Jackdaw loads glTF natively (`GltfSource{path,scene_index}` in `jackdaw_jsn`). So import is
mostly "run the suite's glTF task, open the result, attach our metadata components".

### 4. Our model stays in our format ⚠️→mitigation
Keep the authoritative CAD/BIM model in **our own** serialization; project meshes into Jackdaw
scenes for editing only. Do **not** couple our data to `.jsn`/BSN internals. This insulates us
from the single biggest risk: the scene format is the unfinished, churning part on *both* sides
(Jackdaw `.jsn`→BSN swap + Bevy's BSN itself, PR #23413, still pre-stable).

## What the public extension API supports ✅ (verified in v0.4.1)

| Capability | Verdict | Entry point |
|---|---|---|
| Custom IFC-metadata components, editable in Inspector | ✅ | `#[derive(Reflect)]` auto-appears; custom sections via `ExtensionContext::extend_window()` |
| "Import IFC/STEP" menu action + pipeline | ✅ | `register_operator` + `register_menu_entry(File)`, operators get `&mut World` |
| Persist custom components | ✅ | scene saver serializes all non-skipped reflect components |
| Persist imported geometry entities | ✅ via host binary | `jackdaw::scene_io::register_entity_in_ast` (not in `jackdaw_api` → host-binary only) |

## Build prerequisites (this fork) ✅

mise + rustup are not self-contained — the build also needs host system deps mise
can't manage. `mise run setup:deps` (OS-dispatching, idempotent) installs them, and
the build tasks depend on it, so `mise run build` is self-sufficient on each desktop.

- **nightly toolchain** — pinned in `rust-toolchain.toml` (`nightly-2026-03-05`, `try_trait_v2`). Owned by rustup, not mise.
- **cmake** — `manifold-csg-sys` (the Manifold CSG kernel) build script needs it. Declared in `mise.toml [tools]`.
- **host system deps** (via `setup:deps`):
  - Linux: `libasound2-dev libudev-dev libwayland-dev` (Bevy default features) + `build-essential` + `pkg-config`
  - macOS: Xcode Command Line Tools (clang)
  - Windows: MSVC Build Tools (C++)
- `mise run build` → 334 MB debug binary in ~8.5 min from cold (macOS, verified).

## Roadmap (phases)

1. **Editor runs** ✅ — fork on upstream 0.4.1, `mise run build/open` works, zero core patches.
2. **Prove the import seam** 🔬 — load a real `ifc-ubuntu` `.glb` (cadrum STEP + IFC) into a running
   Jackdaw and edit it. Empirical, not yet done.
3. **`jackdaw_ifc` extension** 🔬 — import operator(s) + an IFC inspector panel reading baked `source`/`GlobalId`.
4. **`jackdaw_host` binary** 🔬 — `EditorPlugin + JackdawIfc`; later add `bevy_pmetra` (also Bevy 0.18) for parametric edit.
5. **Onboard as `ifc-ubuntu` module #10** 🔬 — overlay pattern (`repos/jackdaw/` + `tasks/jackdaw/`), referencing this fork.

## Maintenance model ✅

- Fork = `main` (pristine upstream) + `joeblew999` (mise overlay + these docs). **Zero code patches.**
- Rebase onto each Jackdaw/Bevy release (~quarterly Bevy cadence). Only `mise.toml` could ever conflict.
- Budget one larger "BSN cutover" event whenever upstream finally executes the `.jsn`→BSN swap.
