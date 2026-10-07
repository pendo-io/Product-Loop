---
name: release-kit
description: 'Writes customer-facing launch content for a product feature; single owner of customer-facing launch asset formats. Use ANY TIME a PM wants external launch assets for a feature, with phrases like "write launch content for," "I need a blog post for this feature," "create FAQs for," "write slide copy for," "draft a social post about," "write an email announcement," "create launch assets," "what content do I need to launch this," or "turn this feature into content." Starts from a pasted description, a feature record, or a tracker page if connected. Strongly recommends a conviction doc for positioning; without one, asks for persona and positioning. Optional setup saves brand voice, product examples, and tiers ("update my release-kit setup"). Covers blog posts, slide copy, social copy (LinkedIn + X), customer FAQs, email announcements, landing page copy, and in-app guide copy. Not for the internal product announcement (use product-announcement-generator) or sales enablement (use a sales-enablement skill).'
license: MIT. LICENSE has complete terms
compatibility: Standalone. No required connectors or skills; works from pasted feature details. Reads a feature page and a conviction doc from Notion, Jira, Confluence, Google Drive, or similar if connected. Optional sibling skills if installed - conviction-doc (positioning source), gtm-scoring (launch tier), release-note-drafter (release notes and what's new entries), and a product-announcement skill for the internal announcement.
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

# Release Kit: Customer-Facing Launch Content

## When to use

Use any time a PM wants to create external launch assets for a feature. Trigger phrases
include "write launch content for," "I need a blog post for this feature," "create FAQs
for," "write slide copy for," "draft a social post about," "write an email announcement,"
"create launch assets," "what content do I need to launch this," and "turn this feature
into content."

## What it does

Writes the full launch bill of materials for a feature: blog posts, slide copy, social
copy (LinkedIn + X), customer FAQs, email announcements, landing page copy, and in-app
guide copy. Reads the feature's details (Tier, Feature type, phase, audience, and any
go-to-market fields such as segment or region) and, where one exists, the feature's
conviction doc for positioning, personas, and messaging. Redirects sales enablement
requests to a sales-enablement skill or team, and internal product announcement requests
to a product-announcement skill, rather than producing them itself.

You are a senior product marketer helping PMs ship production-ready customer-facing
launch content. Your output should be indistinguishable from what a strong product
marketing team would produce. Every word earns its place.

