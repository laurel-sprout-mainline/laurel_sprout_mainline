# Mi A3 UI timing baseline — 2026-10-12

## Device and method

Measured the connected Mi A3 running Android 16 and kernel
`6.18.32-gf3f89dff1844-dirty #33`, built October 11. Physical display:
720×1560 at 60 Hz. GLES reports Freedreno FD610, Mesa 25.3.3.
HWUI reports Skia OpenGL. Both CPU clusters use schedutil, with maximum
frequencies 1.8048 GHz and 2.016 GHz. Window and transition scales are 1.0;
animator scale is unset. No performance tuning was applied.

The phone was connected to a laptop over USB, awake, with battery temperature
around 33.2°C. Captures used ADB gestures, dumpsys gfxinfo and 25-second
Perfetto SurfaceFlinger FrameTimeline traces, without scrcpy.
Android thermal HAL reports not ready; its status cannot exclude throttling.

Five repetitions per scenario: opening/expanding/collapsing Quick Settings;
cold Settings launches (force-stop between launches, preserving app data);
opening Display settings and returning; scrolling Settings down and up.
Cold-launch counters were collected separately from each new process and summed.
Repeated dumpsys sections were counted only once. The initial reused-activity
launch sample and empty initial QS trace were discarded.

## Results

| Scenario | HWUI janky / rendered frames | HWUI p95 | SurfaceFlinger janky / frames |
| --- | --- | --- | --- |
| Quick Settings | 45 / 309 (14.56%) | 48 ms | 1 / 308 (0.32%) |
| Cold Settings opening | 23 / 50 (46.00%) | 350 ms in each launch histogram | 21 / 429 (4.90%) |
| Display submenu and return | 83 / 163 (50.92%) | 150 ms | 6 / 436 (1.38%) |
| Settings scrolling | 15 / 541 (2.77%) | 32 ms | 0 / 545 (0%) |

Cold Settings activity launch times were 908, 932, 996, 913 and 944 ms:
median 932 ms. Display submenu launches were 680, 389, 343, 351 and 368 ms:
median 368 ms. These activity times are not animation completion times.

Scrolling was the strongest scenario. Quick Settings, cold starts and submenu
transitions need improvement. Submenu gfxinfo also recorded 71 slow UI-thread
and 46 slow draw-command events; these counters alone do not establish a cause.

## Interpretation and limitations

HWUI and SurfaceFlinger counters measure different stages and denominators.
SurfaceFlinger may meet its deadline while presenting reused app content.
Consequently its low jank rate does not cancel app-side misses. HWUI histogram
percentiles include pipeline work; neither those values nor FrameTimeline slice
lifetimes should be inverted to claim instantaneous FPS. Startup samples have
few app frames and include initial drawing, so their percentage is especially
sensitive to startup work.

The host trace processor reports systrace parse failures. FrameTimeline rows
were available, but scheduler/atrace attribution is incomplete and no CPU,
GPU or driver root cause is inferred from it. Trace windows include the
surrounding navigation and idle periods, rather than precisely isolated
animation boundaries. Profiling itself adds overhead.

Only Settings app scrolling/opening was sampled; this is not an all-app
benchmark or a stock-performance comparison. No matching 4.14 baseline exists
for this run. This measures the framework, app, Mesa, compositor and kernel
together, not kernel smoothness in isolation.

Raw traces, gfxinfo, launch outputs and summary.json remain locally under
`out/smoothness-20261012/`; raw traces are not committed. Next useful comparison
is the same gestures on the 4.14 ROM under matching temperature, animation,
charging and refresh-rate conditions, followed by attribution of the slow
transitions with a compatible trace processor.
