# ExoWorld Habitability Tuner

**Live demo: [eagnespuerto.github.io/exoworld-tuner](https://eagnespuerto.github.io/exoworld-tuner/)**

ExoWorld offers insight into the possible habitability of any exoplanet you can imagine. Be it for research or for science fiction writing, this tool is handy in ensuring you are working with a planet that may or may not be habitable. It can also be used as an interactive teaching tool for science educators looking to share knowledge on how worlds can or cannot be habitable.

A single-file static site — open `index.html` in any modern browser, or host it on GitHub Pages. The tuner is a UI port of the `VetStar` / `STEHM` habitability scoring engine in [eagnespuerto/vetstar](https://github.com/eagnespuerto/vetstar) (`backend/app/habitability.py`).

## How to use

Drag the sliders to set a hypothetical planet (orbital distance, radius, mass) and pick a host star spectral class plus pipeline/disposition flags. The **Habitability Chance Index (HCI)** recomputes live, breaking the result into six weighted sub-scores and a density modifier.

Presets are included for an Earth twin, the full TRAPPIST-1 system, a hot Jupiter, a super-Earth, and an eclipsing-binary false positive.

### Themes

The header has a four-way theme switch:

| Theme  | Use case                                       |
|--------|------------------------------------------------|
| Night  | Default — observatory dark palette             |
| Black  | OLED-friendly pure black                       |
| Warm   | Sepia / paper-lamp tones for low-light reading |
| Light  | Daytime / high-contrast monitors               |

The chosen theme is persisted in `localStorage`. The system-diagram stage stays dark in every theme — space doesn't have a light mode, and locking it preserves the meaning of the spectral colour ramp.

### Host star

Three ways to set the host:

1. **Spectral-class buttons** (M / K / G / F / A) — uses a class-average `Teff` and `R★` (good for quick what-ifs).
2. **Custom** — type an arbitrary `Teff` (K) and `R★` (R⊙). The HZ ring, stellar-type sub-score, and the star's rendered colour all update from `Teff`.
3. **Known star dropdown** — picks a real host's measured `Teff` and `R★` from the literature (Sun, Proxima Centauri, TRAPPIST-1, Alpha Centauri A/B, Tau Ceti, the Kepler/TOI hosts, etc.) and switches the selector to Custom. This is the right choice when you want to speculate about hypothetical planets in a real system: pick the host, then drag the orbit/radius sliders to place a fictional world around it.

Planet presets also auto-populate their real host star — picking *TRAPPIST-1e* now sets `Teff = 2566 K, R★ = 0.119 R⊙` (the actual ultracool dwarf), not the generic M-dwarf class average. This makes the HZ edges and stellar sub-score noticeably more honest for compact systems.

Whenever you change *just the star* (a spectral button or a known-star pick — not a full planet preset), the orbital-distance slider snaps to the centre of that star's conservative HZ. So the diagram opens on a habitable orbit by default, and you can drag inward/outward from a sensible starting point.

### Preset auto-revert

The "known exoplanet preset" dropdown is a *snapshot*. The moment you tweak any planetary observable (orbit, radius, mass, disposition, vetting flag, sectors) the dropdown reverts to "— select a template —", so the UI never claims you're still looking at TRAPPIST-1e once you've moved the sliders.

### System diagram

The mini-system panel above the gauge is a **top-down view of the orbital plane**. The star sits at the centre of the projection, the orbit and the conservative habitable zone are drawn as concentric circles (the HZ band literally curves around the star), and the planet is rendered as a half-lit disc with its day side facing the star. The dashed green circles mark the inner and outer conservative HZ edges from Kopparapu et al.

---

## The HCI calculation

The HCI is a weighted average of six independent sub-scores (each on `[0, 1]`), scaled to a `0–100` index, then nudged by a bulk-density modifier. A hard override caps obvious false positives.

```
HCI = (Σ wᵢ · sᵢ) / (Σ wᵢ) · 100  +  density_modifier
```

| Component        | Weight | What it measures                              |
|------------------|:------:|-----------------------------------------------|
| Planet size      |  30%   | STEHM atmosphere-retention regime             |
| Habitable zone   |  25%   | Insolation relative to Kopparapu HZ edges     |
| Stellar type     |  15%   | How well STEHM calibration transfers          |
| TOI disposition  |  15%   | ExoFOP confidence level                       |
| Vetting flags    |  10%   | Pipeline verdict (planet / EB / blend)        |
| Multi-sector     |   5%   | Detection consistency across TESS sectors     |

### 1. Planet size (30%) — STEHM

Uses thresholds from Hill et al. (2026), *STEHM*, `arXiv:2605.00170`:

- `Rp > 2.2 R⊕` → `s = 0.05` (likely sub-Neptune, not rocky)
- `Rp < 0.5 R⊕` → `s = 0.02` (atmosphere retention ≈ 0)
- `Rp ≥ 0.8 R⊕` → `s = 0.75 + 0.25 · min((Rp − 0.8)/0.2, 1)` (favourable; can retain long-term CO₂)
- `0.7 ≤ Rp < 0.8` → `s = 0.35 + 0.40 · (Rp − 0.7)/0.1` (marginal)
- `0.5 ≤ Rp < 0.7` → `s = 0.05 + 0.30 · (Rp − 0.5)/0.2` (unlikely)

If mass is provided, the size score is multiplied by a density-verdict factor:
- `ρ/ρ⊕ ≥ 0.6` → ×1.1 (rocky-consistent)
- `ρ/ρ⊕ < 0.4` → ×0.35 (volatile-rich)
- otherwise → ×0.7 (intermediate)

### 2. Habitable zone (25%) — Kopparapu et al. (2013/2014)

The HZ edges scale with stellar luminosity `L★ = R★² · (Teff / 5778 K)⁴`:

```
limit(au) = S₀(Teff) · √L★
```

where `S₀(Teff)` is a 4th-order polynomial in `(Teff − 5780)` for each of:
- `recent_venus` and `runaway_greenhouse` → optimistic & conservative inner edges
- `maximum_greenhouse` and `early_mars` → conservative & optimistic outer edges

Scoring, with `iO, iC, oC, oO` denoting the four edges:

- `a < iO` → `s = 0.05` (too hot, runaway greenhouse)
- `a > oO` → `s = 0.10` (too cold, frozen)
- `iO ≤ a < iC` → `s = 0.30 + 0.25 · (a−iO)/(iC−iO)` (warm OHZ edge)
- `oC < a ≤ oO` → `s = 0.65 + 0.30 · (oO−a)/(oO−oC)` (cool OHZ edge)
- `iC ≤ a ≤ oC` → `s = min(0.75 + 0.20 · (1 − |f − 0.5|·2), 1)` with `f = (a−iC)/(oC−iC)` (CHZ, peaks mid-zone)

### 3. Stellar type (15%)

Coarse buckets by `Teff`:

| Regime                      | Teff (K)      | Score |
|-----------------------------|---------------|:-----:|
| G dwarf (solar analog)      | 5000 – 6000   | 0.90  |
| F dwarf                     | 6000 – 7500   | 0.80  |
| K dwarf                     | 3700 – 5000   | 0.65  |
| Hot star (A or earlier)     | > 7500        | 0.40  |
| M dwarf                     | < 3700        | 0.30  |

STEHM was calibrated for Sun-like stars; lower scores for M dwarfs reflect harsher XUV and flaring.

### 4. TOI disposition (15%) — ExoFOP

| Code      | Meaning                        | Score |
|-----------|--------------------------------|:-----:|
| CP / KP   | Confirmed / Known planet       | 1.00  |
| PC / APC  | Planet candidate / ambiguous   | 0.75  |
| TOI (unc.)| Unclassified TOI               | 0.55  |
| (none)    | No TOI designation             | 0.50  |
| FP / FA   | False positive / false alarm   | 0.05  |

### 5. Vetting flags (10%) — pipeline verdict

| Verdict                                | Score |
|----------------------------------------|:-----:|
| Planet candidate — on-target centroid  | 0.85  |
| Planet candidate — centroid unclear    | 0.55  |
| Ambiguous                              | 0.45  |
| No significant signal                  | 0.40  |
| Eclipsing-binary candidate / blend     | 0.05  |

### 6. Multi-sector (5%)

Let `obs` = sectors observed, `det` = sectors showing the dip:

- `obs = 1` → `s = 0.40` (single sector, period unconstrained)
- `det = 0` → `s = 0.30` (no detections — inconsistent with a transit)
- otherwise → `s = min(0.40 + 0.60 · det/obs, 1)`

### Bulk-density modifier (±10 pts, post-average)

After the weighted average is scaled to 0–100:

- `ρ/ρ⊕ ≥ 0.6` (terrestrial, ρ ≳ 3.3 g/cm³) → **+10**
- `ρ/ρ⊕ < 0.4` (gas-giant, ρ ≲ 2.2 g/cm³) → **−10**
- otherwise → 0

### Hard override

If the pipeline verdict is `eclipsing_binary_candidate` or `false_positive_blend`, the final HCI is capped at **12** regardless of other inputs.

### Tier mapping

| HCI         | Tier             |
|-------------|------------------|
| ≥ 70        | Promising        |
| 45 – 69     | Marginal         |
| 20 – 44     | Unlikely         |
| < 20        | Very unlikely    |

---

## Caveats

- STEHM models a pure-CO₂ stagnant-lid planet as a best case. Non-thermal escape, magnetic fields, and plate tectonics are excluded — treat the HCI as a first-order estimate, **not** a vetting verdict.
- The M-dwarf regime is intentionally penalised: STEHM is not calibrated for M-dwarf XUV environments, and the safe-size threshold may exceed `0.8 R⊕` there.
- Disposition and vetting components encode *observational confidence*, not intrinsic habitability — a confirmed hot Jupiter still scores poorly because size and zone dominate.

## References

- Hill et al. (2026), *STEHM*, `arXiv:2605.00170`
- Kopparapu et al. (2013, 2014) — habitable-zone limits
- TESS / ExoFOP TOI dispositions
- Source scoring: [`backend/app/habitability.py`](https://github.com/eagnespuerto/vetstar/blob/main/backend/app/habitability.py) in `eagnespuerto/vetstar`
