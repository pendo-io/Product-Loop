---
name: product-announcement-generator
description: 'Generates an internal product announcement for your team''s announcement channel using a short, punchy three-question format (Why care? Who''s affected? What''s next?), written so internal teams can scan it in under 10 seconds and act. Use when the user wants to create or draft a product announcement, write a launch announcement, prepare a GA / Beta / EA announcement, write a feature release post, or post to the team''s announcement channel. Triggers include "product announcement," "announce this feature," "launch post," "release announcement." Gathers context from tickets, docs, slides, Slack threads, and uploaded files when those are connected or provided. The output is a single concise announcement in markdown. Single owner of the internal announcement format and its dash, date, and ticket-id check. Not for the customer-facing release note; use release-note-drafter. Not for external launch copy such as blog posts or emails; use release-kit.'
license: MIT. LICENSE has complete terms
compatibility: Standalone. No required connectors or skills. Output is markdown in chat. Uses Jira, Confluence, Google Drive, Notion, and Slack if connected to read sources, a browser tool if available to capture a screenshot, and can save the approved draft to a record in a connected tracker such as Notion, Jira, or Airtable (opt-in). Reuses a brand voice from an installed style skill or your instructions if one exists. Works from pasted material otherwise.
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

# Product Announcement Generator

Generate internal product announcements using a short, punchy template. The goal is an announcement an internal reader (sales, customer success, product management, support) can scan in under 10 seconds and act on immediately. The post goes to your team's announcement channel (for example `#<your-announcement-channel>`); a person posts it, not this skill.

The format is built around three questions the reader actually has:

1. Why should I care about this?
2. Which of my customers does this affect?
3. What do I do next?

A **conviction doc** is a short pre-build statement of the problem, who has it, the evidence, and the bet. **EA** is early access. "Who cares most" names one or more roles at a company, the people who care most about this change (for example PM, CPO, CEO, CTO, CSO, sales rep, or support lead). It is not an internal team. **Who reads this** is your internal teams (sales, customer success, product management, support); **who cares most** is the roles they should think of when they act on it.

There is one format. If a user asks for a longer version, explain that this skill writes the one short format, and that extra detail goes in a thread reply (see Step 2).

## Done

