<h1 align="center">oklch-neutral</h1>

<h3 align="center">
  Perceptually uniform neutral scales with WCAG 2.1 and APCA contrast auditing.
</h3>

<p align="center">
  By <a href="https://github.com/walebuilds">@Olawale Balo</a> — Product Designer + Design Engineer
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/oklch-neutral"><img src="https://img.shields.io/npm/v/oklch-neutral?color=20C55C&label=oklch-neutral" alt="npm version" /></a>&nbsp;
  <img src="https://img.shields.io/badge/license-MIT-20C55C" alt="MIT license" />&nbsp;
  <img src="https://img.shields.io/badge/TypeScript-strict-20C55C" alt="TypeScript strict" />&nbsp;
  <img src="https://img.shields.io/badge/Dependencies-0-20C55C" alt="Zero dependencies" />
</p>

---

## What is oklch-neutral?

Most color scale tools let you pick a hue and nudge RGB or HSL values. The problem: HSL lightness is not perceptually uniform, so your AA boundary shifts unpredictably across hues, and the toned variants feel visually inconsistent with the original.

oklch-neutral works entirely in the OKLCH color space. It keeps L (lightness) identical per step, same perceived brightness, guaranteed, then introduces a small, tapered chroma at a fixed hue angle. You get warm and cool toned neutrals where the WCAG boundary stays locked and every step looks exactly as bright as its source.

Built for design system authors, token pipeline builders, and Figma plugin developers who need accessible, perceptually sound neutral scales without pulling in a color science framework.

---

## Why OKLCH?

HSL's lightness channel is a lie. 50% HSL yellow looks dramatically brighter than 50% HSL blue, the eye responds differently to different wavelengths at the same mathematical lightness.

OKLCH is perceptually uniform: equal L steps look equal to the human eye across the entire hue range. That's why Tailwind v4, Radix UI, and Linear all moved to OKLCH for their color systems. oklch-neutral gives you the same color science in a zero-dependency library that runs anywhere, Node, browser, Deno, Figma plugins.

---

## Install

```bash
npm i oklch-neutral
```

---

## Quick Start

Generate a slate scale and audit it for WCAG compliance in three lines:

```ts
import { generateScale, auditScale, firstPassingStep, HUE } from 'oklch-neutral'

const gray = {
  "0":   "#FFFFFF",
  "50":  "#FAFAFA",
  "100": "#F5F5F5",
  "200": "#E5E5E5",
  "300": "#D3D3D3",
  "400": "#A1A1A1",
  "500": "#757575",
  "600": "#5C5C5C",
  "700": "#3D3D3D",
  "800": "#262626",
  "900": "#1C1C1C",
  "950": "#0A0A0A",
}

// Generate a cool blue-slate toned scale
const slate = generateScale(gray, { hue: HUE.slate })

// Audit every step against white — OKLCH values + WCAG results
const audit = auditScale(slate)

// Find the first step that passes WCAG AA (4.5:1)
const boundary = firstPassingStep(slate)
// → { step: "500", hex: "#71767B", ratio: 4.59 }
```

---

## Hue Presets

| Preset | Hue angle | Personality |
|--------|-----------|-------------|
| `HUE.sand` | 68° | Yellow-amber warm — earthy, paper-like |
| `HUE.amber` | 68° | Yellow-amber warm — same angle as sand |
| `HUE.stone` | 75° | Orange-warm — warmer than sand, more terracotta |
| `HUE.rose` | 20° | Red-warm — dusty rose, muted red |
| `HUE.slate` | 255° | Blue-slate cool — the workhorse neutral |
| `HUE.sky` | 220° | Lighter blue cool — airy, cloud-like |
| `HUE.teal` | 195° | Teal cool — green-adjacent, calm |
| `HUE.mauve` | 310° | Purple-adjacent — lavender, editorial |

```ts
import { generateScale, HUE } from 'oklch-neutral'

const sand  = generateScale(gray, { hue: HUE.sand  })
const rose  = generateScale(gray, { hue: HUE.rose  })
const slate = generateScale(gray, { hue: HUE.slate })
const mauve = generateScale(gray, { hue: HUE.mauve })
```

---

## Custom Chroma

The default chroma curve tapers at the extremes to preserve white and near-black fidelity. Override per step to dial in exactly how much toning you want:

