---
name: prototype-fidelity
description: 'Use this skill whenever building, recreating, or improving a UI prototype from a screenshot, or when improving an existing prototype''s fidelity against its source. Enforces pixel-perfect reproduction: every color, font, spacing, border, shadow, and component state must match the screenshot exactly unless explicitly told to deviate. Uses a pluggable design system (a neutral default ships with the skill, and you can add your own product''s tokens, app shell, and reference screenshots), and offers a usability audit after fidelity is confirmed. Always trigger when a user provides a screenshot and says "build", "recreate", "prototype", "mock up", "improve fidelity", or "make this look like", even without explicit mention of fidelity. The screenshot is always the spec, not inspiration. Single owner of the screenshot-to-prototype fidelity rules and the deviations log. Not for production code, backend work, or non-UI tasks; use your normal engineering workflow.'
license: MIT. LICENSE has complete terms
compatibility: Standalone. No connectors or other skills required, and the neutral default design system loads nothing from the network. Optional extras if available - browser tools (for example Claude in Chrome) for the render check, and the gh CLI for fetching a real component from your own repo.
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

# Prototype Fidelity

The screenshot is the spec, not inspiration. Every detail must match exactly.
Deviate only when explicitly instructed.

## Done

**Result:** one HTML prototype file saved in the user's workspace (a `prototypes/` subfolder is
a good home), plus, in chat, the page-specific HTML and CSS and the deviations log from
`references/fidelity-checklist.md`.
**Done:** the file exists at the saved path and opens; `references/fidelity-checklist.md` has been
run region by region against the screenshot (or against the description for a Stage 0 build); the
deviations log is in the reply, with one row per intentional deviation or "None".
**Intent:** the screenshot is the spec, not inspiration. When the steps do not cover a case, match
the screenshot and deviate only when explicitly instructed.

## Boundaries and authority

- Never read the design system's `shell.html` into context, because it is a fixed, large file whose
  SVGs and tokens are already correct; copy it with `cp` and patch its markers with `sed` (Stage 2).
- Never apply the app shell to a branded non-app page or an unrelated screenshot (cases B and C),
  because your logo alone does not mean the layout is the app's; start from a blank page and read
  the layout from the screenshot in Stage 1.
- Never add a second `<head>`, a font link, a CDN script tag, or an icon-library script to the
  neutral shell, because the neutral default loads nothing from the network; use its CSS variables
  and inline SVG icons.
- Never fetch a repo's whole tree (`git/trees/HEAD?recursive=1`), because it pulls the entire tree
  of a large repo and is extremely slow; use a targeted `gh api .../contents/<path>` fetch, and fetch
  from the repo at all only when the user explicitly asks for the real component (Stage 2b).
- Never substitute a "better" state, or apply usability audit findings without asking, because the
  screenshot is the spec; reproduce the state shown, and present audit findings and ask (Stage 4).
- Never implement a request that creates a usability or accessibility problem without first
  explaining it and proposing an alternative, because the user stays in control with the
  information to decide; wait for their answer (Iteration mode).

**Authority:** writes only local files in the user's own workspace: the copied and patched
`prototype.html` in the working folder, the finished prototype saved to the workspace, edits to an
existing prototype in improvement and iteration modes, the UX-review offer notes file
`<prototype-folder>/.ux-review-offers.md` (UX Review mode), and, if the user agrees, a new
design-system folder under `references/design-systems/`. `gh api` and the browser tools (`navigate`, screenshot)
are used read-only. Nothing is sent to a third party.

## Judgment vs mechanics

The model judges which case a screenshot is (A, B, or C), reads colors, type, spacing, and states
from the image, decides what is a blocking delta or polish, and phrases push-back and audit
findings. Code does the fixed parts: `cp` copies the shell and `sed` patches its markers, the
browser tools render and screenshot the file, and `gh api` fetches a component. Still done in prose
by the model: value extraction from the screenshot, the fidelity checklist comparison, contrast
ratios (`references/usability-principles.md` says to compute them, but no script ships), and the
three-iteration bound on the render loop.

## Honesty

