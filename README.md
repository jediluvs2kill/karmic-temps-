# Karmic Temps

**KOO's Windows temperature monitor with a dot-matrix dashboard and an always-on desktop overlay.**

Nothing OS-inspired typography meets compact, Winamp-inspired controls: CPU and NVIDIA GPU temperatures, GPU hotspot difference, temperature history, and PresentMon-based FPS integration.

This repository contains the consolidated **v6 development prompt** and screenshots of the working local app.

## Screenshots

### Dashboard

![Karmic Temps dashboard with dot-matrix readouts, temperature-colored history, timeline controls, and overlay settings](docs/screenshots/dashboard.png)

### Desktop overlay

![Karmic Temps compact overlay showing FPS, CPU, GPU, hotspot, and hotspot difference](docs/screenshots/overlay.png)

These are actual v6 app captures, not mockups. They were captured in a restricted development session: GPU and hotspot readings were available, while CPU access required elevation and FPS capture was inactive. CPU temperatures were separately confirmed working by the owner after installing PawnIO. Live in-game FPS remains to be verified.

## Features

- Live CPU and NVIDIA GPU temperature readouts.
- **HOT:** GPU hotspot temperature. **ΔHOT:** hotspot minus core temperature, from the same GPU and sampling cycle.
- Up to one hour of temperature history; selectable **30 seconds, 1, 5, 15, 30, or 60 minutes**.
- A large, resizable graph with **CPU solid** and **GPU dashed** lines.
- Shared temperature colors: **white below 60°C**, **amber from 60°C to below 80°C**, and **red at 80°C or above**. These are visualization bands, not hardware-specific safety limits.
- A movable, click-through overlay that stays above ordinary desktop apps when the dashboard is minimized.
- PresentMon FPS integration with foreground-process detection or manual process selection.
- Pause/resume temperature recording and CSV export, including hotspot and difference.
- Custom vector dot-matrix lettering, dark inset panels, red KOO branding, and compact beveled controls.

## Controls

| Control | Action |
| --- | --- |
| `Ctrl+Shift+O` | Show/hide the overlay and start/stop FPS capture |
| `Ctrl+Shift+M` | Unlock/lock the overlay; drag it while unlocked |
| Timeline | Change the visible history window without clearing retained readings |
| Expand graph | Maximize/restore the dashboard |
| Refresh games | Refresh the process selector |
| Export CSV | Save retained temperature history |

## Development prompt

Use **[DEVELOPMENT_PROMPT.md](DEVELOPMENT_PROMPT.md)** to reproduce or continue developing the app. It consolidates the latest behavior, visual design, sensor requirements, overlay rules, and validation criteria.

## Runtime requirements

- 64-bit Windows with .NET Framework 4.7.2 or later.
- Administrator access for CPU sensors and Windows frame-event capture.
- **LibreHardwareMonitor 0.9.6** plus **PawnIO 2.2.0** for hardware sensor access.
- **PresentMon 2.6.0** console collector for FPS; NVIDIA's installed `nvidia-smi` is a fallback for GPU core temperature.

The overlay targets desktop, windowed, and borderless applications. Exclusive fullscreen, Windows security prompts, and protected surfaces may cover it. Missing sensor or frame data must appear as unavailable, never as invented readings.

## Project status

The local v6 app has been built and checked for live GPU/hotspot readings, graph rendering, timeline switching, CSV output, and overlay behavior. FPS calculations have controlled-data tests; live in-game capture needs an elevated test with a real game. Source and downloadable binaries are not included in this documentation update.

## Credits

- [LibreHardwareMonitor](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor) — hardware telemetry.
- [PawnIO](https://github.com/namazso/PawnIO.Setup) — CPU sensor driver dependency.
- [Intel PresentMon](https://github.com/GameTechDev/PresentMon) — frame presentation telemetry.

Karmic Temps uses its own KOO branding and vector lettering. It is not affiliated with Nothing or Winamp.
