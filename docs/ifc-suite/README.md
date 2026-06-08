# Jackdaw → IFC/CAD suite integration

Tracking folder for using **Jackdaw** (this fork) as the desktop editor for the
IFC/CAD suite in [`ifc-ubuntu`](https://github.com/joeblew999/ifc-ubuntu).

- [PLAN.md](./PLAN.md) — what we're building and the architecture decisions (with the evidence behind them).
- [TRACKING.md](./TRACKING.md) — live status: our action items, upstream issues/PRs we watch, and our upstream PRs.

## One-paragraph summary

Jackdaw is a Bevy 0.18 desktop level editor. We use it as the **authoring surface**
for the suite's CAD (STEP, via `cadrum`) and BIM (IFC) geometry, bridged through
**glTF** — which `ifc-ubuntu` already produces (`cadrum-gltf` + `IfcConvert`, with
element identity baked into node extras/names). Our work lives **downstream** as a
host binary + extension; Jackdaw core stays **pristine upstream** (zero patches).
The authoritative CAD/BIM model stays in **our** format — Jackdaw is for editing,
not the system of record — to insulate us from upstream's `.jsn`→BSN churn.

## Repo layout (current)

- `main` — pristine mirror of upstream `jbuehler23/jackdaw` (0 divergence).
- `joeblew999` — our overlay branch: **only** `mise.toml` (tooling) + this `docs/ifc-suite/` folder. No code patches.

Last updated: 2026-06-08.