- "Could not check" is not "checked": with no `gh` CLI, no named repo, or a component not found
  after two direct paths, build from tokens and tell the user it is an approximation of the
  production component. With no browser tools, hand the file over for the user to open and do not
  claim a rendered check.
- Estimates are labeled: Stage 1 spacing is estimated in px from the image, and contrast ratios are
  computed, not estimated.
- Every intentional deviation from the screenshot goes in the deviations log; if there are none,
  say "None".
- Read back each write before reporting it done: re-read the saved prototype at its saved path,
  and re-read `.ux-review-offers.md` after adding an offer to it.

## When to use

Use whenever someone provides a screenshot and wants it built, recreated, prototyped, mocked
up, or improved for fidelity (even without the word "fidelity"), and whenever they ask to
build a UI in their design system from a description. Trigger phrases: "build this", "recreate this
screen", "prototype this", "mock this up", "make this look like", "improve the fidelity",
"match this screenshot". The screenshot is always the spec, not inspiration. Not for
production code, backend work, or non-UI tasks.

## The active design system

Everything below that says "the design system" means the **active design system**: a folder
anywhere on disk. A relative value is a folder under `references/design-systems/`; an
absolute path is used as is. Pick it in this order:

1. The one the user names in the request.
2. `design_system` in this skill's `config.json` (see `config.example.json`): a folder
   name under `references/design-systems/` or an absolute folder path.
3. Otherwise `neutral`, the default that ships with the skill.

How to add your own (tokens, app shell, screenshots, icon rules, component repo):
`references/design-systems/README.md`. Read the active folder's `design-system.md` before
building. Do not read its `shell.html` into context (see Stage 2).

## Reference files
| File | Covers |
|------|--------|
| `references/design-systems/README.md` | How to plug in a design system, and what a design-system folder contains |
| `references/design-systems/neutral/design-system.md` | Default tokens, critical values, shell markers, contrast checks |
| `references/design-systems/neutral/shell.html` | Default app shell. Copy it verbatim as a starting point, never read it in |
| `<active design system>/screenshots/` | Your product's reference screenshots and their `index.md`, if you added them |
| `references/usability-principles.md` | Gestalt, Laws of UX, WCAG 2.1 AA, Nielsen's 10 |
| `references/fidelity-checklist.md` | Checklist to run before delivery, plus the deviations log |
| `references/ux-review.md` | Opt-in UX review mode |

**Before writing any code: if the active design system has screenshots, read them visually.**
Use them to verify exact font sizes, spacing, component density, and color values. Do not rely
solely on the tokens in `design-system.md`. If a screenshot contradicts a token value, the
screenshot wins.

---

## UX Review mode (opt-in only)

At natural transition points during prototyping (after the first meaningful version is complete, before expanding to additional screens, when the user seems to be polishing rather than progressing, or when the prototype appears to be solving multiple problems at once), offer the UX review once, after completing the requested work:

> The prototype is at a useful point for a quick UX review. Would you like me to review whether the flow is focused on the right problem and hypotheses before we add more detail?

If the user says yes (or says "UX review", "review the product thinking", "challenge this concept", "what assumptions are we making?"), **read `references/ux-review.md`** and follow its instructions. Do not load this file during normal prototyping.

Do not offer the review after small changes (spacing, labels, icons, fidelity fixes). Do not offer more than once every three substantial iterations unless the user requests it. If the user declines, continue building normally.

Track offers in a small notes file next to the prototype, `<prototype-folder>/.ux-review-offers.md`, so the three-iteration rule does not rely on chat history. Read it before offering. Each time you offer, add one line with the date and the iteration number of the offer.

---

## Stage 0: Intake (no screenshot provided)

If the user sends a prompt describing a UI they want built **but includes no screenshot**, do not start building. Instead:

1. **Ask if they meant to include a screenshot.** Example: "It looks like you didn't attach a screenshot. Did you mean to include one? If so, share it and I'll build from that."

