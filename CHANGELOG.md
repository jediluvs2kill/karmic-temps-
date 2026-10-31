# Changelog

## v8 — 2026-10-31

- Expanded from three to seven shared temperature bands, with boundaries at 50, 60, 70, 80, 90, and 100°C. The scale was refined after reviewing exported temperature readings.
- Preserved CPU solid / GPU dashed lines and split historical segments at each color boundary.
- Reduced the widget from 300 × 244 to 250 × 220 pixels, bringing values 16 pixels closer to their labels and tightening rows.
- Added a 0–80% widget transparency slider, defaulting to 10%, with live updates and persistence after normal app closure.
- Verified color boundaries, graph transitions, line styles, minimum/default layouts, transparency persistence and overlay state changes. Refreshed genuine dashboard and widget screenshots.

## v7 — 2026-10-29

- Fixed FPS data being discarded because the parser expected `MsBetweenPresents` while the bundled PresentMon collector emits `msBetweenPresents`.
- Made header matching insensitive to casing and surrounding whitespace, tolerated a BOM, and cached column positions.
- Cleared capture state on restart and added clearer messages for unsupported columns, stopped capture and missing game frames.
- Reproduced the old failure with PresentMon's actual legacy header and verified changing FPS, separate swap chains, process switching, invalid data and stale expiry.
- The owner confirmed FPS working in games after the fix. A separate simulated 240 FPS test verified that the app does not cap readings at 120.

## v6

- Introduced shared temperature-based graph colors while keeping CPU solid and GPU dashed.
- Published the consolidated development prompt and genuine app screenshots.
