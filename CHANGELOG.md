# Changelog

All notable changes to this project will be documented in this file.

## [1.0.2] — 2026-03-20

### Changed

- Removed `src/` from published files — only `dist/`, `README.md`,
  and `LICENSE` are now included in the install
- Added `repository`, `bugs`, and `homepage` fields — issue tracker
  and source links now appear correctly on the npm page
- Expanded keywords for better discoverability on npm
- README rewritten with hero section, badges, Why OKLCH explanation,
  full API reference, and Style Dictionary integration example

## [1.0.0] — 2026-03-11

### Added

- Full OKLCH color pipeline — sRGB → OKLab → OKLCH and full reverse,
  zero dependencies
- `generateScale()` — perceptually uniform toned neutral scale from
  a pure neutral. Preserves L per step, introduces tapered chroma
  at a fixed hue angle
- Default chroma taper curve — peaks at midtones, tapers at extremes
  so whites and near-blacks stay clean
- `chromaMap` option — override chroma per step independently
- `HUE` presets — sand, amber, stone, rose, slate, sky, teal, mauve
- `contrastAudit()` — combined WCAG 2.1 + APCA audit in one call
- `wcagAudit()` — contrast ratio, passAA, passAALarge, passAAA
- `apcaAudit()` — body text, large text, UI elements, placeholder
  thresholds per APCA-W3 0.0.98G-4g
- `apcaLc()` — signed Lc value, positive for dark-on-light
- `auditScale()` — audit every scale step with full contrast results
- `firstPassingStep()` — find the lightest WCAG AA passing step
- Color conversion utilities — `hexToOklch`, `oklchToHex`,
  `hexToOklab`, `oklabToHex`, `relativeLuminance`, `contrastRatio`
- TypeScript interfaces for all core shapes — `OKLCHColor`,
  `OKLabColor`, `NeutralScale`, `WCAGResult`, `APCAResult`,
  `ContrastResult`, `ScaleAudit`
- Full declaration maps for IDE go-to-definition support