2. **If they confirm no screenshot**, ask: "Would you like me to build this using [the active design system's name] (fonts, colors, components, and shell layout)?"

   - **Yes → use this skill** in full: apply the design system's shell (if it has one), tokens, and all rules below. Skip Stage 1 (there is no screenshot to analyze) and go straight to Stage 2, using the user's written description as the spec in place of a screenshot. If the design system has no `shell.html`, follow Stage 2's case B steps.
   - **No → follow their lead** without applying the design system. Build what they describe using general best practices. Do not apply the shell, tokens, or any design-system-specific rules unless they ask.

Do not ask both questions at once, because the styling question only applies when there is no screenshot. Ask about the screenshot first, then the styling question only if they confirm there is no screenshot.

---

## Is this a screen from your design system, and is it the *app*?

Two separate questions, because they lead to different builds. **Your logo alone does
NOT mean "use the app shell."** The active `design-system.md` says what identifies your app.

**A) Is it your product's *app*?** YES only if it shows what `design-system.md` names as the
app's identifying chrome (typically the app's left sidebar nav or its top header). For an app
screen, use the **app shell**: skip the color/font/icon extraction steps below, copy the
design system's `shell.html` verbatim (Stage 2), and use the `design-system.md` values directly.
Don't approximate what you can already know exactly. If the design system has no `shell.html`,
treat it as case B.

**B) Is it *branded* with your design system but NOT the app?** A page with your logo and brand
palette but different chrome: marketing pages, standalone internal tools, login / sign-out pages,
embedded or full-bleed views, email templates. Here you still get the brand for free: use the
design system's **tokens** (fonts, color palette, radius, spacing scale), **but do not apply
`shell.html`** (see Boundaries), and **DO run Stage 1 layout extraction** from the screenshot, because the layout
is not the app's and can't be known without looking. If the app's identifying chrome is absent,
it's case B: brand tokens, screenshot-driven layout, no shell.

**C) Not from your design system at all.** A screenshot of some other product or site: full
Stage 1 extraction, and no design-system tokens unless the user asks for them. With the neutral
default, every screenshot is case C.

---

## Stage 1: Screenshot Analysis

**For a Stage 0 build from a written description: skip Stage 1** and go to Stage 2. **For an app screen (case A): skip Stage 1 entirely.** The shell already carries the layout and all tokens; go straight to Stage 2. **For a branded non-app page (case B) or an unrelated screenshot (case C): run Stage 1** to capture the layout from the screenshot. In case B you can still reuse the known color/font/spacing tokens rather than re-extracting them. It's only the *layout* you must read from the image.

For case C screenshots, complete this full inventory before writing any code:

### 1a. Layout structure
- Overall composition: columns, rows, fixed vs. fluid
- Outer shell (sidebar + header + content? full bleed? split pane?)
- Every distinct region and approximate dimensions

### 1b. Color extraction
Every unique color as hex: backgrounds, text, borders, shadows, icons, badge bg+text, buttons.

### 1c. Typography
Per text style: font family, size, weight, letter-spacing, line-height, color.

### 1d. Spacing
Page outer padding, card inner padding, nav item gaps, section gaps, estimated in px.

### 1e. Component inventory
Every UI element: nav items (labels, icons, states), buttons, inputs, cards, tables, badges.

### 1f. Borders, radius, shadows
Per element type. Shadow: x, y, blur, spread, color, opacity.

### 1g. States visible
Reproduce the exact state shown. Never substitute a "better" state.

---

## Stage 2: Code Generation

### For an app screen (case A), or a Stage 0 build in the design system: copy the shell, then patch. Never read it into context

The shell is a fixed file and can be large. Do not read it. Instead:

> **Cases B and C do not use the shell** (see Boundaries). They follow the same three steps with the differences in "For case B or C" below.

**Step 1: Copy it at the filesystem level** into your working folder (you deliver it in the Output step below):
```bash
cp <active design system folder>/shell.html <working-folder>/prototype.html
```
(Resolve the path from wherever the skill is installed. Do not hardcode an install path.)

