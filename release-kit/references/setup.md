# Setup

Run this when the `release_kit` config section doesn't exist, or when the user
asks to redo part of their setup ("update my release-kit setup"). It fills four
things the skill otherwise has to guess: your brand voice, real product
examples, the layout of one of your product pages, and your launch tier names.
Each step is one focused question. Use a multiple-choice tool for closed choices
if the client has one, prose for free text.

Every step is skippable. A skipped step saves `null` and the skill falls back to
its neutral default (listed under each step). Save after each step, so an
interrupted setup resumes from the first missing field.

## Rules for every step

- **Never invent a voice or product facts.** Anything you derive from a website,
  a post, or a doc is a proposal. Show it, say where each part came from, and
  save it only after the user confirms or edits it. If they reject it, save
  `null` for that step.
- **Only fetch what the client can fetch.** Use a web fetch tool only if one is
  available, and a doc store (Google Drive, Notion, Confluence, or similar) only
  if a connector is. If neither is available, ask the user to paste instead.
- **Say what you couldn't read.** If a page won't load or a search finds
  nothing, say so and move on. Don't fill the gap from general knowledge.
- **Keep it short.** The whole setup should take about 5 minutes.

## Initial check

Explain in one sentence what setup does, then ask:

> Before we start, want to set up your brand voice, product examples, and launch
> tiers? It takes about 5 minutes and later runs reuse it.
> 1. Yes, set up now
> 2. Skip setup, use the defaults

If "Skip setup" (or the user says "skip setup" at any point), stop here, write
no config, and go straight to Step 1 of SKILL.md with the defaults. Don't ask
again in this conversation. On the next run with no config, ask once more.

## Step A: Brand voice

### A1. Look for a voice the user has already specified

Before asking, check what you can already see, in this order:

1. **This skill's saved config.** If `release_kit.voice` exists (for example
   during a partial re-setup), show it and ask whether to keep it or redo it.
2. **Installed skills that define a voice.** Look through the skills available in
   this session for one whose name or description mentions brand voice, voice,
   tone, style guide, or writing style. Read it and take its voice or style
   rules as the source.
3. **Memory and standing instructions.** Check Claude's memory and standing
   instructions for this user (memory files, CLAUDE.md or project instructions,
   saved user preferences) for voice, tone, or style rules.

