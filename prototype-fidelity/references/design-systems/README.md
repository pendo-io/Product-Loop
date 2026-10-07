# Plugging in your design system

The skill builds with one design system at a time. It ships with
`neutral/`, and you can add your own product's design system next to it.

## Choosing the active design system

In order:
1. The user names one in the request ("use our design system", "use the
   acme design system").
2. `design_system` in the skill's `config.json` (see `config.example.json`),
   a folder name under `references/design-systems/` or an absolute path.
3. Otherwise `neutral`.

If the user asks for "our design system" and none is configured, ask once
where it lives (a folder, a tokens file, a Figma export, a style guide
page, or screenshots) and offer to save it as a folder here.

## What a design-system folder contains

Only `design-system.md` is required. Copy `neutral/design-system.md` as a
starting point.

| File | Required | Purpose |
|---|---|---|
| `design-system.md` | yes | Name, fonts (and where they load from), color and spacing tokens, type scale, radius, the "critical values" models most often get wrong, and accessibility quick checks for your palette |
| `shell.html` | no | Your product's app shell (sidebar, header, content region) as a single HTML file. If present, app screens are built by copying it and patching only the page region. `design-system.md` must name its markers (page title, active nav class, page CSS marker, page content element). |
| `screenshots/` | no | Reference screenshots of your product (nav expanded and collapsed, a list page, a detail page, an empty state, your logo), plus an `index.md` that says what each one shows and the corrections models commonly get wrong |
| Icon rules | no | A section in `design-system.md` if your nav icons have rules (stroke vs fill, which icons are exceptions) |
| Component repo | no | A section in `design-system.md` naming the repo and the likely component paths, for Stage 2b |

## How the skill tells an app screen from a branded page

`design-system.md` should say what identifies your *app* (for example "the
workspace switcher at the top of the left sidebar" or "the header with the
global search box and account menu"). A screenshot with only your logo or brand palette is a branded
page, not the app: it gets your tokens but its own layout.

## Fonts

If your brand fonts load from your own CDN, put the `@font-face` rules in
`shell.html` and say in `design-system.md` that no other font link may be
added. If the fonts are not reachable from where the prototype is opened,
note the fallback stack.
