# Mangmi AIR Y Pro performance check (2026-11-11)

Device: Mangmi AIR Y Pro, Adreno 650, 1280x960 at 60 Hz. The test used the saved level 9 Paladin in the Rogue Encampment with the same renderer settings and scene throughout.

## CPU affinity

The default fastest-core mask was `0x80` (CPU7, 2.84 GHz). It initially succeeded, but later affinity checks were repeatedly rejected with `EINVAL` after Android changed the app's allowed CPU set. An in-process A/B confirmed that a `0x40` mask (CPU6, 2.42 GHz) was accepted.

During stable gameplay, CPU6 pinned versus released phases averaged about 47.1 versus 46.9 FPS, 12.51 versus 12.58 ms CPU submit time, and 18.80 versus 19.09 ms GPU time. This is a small difference in a GPU-limited scene, not evidence of a broad performance or battery improvement.

The default affinity path now falls back once to the next fastest core class after `EINVAL` (on this device, `0x70`, CPUs4-6). An explicit `gfx-pin-cpu` marker remains strict and does not fall back.

## Power measurement limits

The device remained connected to USB. Its reported battery current varied substantially, while the smoothed current readings were effectively the same in pinned and released phases. This does not establish a battery-life benefit or penalty. No resolution, frame-rate, or image-quality setting was changed.

## Full-resolution GPU profile

The saved quality-priority setting pins the internal 3D target to the full 1280x960 output. In the Rogue Encampment, stable windows measured about 40 FPS, 11.8 ms CPU submit time, and 24.5 ms GPU time. The Adreno 650 was at its reported 800 MHz maximum; Android thermal status was 0. The device was USB-powered, so this run does not establish battery draw.

A one-frame pass profile in the camp measured 23.97 ms across graphics passes and 0.68 ms across compute, with a 24.89 ms timestamp span. The main 1280x960 scene pass alone took 13.04 ms for 267 draws and 365,325 vertices. The measured GPU time is well above the 16.67 ms budget for 60 FPS, while CPU submission is below that budget.

## Adreno 650 SPIR-V optimization A/B

Tested `spirv-opt -O` on the four dominant camp fragment shaders. Optimizing the raw modules first changed the helper shape enough that the existing Adreno fragment-subgroup rewrite left subgroup operations in `5c1ca4e7c3e4657c`; the device rejected it at startup. Those temporary overrides were removed. Repeating the experiment in the renderer's order (apply the existing subgroup rewrite, then optimize) passed `spirv-val`, started successfully, and rendered the camp.

At the same camp scene and full 1280x960 resolution, the profiled 1280x960 main pass was 1.066 ms before and 1.081 ms with the candidate (264 versus 262 draws; 356,334 versus 356,199 vertices). This single-frame comparison shows no measurable benefit and is slightly slower within expected run-to-run variation. The optimized candidate was not retained. A sustained gameplay soak and battery comparison were not performed.

No renderer code change was made: this semantics-preserving shader optimization did not improve the measured pass, and the existing renderer already enables the tested exact shader cleanups and Android GPU optimizations. The full-resolution GPU-bound baseline remains about 40 FPS / 24.5 ms GPU time, so further work needs to target a measured source of GPU cost. Lowering the 3D scale can reduce that load, but would override the saved quality-priority behavior and was intentionally not tested.
