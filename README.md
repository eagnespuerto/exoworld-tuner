# ExoWorld Habitability Tuner

[![license](https://img.shields.io/badge/license-CC0-blue)](LICENSE.md)
[![made with](https://img.shields.io/badge/made%20with-HTML5-E34F26?logo=html5&logoColor=white)](https://github.com/eagnespuerto/exoworld-tuner)
[![website](https://img.shields.io/badge/website-live-brightgreen)](https://eagnespuerto.github.io/exoworld-tuner/)
[![Ko-fi](https://img.shields.io/badge/Ko--fi-support%20me-FF5E5B?logo=kofi&logoColor=white)](https://ko-fi.com/eagnespuerto)
[![Linktree](https://img.shields.io/badge/Linktree-soitsagnes-43E55E?logo=linktree&logoColor=white)](https://linktr.ee/soitsagnes)

**Live demo: [eagnespuerto.github.io/exoworld-tuner](https://eagnespuerto.github.io/exoworld-tuner/)**

ExoWorld scores how habitable a planet might be. Use it to sanity-check a
real candidate, to build a believable world for a story, or to show a class
why some planets can hold liquid water and others can't.

It is one static file. Open `index.html` in any modern browser, or host it on
GitHub Pages. The scoring is a port of the **HCI v2** engine in VetStar
([`vetstar_core/habitability.py`](https://github.com/eagnespuerto/vetstar/blob/release/v2/vetstar_core/habitability.py)
on `release/v2`). The look is set out in [DESIGN.md](DESIGN.md).

## What changed with HCI v2

VetStar v2 moved to a new version of the Habitability Chance Index, and
ExoWorld follows it.

| v1 (removed) | Weight | v2 (added) | Weight |
|---|:-:|---|:-:|
| ExoFOP TOI disposition | 15% | Cosmic shoreline | 10% |
| TESS sectors with a dip | 5% | Tidal locking | 5% |
| | | System age | 5% |

The two v1 terms measured how sure we are of the *detection*, not anything
about the *planet*. A confirmed hot Jupiter got full marks for being
confirmed. The three v2 terms are physical: can the planet hold on to its air
under its star's XUV, does it spin, and is the star settled.

The disposition menu is still there. It no longer changes the score, but an
FP or FA disposition adds a warning to the caveats. Planet size, habitable
zone, stellar type, vetting flags, the density modifier and the EB cap are
the same as before.

## How to use

Pick a known planet from the menu, or build your own with the sliders: orbit,
radius, mass, host star, system age and vetting verdict. The HCI updates as
you move them. Click any of the seven score rows to see how that number was
worked out.

Presets cover the Solar System, the seven TRAPPIST-1 planets, the best-known
habitable-zone candidates, sub-Neptunes, hot Jupiters and an eclipsing-binary
false positive. Each one loads its real host star.

### Host star

Three ways to set the host:

1. **Spectral-class buttons** (M / K / G / F / A) use class-average `Teff`,
   `R★` and `M★`. Good for quick what-ifs.
2. **Custom** lets you type `Teff` (K), `R★` (R⊙) and `M★` (M⊙). The HZ,
   the stellar-type score and the star's colour follow `Teff`. Mass sets the
   tidal-locking radius and the main-sequence lifetime.
3. **Real star menu** loads measured values from the literature (Sun,
   Proxima Centauri, TRAPPIST-1, Alpha Centauri A/B, Tau Ceti, the Kepler and
   TOI hosts and more) and switches to Custom. Use this to put a made-up
   planet around a real star: pick the host, then drag the orbit and radius.

Changing only the star (a class button or a real-star pick) moves the orbit
slider to the middle of that star's conservative HZ, so you start on a
habitable orbit and drag in or out from there.

### System age

The age slider runs from 0.05 to 13 Gyr. Stars with a well-established age
(Sun, Alpha Centauri, Proxima, TRAPPIST-1, Kepler-186, -442 and -452) load
it. For the rest, *Age unknown* is ticked: the age term goes neutral (0.5)
and tidal locking assumes a mid-life age, the smaller of 4.5 Gyr and half
the main-sequence lifetime.

### Preset clears on edit

The preset menu is a snapshot. Touch any planet or detection control and it
resets, so the page never claims you are looking at TRAPPIST-1e after you
have moved the sliders.

### System diagram

The panel above the gauge is a top-down view of the orbital plane. The star
sits at the left, and the conservative and optimistic HZ are drawn as bands
around it. The green dashed circles are the conservative HZ edges (Kopparapu
et al.), and the salmon dashed circle is the tidal-locking radius. The planet
is a half-lit disc with its day side toward the star.

### Themes

Dark (navy) and Light. The page follows your system setting until you press
the toggle, and remembers the choice in `localStorage`. The diagram stays
navy in both, because space doesn't have a light mode and the spectral
colours only read right on a dark sky.

---

## The HCI v2 calculation

The HCI is a weighted average of seven sub-scores, each on `[0, 1]`, scaled
to `0–100`. A bulk-density modifier then adds or subtracts up to 10 points,
and a hard cap catches eclipsing binaries.

```
HCI = (Σ wᵢ · sᵢ) / (Σ wᵢ) · 100  +  density_modifier
```

| Component | Weight | What it measures |
|---|:-:|---|
| Planet size | 30% | STEHM atmosphere-retention regime |
| Habitable zone | 25% | Orbit against the Kopparapu HZ edges |
| Stellar type | 15% | How well the STEHM calibration carries over |
| Vetting flags | 10% | Pipeline verdict (planet / EB / blend) |
| Cosmic shoreline | 10% | XUV instellation against escape velocity |
| Tidal locking | 5% | Orbit against the tidal-locking radius |
| System age | 5% | Age against the star's main-sequence life |

### 1. Planet size (30%), STEHM

Thresholds from Hill et al. (2026), *STEHM*, `arXiv:2605.00170`:

- `Rp > 2.2 R⊕` → `s = 0.05` (likely sub-Neptune, not rocky)
- `Rp < 0.5 R⊕` → `s = 0.02` (atmosphere retention ≈ 0)
- `Rp ≥ 0.8 R⊕` → `s = 0.75 + 0.25 · min((Rp − 0.8)/0.2, 1)` (favourable)
- `0.7 ≤ Rp < 0.8` → `s = 0.35 + 0.40 · (Rp − 0.7)/0.1` (marginal)
- `0.5 ≤ Rp < 0.7` → `s = 0.05 + 0.30 · (Rp − 0.5)/0.2` (unlikely)

For planets between 0.5 and 2.2 R⊕, the size score is multiplied by a
density factor:

- `ρ/ρ⊕ ≥ 0.6` → ×1.1 (rocky-consistent)
- `ρ/ρ⊕ < 0.4` → ×0.35 (volatile-rich)
- otherwise → ×0.7 (intermediate)

### 2. Habitable zone (25%), Kopparapu et al. (2013, 2014)

HZ edges scale with luminosity, `L★ = R★² · (Teff / 5778 K)⁴`:

```
edge(au) = k · √L★
```

with `k` = 0.75 (recent Venus), 0.95 (runaway greenhouse), 1.676 (maximum
greenhouse) and 1.765 (early Mars). Call the four edges `iO, iC, oC, oO`:

- `a < iO` → `s = 0.05` (too hot)
- `a > oO` → `s = 0.10` (too cold)
- `iO ≤ a < iC` → `s = 0.30 + 0.25 · (a−iO)/(iC−iO)` (warm edge)
- `oC < a ≤ oO` → `s = 0.65 + 0.30 · (oO−a)/(oO−oC)` (cool edge)
- `iC ≤ a ≤ oC` → `s = min(0.75 + 0.20 · (1 − |f − 0.5|·2), 1)`, with
  `f = (a−iC)/(oC−iC)` (conservative HZ, peaks mid-zone)

### 3. Stellar type (15%)

| Regime | Teff (K) | Score |
|---|---|:-:|
| G dwarf (solar analogue) | 5000 – 6000 | 0.90 |
| F dwarf | 6000 – 7500 | 0.80 |
| K dwarf | 3700 – 5000 | 0.65 |
| Hot star (A or earlier) | > 7500 | 0.40 |
| M dwarf | < 3700 | 0.30 |

STEHM was calibrated for Sun-like stars. M dwarfs score low because of
harsher XUV and flaring.

### 4. Vetting flags (10%)

| Verdict | Score |
|---|:-:|
| Planet candidate, on-target centroid | 0.85 |
| Planet candidate, centroid unclear | 0.55 |
| Not vetted | 0.50 |
| Ambiguous | 0.45 |
| No significant signal | 0.40 |
| EB indicators (odd-even, secondary, centroid offset) | 0.10 |
| Eclipsing-binary candidate / blend | 0.05 |

### 5. Cosmic shoreline (10%), new in v2

Zahnle & Catling (2017) found that bodies with and without atmospheres split
along `I ∝ v_esc⁴`. HCI v2 puts the line through Mars and uses their XUV
scaling, which is harsher around dim stars:

```
v_esc = 11.186 km/s · √(M/R)
I_xuv = (L★ / a²) · L★^−0.6
q     = I_xuv / (I_Mars,shore · (v_esc / v_esc,⊕)⁴)
s     = clamp(1 − 0.95 · (log₁₀ q + 1) / 2, 0.05, 1)
```

Full marks a decade below the line, near zero a decade above. Earth sits at
`q ≈ 0.1`, so it scores 1.0.

### 6. Tidal locking (5%), new in v2

Kasting, Whitmire & Reynolds (1993), after Peale (1977). The locking radius
grows slowly with age and with stellar mass:

```
r_T = 0.46 AU · (t / 4.5 Gyr)^(1/6) · (M★ / M⊙)^(1/3)
```

- `a ≥ 1.5 r_T` → `s = 1.0` (free rotator)
- `r_T ≤ a < 1.5 r_T` → `s = 0.4 + 0.6 · (a/r_T − 1)/0.5`
- `a < r_T` → `s = 0.4` (likely locked)

A locked planet is marked down, not ruled out. Climate models still allow
habitable locked worlds.

### 7. System age (5%), new in v2

Main-sequence lifetime `t_MS = 10 Gyr · (M★/M⊙)^−2.5`.

- `t < 0.1 Gyr` → `s = 0.10` (very young)
- `0.1 ≤ t < 1` → `s = 0.10 + 0.60 · log₁₀(t / 0.1)` (young, still flaring)
- `1 ≤ t < 3` → `s = 0.70 + 0.30 · (t − 1)/2` (maturing)
- `t ≥ 3` → `s = 1.0` (mature)
- past `0.8 t_MS`, the score falls to as low as 0.3 as the star brightens
- unknown age → `s = 0.5`

VetStar can also estimate age from a rotation period (gyrochronology,
Mamajek & Hillenbrand 2008). ExoWorld takes the age directly.

### Bulk-density modifier (±10 pts)

After the weighted average is scaled to 0–100:

- `ρ/ρ⊕ ≥ 0.6` (terrestrial, ρ ≳ 3.3 g/cm³) → **+10**
- `ρ/ρ⊕ < 0.4` (gas-giant, ρ ≲ 2.2 g/cm³) → **−10**
- otherwise → 0

### Hard cap

If the vetting verdict is an eclipsing-binary candidate or a background
blend, the HCI is capped at **12**.

### Tiers

| HCI | Tier |
|---|---|
| ≥ 70 | Promising |
| 45 – 69 | Marginal |
| 20 – 44 | Unlikely |
| < 20 | Very unlikely |

---

## Caveats

- STEHM models a pure-CO₂ stagnant-lid planet as a best case. It leaves out
  non-thermal escape, magnetic fields and plate tectonics, so read the HCI as
  a first-order estimate, **not** a vetting verdict.
- M dwarfs are penalised on purpose. STEHM is not calibrated for M-dwarf
  XUV, and the safe size there may be above `0.8 R⊕`.
- The HZ edges use fixed Solar-System distance ratios scaled by `√L★`, not
  the full Teff-dependent Kopparapu flux limits. Around cool stars this is
  an approximation.
- Earlier versions floored luminosity at 0.01 L⊙, which pushed the HZ of
  Proxima and TRAPPIST-1 outward and scored their HZ planets "too hot".
  ExoWorld drops the floor, matching
  [eagnespuerto/vetstar#68](https://github.com/eagnespuerto/vetstar/pull/68).

## References

- Hill et al. (2026), *STEHM*, `arXiv:2605.00170`
- Kopparapu et al. (2013, 2014), habitable-zone limits
- Zahnle & Catling (2017), ApJ 843, 122, cosmic shoreline
- Kasting, Whitmire & Reynolds (1993), Icarus 101, 108, tidal locking
- Mamajek & Hillenbrand (2008), ApJ 687, 1264, gyrochronology
- Anugu et al. (2026), `arXiv:2610.33373`, Proxima Centauri Teff and radius
- Scoring source: [`vetstar_core/habitability.py`](https://github.com/eagnespuerto/vetstar/blob/release/v2/vetstar_core/habitability.py) in `eagnespuerto/vetstar`