```ts
// Quieter — barely perceptible toning
const subtle = generateScale(gray, {
  hue: HUE.slate,
  chromaMap: {
    "500": 0.005,
    "600": 0.005,
  }
})

// Stronger — visibly tinted midtones
const strong = generateScale(gray, {
  hue: HUE.sand,
  chromaMap: {
    "400": 0.018,
    "500": 0.020,
  }
})
```

Default chroma curve:

| Step | Chroma |
|------|--------|
| 0    | 0.000  |
| 50   | 0.004  |
| 100  | 0.004  |
| 200  | 0.007  |
| 300  | 0.007  |
| 400  | 0.010  |
| 500  | 0.010  |
| 600  | 0.009  |
| 700  | 0.009  |
| 800  | 0.005  |
| 900  | 0.005  |
| 950  | 0.005  |

---

## Style Dictionary Integration

Drop a generated scale directly into a Style Dictionary token build:

```js
// build-tokens.js
import { generateScale, HUE } from 'oklch-neutral'
import StyleDictionary from 'style-dictionary'

const gray = { /* your pure neutral scale */ }

const slate = generateScale(gray, { hue: HUE.slate })
const sand  = generateScale(gray, { hue: HUE.sand  })

const tokens = {
  color: {
    neutral: Object.fromEntries(
      Object.entries(slate).map(([step, hex]) => [
        step,
        { value: hex, type: 'color' }
      ])
    ),
    warm: Object.fromEntries(
      Object.entries(sand).map(([step, hex]) => [
        step,
        { value: hex, type: 'color' }
      ])
    ),
  }
}

const sd = new StyleDictionary({ tokens, platforms: { css: { /* ... */ } } })
await sd.buildAllPlatforms()
```

The output is plain hex, Style Dictionary handles the rest: CSS custom properties, Tailwind config, JSON, whatever your pipeline needs.

---

## API Reference

### `generateScale(pure, options)`

Generate a toned neutral scale from a pure neutral.

```ts
generateScale(pure: NeutralScale, options: { hue: number; chromaMap?: Partial<ChromaMap> }): NeutralScale
```

| Param | Type | Description |
|-------|------|-------------|
| `pure` | `NeutralScale` | Source scale — `{ "500": "#757575", ... }` |
| `options.hue` | `number` | OKLCH hue angle 0–360 |
| `options.chromaMap` | `object?` | Override default chroma per step |

Returns `NeutralScale` — same shape as input with toned hex values.

---

### `auditScale(scale, background?)`

Audit every step against a background color. Returns OKLCH values, relative luminance, and WCAG contrast results per step.

```ts
auditScale(scale: NeutralScale, background?: string): ScaleAudit[]

// ScaleAudit shape:
// {
//   step: string
//   hex: string
//   oklch: { l: number; c: number; h: number }
//   contrast: {
//     wcag21: { ratio: number; passAA: boolean; passAALarge: boolean; passAAA: boolean }
//     apca:   { lc: number; lcAbs: number; passBodyText: boolean; passLargeText: boolean; passUIElement: boolean; passPlaceholder: boolean }
//   }
// }
```

`background` defaults to `#FFFFFF`. Pass any hex value to audit against dark backgrounds.

---

### `firstPassingStep(scale, background?)`

Find the lightest step that passes WCAG AA (4.5:1). Useful for determining your minimum accessible text color.

```ts
firstPassingStep(scale: NeutralScale, background?: string): { step: string; hex: string; ratio: number } | null

const boundary = firstPassingStep(slate)
// → { step: "500", hex: "#71767B", ratio: 4.59 }
```

Returns `null` if no step passes.

---

### `contrastAudit(foreground, background?)`

Combined WCAG 2.1 + APCA audit in one call. The right function when you need both standards at once.

```ts
contrastAudit(foreground: string, background?: string): ContrastResult

const result = contrastAudit('#71767B', '#FFFFFF')

// WCAG 2.1
result.wcag21.ratio         // 4.59
result.wcag21.passAA        // true
result.wcag21.passAALarge   // true
result.wcag21.passAAA       // false

// APCA (WCAG 3.0 draft)
result.apca.lc              // 68.5  (signed — positive = dark on light)
result.apca.lcAbs           // 68.5
result.apca.passBodyText    // false — needs 75 Lc
result.apca.passLargeText   // true  — passes 60 Lc
result.apca.passUIElement   // true  — passes 45 Lc
result.apca.passPlaceholder // true  — passes 30 Lc
```

---

### `apcaLc(foreground, background?)`

Raw APCA Lc value. Signed, positive means dark text on light background, negative means light text on dark background.