- **One found:** summarize it into the structure below, say where it came from
  (the skill's name, or "your memory" / "your project instructions"), and ask:
  "Use this as your brand voice?" Note any of its rules the skill's hard rules will
  set aside (see "The voice governs tone and word choice only" in `SKILL.md`). Save only
  after the user confirms or edits it.
- **Several found:** show each summary with its source and let the user pick
  one or merge them. Save the confirmed result.
- **None found, or the user declines what was found:** go to A2.

Never copy a whole skill's or instruction file's content into config. Save a
reference to the source (`source_skill`, or `source_refs` for memory or
instructions) plus the confirmed summary. If the source is a skill, re-read it
at run time and let its current rules win over the saved summary where they
differ; tell the PM if they do.

### A2. Ask or derive

Ask:

> Do you have a brand voice or writing style guide? Paste it, upload it, or send
> a link.
> 1. I'll paste or link it
> 2. I don't have one, derive it from our website and posts
> 3. Skip, use the default voice

- **Pasted, uploaded, or linked:** read it (fetch a link if you can; otherwise
  ask them to paste it). Summarize it into the structure below, keeping the
  guide's own words where possible.
- **Derive it:** ask for the company website URL. Then, using what's available:
  - With a web fetch tool: fetch the homepage and 2 to 3 existing blog posts or
    announcements (ask the user for the links, or take them from the site's blog
    or news page).
  - With a doc store connector: search it for a brand guide, voice guide, style
    guide, or messaging doc. If you find one, show its title and link and ask
    whether to use it.
  - With neither: ask the user to paste 2 to 3 pieces of copy they consider on
    brand.
  Summarize the voice from what you read. Cite the page or doc each trait came
  from.
- **Skip:** save `null`. The default voice in `references/voice-examples.md`
  applies.

Summarize in the same structure as "Your brand voice" in
`references/voice-examples.md`:

- **Voice in one sentence:** how the brand sounds
- **We are:** 3 to 5 traits
- **We are not:** 3 to 5 traits
- **Readers we write for:** personas
- **Example line that sounds like us:** one real line, quoted exactly from their
  own copy (never written by you)

Show the summary and ask: "Does this sound like you? Edit anything that's off."
Save only the confirmed version. If a field has no support in what you read,
leave it `null` and say so rather than filling it.

If the guide names words to avoid or competitor terms, offer to save them as
`voice.terms_to_avoid` (same shape as "Your own terms to avoid").

```json
"voice": {
  "source": "pasted",
  "source_skill": null,
  "source_refs": ["<url or doc title>"],
  "one_sentence": "Plain, practical, and on the side of the person doing the work.",
  "we_are": ["direct", "practical", "warm"],
  "we_are_not": ["hype-driven", "jargon-heavy", "cute"],
  "readers": ["operations leads", "team managers"],
  "example_line": "<one line quoted from your own copy>",
  "terms_to_avoid": [{ "avoid": "<term>", "use_instead": "<term>" }],
  "confirmed_at": "2026-01-01T00:00:00Z"
}
```

`source` is `skill`, `memory`, `pasted`, `linked`, `derived`, or `doc_store`.
`source_skill` is the installed skill's name when `source` is `skill`, else
`null`.

## Step B: Product examples

The good/bad examples in `references/voice-examples.md` use a fictional product.
Real ones make the calibration fit your copy. Ask:

> Give me 2 or 3 of your real products or features, each with a one-line
> description. I'll use them in place of the fictional examples.
> 1. I'll list them
> 2. Pull them from our website (if you fetched it in Step A, reuse it)
> 3. Skip

If you pull them from the site, use only names and descriptions that appear on
the page, cite the page, and show them for confirmation. Don't write
descriptions for features the site doesn't describe.

Default when skipped: the fictional work-tracking examples.

```json
"product_examples": [
  { "name": "<Feature name>", "description": "<one line, in your words>", "source": "user" }
]
```

These are calibration only. Don't mention them in launch copy for a different
feature unless the PM asks (for example, a related-features footer).

## Step C: Landing-page spec

Spec 6 in `references/asset-formats.md` uses a generic hero, capabilities,
social proof, CTA structure. Ask:

> Share one of your real product pages (a link, or paste its sections) and I'll
> match its section order for landing page copy.
> 1. Here's a link
> 2. I'll paste the sections
> 3. Skip, use the generic structure

- **Link:** fetch it if you can. List its sections top to bottom as short
  labels (for example `Eyebrow`, `Hero headline`, `Customer logos`,
  `Capabilities (3)`, `Quote`, `Pricing teaser`, `CTA`), with rough length where
  it's obvious. If you can't fetch it, ask them to paste.
- **Pasted:** turn it into the same list.

Show the list and ask them to confirm or reorder. Save only the section order
and notes, not the page's copy.

Default when skipped: spec 6 as written.

```json
"landing_page": {
  "source_url": "<product page URL>",
  "sections": [
    { "label": "Eyebrow", "notes": "2 to 4 words, all caps" },
    { "label": "Hero headline", "notes": null },
    { "label": "Customer logos", "notes": "no copy needed" },
    { "label": "Capabilities", "notes": "3 blocks" },
    { "label": "Customer quote", "notes": null },
    { "label": "CTA", "notes": null }
  ]
}
```

When a saved spec exists, spec 6's length and voice rules still apply to each
section it shares with spec 6. Sections spec 6 doesn't have get the length the
user noted, or a short default, and the PM is told which ones.

## Step D: Tier vocabulary

The skill uses three tiers (Tier 1, Tier 2, Tier 3 / Release) and a sample list
of assets for each. Ask:

> How do you name your launch tiers, and what does each one include?
> 1. Keep the default three tiers
> 2. I'll describe ours

If they describe theirs, record each tier's name, a one-line meaning, and the
assets it includes. Map each to the closest default tier (1, 2, or 3) so the
rest of the skill (and the optional `gtm-scoring` skill, which maps a score to
Tier 1 to 3) still works. If a tier doesn't map cleanly, ask which default it's
closest to.

Show the saved menu back and confirm. Keep the scope boundary: sales-enablement
assets and the internal product announcement stay redirects even if their menu
lists them.

Default when skipped: the three-tier menu in SKILL.md Step 2.

```json
"tiers": [
  {
    "name": "Major launch",
    "maps_to": 1,
    "meaning": "Coordinated launch across marketing, sales, and support",
    "assets": ["Positioning doc", "Blog post", "Email (prospect + installed base)", "Landing page copy", "In-app guide copy"]
  }
]
```

## Wrap up

1. Add `"version": "1.0.0"`, `created_at`, and `updated_at`.
2. Write the section to the config file named in SKILL.md → Config, under
   `release_kit` (create the file if absent; leave other sections alone). Read it
   back to confirm it parses.
3. If there's no writable folder, say settings won't persist, show the section
   as JSON so the user can save it, and accept it pasted back next time.
4. Confirm in one short paragraph what was saved, what uses the default, and how
   to change it ("update my release-kit setup", or "update my brand voice").
5. Continue to Step 1 of SKILL.md with the original request.

## Partial re-setup

- "update my release-kit setup" → ask which of the four to redo (or all)
- "update my brand voice" → Step A
- "update my product examples" → Step B
- "update my landing page spec" → Step C
- "update my launch tiers" → Step D
- "reset my release-kit setup" → confirm, then clear the `release_kit` section

Always read the current section first, show what's there, and update
`updated_at`. Don't touch other fields or other skills' sections.
