# Karmic Temps — consolidated v8 development prompt

Build or continue **Karmic Temps**, a small native Windows desktop temperature monitor branded **KOO**. Deliver a runnable application, editable source, build instructions, and genuine screenshots. Use the screenshots in `docs/screenshots/` as the current visual reference. This prompt describes the latest agreed behavior; implement it rather than producing only a design or plan.

## Platform and packaging

Use C# WinForms targeting 64-bit Windows and .NET Framework 4.7.2 or later. The current implementation is organized as `KarmicTemps.cs`, `Overlay.cs`, and `Theme.cs`, with `app.manifest`, `KarmicTemps.exe.config`, `koo.ico`, and `build.ps1`. These names describe the existing local implementation; this repository update supplies the brief and screenshots, not those source files.

Produce a portable ZIP that runs after extraction. Keep sensor dependencies in `sensors/` and the PresentMon console executable in `fps/`. Use an application manifest requesting administrator rights. Bundle dependency licenses and source links. Download dependencies only from their official releases, pin versions, and verify provided checksums or signatures. Do not install drivers silently.

## Real hardware readings

1. Poll approximately once per second on a background task. Keep the UI responsive, prevent overlapping polls, and release resources on shutdown.
2. Read CPU temperature through LibreHardwareMonitor. Show the hottest valid CPU temperature, excluding sensors named “Distance to TjMax” or other distance-to-limit measurements.
3. Read NVIDIA GPU **core** temperature and **GPU Hot Spot** from the same adapter and hardware update.
4. Calculate **hotspot difference = hotspot − core**, in °C. Display hotspot and difference to one decimal place. Do not confuse hotspot with memory-junction temperature.
5. For multiple NVIDIA GPUs, select the adapter with the highest valid core reading and retain that adapter's paired hotspot. Never subtract readings from different adapters.
6. Use `nvidia-smi` only as a GPU core fallback when the library has no valid core reading. Do not infer a hotspot from that fallback.
7. Missing, zero, invalid, stale, or inaccessible readings must appear as a dash/unavailable state and as gaps in history. Never substitute fake temperatures or silently treat missing values as zero.
8. Detect the missing PawnIO dependency. Offer **Set up CPU** using the official installer and clear instructions to reopen the app afterward. Explain actual sensor errors instead of repeatedly suggesting administrator mode when the driver is absent.

The development machine is an Intel Core i9-13900HX / NVIDIA GeForce RTX 4070 Laptop GPU. CPU readings were confirmed working by the owner after PawnIO installation. Support other compatible hardware without hardcoding that machine's sensor values.

## Dashboard and graphs

- Default size approximately 1200×820; resizable with a minimum size around 900×600.
- Give the history graph most of the window. Include **Expand graph / Restore size**.
- Show prominent CPU and GPU core readings, plus GPU hotspot and hotspot difference.
- Retain up to one hour of timestamped temperature samples in memory. History starts fresh on launch.
- Timeline selector: **30 sec, 1 min, 5 min, 15 min, 30 min, 1 hour**. Switching views must not discard retained samples.
- Use a shared 0–120°C vertical scale, clear time labels, and a visible legend.
- Distinguish devices by line style: **CPU solid; GPU dashed**. Dashes must remain visible even with dense one-second samples; do not restart a dash at every tiny segment.
- Both series share the same temperature-based colors:

  | Temperature | Color |
  | --- | --- |
  | Below 50°C | Blue (`#70A8FF`) |
  | 50°C to below 60°C | Cyan (`#51D7EB`) |
  | 60°C to below 70°C | Green (`#86DE84`) |
  | 70°C to below 80°C | Yellow (`#F2D763`) |
  | 80°C to below 90°C | Orange (`#FF9E4D`) |
  | 90°C to below 100°C | Red (`#FF4B40`) |
  | 100°C and above | Pink (`#F174DA`) |

- Color each historical section according to its temperature, splitting segments at all six boundaries (50, 60, 70, 80, 90, and 100°C) in either direction. Preserve gaps across missing readings and recording pauses. These are display bands, not universal thermal safety thresholds.
- Provide **Pause / Resume** and **Export CSV**. CSV columns: timestamp, CPU core/hottest sensor, NVIDIA core, NVIDIA hotspot, hotspot difference. Use ISO timestamps, invariant decimal formatting, and blank fields for unavailable values. Export all retained temperature samples. FPS export is not part of v8.

## Always-on overlay

- Show the overlay automatically on normal launch. It stays visible when the dashboard is minimized and closes when the app exits.
- Display **FPS, CPU, GPU, HOT, and ΔHOT**, plus the tracked application and a compact status line.
- Use a borderless, topmost, slightly translucent window, initially around 250×220 pixels near the top-left of the display.
- Use compact label/value columns (labels near x=12, values near x=58) and approximately 28-pixel row spacing. Fit readings such as `108.4 °C` without clipping.
- Add a dashboard **Widget transparency** slider: 0–80%, default 10%, where 0% is fully opaque. Apply it immediately to the entire widget, including text, without changing dashboard opacity. Keep at least 20% opacity so the widget cannot become completely invisible.
- Save the transparency setting on normal app closure in `%LOCALAPPDATA%/KarmicTemps/widget-transparency.txt`; restore it at startup. Clamp out-of-range values, fall back to 10% for missing/invalid settings, and tolerate storage errors. Preserve the chosen opacity when hiding/showing or locking/unlocking the widget.
- Keep the transparency slider and capture status visible at both 900×600 and 1200×820 dashboard sizes.
- Locked mode must be click-through and must not steal keyboard focus.
- `Ctrl+Shift+O`: toggle visibility and FPS capture.
- `Ctrl+Shift+M`: unlock/lock; allow dragging while unlocked. Provide equivalent dashboard controls if hotkeys are unavailable.
- Keep the overlay above ordinary desktop applications without activating it. Permit dragging between monitors.
- Be honest about the boundary: exclusive fullscreen, secure Windows prompts, other desktops, and protected surfaces can cover a normal desktop overlay. Recommend windowed/borderless game mode where necessary. Do not claim universal visibility.
- Pausing temperature sampling must not show old temperatures as live. Mark them paused/unavailable while FPS capture can continue.

