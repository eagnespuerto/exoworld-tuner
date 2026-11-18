# ExoWorld design system

This file sets ExoWorld's font, colours and component rules. Everything lives
in one file, `index.html`. Colours are CSS variables on `:root`, so change a
role there and every panel follows.

## Font (Google Fonts)

| Role | Family | Weights | Use |
|---|---|---|---|
| Everything | [Manrope](https://fonts.google.com/specimen/Manrope) | 400, 500, 600, 700, 800 | Headings, body text, controls, numbers |

One family keeps the page light and consistent. Manrope ships tabular
figures, so every readout (`.val`, the gauge, the stage labels) sets
`font-variant-numeric: tabular-nums` and numbers stay put while a slider
moves. Its `1`, `l` and `I` are distinct enough for values like `1.11 R⊕`.

Scale (px): 11 caption · 12.5 control label · 14 body · 15 panel title ·
48 gauge number · `clamp(28px, 4.2vw, 44px)` page title. Titles use weight
800 with -0.02em tracking. Small uppercase labels use 600 with +0.14em.

## Brand colours

Four owner-chosen colours, dark to light:

| Name | Hex | RGB |
|---|---|---|
| Navy | `#09122C` | `rgb(9, 18, 44)` |
| Wine | `#872341` | `rgb(135, 35, 65)` |
| Crimson | `#BE3144` | `rgb(190, 49, 68)` |
| Salmon | `#E17564` | `rgb(225, 117, 100)` |

**How they work together.** Navy is the night sky: the dark-mode page and the
system diagram. Wine, crimson and salmon run as a warm ramp from deep to
bright, so they read as one family. Each one does a single job:

- **Salmon** is the dark-mode primary: slider thumbs, the selected spectral
  button, value chips, focus rings.
- **Crimson** is the light-mode primary and the brand mark. On navy it fails
  text contrast (3.3:1), so in dark mode it only fills shapes and borders.
- **Wine** is the light-mode link and heading accent. It also sits at the
  deep end of the title gradient.

## Derived colours

The tier colours carry meaning (good / borderline / poor), and a palette of
four warm reds cannot signal "good". So the tiers get two colours of their
own:

| Name | Dark | Light | Built from | Role |
|---|---|---|---|---|
| `tide` | `#5CC8B0` | `#1C7462` | Crimson's complement (hue ≈ 170°) | Promising tier, conservative HZ |
| `ochre` | `#E8B25A` | `#8A5800` | Salmon turned toward yellow | Marginal tier |
| `salmon` / `crimson` | `#E17564` | `#BE3144` | Brand | Unlikely tier |
| `dust` | `#9A97B0` | `#5B5F78` | Navy lifted, desaturated | Very unlikely tier |

The spectral ramp (M red → O blue) on the star buttons and the diagram is
**data colour**. It shows real stellar temperature, and themes never
override it.

## Roles

| Role | Dark | Light |
|---|---|---|
| `--bg` | `#09122C` | `#FBF3F0` |
| `--surface` | `#0F1A3A` | `#FFFFFF` |
| `--surface-2` | `#16234A` | `#F5E6E1` |
| `--line` | `#26345F` | `#E6D2CB` |
| `--ink` | `#F6ECE8` | `#09122C` |
| `--muted` | `#B9B3C6` | `#4E5470` |
| `--faint` | `#8F8BA6` | `#62667F` |
| `--primary` | `#E17564` | `#BE3144` |
| `--on-primary` | `#09122C` | `#FFFFFF` |
| `--link` | `#EE9A8B` | `#872341` |
| `--brand-deep` | `#872341` | `#872341` |
| `--brand` | `#BE3144` | `#BE3144` |

## Contrast (WCAG 2.1)

Every text pair clears AA (4.5:1) on all three surfaces. Measured on the
darkest surface each role sits on:

| Pair | Dark | Light |
|---|---|---|
| `ink` on `surface-2` | 13.2 | 15.2 |
| `muted` on `surface-2` | 7.5 | 6.1 |
| `faint` on `surface-2` | 4.7 | 4.6 |
| `primary` on `surface-2` | 5.1 | 4.7 |
| `on-primary` on `primary` | 6.1 | 5.7 |
| tier colours on `surface` | ≥ 5.6 | ≥ 5.6 |

Salmon on light backgrounds (2.8:1) and crimson on navy (3.3:1) are
**non-text only**.

## Themes

Two themes, Dark (navy) and Light (blush). The page follows the system
setting until someone presses the header toggle. The choice is kept in
`localStorage` as `exoworld-theme`. The system diagram stays navy in both,
because the spectral colours only read correctly on a dark sky.

## Components

- **Panels**: `--surface`, 1px `--line` border, 20px radius, no gradient.
  The header strip of each panel holds a 16px icon in `--primary` and a
  600-weight title.
- **Sliders**: 6px track in `--surface-2`, filled part in `--primary` up to
  the thumb (set by `--fill` from JavaScript). 18px thumb.
- **Segmented buttons** (spectral class): the selected one fills with its
  spectral colour and uses navy text.
- **Value chips**: `--primary` text on a 12% tint of `--primary`, tabular
  figures.
- **Sub-score rows**: name, label and weight on one line, then a 7px bar
  coloured by score (tide ≥ 0.7, ochre ≥ 0.45, salmon ≥ 0.2, crimson below).
  Click a row to show its explanation. Rows are `<button>`s, so they work
  from the keyboard.
- **Focus**: 2px `--primary` outline, 2px offset, on every control.
- **Icons**: [Lucide](https://lucide.dev) 0.460.0 from unpkg, 16px.

## Motion

Bars and the gauge ease over 400 ms. Under `prefers-reduced-motion: reduce`
all transitions are off.