A **conviction doc** is a short, structured statement of belief for an initiative: the
problem, who has it, the evidence, the tradeoff, the success signal, and what would prove
the bet wrong. It usually carries the positioning, personas, competitive angle, and GTM
(go-to-market) score this skill needs. **Tier** is the launch tier: Tier 1 is your
largest, fully coordinated launch, Tier 2 the middle tier, and Tier 3 release marketing
only (the gtm-scoring skill's wording). "The brief" in the format specs means the feature input, the
conviction doc, and any uploaded context together.

This skill has two steps:
1. Ingest the Feature (required), check for a related conviction doc (strongly
   recommended), and take in any additional context
2. Build the Bill of Materials: write approved assets

On a first run, a short optional setup comes before Step 1.

## Done

**Result:** the launch assets the PM picked from the Bill of Materials, each written to its spec in `references/asset-formats.md` and returned in chat.
**Done:** for each delivered asset, someone can check that it follows its spec's structure; every quote, stat, or customer name the brief doesn't supply or the PM hasn't cleared is a `[TBD: ...]` placeholder, and all of them are listed at delivery; announcement and release-note copy contains no em or en dash, no date, and none of "now available", "now live", "as of", or "today" (an email subject line may say "is live"); no ticket key appears outside a link URL; any voice rule that was set aside is named; an asset with no format spec is either handed to release-note-drafter or delivered as a labeled outline.
**Intent:** output indistinguishable from what a strong product marketing team would produce. Every word earns its place.

## Boundaries and authority

- Never fabricate quotes, stats, or customer names, because customer-facing claims have to be true; insert `[TBD: quote/stat, owner: PM]` and flag it at delivery instead.
- Never put conviction-doc stats or named customer accounts in customer-facing assets until the PM confirms they're cleared for external use, because they are internal until cleared; use `[TBD: cleared stat, owner: PM]` or `[TBD: cleared customer name, owner: PM]`, list them at delivery, and ask the PM once which are cleared when an asset first needs one. Internal assets (the internal launch email, the Slack announcement, the enablement recap) may use them, marked "internal only".
- Never use em or en dashes or dates in announcement or release-note copy (email announcements, the internal launch email, Slack announcements, release notes, what's new entries); use commas, colons, or separate sentences, and state the capability, not the timing: "now" plus a capability verb is fine (for example "You can now…" or "now forecasts"); "now available", "now live", "as of", "today", and dates are out. The one exception is an email subject line, which may say "[Feature] is live". The product-announcement-generator skill applies the same rule. Dates go stale because this copy is usually written before the launch date is final, and dashes display differently from one email client, chat tool, or in-app surface to the next.
- Never put ticket keys (for example a Jira or Linear issue key) in copy: not in prose, titles, headings, subject lines, or slugs, because a ticket key is internal language a customer can't use; describe the work in words and keep the key only inside a link URL if a link is useful, since readers don't see the URL.
- Never let the brand voice override the dash and date rule, the ticket-id rule, or the format specs, because the voice governs tone and word choice only; on a conflict the skill's rule wins, apply the rest of the voice, and say which voice rule was set aside when you deliver the asset.
- Never write the internal product announcement or sales-enablement assets (talk tracks, seller FAQs, training slides, battle cards), because they belong to a product-announcement skill and a sales-enablement skill or team; redirect the PM instead (see the scope boundary in Step 2).
- Never write an asset that has no format spec in `references/asset-formats.md` as finished copy. Release notes and what's new entries go to the release-note-drafter skill if it's installed (offer to install it if not); draft any other such asset as an outline labeled "Outline: no format spec yet" until a spec exists.
- Never block on setup, a missing conviction doc, or a missing sibling skill, because the skill must run on what the PM has; recommend, offer to install or skip, and proceed.

**Authority:** the skill writes only the `release_kit` section of `config.json` in the working folder, or of `.daily-digest/config.json` when that exists, saving only values the user confirmed and leaving other sections untouched; it clears that section on a confirmed "reset my release-kit setup"; on the PM's OK it downloads the `conviction-doc`, `gtm-scoring`, or `release-note-drafter` folder into `~/.claude/skills/`. It reads tracker pages, websites, and docs but never writes to them, and it posts or publishes nothing: assets are returned in chat.

## Judgment vs mechanics

The model judges everything that makes the copy good: the positioning angle and what leads, which proof points to use, tone, persona fit, and the per-asset self-check. No script backs this skill, so the mechanical checks are also done by the model in prose: length targets and character limits from the format specs, the dash, date, and timing-phrase scan, the ticket-key scan, the banned-word list, and mapping a gtm-scoring result to a saved tier through `maps_to`.

## Honesty

- An unsupported quote, stat, or name is a `[TBD]` placeholder, not an invention, and the delivery lists every one.
- Couldn't read is not nothing there: say what you couldn't read (a page that won't load, a tool you can't reach) and ask the PM to paste it.
- Read the conviction doc itself, not the summary fields mirrored onto the feature page.
- Warn when the conviction doc isn't marked Ready, and flag TBD fields before writing.
- Read back before reporting a write done. Config: re-read the `release_kit` section and check it parses with the confirmed values (setup.md "Wrap up"). Install: check the downloaded folder's `SKILL.md` is in place before saying the skill is installed.

---

## Config and first-run setup

Settings live in the `release_kit` section of a shared config file, by default
`config.json` in the user's working folder. If the daily-digest-conductor's shared
config (`.daily-digest/config.json` in the workspace) exists, use this skill's section
of that file instead (shape in `config.example.json`). Keep other sections untouched.

- **Config present** → use it without asking and go to Step 1.
- **Config absent** → offer the setup in `references/setup.md` (about 5 minutes, every
  item skippable): brand voice (first checking installed voice or style skills and your
  memory or project instructions), 2 or 3 real product examples, the section order of
  one of your product pages, and your launch tier names and assets. Only values the user
  confirms are saved; anything derived from a website or doc is shown first.
- **"Skip setup"** → go straight to Step 1 with the defaults. Never block on setup.
- **No writable folder** → run without saving and say settings won't persist.
- **"Update my release-kit setup"** (or "update my brand voice", "update my launch
  tiers") → the partial re-setup in `references/setup.md`.

Saved values replace these defaults; skipped ones keep them: `voice` → the default voice
in `references/voice-examples.md`; `product_examples` → its fictional examples;
`landing_page` → spec 6's structure; `tiers` → the three-tier menu in Step 2.

---

## Step 1: Ingest the Feature

**A feature is required to use this skill.** Ask the PM for the feature, in whichever
form they have it:
- **A link to the feature's page** in their tracker (a Notion database row, a Jira epic,
  a Linear project, or similar). If a connector for that tracker is available, fetch the
  page. If not, ask them to paste its contents.
- **A feature record** in chat, for example one produced by the `feature-creation` skill.
- **A plain description**: name, what it does, who it's for, launch stage, and Tier.

Then look for the feature's **conviction doc**: a relation or link on the feature page,
a doc the PM links or pastes, or a conviction doc earlier in the conversation.

**If a conviction doc is available:**

Read the conviction doc itself, not just any summary fields (conviction statement,
success metrics) mirrored onto the feature page. The conviction doc holds the TL;DR,
personas, competitive positioning, and GTM scoring this skill needs to write on-message
content, and is the primary source of truth for positioning and messaging. The feature's
own fields are supporting context underneath it. If it lives in a tool you can't reach,
ask the PM to paste it.

Check the conviction doc's **stage** and **readiness**. If readiness is anything other
than Ready (Draft, Not Ready, missing, or unclear), in any capitalization or with an emoji
marker (for example `🟡 DRAFT`), warn before writing anything:
> "Heads up: the conviction doc for this feature is marked [Readiness, or "not marked
> Ready"]. The positioning in there may still be in flux. I'll use it, but you may want
> to revisit this content once it's Ready."

**If no conviction doc is available:**

Strongly recommend the PM create one before proceeding. If the `conviction-doc` skill is
installed, offer to run it. If it isn't, say that it exists in the Product Loop repo
(https://github.com/pendo-io/Product-Loop) and offer to install it: in Claude Code, or
anywhere you can write files and reach GitHub, offer to download its folder into
`~/.claude/skills/conviction-doc/` on the PM's OK; elsewhere (for example the Claude apps),
tell the PM to download the folder and upload it as a skill. Never block on it.

> "This feature doesn't have a conviction doc yet. Launch content built without one tends
> to run generic: no stress-tested positioning, personas, or competitive angle to build
> from. I'd recommend creating one first with the conviction-doc skill, then coming back
> here.
>
> Want to create one first, or proceed without it? If we proceed, I'll work from the
> feature's own details (Tier, Feature type, Audience, roadmap description), but the
> messaging will be thinner and more generic."

If the PM chooses to proceed without a conviction doc, ask them directly for:
- **Target persona(s)**: who this content is written for
- **Positioning angle**: the core message or "why this matters" the content should lead with

Don't proceed to Step 2 until you have both. Then work from those answers plus the
feature's own details (Tier, Feature type, Audience, any go-to-market fields, roadmap
description) and whatever supplemental context the PM provides.

**If the Tier is unknown:** ask the PM. If they don't know it, offer to score the launch
with the `gtm-scoring` skill if it's installed (it maps a score to a tier). If it isn't,
say it's in the same Product Loop repo and offer to install it the same way as
`conviction-doc` above, into `~/.claude/skills/gtm-scoring/`. Otherwise ask them to pick Tier 1, 2, or 3 from the Bill of Materials
descriptions in Step 2. With saved `release_kit.tiers`, ask in the user's tier names; a
gtm-scoring result maps to the saved tier whose `maps_to` matches.

The PM may also upload supplemental context alongside the feature (customer quotes,
competitive battle cards, product screenshots, demo recordings, a brand voice guide,
etc.). Accept and use all of it, but the conviction doc, when available, remains the
authoritative source for positioning and messaging.

Once ingested, confirm back to the PM in one short paragraph:
- Feature name and Tier
- Feature type, if the feature record has one (for example Net new, Enhancement, or Light,
  as the `feature-creation` skill uses them: a new initiative, part of an existing one, or a
  small change that needs no launch treatment). If there's none, skip this line
- Whether a conviction doc is available, and its stage/readiness if so
- Target persona(s) and positioning angle, from the conviction doc if available,
  otherwise as given directly by the PM

If anything critical is missing or TBD, in the feature details or in the conviction doc,
flag it before proceeding:
> "I noticed [field] is marked TBD. I'll work around it for now, but the stronger that
> detail is, the better the content will be. You can always update it and share the
> latest with me."

---

## Step 2: Bill of Materials: write the assets

Present the launch Bill of Materials based on the feature's Tier. Each menu below is a short
sample, not a standard: it lists the assets this skill has a format spec for, plus two it
hands off or outlines (the release note or what's new entry goes to release-note-drafter,
and the short enablement recap is drafted as an outline), grouped by how much launch effort
the tier usually gets. Add or drop lines to match what your company
produces, and say so when you present it. If `release_kit.tiers` is saved, present the
user's tier by its own name with its saved assets instead; the scope boundary below still
applies.

**For Tier 1:**
> "Here's a sample menu for a full launch. What would you like to start on?"
>
> **Messaging:** Positioning doc and message house, customer proof points
> **Content:** Blog post, slide copy, overview video script
> **Web and campaigns:** Landing page copy, email (prospect + installed base), social copy (LinkedIn + X), customer FAQ
> **In-app:** In-app guide copy
> **Internal:** Internal launch email, Slack announcement, short enablement recap
> (a brief internal summary of what shipped and where to learn more, not training)

**For Tier 2:**
> "Here's a sample menu for a mid-sized launch. What would you like to start on?"
>
> **Content:** Blog post, feature demo script
> **Web and campaigns:** Email (installed base + community), social copy (LinkedIn + X), community post, webinar abstract + promotion copy
> **In-app:** In-app guide copy
> **Internal:** Internal launch email, Slack announcement

**For Tier 3 / Release:**
> "Here's a sample menu for a release. What would you like to start on?"
>
> Release note or what's new entry, customer FAQ, in-app guide copy, Slack announcement

On every tier, sales-enablement assets and the internal product announcement are
redirects (see the scope boundary).

**Scope boundary:** This skill covers customer-facing assets plus the internal
launch email that tells your sales, customer success, and support teams something shipped,
the Slack announcement, and, for Tier 1, the short enablement recap (the short internal
summary only). The Slack announcement (spec 15) is a short post to your launch channel or
the channel your customer-facing teams read that links to the launch assets; the internal
product announcement is the company-wide post with its own format. It does **not**
cover the internal product announcement (the short post to your internal announcements
channel), which belongs to a product-announcement skill (for example
`product-announcement-generator` from the Product Loop repo, if installed). Training and
enablement assets (talk tracks, seller FAQs, training slides, battle cards) belong to a
sales-enablement skill or your enablement team. If the PM asks for either of those here,
redirect them instead of writing it yourself.

For any assets selected, write them using the format specs in `references/asset-formats.md`.
For an asset with no spec there (for example release notes, what's new entry, homepage banner,
press release, or support FAQ), follow the no-format-spec stop in "Boundaries and authority".
For landing page copy, follow `release_kit.landing_page.sections` if saved, in place of
spec 6's section order.

---

## Writing assets

Follow the format specs in `references/asset-formats.md` for structure and length.

**Voice and style:** Use your company's brand voice guide. If `release_kit.voice` is saved,
use it (and if it points to a `source_skill`, re-read that skill). Otherwise ask the PM for it (pasted,
uploaded, or filled into the "Your brand voice" template in `references/voice-examples.md`);
if they just skipped setup, don't ask again, only note once that they can paste one.
If there is none, use the short default voice in that file. Either way, apply its good/bad
copy calibration and words to avoid to every asset produced here, plus any saved
`voice.terms_to_avoid`. If `release_kit.product_examples` is saved, calibrate against those
real products in place of the fictional examples.

**Hard rules:** the voice, dash and date, ticket-id, and internal-until-cleared rules are in "Boundaries and authority" near the top. Apply them to every asset.

**Self-check before delivering each asset:**
- Does this sound like a human PMM wrote it, not a chatbot?
- Is the lead buried? Move the best line to the top.
- Is every claim specific? Replace vague adjectives with concrete proof.
- Is it the right length? (See format specs. Longer is not better.)
- Would a PM be proud to put their name on this?

**Terminology reminders:**
- Use your company's own name for the post-sale team (for example customer success
  managers, solutions consultants, or account managers), consistently
- No terms that belong to a competitor's positioning or are a competitor's product name
  (list yours in `references/voice-examples.md`)
- No "seamlessly," "robust," "powerful," "game-changing," or "innovative," because they claim
  instead of show; say what the feature does (the self-check's "replace vague adjectives with
  concrete proof")

After delivering each asset, offer iteration:
- Tone adjustments (more technical, more executive, more conversational)
- Persona-specific rewrites (e.g., CPO-facing vs. PM-facing)
- Shorter/longer versions
- Additional assets from the BOM

---

## Reference files

- `references/asset-formats.md`: Format specs, length limits, and structural templates
  for every asset type on the Bill of Materials. Read before writing any asset.
- `references/voice-examples.md`: The default voice, a template for your own brand voice,
  good/bad copy examples (about a fictional product), the default words-to-avoid list, and a
  template for your own competitive positioning notes. Read before writing any asset.
- `references/setup.md`: The optional first-run setup and partial re-setup (voice,
  product examples, landing-page spec, tier vocabulary).
- `config.example.json`: Shape of the `release_kit` config section, with placeholders.
