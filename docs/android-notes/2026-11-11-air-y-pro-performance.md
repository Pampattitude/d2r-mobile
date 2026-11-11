# Mangmi AIR Y Pro performance check (2026-11-11)

Device: Mangmi AIR Y Pro, Adreno 650, 1280x960 at 60 Hz. The test used the saved level 9 Paladin in the Rogue Encampment with the same renderer settings and scene throughout.

## CPU affinity

The default fastest-core mask was `0x80` (CPU7, 2.84 GHz). It initially succeeded, but later affinity checks were repeatedly rejected with `EINVAL` after Android changed the app's allowed CPU set. An in-process A/B confirmed that a `0x40` mask (CPU6, 2.42 GHz) was accepted.

During stable gameplay, CPU6 pinned versus released phases averaged about 47.1 versus 46.9 FPS, 12.51 versus 12.58 ms CPU submit time, and 18.80 versus 19.09 ms GPU time. This is a small difference in a GPU-limited scene, not evidence of a broad performance or battery improvement.

The default affinity path now falls back once to the next fastest core class after `EINVAL` (on this device, `0x70`, CPUs4-6). An explicit `gfx-pin-cpu` marker remains strict and does not fall back.

## Power measurement limits

The device remained connected to USB. Its reported battery current varied substantially, while the smoothed current readings were effectively the same in pinned and released phases. This does not establish a battery-life benefit or penalty. No resolution, frame-rate, or image-quality setting was changed.