Implements **APCA-W3 0.0.98G-4g** (current Bronze/Simple mode spec).

```ts
apcaLc(foreground: string, background?: string): number

apcaLc('#3A3D42', '#FFFFFF')  // →  91.5 Lc  — fluent body text
apcaLc('#71767B', '#FFFFFF')  // →  68.5 Lc  — large text / subheadings
apcaLc('#9DA2A7', '#FFFFFF')  // →  47.8 Lc  — UI components
apcaLc('#FFFFFF', '#0A0A0A')  // → -104.0 Lc — light on dark
```

---

### `apcaAudit(foreground, background?)`

Full APCA audit — returns Lc value plus all threshold checks.

```ts
apcaAudit(foreground: string, background?: string): APCAResult

// {
//   lc: number
//   lcAbs: number
//   passBodyText: boolean     // 75 Lc
//   passLargeText: boolean    // 60 Lc
//   passUIElement: boolean    // 45 Lc
//   passPlaceholder: boolean  // 30 Lc
// }
```

---

### `wcagAudit(foreground, background?)`

Full WCAG 2.1 audit — contrast ratio plus all level checks.

```ts
wcagAudit(foreground: string, background?: string): WCAGResult

// {
//   ratio: number
//   passAA: boolean       // 4.5:1 — normal text
//   passAALarge: boolean  // 3:1   — large text / UI
//   passAAA: boolean      // 7:1   — enhanced
// }
```

---

### `hexToOklch(hex)` / `oklchToHex(color)`

Direct hex ↔ OKLCH conversion. Full pipeline: sRGB → linear RGB → OKLab → OKLCH and back.

```ts
hexToOklch(hex: string): { l: number; c: number; h: number }
oklchToHex(color: { l: number; c: number; h: number }): string

hexToOklch('#71767B')
// → { l: 0.512, c: 0.008, h: 255.3 }

oklchToHex({ l: 0.512, c: 0.008, h: 255 })
// → "#71767B"
```

---

### `hexToOklab(hex)` / `oklabToHex(color)`

Direct hex ↔ OKLab conversion. Useful when you need the intermediate Lab representation.

```ts
hexToOklab(hex: string): { l: number; a: number; b: number }
oklabToHex(color: { l: number; a: number; b: number }): string
```

---

### `relativeLuminance(hex)`

WCAG relative luminance of a hex color. Linearizes sRGB and applies the WCAG 2.1 formula.

```ts
relativeLuminance(hex: string): number

relativeLuminance('#FFFFFF')  // → 1.0
relativeLuminance('#000000')  // → 0.0
relativeLuminance('#71767B')  // → 0.194
```

---

### `contrastRatio(foreground, background)`

WCAG 2.1 contrast ratio between two hex colors.

```ts
contrastRatio(foreground: string, background: string): number

contrastRatio('#71767B', '#FFFFFF')  // → 4.59
```

---

### `apcaLuminance(hex)`

APCA luminance (Y) of a hex color. Used internally by `apcaLc` — exposed for custom APCA calculations.

```ts
apcaLuminance(hex: string): number
```

---

## APCA Thresholds

APCA uses a signed Lc scale instead of a ratio. Higher absolute value = more contrast.

| Lc (abs) | Use case |
|----------|----------|
| 90+      | Fluent body text, long-form reading |
| 75+      | Body text, normal weight columns |
| 60+      | Large text 18px+ / subheadings |
| 45+      | UI components, icons, non-text |
| 30+      | Placeholder, disabled, decorative |
| 15+      | Incidental / invisible-pass minimum |

> APCA is a draft standard under WCAG 3.0. WCAG 2.1 remains the current legal requirement. Use both for forward-compatible, perceptually grounded decisions, `contrastAudit` returns both in one call.

---

## Output Sample

```
gray  neutral-0     #FFFFFF
gray  neutral-50    #FAFAFA
gray  neutral-500   #757575   4.61:1 ✓

sand  neutral-0     #FFFFFF
sand  neutral-50    #FCFAF7
sand  neutral-500   #79746F   4.62:1 ✓

slate neutral-0     #FFFFFF
slate neutral-50    #F8FAFD
slate neutral-500   #71767B   4.59:1 ✓
```

Every toned step that passes AA does so at the same perceptual lightness as the source neutral, no brightness drift, no shifted AA boundary.

---

## License

MIT © [Olawale Balo](https://github.com/walebuilds)