**Step 2: Patch only the things that change**, using the markers `design-system.md` names, with targeted `sed` or a minimal script, never by reading the file in:
1. **Page title**: replace the title marker (`PAGE TITLE HERE` in the neutral shell) with the screenshot's title. For a Stage 0 description build, use the page title the description gives; if it gives none, ask the user for one.
2. **Active nav**: add the active class to the matching nav item, found by the marker `design-system.md` names (in the neutral shell, add `active` to the `.nav-option` whose `data-nav` matches, for example `data-nav="settings"`).
3. Leave everything **outside the page content region untouched**. The shell structure is fixed.

**Step 3: Append page-specific CSS and page content directly**: add page CSS just **below** the page-CSS marker (`/* ── PAGE-SPECIFIC STYLES GO BELOW THIS LINE ── */` in the neutral shell), and replace only the contents of the page content region (`<main class="page-content">` in the neutral shell) with the page-specific HTML.

Do not read the shell file into context at any point. Its SVGs, tokens, and boilerplate never need to be in working memory; they are already correct and unchanged.

After patching, show the user **only** the page-specific additions: the page content HTML and any page CSS. That is what they need to review, not the shell boilerplate they already know.

**Critical values.** `design-system.md` lists the values models most often get wrong (heading vs body font, text colors, border color, primary button color, radius, layout sizes). Always use its CSS variables rather than raw px or hex values. Also follow its rules on what not to add; for the neutral default:
- **Do NOT add another `<head>`, a font link, or a script tag from any CDN**
- **No icon-font or icon-library scripts.** Icons are inline SVGs
- The sidebar and header are in normal flex flow. Do NOT use fixed/absolute positioning for them

If `design-system.md` has **icon rules** (which icons are outline vs filled, exceptions, sub-navigation states, special buttons), follow them exactly.

### For case B (branded, not the app) or case C (not from your design system): same steps, blank page instead of the shell

Follow Steps 1 to 3 above, with these differences:

- **Step 1:** start from a blank page (a minimal HTML document with its own `<head>` and `<body>`) instead of copying `shell.html`. There is no app frame.
- **Step 2:** set the page title. There is no shell nav to patch; if the screenshot has its own nav, build it in Step 3 with the active state it shows.
- **Step 3:** build the screenshot's own layout (from Stage 1, or from the written description for a Stage 0 build) as the page CSS and page content.
- **Values:** case B uses the design system's tokens (fonts, colors, radius, spacing) from `design-system.md`, and its icon rules where they apply. Case C uses the values extracted in Stage 1 and no design-system tokens unless the user asks for them.

After building, show the user the page CSS and content, as for case A. You wrote the whole file, so there is no boilerplate to leave out.

---

## Stage 2b: Component sourcing from the repo (explicit request only)

Only do this when the user **explicitly asks to fetch or use the real production component** (for example "fetch the modal from the repo", "use our actual tooltip component") **and** `design-system.md` names a component repo. Do NOT run this automatically for every component mention. For standard components like buttons, modals, tooltips, and dropdowns, build them using the design system's tokens and shell conventions. No repo fetch needed.

When the user has asked for it:

> **If the `gh` CLI is missing or not signed in**, or the design system names no repo, skip the repo fetch entirely and build the component from the design system's tokens and shell conventions, telling the user it's an approximation of the production component.

1. Fetch the component directly by guessing its likely path from the paths `design-system.md` lists. Do NOT use `git/trees/HEAD?recursive=1` (pulls the entire tree of a large repo and is extremely slow). Instead use a targeted fetch:
   ```
   gh api repos/<owner>/<repo>/contents/<component path from design-system.md>
   ```
   If that path 404s, try the next listed path. Only fall back to a tree search if both direct paths fail.

2. Read the component file. Extract template structure, class names, and styles.

3. Translate to static HTML/CSS using the exact class names and the design system's tokens.

If the component genuinely cannot be found after two direct attempts, build it with the design system's tokens and tell the user it's an approximation.

---

## Output: deliver the prototype (final step)

Once the file is built, deliver it. Never leave it in an ephemeral working folder, because that folder is temporary and the user would lose the file:
1. **Save** the finished HTML into the user's workspace or project folder (a `prototypes/`
   subfolder is a good home). If no workspace folder is in scope, say so and offer to hand the
   file over another way (download / paste the path).
