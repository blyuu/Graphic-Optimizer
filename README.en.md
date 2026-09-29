<div align="center">

[한국어](README.md) · **English**

# Graphics Optimizer

**Unreal Engine 5.5 plugin for automated graphics scalability benchmarking**

Instead of hand-tuning Scalability settings to hit a target FPS, this runtime/editor plugin measures, searches, and validates the setting combinations for you, inside PIE.

</div>

---

## Overview

`Graphics Optimizer` measures **Shadow / Global Illumination / Reflection / Post Process / Texture / Effects / Resolution Scale** combinations directly in PIE, and finds the setting combination that hits a target FPS with the smallest visual-quality loss.

- **Measured search, not guesswork** — every combination's FPS, 1% low, and Frame/Game/Draw/GPU timings are actually measured and compared.
- **Quality-recovery pass** — once a target-meeting configuration is found, it tries restoring each changed setting one at a time, keeping only restorations that still meet the target.
- **Repeat validation** — the final candidate is re-measured three times; it is only confirmed as "target met" when all three passes agree.
- **Always restores** — every experiment restores the original graphics settings and pause state on completion, cancellation, or PIE teardown.

---

## Key Features

| Feature | Description |
|---|---|
| **Measured setting search (Phase 2)** | Ranks single-setting reductions by measured gain, combines the useful ones, then runs a quality-recovery pass once the target is met |
| **Repeat validation** | Re-measures the selected candidate three times, reports median/range, and only calls it "repeatably met" if all three passes hit the target |
| **GPU pass A/B & trace (Phase 3)** | Independent GI/Shadow reduction A/B, `.utrace` capture with Unreal Insights integration, timing breakdown for major render passes (deferred lighting, Lumen, shadow depths) |
| **Shadow caster scan** | Inventories static-mesh groups currently casting dynamic shadows, then measures a per-group `CastShadow` on/off A/B |
| **VSM / instancing candidate scan** | Detects movable shadow-casting lights, WPO casters, and repeated mesh/material groups (a candidate list, not a measurement) |
| **Lumen screen-probe A/B** | Compares FPS/GPU/image before and after adjusting `r.Lumen.ScreenProbeGather.DownsampleFactor` |
| **Visual A/B capture** | Captures baseline and selected settings from the same camera and produces a side-by-side PNG |
| **Motion A/B (optional)** | Drives automatic A/D input for two 4-second passes and encodes a side-by-side MP4 via `ffmpeg` if it's on PATH |
| **Apply / save preset** | Apply the selected combination temporarily to PIE, or save/reload it as a project-local `.ini` preset; a game-instance subsystem API also supports save/load in packaged builds |
| **Report history** | Every completed or interrupted study is archived under `Saved/GraphicsOptimizer/ReportHistory/`, with project-wide accumulated report copying |

For the full behavioral reference (every panel button, frame-cap handling, render-thread timing caveats, etc.), see [`Plugins/GraphicsOptimizer/README.md`](Plugins/GraphicsOptimizer/README.md).

---

## Getting Started

1. Install **Unreal Engine 5.5**.
2. Clone this repository, or copy the `Plugins/GraphicsOptimizer` folder into your target project's `Plugins/` directory.
3. Right-click `GraphicsOptimizer.uproject` → **Generate Visual Studio project files**.
4. Open `GraphicsOptimizer.sln` and build **Development Editor / Win64**.
5. Launch the editor and confirm the plugin is enabled (`Edit > Plugins > Graphics Optimizer`).
6. Start **Play In Editor** with a fixed camera on a stable scene.
7. Open **Tools > Graphics Optimizer** from the menu.

> To enable motion A/B video encoding, `ffmpeg.exe` must be on the system PATH (PNG frames and the FPS report are still produced without it).

---

## Usage

1. The panel is organized by purpose — **Quality settings** (setting search, result selection, comparisons, apply/save), **Shadow casters and GPU traces**, **Lumen detail and scene structure**, and **Complete analysis and final decision** (runs every study together). Expand only the area you need.
2. Enter a target FPS and measurement duration, then run **Run All Analyses + Copy Report** (everything) or **Run Optimization** (setting search only).
3. During a run, the engine's `stat unit` overlay is enabled automatically and per-viewport raw Frame/Game/Draw/GPU values are sampled. The log category is `LogGraphicsOptimizer`.
4. After completion, use **Copy Settings/Shadow/GPU/Scene/Lumen Report** for one study, or **Copy Accumulated Project Reports** for every archived run in the project.
5. Select a result row to run **Capture A/B: Baseline vs Selected**, **Apply Selected to PIE**, or **Save Selected Preset**. All outputs (PNGs, reports, presets) are written under the project's `Saved/GraphicsOptimizer/`.

> This is a **PIE runtime experimentation tool**. It never saves mesh or level changes, and applying settings or saving a preset is always a separate, explicit action.

---

## Project Structure

```
Graphic-Optimizer/
├─ GraphicsOptimizer.uproject
├─ Source/GraphicsOptimizer/        Base project game module
├─ Plugins/GraphicsOptimizer/
│  ├─ Source/
│  │  ├─ GraphicsOptimizerRuntime/  Measurement/search/apply logic (Runtime module)
│  │  └─ GraphicsOptimizerEditor/   Editor panel UI (Editor module)
│  ├─ EnginePatches/UE5.5/          Reference notes for engine-header related fixes
│  ├─ Docs/BenchmarkResults/        Archived run reports (not tracked by git)
│  └─ README.md                    Detailed plugin usage reference
├─ Config/                          Project settings
└─ Content/                         Test content (assets)
```

---

## Requirements

| | |
|---|---|
| **Engine** | Unreal Engine 5.5 |
| **Language** | C++17 |
| **Platform** | Windows |
| **Optional dependency** | `ffmpeg.exe` on PATH — for motion A/B video encoding |
| **Status** | Beta (`IsBetaVersion: true`) |

---

## Notes

- Every benchmark restores the original graphics settings and pause state on completion, cancellation, or PIE teardown.
- GPU timing can be unavailable on some RHI/editor configurations; GPU comparisons are omitted in that case.
- If the engine's effective max tick rate is near the target FPS, results are reported as "capped / target inconclusive."
- Generated output such as `Docs/BenchmarkResults/`, `Saved/`, `Binaries/`, and `Intermediate/` is excluded via `.gitignore`.