## Actual FPS measurement

- Integrate the official **PresentMon 2.6.0 x64 console collector**, using Windows graphics frame events. No separate full PresentMon UI installation should be needed.
- Measure the selected application's presentation rate, not display refresh rate, graph redraw rate, or the app's own timer frequency.
- Support automatic foreground-process tracking and an explicit process selector with **Refresh games**.
- The current integration reads PresentMon CSV using `--output_stdout --no_console_stats --v1_metrics --no_track_gpu --no_track_input --no_track_display` and a unique `--session_name`.
- Parse CSV robustly, including quoted fields, header whitespace, and a possible BOM. Match column names case-insensitively and cache column positions. Filter by the selected `ProcessID`. PresentMon 2.6.0's legacy output uses `TimeInSeconds`, **`msBetweenPresents`** (lowercase `ms`), and `SwapChainAddress`. The earlier case-sensitive expectation `MsBetweenPresents` discarded every frame; preserve the v7 fix and test with the exact collector header.
- Reset both header state and frame samples when capture restarts. Distinguish unsupported CSV columns, stopped capture, and no recent frames from the selected process. Name the selected process in dashboard status.
- Calculate approximately one-second average app-present FPS as `1000 / mean(valid frame interval in milliseconds)` for the most active swap chain. Do not add rates from different swap chains.
- Clear old process measurements when the target changes. Expire stale frame data after roughly 2.5 seconds. Desktop FPS is not applicable; no recent valid data means a dash.
- Show capture errors clearly, including access-denied errors. Treat app-present FPS as distinct from displayed FPS, especially with dropped or generated frames.
- Keep only the needed in-memory frame data. Stop the app's own named capture session and child process when disabling the overlay or exiting. Avoid affecting other PresentMon users/sessions.
- The owner confirmed working in-game FPS after the v7 parser fix. Automated coverage still uses controlled frame streams; do not claim universal game compatibility. There is no 120 FPS display cap. Preserve support for 240 FPS and above without confusing the monitor's refresh rate with measured game FPS.

## Visual direction: dot matrix × classic Winamp

Use a **Nothing OS-inspired dot-matrix display combined with a compact Winamp-style equipment panel**. Retain original KOO branding and Karmic Temps naming; do not copy another product's logo.

- Graphite background around `#151515`, inset displays around `#0A0A0A`.
- Warm off-white text around `#EBE9E1`, muted gray around `#999994`, red accent around `#FF4B40`, amber around `#FFB84D`.
- Render headings, key readings, and the KOO mark as scalable 5×7 vector dot-matrix glyphs. No font installation should be required.
- Use a readable monospace font for controls, legends, status messages, and detailed values.
- Header: red dotted KOO mark, dotted KARMIC TEMPS title, subtle horizontal grille lines.
- Compact rectangular buttons with understated bevels, visible keyboard focus, dark dropdowns, and inset graph/readout panels.
- Apply the same theme to the overlay. Temperature readouts can use the same shared color bands; the hotspot *difference* is a separate metric and must not be treated as an absolute GPU temperature.
- Keep decoration restrained. No fake animated equalizer, invented sensor activity, promotional cards, or decorative data.
- Retain usable labels, keyboard controls, accessible control names, adequate contrast, and clean resizing. Temperature color is supplemented by numeric labels, and device identity by line style.

## Validation and handoff

Build the release executable and portable ZIP. Verify live GPU/core/hotspot readings where accessible and distinguish actual readings from test fixtures. Test missing-sensor behavior and CSV output across locales; test values just below and exactly at every 50/60/70/80/90/100°C color boundary, line styles over dense samples, color crossings, missing-data gaps, all timeline views, and minimum-size layout. Check overlay show/hide, click-through locking, dragging, hotkeys, and visibility while the dashboard is minimized. Check transparency endpoints, immediate slider updates, saved-setting roundtrips, corrupt/missing setting fallback, and opacity across lock/show transitions. Check FPS using the exact PresentMon 2.6.0 legacy CSV header, changing frame rates including 240 FPS, process filtering, swap-chain handling, invalid CSV, target changes, capture reset, unsupported-column diagnostics, and stale-data expiry. Run an elevated game capture when available.

Take genuine screenshots of the final dashboard and overlay. If the capture environment lacks CPU access or active FPS capture, state that in the caption instead of editing in plausible values. Keep test-only simulated graphs out of normal app screens and release screenshots.

Deliver a concise change summary, screenshots, setup/build instructions, dependency attribution, validation results, and any remaining limitations. Preserve working sensor and overlay behavior while iterating on the visual design.
