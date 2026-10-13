# normgamlss.github.io

Landing page for the NormGAMLSS Python package. Plain HTML/CSS — no build step, no framework, no external dependencies. Open `index.html` directly, or drop the folder on any static host (Cloudflare Pages, GitHub Pages, Netlify) with zero config.

Every section was rendered in headless Chromium and checked visually at 1440px and 390px before shipping.

## What this page is

A marketing/landing page only. The real API documentation is your separate Jupyter Book site — this page links to it (`NormGAMLSS/`).

## Structure

```
index.html
fonts/     IBM Plex Sans + Mono (self-hosted woff2, only the weights used)
images/
  logo-mark.png   wordmark only — nav
  logo.png        full logo with tagline — footer
  icon_*.svg      feature icons (6)
  fig_*.svg       feature figures, drawn in the site's own style
  fig_brain.png   real nilearn output, cropped to match the others' aspect
  demo_anim.svg   animated mini chart in the Example section
```

## Notable decisions

- **Fonts are self-hosted.** No Google Fonts request, so the page renders identically offline and doesn't depend on a third-party CDN.
- **The logo was fixed, not just resized.** The original had a *white* background (not transparent), so on the cream paper it rendered as a visible white rectangle. Alpha was rebuilt from luminance and the built-in padding cropped. It was also split in two: the tagline is illegible below ~90px, so the nav uses a wordmark-only crop (letting the wordmark itself be far larger in the same bar) and the footer uses the full logo at 88px.
- **Ghost curves are back.** The four drifting Gaussians from your original dark site — one per GAMLSS parameter — rebuilt for the light theme as pure SVG + CSS. No canvas, no JS.
- **Figures are drawn in the site's own style** rather than pasted-in matplotlib exports, which brought their own fonts, baked-in titles, and large dead margins that fought the layout. Each is grounded in the real pattern from your tutorial notebook. The brain figure is the exception — it's the genuine nilearn render (tightly cropped), because a redrawn brain silhouette loses the credibility the real image has.
- **The site-transfer figure uses a shared y-scale across both panels.** The earlier version scaled each panel independently, which made the before/after look convincing without being a fair comparison.
- **Animations degrade safely.** Every animated element's *base* CSS state is the finished state, and animations run once (`fill-mode: both`) instead of looping. If CSS animation is unsupported, disabled, or already finished, the chart still renders complete — it can never be blank or stuck mid-draw. (`prefers-reduced-motion` is not reliably honoured inside an SVG loaded via `<img>`, which is why the base state carries the safety rather than the media query.)

## Before you publish

1. **GitHub URL** — links point to `github.com/NormGAMLSS/NormGAMLSS` (done).
2. **Documentation** — the docs live in the `NormGAMLSS/` folder (https://normgamlss.github.io/NormGAMLSS/) and are copied there automatically from the NormGAMLSS repository. Do not edit that folder by hand.
3. **Figure captions** say "tutorial output (synthetic data)". Confirm that's accurate for all five. If any figure derives from real patient data, the caption must change before this is public.
4. **`pip install`** — the hero shows a status pill reading "releasing soon". Replace with the real command once on PyPI.
5. **Counts** — the stats bar claims 5 modalities and 6 publications; the publications figure matches the six paper cards. Update both together if papers change.
6. **The "not the full API" note** below the feature grid lists the capabilities not given their own card. Trim or extend it as the package changes, and point it at the docs URL once that's live.
7. **Modalities section** — shows the 4 modalities actually backed by a paper (Structural MRI, fMRI, EEG, CSF/glymphatic dynamics via the Kand et al. paper), plus an "In progress" tile that names cognitive data as unpublished work rather than claiming it as validated. The stats bar still says "5 Modalities validated" against 4 confirmed here — worth reconciling once the cognitive paper (or whatever the 5th is) is real. `icon_pet.svg` and `icon_cognitive.svg` are built and sitting unused in `images/` if PET work ever gets a paper behind it.
8. **Figures are regenerable** — the SVGs match your tutorial's patterns. Swap in production numbers keeping the same language: teal `#00897B` = reference/normal, coral `#E2542C` = flagged/observed.

## Design tokens

| token | value |
|---|---|
| paper | `#FBFAF7` |
| paper (alt bands) | `#F3F1EA` |
| ink | `#1C1C1A` |
| teal (reference/normal) | `#00897B` |
| coral (flagged/observed) | `#E2542C` |
| type | IBM Plex Sans / IBM Plex Mono |
