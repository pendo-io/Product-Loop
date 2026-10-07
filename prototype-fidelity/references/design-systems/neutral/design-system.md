# Neutral default design system

The design system this skill uses when you haven't plugged in your own. It is
deliberately plain: system fonts, a gray and blue palette, and a standard
sidebar + top bar app shell. Nothing loads from the network.

- **Name:** the neutral default design system
- **Shell:** `shell.html` in this folder
- **Screenshots:** none
- **Component repo:** none
- **What identifies the app:** nothing. The neutral default is not any real product, so a screenshot is never an app screen (case A) under it. The shell is used only when the user asks for a new screen from a description (Stage 0) and wants the default styling.

## Shell layout

```
.app-content                 ← full viewport, flex row
  .side-nav                  ← 240px wide (add .collapsed for 64px, icons only)
  .main-container            ← white, fills the rest
    .top-nav                 ← 56px tall header with search and avatar
    .page-content            ← ONLY THIS CHANGES PER PAGE (24px padding)
```

Shell markers the skill patches:
- Page title: `PAGE TITLE HERE` in `<title>`
- Active nav: each `.nav-option` has a `data-nav` attribute (`home`, `dashboards`, `reports`, `people`, `settings`). Add `active` to the class of the one to highlight, for example:
  `sed 's/class="nav-option" data-nav="settings"/class="nav-option active" data-nav="settings"/' prototype.html > p.tmp && mv p.tmp prototype.html`
  To add a nav item, copy an existing `<li>` and give it a new `data-nav` value.
- Page CSS: below `/* ── PAGE-SPECIFIC STYLES GO BELOW THIS LINE ── */`
- Page HTML: replace the contents of `<main class="page-content">`

The sidebar and header are in normal flex flow. Do not use fixed or absolute
positioning for them.

## Critical values

Always reference the CSS variables, never raw px or hex values.

- **Heading font:** `var(--font-display)` (system UI stack)
- **Body/UI font:** `var(--font-ui)` (system UI stack)
- **Section labels:** `var(--font-mono)`, uppercase, `var(--letter-spacing-nav-eyebrow)`
- **Type scale:**
  - H1 600: `var(--font-size-h1)` = 32px, lh 40px
  - H2 600: `var(--font-size-h2)` = 24px, lh 32px
  - H3 600: `var(--font-size-h3)` = 20px, lh 28px ← default `.page-title`
  - H4 600: `var(--font-size-h4)` = 16px, lh 24px
  - Body 400: `var(--font-size-body)` = 14px, lh 20px
  - Body small 400: `var(--font-size-body-sm)` = 12px, lh 16px
  - Buttons, weight 600: `var(--font-size-btn)` (14px), medium height `var(--btn-height-md)` (36px), small height `var(--btn-height-sm)` (32px)
  - Metrics, weight 700: large `var(--font-size-metric-lg)` (32px), medium `var(--font-size-metric-md)` (24px)
- **Text primary:** `#1F2328`
- **Text secondary:** `#59636E`
- **Text muted:** `#656D76`
- **Brand / primary button:** `#2563EB`, hover `#1D4ED8`
- **Sidebar bg:** `#F6F8FA`
- **Hover bg:** `#EEF1F4`
- **Border:** `#D0D7DE` (dividers, cards)
- **Input border:** `#8C959F` (`var(--input-border)`, form controls)
- **Border radius:** `6px` buttons and inputs, `8px` cards
- **Layout:** sidebar `240px`, top nav `56px`, content padding `24px`
- **Do not** add a second `<head>`, font link, or CDN script tag.
- **Icons:** inline SVG, stroke `currentColor`, 18px in the nav. Do not add an icon-font or icon-library script.

## Accessibility quick checks (on white)

- `#1F2328` = 15.8:1 ✓
- `#59636E` = 6.1:1 ✓
- `#656D76` (text muted) = 5.25:1 ✓ (4.93:1 on the `#F6F8FA` sidebar ✓, so nav section labels pass AA)
- `#6E7781` (`--gray-70`) = 4.55:1 ✓ (barely AA on white; fails AA on `#F6F8FA` at 4.27:1, so not for sidebar text)
- `#8C959F` = 3.04:1 (fails for text: placeholder/disabled text only; passes 3:1 as the input border)
- `#2563EB` = 5.2:1 ✓ (white text on it is also 5.2:1)
- `#D0D7DE` = 1.45:1 (dividers only; not enough for an input border that must meet 3:1)