2. **Present** the saved file so the user can open it (the host's file-preview surface, if it
   has one).
3. In chat, show **only** the page-specific additions (the page content HTML and any page
   CSS), not the shell boilerplate, plus the deviations log from
   `references/fidelity-checklist.md` (or "None").

Then offer the two optional passes below.

## Stage 3: Browser Render Loop (opt-in)

After delivering the prototype, ask once:
> Want me to open this in a browser and verify the rendering against the original?

If yes, run the verification loop. **Primary: browser tools (for example Claude in Chrome), when
they are available:**
1. Open the saved file with the browser tools (`navigate` to its `file://` path)
2. Take a screenshot

**Backup: no browser tools.** Don't force them. Present the file for the user to open and
eyeball, and offer a self-check against `references/fidelity-checklist.md` instead. Either path, then:
3. Check each region against the original screenshot, using the values read from it (the screenshot
   is the spec, so where a design-system token differs from what the screenshot shows, the screenshot wins):
   - Fonts: family, sizes, and weights match the screenshot
   - Colors: sidebar, border, and primary button colors match the screenshot
   - Nav: all items present, correct icons, correct active state
   - Spacing: content padding, top nav height, and sidebar width match the screenshot

   For a description build (Stage 0, no screenshot), check these against `design-system.md`'s
   critical values instead.
4. List every delta: what it shows · what it should show · fix
5. Apply fixes, re-render, take a second screenshot
6. Repeat up to 3 iterations until no blocking deltas remain

If the user says no or doesn't respond, skip this stage entirely.

---

## Stage 4: Usability Audit (opt-in)

After delivering the prototype, ask once:
> Want me to run a usability audit (Gestalt, WCAG 2.1 AA, Nielsen's 10)?

If yes, read `references/usability-principles.md` and check against Gestalt, Laws of UX, WCAG 2.1 AA, and Nielsen's 10. Use the contrast quick checks in the active `design-system.md` where it has them. Report each issue with severity (Critical/Major/Minor), principle violated, what's wrong, and suggested fix. Do NOT auto-apply. Present and ask.

If no or no response, skip.

---

## Improvement mode (existing code + screenshot)

1. Read the code, identify every fidelity gap vs. screenshot
2. List gaps: what's wrong · what it should be · severity (blocking / polish)
3. Fix all blocking gaps immediately
4. List polish gaps and ask whether to fix
5. Ask once if the user wants a usability audit. Do not run it automatically

---

## Iteration mode (changes to an existing prototype)

Once a prototype exists, treat every follow-up request as an iteration. Keep iterations fast:

**Only modify what's changing.** Read the existing file, make the smallest surgical edit that satisfies the request. Do not rewrite the whole file. Output only the changed section(s) in the conversation, not the full file.

**Apply UX best practices to the requested change only.** While implementing the requested change, ensure the parts the user asked to change follow established UX principles (clear hierarchy, consistent spacing, accessible contrast, appropriate affordances, Fitts's Law for targets, etc.). Leave the rest of the prototype as it is. Log each best-practice adjustment in the deviations log, so nothing moves away from the screenshot without a record.

**Push back on requests that conflict with UX best practices.** If the user asks for something that would create a usability or accessibility problem (for example low-contrast text, ambiguous labels, destructive actions without confirmation, critical content in a hover-only state), do not implement it immediately. Instead:

1. Briefly explain why the request creates a problem (one or two sentences, no lecturing)
2. Propose a better alternative that achieves their intent
3. Ask whether they'd like to go with your proposal or proceed with their original preference

Wait for their response before making any change. This keeps the user in control while ensuring they have the information to make a good decision.

**Example (neutral default):**
> User: "Drop the field labels on the timesheet form and just use placeholder text"
> Response: "Placeholder-only fields lose their label as soon as someone starts typing, and screen readers may not announce them, so the form would miss the WCAG 2.1 AA label requirement. I'd keep a short visible label above each field and use the placeholder for an example value, like `e.g. 7.5`. Want me to do that, or go placeholder-only anyway?"

Do not push back on purely aesthetic preferences (color choices, layout style, component selection) unless they create a direct usability or accessibility problem.