- **Result:** one announcement in the three-question format, shown in chat as markdown, ready for a person to post. If the user asked to save it, it is also on their tracker record.
- **Done check:** when the source included tickets, the shipped vs not-shipped split was echoed and the user did not correct it (Step 1a; a run with no tickets has no split to echo); the pre-publish checklist and the mechanical check came back clean (`ok` is true, or every item was checked by hand when code can't run), or after two fix passes the draft is handed back marked as not passed with each remaining hit named; every remaining `[TBD]` gap and missing source is named at hand-back; if saved, the record was re-read, the header rendered bold, and the record link is given.
- **Next consumer:** the person who posts it, and the internal teams who read it. When a feature-creation skill calls this one, that skill stores the approved draft (Step 3.3).
- **Intent:** an announcement an internal reader can scan in under 10 seconds and act on, describing only what shipped. When the steps don't cover a case, choose the shorter, more accurate post.

## Boundaries and authority

- **Announce only what shipped.** Internal teams repeat these posts to customers, so a wrong claim has real cost. If the work hasn't shipped, draft a `Coming Soon` announcement instead and recheck the split before anyone posts it (Step 1a).
- **Never post.** A person posts the announcement to the channel. Hand over the draft in chat instead.
- **Never invent links, dates, or images.** Ask the user, or leave a `[TBD]` and name it at hand-back. Only use an image you captured or the user provided (Step 3.4).
- **Never create a tracker property or overwrite a non-empty value without asking.** The tracker belongs to the user and its existing content may be someone else's work. Write only to an existing text field, and ask before replacing what is there (Step 3.2).
- **The voice never overrides the format rules** (three-question format, no em or en dashes, no dates, ticket ids only inside URLs, the checklist, the mechanical check). Apply the voice to tone and word choice, and name any voice rule you set aside at hand-back (Step 1b).
- **No em or en dashes, and no dates, in the announcement.** Dates go stale because this copy is usually written before the launch date is final, and dashes display differently from one email client, chat tool, or in-app surface to the next (see Formatting).

**Authority:** this skill may write (1) the `product_announcement` section of `config.json`, after the user confirms the values; (2) an existing text field on a tracker record the user names, only when they ask to save; (3) a browser session to navigate to the feature and capture a screenshot, only when the user accepts the offer; (4) a sibling skill folder into `~/.claude/skills/<skill-name>/`, only on the user's OK. It never posts to any channel.

## Judgment vs mechanics

The model judges the value hook, the body wording, the roles under "Who cares most", which links help the reader most, and the voice. It also reads the tickets and decides the shipped vs not-shipped split; that split is done in prose from status and resolution, not by code, so it is echoed to the user before drafting. The mechanical check code in Step 2 owns the dash, date, ticket-key, and "now available" / "now live" hits; when code can't run, the same check is done by hand. The header pattern is written by the model, not printed by code.

## Honesty

- **Nothing shipped is a valid answer.** If no source shows shipped work, say so and offer a `Coming Soon` draft instead of a shipped announcement.
- **Couldn't read is not nothing there.** Name every source that failed and mark what it would have supplied as `[TBD]` (Access issues).
- **Ticket status is a claim.** Done or resolved alone doesn't mean shipped; check the resolution (Step 1a).
- **Read back what you saved** before giving the record link (Step 3.2).
- **Read back every other write.** Config: re-read the `product_announcement` section and check it parses with the confirmed values. Install: check the downloaded folder's `SKILL.md` is in place before saying the skill is installed.
- **No time estimates** for your own work; describe what is left instead.

## Setup (optional, ask once if useful)

Nothing is required. If the user gives these once, reuse them for the rest of the session:

- **Announcement channel** name (for example `#<your-announcement-channel>`).
- **Questions channel pattern**, if your team has one per product area (for example `#<area>-launch-questions`).
- **Product areas** and one emoji per area, so headers read consistently.
- **Status values**: your launch stages. The default set is `GA`, `Open Beta`, `Closed Beta`, `EA`, `Coming Soon`; replace it with your own stage names if they differ. Use `Coming Soon` for an announcement written before launch; switch it before posting (see Formatting).
- **Terminology glossary**, if your team keeps one: official product and feature names, internal shorthand to replace with plain terms, and verified names the mechanical check should skip (`allowed_names`, for example a name that contains a dash and digits).
- **Regions** your product ships to, if availability can differ by region.
- **Brand voice**, found by the voice lookup (Step 1b) and saved in `config.json` once confirmed.

When the user confirms these, offer to save them in the `product_announcement` section of the same `config.json` the voice uses (location rule in `references/voice.md`; shape in `config.example.json`), and read them from there next time. In a long session or after the conversation is compacted, re-read `config.json` rather than recalling values from chat. Saved values replace the defaults and the bundled examples, which use a fictional work-tracking product and are only there to show the shape.

## Workflow

### Step 1: Gather context sources

Ask the user to provide context sources for the announcement. If the feature already has a **feature record** (for example one produced by a feature-creation skill, or a row in the user's tracker), check it first: it often already holds the launch essentials (roadmap description, release note, status, regions, audience). Prefer the PM's own framing there. If Slack is connected and the area has a launch or go-to-market channel, check it too for recent context and help-article or demo links.

Supported sources, all optional:

- **Feature record**, markdown or a tracker page
- **Docs**: conviction docs, launch briefs, specs, marketing launch packets (Google Docs, Notion, Confluence)
- **Tickets**: feature tickets and epics (Jira or another tracker)
- **Slides** with release information
- **Slack threads** where the feature was discussed
- **Uploaded files** such as PDFs or other documents

Fetch sources with whatever connector is present (direct fetch by URL or ID, or search by name). If a fetch fails, try the connector's search by title or key, then ask the user to paste the content. If no connector is present, ask the user to paste the material.

Fetch and read all provided sources to extract:

- **Product area**
- **Feature name**
- **Status** (one of your saved status values, or the default set: GA, Open Beta, Closed Beta, EA, Coming Soon)
- **What changed**: the core thing the reader needs to know
- **Why it matters**: the value hook in one line
- **Who cares most**: the one or more roles at a company that care most about this change (for example PM, CPO, CTO, sales rep, or support lead), not an internal team
- **What to do next**: external resource (help article, blog post, marketing page, in-app path) or first action
- **Questions channel**
- **Attachments**: screenshots, GIFs, or short videos showing the feature, if available
- **Regional availability**: flag if the feature is *not* available in all regions

### Step 1a: Ticket sourcing and the shipped-accuracy gate

When the source is one or more tickets (an epic, story, task, or link), read them properly before drafting. This applies to any product area.

1. **Fetch each issue** (summary, description, status, issue type, resolution, fix versions, parent, labels).
2. **If an issue is an epic,** fetch its children (in Jira, search `parent = <KEY>`, oldest first). The children with a done-type resolution are what concretely shipped.
3. **If a description links a conviction doc or spec**, fetch it: the problem and bet framing gives the benefit framing (tickets describe mechanism; the conviction doc or spec describes outcome). Only read these; never create a PRD.

**The accuracy gate (most important):** an announcement must describe only work that **actually shipped**, must not overstate availability, and must not invent links or dates. Internal teams act on these posts, often in front of customers, so a wrong claim has real cost. From the tickets, split shipped from in-flight (still open) and from closed-without-shipping. A ticket counts as shipped only when its status is done or resolved (or it's in a released fix version) **and** its resolution is a done-type value such as Done or Fixed. Status alone isn't enough: a done or resolved ticket, or an epic, with resolution Won't Do, Duplicate, Cannot Reproduce, Incomplete, or similar did not ship. If the resolution is empty or unclear, ask. **Announce only the shipped part**; stay silent on the rest. The one exception is a `Coming Soon` draft written before launch (see Formatting): it describes the intended scope, and the shipped split is rechecked before anyone posts it. Then echo back the shipped vs not-shipped split in 2 to 3 lines so the user can correct course before you draft.

### Step 1b: Find the voice (optional)

Before drafting, run the lookup in `references/voice.md`: this skill's saved config, then installed skills that define a brand voice or writing style, then Claude's memory and standing instructions. If one is found, summarize it, name the source, and confirm once; save the pointer and summary in config. If several, let the user pick or merge. If none, use the built-in style below and offer (never require) to set one.

**The voice governs tone and word choice only.** It never overrides the three-question format, no em or en dashes, no dates, ticket ids only inside URLs, the pre-publish checklist, or the mechanical check. When a voice rule conflicts, the skill's rule wins; say which voice rule was set aside when you hand back the draft.

### Step 2: Draft the announcement

See `references/announcement-template.md` for the canonical template and field guidance, and `references/examples.md` for worked examples. If the user has a terminology glossary, check product names and terms against it.

The drafting order:

1. **Write the value hook first.** One line, concrete, reader-facing. Decide the single reason a reader should care; this anchors everything else.
2. **Draft the 2 to 3 sentence body.** Cover what changed, why it matters, what it unlocks. Resist a fourth sentence. Tone: telling a smart colleague over coffee, adjusted to the confirmed brand voice if there is one (Step 1b).
3. **Name the roles for "Who cares most."** One or more roles at a company (for example PM, CPO, CEO, CTO, CSO, sales rep, or support lead) that care most about this change, not an internal team. The announcement's readers stay your internal teams. Keep it to the roles that care most; if the line turns into a long list, cut to the ones who'd be most disappointed to miss it.
4. **Choose links thoughtfully.** An external resource (help article, blog post, marketing page) is almost always the right first link. Skip the internal-resource line unless it genuinely changes what the reader does next.
5. **Note regional availability.** If the feature isn't available in all regions, add a line in the body: *"{Feature Name} will be available in {List of Regions}."*
6. **Note any existing visual; source a new one after the draft is approved.** If the feature record already has an image, reference it. Sourcing a *new* screenshot happens in Step 3, not during drafting. A visual is expected on every announcement.
7. **Move supplementary detail to a thread.** If the user has extra context that doesn't fit the 2 to 3 sentence body (caveats, edge cases, deeper dives, FAQ-style detail), that goes in a thread reply on the posted announcement, not in the main post. Note this to the user when you hand back the draft.
8. **Run the 10-second test.** Read the draft start to finish. If it takes longer than 10 seconds, cut something or move it to the thread.

Before handing it over, run `references/pre-publish-checklist.md`, then the mechanical check below.

### Mechanical check (required, before hand-back)

Don't rely on reading the draft back. Scan the visible announcement text (header and body, not your notes to the user) literally for:

- **Dashes:** `—` (em dash) and `–` (en dash).
- **Dates:** month names (January to December, and abbreviations such as "Sep") in date-like context, meaning followed by a day number or a four-digit year, or preceded by "in", "on", "by", "since", or "until" ("in March"), plus four-digit years and numeric date patterns (year, month, and day joined by dashes, or day and month joined by a slash). A capitalized "May" that starts a sentence isn't a date.
- **Ticket ids:** a ticket key (letters, a dash, and digits, for example ABC-123) anywhere in the visible text, including link text. A key inside a link's URL, `[text](url)`, is fine, since readers don't see it.
- **Timing phrases:** "now available" and "now live". "Now" plus a capability verb ("You can now…", "now forecasts") passes.

**Skip placeholder text.** Remove every `[TBD…]` or `[PLACEHOLDER: …]` span (and any `(url)` right after it) before scanning, and name the remaining gaps at hand-back. **Allow-listed names:** a verified product or feature name, or a term such as "COVID-19" that looks like a ticket key, is skipped by every check. The allow-list is `product_announcement.glossary.allowed_names` in config plus any name you verified against the sources this session; never add one to make a hit go away. If code execution is available, run the check as code, for example:

```python
import re
ALLOWED_NAMES = []  # product_announcement.glossary.allowed_names from config, plus names verified this session
MONTH = (r"(January|February|March|April|May|June|July|August|September|October|November|December"
         r"|Jan|Feb|Mar|Apr|Jun|Jul|Aug|Sep|Sept|Oct|Nov|Dec)")
# Case-sensitive month names in date-like context, so "you may" and a sentence-initial "May" pass.
MONTHS = (rf"\b{MONTH}\.?,?\s+'?\d"
          rf"|(?i:\b(?:in|on|by|since|until)\s+){MONTH}\b")
DATES = r"\b(19|20)\d{2}\b|\b\d{1,2}/\d{1,2}(/\d{2,4})?\b|\b\d{4}-\d{2}-\d{2}\b"
PLACEHOLDER = r"\[(?:TBD|PLACEHOLDER)\b[^\]]*\](\([^)]*\))?"

placeholders = len(re.findall(PLACEHOLDER, draft))   # gaps to name at hand-back, not a failure
visible = re.sub(PLACEHOLDER, "", draft)              # skip placeholder text
visible = re.sub(r"\]\([^)]*\)", "]", visible)        # drop the URL inside [text](url), keep the text
scan = visible
for name in sorted(ALLOWED_NAMES, key=len, reverse=True):
    scan = scan.replace(name, "NAME")                 # skip verified names
dash_hits = [(m.group(), m.start()) for m in re.finditer(r"[—–]", scan)]
ticket_hits = re.findall(r"\b[A-Z][A-Z0-9]*-\d+\b", scan)
phrase_hits = [m.group() for m in re.finditer(r"\bnow\s+(?:available|live)\b", scan, re.I)]
date_hits = [m.group() for m in re.finditer(MONTHS + "|" + DATES, scan)]
ok = not (dash_hits or ticket_hits or phrase_hits or date_hits)
```

If code execution isn't available, go through the draft for each item above by hand. Fix every hit and re-run, for at most two fix passes. If hits remain after the second pass, hand the draft back marked as not passed, with each remaining hit named, rather than as ready to post. Flag names, "as of", and "today" have no pattern here; the checklist covers them.

### Step 3: Hand over, then get a visual

1. **Show the draft in chat** as markdown (see Formatting). This is the default deliverable.
2. **Save it (optional, opt-in).** If the user asks to save it to a record in a connected tracker (for example a feature record in Notion, Jira, or Airtable): ask for the record link or match by feature name (one confident match, use it; zero or several, ask which). Read the record's schema and current value first, and write to an existing text field for announcement copy (for example `Product Announcement`). Never create a property. **Never overwrite** a non-empty value without asking. After saving, re-read the record and check the header rendered bold; if it didn't, fix the markdown and save again. Confirm with the record link. If the write fails, say so and hand over the text.
3. **Called by another skill.** When a feature-creation skill (or similar) calls this one for its Product Announcement field, return the approved draft and let the caller store it. Don't save anywhere yourself in that case.
4. **Then get the visual.** A visual is expected on every announcement.
   - If the feature record already holds an image, use it and skip the rest.
   - Otherwise, if a browser tool is available in this session, **offer to capture it**: navigate to the feature and take a screenshot.
   - **If you don't know which environment to capture from**, don't guess. Ask the user for the URL of the right test, demo, or staging environment (and the path to the feature), or to paste or attach the screenshot themselves.
   - Hand the image back to the user to attach to the post. Never invent or hotlink a random image; only use one you captured or the user provided.

Then ask **"What next?"** with these options:

- *Regenerate with changes* (user describes edits in their next message)
- *Looks good, I'll post it*
- *Question about posting*

See `references/workflow.md` for the full flow.

## Asking the user for input

For any question with a closed set of choices (confirm or correct, single-select from a defined list, multi-select), use the surface's tappable-choice tool if it has one (for example `ask_user_input_v0`). Otherwise, show a short numbered list the user can answer with a number.

Use closed choices for:
- Confirming inferred values (status, regional availability, who cares most when picked from a short list)
- Resolving conflicts between sources (for example, "Sources disagree on status: which is correct?")
- Choosing the destination channel from known options
- Asking what to do after the draft is ready
- Choosing language(s) when sources are non-English

Don't use closed choices for:
- Free-text content the user has to write (value hook, body, "Who cares most" line, help-article URL)
- Open-ended source gathering ("paste your launch brief")
- Anything where the answer space is unbounded

Keep the introductory message brief. Don't list the options in prose and then also show them as buttons.

## Edge cases

### Missing information

When source documents don't contain required fields:

- **High confidence inference**: proceed with the inferred value, clearly mark it as inferred, and ask the user to confirm. For closed-choice fields (status, regional availability), offer the candidate value pre-selected with *"That's right"* / *"Let me correct it"*. For free-text fields, state the inferred value and ask the user to confirm or correct in their next message.
- **Low confidence inference**: ask the user for the missing information before generating the draft. Offer status as a single choice from the status values (your saved set, or the default five). For value hook, "Who cares most", and other free-text fields, ask in prose.

The two questions worth pushing on most: **what's the value hook** (one line, concrete) and **who cares most** (one or more roles). Without those two, the announcement won't land. Other gaps can use `[TBD]` placeholders.

### Access issues

If a document can't be fetched (no permission, a 404, or a bad URL):

1. Inform the user immediately which source failed
2. Ask for an alternate source or copy/paste of the content
3. **Block only on the main source** (the ticket or brief the announcement is built from). If that one can't be read, don't draft until the user gives an alternate or pastes it. If any other source fails, draft without it, mark what it would have supplied as `[TBD]`, and name the missing source when you hand back the draft

### Conflicting information

When sources contain conflicting information (for example, one says "GA" another says "Open Beta"):

1. State the conflict and indicate which direction you'd lean and why (for example, "The ticket was updated more recently and says Open Beta, so I'd lean toward that")
2. Offer the conflicting values as a single choice. Phrase the question simply, for example *"Which is correct?"* with *"Open Beta"* and *"GA"*
3. Wait for confirmation before proceeding

### Irrelevant sources

If a provided source doesn't appear relevant to the announcement:

- Extract whatever might be useful and proceed
- Mention in the draft review that relevance was unclear for that source
- The user will edit the draft to fit their needs

### Non-English sources

When source documents are in a language other than English, ask *"Which language(s) do you want?"* as a multi-select with these options:

- *English only*
- *Source language only*
- *Both: I'll choose one or combine them*

Generate accordingly based on the user's selection.

### Multiple features in one announcement

When the user wants to combine multiple small features into one announcement:

1. Work with the user to propose a combined header and value hook
2. Once agreed, write a single 2 to 3 sentence body that cohesively covers all items
3. If a single value hook genuinely doesn't fit, push back: multiple announcements may be cleaner than one bloated post

## Formatting

For detailed formatting guidelines, see `references/announcement-template.md`.

**Use standard markdown (no Slack codes):**

| Element        | Format                                              |
| -------------- | --------------------------------------------------- |
| Bold           | `**text**` (double asterisks)                       |
| Italic         | `*text*` or `_text_`                                |
| Links          | `[text](https://url)`                               |
| Bullets        | `* ` or `-`                                         |
| Nested bullets | indent 4 spaces, then `* `                          |
| Channels       | plain text `#channel-name`, or a link; no `![](#id)` |
| People         | plain name; never `<@U12345>`                        |

**Header line:** Single line. Pattern: `[emoji] **[Product Area] | [Feature name] | [Status]:** **[value hook]** [emoji]`. The same emoji bookends both ends. No date in the header. The emoji can be a character or a shortcode such as `:bar_chart:`; shortcodes are fine (the "no Slack codes" rule is about channel and user mention codes).

**Status values:** your saved set if there is one; otherwise the default `GA`, `Open Beta`, `Closed Beta`, `EA`, `Coming Soon`. Use `Coming Soon` for an announcement written before launch; switch it before posting.

**No dates. State the capability, not the timing.** "Now" plus a capability verb is fine (for example "You can now…" or "now forecasts"); "now available", "now live", "as of", "today", and dates are out. Dates go stale because this copy is usually written before the launch date is final, so the header carries Status only. Regional-availability lines are about place, not timing, and are fine. release-kit states the same rule for its announcement copy.

**No dashes.** Never use an em dash (—) or en dash (–) anywhere in the announcement, including the header. Use `|` as the header separator, and commas, colons, or periods in the body.

**Plain language, real product terms.** Write for the internal teams reading the announcement, not only the product team. Replace internal shorthand and acronyms they won't recognize (for example, a project codename or a team acronym: name the product or feature instead). If the user has a terminology glossary, check product names against it; if a term in the source docs conflicts with the glossary, use the glossary term.

**No ticket ids.** Ticket ids and keys (for example a Jira key) and feature-flag names never appear in the announcement body, header, or link text. A ticket reference may appear only inside a link's URL.

## Related skills (optional)

These are separate skills in https://github.com/pendo-io/Product-Loop. None is required.

- **release-note-drafter** writes the customer-facing release note for the same launch.
- **feature-creation** calls this skill for a feature record's Product Announcement field.
- **release-kit** writes the customer-facing launch copy (blog post, email, social, FAQs) for the same launch.

If the user asks for one of those jobs and the skill isn't installed, say it exists and offer to install it. In Claude Code, or anywhere you can write files and reach GitHub, offer to do it yourself, and on the user's OK download that skill's folder from the repo into `~/.claude/skills/<skill-name>/`. Elsewhere, tell the user to download the folder and upload it as a skill. Never block on it.

## Maintenance

When this skill misbehaves, open an issue at https://github.com/pendo-io/Product-Loop/issues. Each fix lands as a named rule plus a line in `references/pre-publish-checklist.md`.
