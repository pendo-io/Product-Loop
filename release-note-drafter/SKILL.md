---
name: release-note-drafter
description: 'Draft the customer-facing release-note entry for a shipped or shipping change, following a bundled release-note standard and matching the length and one-paragraph format of your existing entries. Single owner of the release-note entry and its mechanical check. Takes a ticket reference (for example a Jira epic or story) and reads it, its resolved children or parent, and any linked conviction doc or spec, or works from an SME description, feature brief, spec, doc page, or changelog. Use whenever the user wants the published release note for a change ("draft a release note", "write the What''s New entry", "release note for this epic") or an article update needs its companion release note. Covers new features, enhancements, renames, metric changes, and open beta to GA; not bug fixes or backend-only changes. Not for an internal team announcement; use product-announcement-generator. Not for a documentation-request ticket, help-article bodies, blog posts, or marketing copy.'
license: MIT. LICENSE has complete terms
compatibility: Standalone. No required connectors or skills. Output is markdown in chat. Uses Jira (or another ticket tracker), Notion, Google Drive, or Confluence if connected to read sources, and can save the approved entry to a tracker record you choose (opt-in). Reuses a brand voice from an installed style skill or your instructions if one exists. Works from pasted material otherwise.
metadata:
  author: Pendo Product Operations
  version: "1.0.0"
---

# Release Note Drafter

Drafts a single-paragraph entry for your product's release notes page, following the bundled release-note standard and matching the length and structure of your existing entries so it drops in with minimal editing. A good entry answers, in order: **what customers can now do → how it behaves → why it matters**, plus a closing doc pointer. Short: typically 2 to 4 sentences, 50 to 90 words.

**Default page model.** The standard assumes a single release notes page where each entry is a card, the page (not the entry text) shows the status, product, and timing, and readers see the title and opening sentence before opening a card. If your page works differently (per-product "What's new" pages, no tags), keep the same rules and report the metadata your page does use.

A **conviction doc** is a short pre-build statement of the problem, who has it, the evidence, and the bet. An **SME** is the subject-matter expert (usually the PM or engineer) who can answer questions about the change.

## Done

- **Result:** one release-note entry in chat (bold title plus one paragraph), with card metadata, any flags and assumptions, and a short notes block, in the Step 5 order.
- **Done check:** the Step 4 mechanical check returns `ok` true (run as code, or each item by hand), or after two fix passes the entry is handed back marked as not passed with each remaining hit named, every Step 5 item that applies is present in order, and the "Not ready to publish: N placeholders" line appears whenever placeholders remain. A length warning does not block done.
- **Next consumer:** the user, who copies the entry to the release notes page; or a calling skill such as feature-creation, which stores the returned entry in its Release Note field (Step 5).
- **Intent:** customer-facing copy, not an engineering recap. When the steps don't cover a case, leave a visible placeholder rather than invent a name, benefit, or link.

## Boundaries and authority

- **Never invent** a product, feature, UI, API, or SDK name, a benefit, an availability status, a doc title, or a URL, because a visible gap beats an invented benefit. Use a `[PLACEHOLDER: …]` with a suggested SME question and flag it instead.
- **Never present an entry that hasn't passed the mechanical check** (Step 4) as ready, because reading the entry back misses characters, dates, and ids. Fix each hit and run the check again, for at most two fix passes; if hits remain, hand the entry back marked as not passed with each remaining hit named. A length warning is a note, not a fail.
- **No em dashes, en dashes, ampersands, or dates in the entry**. Em dashes are the most common failure in generated release notes, and dates belong to the card metadata so the status clause is the only part that changes when a feature graduates. Use commas, colons, parentheses, "to", or "and", and report the month as card metadata.
- **Closed beta never publishes.** Draft the entry and lead with a hold flag; it publishes at open beta or GA.
- **Bug fixes, defect corrections, and backend-only changes get no note**, because a release note covers change a customer can notice. Say so and confirm with the user before drafting; this confirm question is the one exception to never pausing after the echo-back (Step 0).
- **In a tracker, write only to the one existing text field the user names.** Never create a property, overwrite a non-empty field, or change any other field on the record. Show the existing value and ask, or hand over the text.
- **Reference conviction docs or specs; never create a PRD.**
- **A brand voice never overrides** the format, length, punctuation, date, ticket-id, or mechanical-check rules. Say in the notes which voice rule was set aside.

**Authority:** the skill can make three writes, each only after the user confirms or asks: (1) the `release_note_drafter` section of `config.json` (or of the conductor's `.daily-digest/config.json`), after setup or voice values are confirmed; (2) one existing text field on a tracker record the user names (Step 6), opt-in; (3) a sibling skill folder downloaded into `~/.claude/skills/<skill-name>/` (Related skills). Everything else is a read. When another skill calls it, it returns text and the caller stores it.

## Judgment vs mechanics

The model judges eligibility, what shipped and how to group it, which names to confirm, the title, the wording, the value pattern, the doc target, and which voice rules to set aside. The mechanical check snippet in Step 4 computes the character, bare URL, ticket id, banned phrase, and date hits, the word count, and the readiness line when code execution is available; without it, the same check is done by hand. Still done in prose every run: the shipped-ticket filter (done status plus a done-type resolution), the length comparison against recent entries, and the Step 5 order. In the Step 5 output, the flags, assumptions, and notes are evidence for the user to judge; the entry and the readiness line are the output to copy as is.

## Honesty

- **An empty result is valid.** A bug fix or backend-only change gets no note; say so rather than draft one.
- **"Couldn't check" is not "nothing there."** If the release notes page, tracker, or doc source can't be reached, say so and ask for a paste; without the user's style files, note that detailed style checking was limited; with no tracker connected, say so and hand over the text.
- **Inferred is not read.** Anything inferred rather than read (scope, names, availability, the article picked) goes under Assumptions.
- **Small references are labeled.** Existing entries are a length reference only; staging content doesn't count.
- **No time estimates** for the work.
- **Read back before reporting a write done.** Config: re-read the `release_note_drafter` section and check it parses with the confirmed values. Tracker: re-fetch the record and check the field holds the saved entry (Step 6). Install: check the downloaded folder's `SKILL.md` is in place.

## Setup (optional, ask once if useful)

Nothing is required. If the user gives these once, reuse them for the rest of the session:

- **Release notes page URL**, used as the length reference.
- **Product tags** your page uses (for example your product or module names).
- **Status labels**, if your page names open beta and GA differently (for example "Preview" and "Available"). The open beta and GA rule still applies; the labels only change what you report.
- **Help center location**, where the doc pointer's articles live.
- **Style files**, if any: a product style guide, UX writing guide, or terminology glossary.
- **Glossary values**, if you want them applied every time: official product and feature names (with capitalization), terms to avoid with what to use instead, the hierarchy words your product uses (for example "workspace" or "team"), extra banned phrases (for example a positioning phrase that belongs in marketing, not release notes), and verified names that contain "&" or a banned word, which the mechanical check then allows.
- **Brand voice**, found by the voice lookup (Step 0a) and saved in `config.json` once confirmed.

When the user confirms these, offer to save them in the `release_note_drafter` section of the same `config.json` the voice uses (location rule in `references/voice.md`; shape in `config.example.json`), and read them from there next time. Saved values replace the bundled examples, which use a fictional work-tracking product and are only there to show the shape.

## Standards reference

Before drafting, read `references/release-note-standards.md` bundled with this skill. It is the standard for release notes: eligibility (what gets a release note), the open beta / GA publishing rule, when to state availability or requirements, entry structure, required body format, the terminology-precision traps list, durability rules, punctuation, placement, and the finalization checklist. Where the reference and this file overlap, the reference wins. When the user's own style files (product style guide, UX writing guide, terminology glossary) are available, apply them too; if they aren't, follow the reference and note that detailed style checking was limited.

## Length reference

Use recent entries on the release notes page, filtered to the same product tag, as the length reference. If you can't fetch the page, ask the user to paste a few recent entries for that product, or work from entries already in the conversation. Staging content doesn't count as a reference.

**Use existing entries for length and cadence only.** Many predate the current standards, so don't copy their wording, openings, terminology, capitalization, or marketing phrasing. Where an existing entry and `references/release-note-standards.md` disagree, the standards win.

## Step 0: Read the source and gather context

**If the user gave a ticket reference** and a ticket tracker (for example Jira) is connected:

1. **Fetch the referenced issue** (summary, description, status, resolution, issue type, fix versions, resolution date, parent). If a call fails, check which sites the connector can reach before retrying.
2. **If an epic**, fetch its children (in Jira: `parent = <KEY> ORDER BY created ASC`, limiting fields to summary, status, resolution, and issue type so the result stays small). **Only shipped children belong in the note once the feature has shipped.** A ticket counts as shipped only when its status is done or resolved **and** its resolution is a done-type value (for example Done or Fixed). Status alone isn't enough: a done or resolved ticket with resolution Won't Do, Duplicate, Cannot Reproduce, Incomplete, or similar did not ship, and neither does an epic with one of those resolutions. If the resolution is empty or unclear, ask. If you're drafting before launch, draft from the intended scope instead (see "Drafting before launch" in Step 1).
3. **If a story or task**, that issue is the change; optionally read the parent epic for framing, but don't expand scope.
4. **If it links a conviction doc or a spec**, fetch it. The problem and bet framing is the best source for benefit framing (tickets describe mechanism; the conviction doc or spec describes outcome). Reference conviction docs or specs; never create a PRD.

If no tracker is connected, ask the user to paste the ticket and its children, or the relevant parts.

**If the user gave non-ticket material** (SME description, brief, spec, doc page, changelog, or a just-finished companion article): read it, then search any connected doc source (Notion, Google Drive, Confluence) for the feature brief, conviction doc, spec, or product marketing material about the change. If none is connected, ask the user to paste the source or point you to it. If a ticket key turns up along the way, follow the ticket path above for it.

**In both cases:**

5. **Pull recent release notes for the same product tag** (for length only) per "Length reference" above.
6. **Confirm names and availability against context, not the source alone.** Use connected docs and the user's glossary (when present) to confirm the real feature, UI, API, and SDK names and their capitalization, plus any beta, tier, or gating signal. If context conflicts with the source, surface the conflict rather than picking one silently.

Echo back what you found in 2 to 3 lines (the change; which children shipped, which are in flight, and which were closed without shipping, if from tickets; the conviction doc or spec framing; any beta or gate signal), then keep going and draft in the same response. **Never pause after the echo-back.** The one exception: if the source describes a bug fix, defect correction, or backend-only change, ask the Step 1 confirm question and wait for the answer before drafting. Anything you inferred rather than read (scope, names, availability, which article to link) goes in the Assumptions list in Step 5, so the user can correct it after seeing the draft.

## Step 0a: Find the voice (optional)

Before drafting, run the lookup in `references/voice.md`: this skill's saved config, then installed skills that define a brand voice or writing style, then Claude's memory and standing instructions. If one is found, summarize it, name the source, and confirm once; save the pointer and summary in config. If several, let the user pick or merge. If none, use the built-in style and offer (never require) to set one.

**The voice governs tone and word choice only.** It never overrides the release-note standard's format and length, no em or en dashes or ampersands, no dates, ticket ids only inside URLs, or the mechanical check. When a voice rule conflicts, the skill's rule wins; say which voice rule was set aside in the notes block.

## Step 1: Identify what actually shipped, and whether it gets a note

Announce shipped behavior, not roadmap. From shipped tickets (done-type resolution, per Step 0) or the source, list the user-facing surfaces that changed (filters, columns, settings, controls, panels, calculations). Group related work into one capability. **Include** anything a customer touches in the UI or any change to how a metric is calculated or reported; **exclude** internal plumbing (flags, pipelines, refactors, cleanup) and anything still in backlog, to do, or in development, or closed as Won't Do, Duplicate, or a similar non-done resolution. Watch for partials: if only part of a spec's scope shipped, cover that and stay silent on the rest (noting what you excluded).

**Without tickets, confirm what shipped.** Briefs, specs, and SME descriptions often describe planned scope. If the feature has shipped and the source doesn't make clear which parts are live for customers, say what you plan to announce in the echo-back, draft it, and list that scope under Assumptions in Step 5 for the user to confirm.

**Drafting before launch.** A draft can start before the feature is built. Write from the intended scope and put a `[PLACEHOLDER: …]` on whatever isn't decided yet (limits, options, availability, and so on). The placeholder count in Step 5 is the only readiness signal: add no warnings or reviewer notes about how early the feature is, and don't ask the user to confirm scope or eligibility for that reason. Comparing the draft with what shipped is a separate check before it publishes.

Then apply the eligibility gate from the standards reference:

- **Bug fixes, defect corrections, and backend-only changes don't get a release note.** If that's what the source describes, say so and confirm with the user before drafting.
- **Closed beta is not eligible to publish.** Draft the entry, but lead the output with a hold flag: publish only at open beta or GA.
- **Default to generally available.** Treat the change as GA unless the source or context shows a beta, a paid tier surfacing outside its home context, an access-gating setting, or a minimum app or SDK version. Actively check only for a beta signal. Don't manufacture a caveat.

## Step 2: Card metadata

The page sets each card's month group, status tag, and product tag, so they never appear in the entry text. Report them alongside the draft instead:

- **Month:** when the customer can use it (the GA or stable-release window), not the last engineering resolution date. Because entries are often pre-drafted, use `[Month TBD, set at publish]` unless the month is confirmed.
- **Status:** open beta or GA (using your saved status labels, if any). Closed beta doesn't publish (see Step 1).
- **Product:** the product tag the entry belongs under (from the user's product tags).

Never put the month, status, or product in the title or body, and never include the metadata lines in a saved copy (Step 6). If drafting several entries, order them by actual release order, most recent first. One capability per entry.

## Step 3: Doc pointer

Every entry ends with "For more information, see [doc title](article URL)." **Hyperlink the article title** (markdown `[title](url)`) to the published help article, wherever the area's help content currently lives. Don't assume a domain: take article URLs from the source, connected docs, or the user. Never leave it as bare text, and never a raw URL. Pick the article a customer reading the entry would open next. Use the area's own help articles; if you can't confidently name one, propose the closest match and flag it. Never invent a title or URL. If the article doesn't exist yet, use a placeholder title and flag that the article is needed. Don't drop the link.

## Step 4: Draft the entry

```
**[Bold sentence-case title, 4 to 8 words, no ending punctuation]**
[One sentence, about 25 words: the most important detail the title doesn't carry (scope, specifics, behavior, or outcome). This is the card preview.]
[Optional sentence: more behavior or scope. Optional "Previously, you could only…" context.]
[Why it matters, using an approved value pattern (see Value). Can be its own sentence or folded into an earlier one.]
[Only if it's a real gate: one short, positively framed availability or requirement clause.]
For more information, see [doc title](article URL).   ← hyperlink the title, never bare text
```

One paragraph (the structure above is its internal flow, not layout). Rules:

- **Title.** Action-based by default, leading with a verb for a capability (`Pause a recurring task`). Status-style is fine for availability changes and renames (`Calendar Sync now generally available`, `Time logs are now Timesheets`). Bold line on its own line, never a Markdown heading (`#`, `##`, `###`), and nothing between the title and the paragraph: no product-area line, no italic subtitle.
- **First sentence adds to the title, never repeats it.** Follow "Title and first sentence: don't repeat" in the reference. The first sentence must add scope, specifics, behavior, or outcome. Test it: if deleting the first sentence loses no information, rewrite it. There's no fixed opening formula; an imperative verb with scope, a subject-first sentence, or "You can now…" followed by new detail all work. Vary openings across entries. Never "We've added…".
- **Value.** Use one of the approved value patterns in the reference ("This helps you…", "This allows you to…", "…so you can…", "…without [the old cost]", "Previously, you had to…", or "Use it to…"). Don't repeat the same pattern in back-to-back entries. The value must pass three tests: it names a concrete task, question, or cost the reader has ("bill each client for the hours logged against their projects", not "improves your workflow"); it traces to the problem statement in the conviction doc, spec, or brief (name the source in the notes); and it would read naturally in a help article. If it fails any of them, use a placeholder and flag it for SME review instead of writing a benefit that merely sounds right.
- **Requirements only when they're a real gate the tags don't show.** The card's status and product tags carry beta or GA and the product, so don't restate them. Note a paid tier the product tag doesn't make clear ("available on the Business plan"), a setting that gates access, or a minimum app or SDK version. Frame positively. For a beta-to-GA graduation, "now generally available (GA)" can appear, spelled out on first use. Most entries need no requirement line.
- **Verify names, never invent them.** Every product, feature, UI, API, and SDK name and its capitalization must be confirmed against context or recent entries for the same product. If you can't confirm one, use a placeholder and flag it.
- **Mark gaps, don't fill them.** Use `[PLACEHOLDER: what's needed. Suggested SME question: "question text"]` inline for missing value, availability, requirements, names, or the companion article. A visible gap beats an invented benefit.
- **Terminology.** Run the terminology-precision traps in the reference (for example: don't use overloaded nouns like "schedule" loosely; don't fuse a product and its object into a marketing phrase; "turn on/off," not "enabled/disabled"; official terms only), with the values from the user's glossary or saved glossary config. Capitalize product and feature names as verified in Step 0 (connected docs, the user's glossary), not as the existing page does; incidental UI nouns lowercase.
- **No dashes or ampersands.** Never use an em dash (—), en dash (–), or ampersand (&) in the entry. Use commas, colons, parentheses, or periods; write ranges as "to"; write "and" instead of "&". The only exception for "&" is an official product or feature name that contains one, verified in Step 0 and on the check's allow-list. Use the Oxford comma.
- **No dates or time-bound phrasing in the entry.** No launch date, no "today", no "as of this update", no "In this open beta release…", no specific month. ("You can now…" describes capability, not timing, and is fine.) When a feature graduates from beta, the status clause should be the only part that needs updating.
- **Length:** 2 to 4 sentences, 50 to 90 words, not counting placeholder text. Match the surrounding entries; if recent ones run short, run short.
- **No marketing puff** ("powerful", "seamless", "revolutionary", "game-changing", "robust", "effortless"). Flat-affect descriptive. Present tense, second person, active voice.
- **No internal language:** no flag names, ticket IDs, file paths, or non-customer-facing tool names.

Illustrative example of the format (from a fictional work-tracking product; copy the *shape*, not the terminology):
> **Pause a recurring task**
> Skip upcoming copies of a recurring task for the number of weeks you pick, and the schedule picks up again on its own afterward. Copies that already exist stay on the board. Use it to hold a weekly review over a holiday without deleting the series and setting it up again. For more information, see [Manage recurring tasks](ARTICLE_URL).

Anti-patterns: a first sentence that restates the title, announcement/excitement framing ("We're excited to announce…"), internal language leaking in, marketing puff, or a multi-paragraph/bulleted entry (the page is single-paragraph entries only).

Before presenting, run the quick checklist at the end of `references/release-note-standards.md`, then the mechanical character check below.

### Mechanical check (required, every entry)

Don't rely on reading the entry back. Scan the visible entry text literally for:

- **Characters:** `—` (em dash), `–` (en dash), and `&` (ampersand) in the title and prose.
- **Bare URLs:** any visible URL. The doc pointer must be a hyperlinked article title.
- **Banned phrases:** the list in "Banned phrases" in the reference (marketing verbs and adverbs, announcement framing including "Introducing…", "quickly," "easily," "simply," and so on), plus any banned phrases in the user's saved glossary, plus exclamation points and question marks.
- **Dates:** month names (January to December, and abbreviations such as "Sep") in date-like context, meaning followed by a day number or year ("June 3", "March 2026") or preceded by "in", "on", "by", "since", or "until" ("in March"), plus years and numeric date patterns (for example YYYY-MM-DD or M/D). A year-shaped number right after a quantity word ("up to 2000 tasks") and "24/7" aren't dates. A capitalized "May" that starts a sentence isn't a date. Month grouping belongs to the card metadata, never the entry.
- **Ticket ids:** a ticket key (letters, a dash, and digits, for example ABC-123) anywhere in the visible text, including a link title. Standard names with the same shape (UTF-8 and other UTF, SHA, ISO, or RFC names) aren't ticket keys. A key inside a link's URL, `[title](url)`, is fine, since readers don't see it.
- **Length (warning, not a failure):** under 50 or over 90 words, not counting placeholder text.

**Skip placeholder text, then count it.** Remove every `[PLACEHOLDER: …]` span (and any `(url)` right after it) before scanning and before counting words, so the quoted SME question in a placeholder doesn't fail the check or pad the length. Count the spans you removed and report them separately (Step 5) as **"Not ready to publish: N placeholders"**. That report is not a check failure: an entry with placeholders can pass. Check the replacement text by running the check again once each placeholder is filled.

**Allow-listed names.** A verified product or feature name that contains "&" or a banned word (for example "Plan & Track" or "Quick Entry") is skipped by the character, phrase, and date checks, and still counts toward length. The allow-list is `release_note_drafter.glossary.allowed_names` in config plus any name you verified in Step 0 this session. Only a name confirmed against context goes on the list; never add one to make a hit go away.

The doc pointer's article URL sits inside the link markup, `[title](url)`, where readers don't see it and where `&` can legitimately appear in a query string. Ignore that hidden URL as well. If code execution is available, run the check as code, for example:

```python
import re
BANNED = [
    r"\bunlock", r"\bempower", r"\bsupercharg", r"\belevat", r"\bstreamlin",
    r"\bseamless", r"\beffortless", r"\bgame[- ]chang", r"\bnext level",
    r"\bpowerful", r"\brobust", r"\brevolutionar", r"\bcutting[- ]edge",
    r"\bquick(ly)?\b", r"\beas(y|ily)\b", r"\bsimpl(e|y)\b", r"\bjust\b",
    r"\bwe'?re excited", r"\bwe'?ve added", r"\bintroducing\b",
]
EXTRA_BANNED = []   # your own banned phrases from config (release_note_drafter.glossary.banned_phrases)
ALLOWED_NAMES = []  # verified names from config (release_note_drafter.glossary.allowed_names) plus names verified this session
BANNED += [r"\b" + re.escape(t) for t in EXTRA_BANNED]
MONTH = (r"(January|February|March|April|May|June|July|August|September|October|November|December"
         r"|Jan|Feb|Mar|Apr|Jun|Jul|Aug|Sep|Sept|Oct|Nov|Dec)")
# Month names only in date-like context: followed by a day number or year ("June 3", "March 2026"),
# or preceded by in/on/by/since/until ("in March"). Case-sensitive, so "you may" and a
# sentence-initial "May users..." both pass.
MONTHS = (rf"\b{MONTH}\.?,?\s+'?\d"
          rf"|(?i:\b(?:in|on|by|since|until)\s+){MONTH}\b")
# Every year-shaped number is a date unless a quantity word comes right before it, which makes
# it a count ("up to 2000 tasks", "more than 1500 rows").
QTY = r"\b(?:up to|more than|fewer than|less than|at least|at most|nearly|almost|maximum of|limit of)\s+$"
YEARS = r"\b(?:19|20)\d{2}\b"
# Numeric dates; "24/7" (around the clock) is not one.
DATES = YEARS + r"|\b(?!24/7(?!/\d))\d{1,2}/\d{1,2}(/\d{2,4})?\b|\b\d{4}-\d{2}-\d{2}\b"
NOT_TICKETS = {"UTF", "SHA", "ISO", "RFC"}   # standard names shaped like a key, such as UTF-8

PLACEHOLDER = r"\[PLACEHOLDER:[^\]]*\](\([^)]*\))?"
placeholders = len(re.findall(PLACEHOLDER, entry))       # reported separately, not a failure
visible = re.sub(PLACEHOLDER, "", entry)                  # skip placeholder text (checks and word count)
visible = re.sub(r"\]\([^)]*\)", "]", visible)            # drop the URL inside [title](url), keep the title
body = visible.split("\n", 1)[-1]                         # word count excludes the title line
words = len(re.findall(r"\b\w[\w'-]*\b", body))           # allow-listed names still count
scan = visible
for name in sorted(ALLOWED_NAMES, key=len, reverse=True):
    scan = scan.replace(name, "NAME")                     # skip verified names in the checks below
char_hits = [(m.group(), m.start()) for m in re.finditer(r"[—–&!?]", scan)]
bare_urls = re.findall(r"https?://\S+", scan)
ticket_hits = [t for t in re.findall(r"\b[A-Z][A-Z0-9]*-\d+\b", scan)   # keys inside [title](url) were dropped above
               if t.split("-")[0] not in NOT_TICKETS]
phrase_hits = [p for p in BANNED if re.search(p, scan, re.I)]
date_hits = [m.group() for m in re.finditer(MONTHS + "|" + DATES, scan)
             if not (re.fullmatch(YEARS, m.group()) and re.search(QTY, scan[:m.start()], re.I))]
ok = not (char_hits or bare_urls or ticket_hits or phrase_hits or date_hits)   # hard fails
length_warning = not 50 <= words <= 90                           # note it; don't pad to fix it
readiness = f"Not ready to publish: {placeholders} placeholder{'s' * (placeholders != 1)}" if placeholders else None
```

If code execution isn't available, go through the entry for each item above by hand. Fix every character, URL, ticket id, phrase, and date hit and re-run, for at most two fix passes. The only exception is an allow-listed name. A length warning isn't a failure: never pad an entry with filler or unverified detail to reach 50 words. Mention the length in the notes instead. If hits remain after the second pass, don't present the entry as ready: hand it back marked as not passed, with each remaining hit named under Flags.

## Step 5: Present and iterate

Present in this order:

1. **Flags** (only if any): unverified names, unsupported benefits, mechanical-check hits left after two fix passes, or a closed-beta hold when the source explicitly says the feature will launch in closed beta. Numbered, each with the question needed to resolve it. Never flag a feature for being early in its lifecycle (discovery, design, not started, in progress); placeholders cover what's undecided.
2. **Assumptions** (only if any): what you inferred rather than read in Step 0 and Step 1 (for example planned scope with no tickets, an unconfirmed name, or the article you picked), one line each, so the user can correct them now that the draft exists.
3. The **card metadata**: month, status, and product.
4. The **entry**, formatted exactly (bold title on its own line, paragraph beneath), with any `[PLACEHOLDER: …]` markers inline.
5. **"Not ready to publish: N placeholders"**, on its own line under the entry, whenever the entry has one or more placeholders (from the mechanical check). Leave it off when there are none. It is a count, not a lifecycle warning.
6. A short **notes block** (3 to 4 bullets max): what you excluded, doc-target uncertainty, and any voice rule set aside (Step 0a), and any publishing steps that apply (the product and status to set on the page, the companion article if it's new or renamed, translation if you localize, a closed-beta hold).

Iterate on feedback. Common revisions are shorter, a different title verb, a different doc pointer, or folding in scope landing in the same window.

**Companion release note for an article update.** When this runs alongside a help-article update, draft from the same context, reference the article being updated as the "For more information, see…" link, and present the entry clearly labeled as the release note summary, separate from the article changes.

**Called by another skill.** When a feature-creation skill (or similar) calls this one for its Release Note field, return the approved entry text (title and paragraph, with any placeholders) plus the flags, assumptions, and placeholder count, and let the caller store it. Don't save anywhere yourself in that case, unless the user asks you directly.

## Step 6: Save the approved entry (optional, opt-in)

By default the entry lives in chat, and the user copies it where it needs to go. If the user asks to save and no tracker is connected, say so in one line and hand over the text. If the user asks to save it to a record in a connected tracker (for example a feature record in a Notion database, a Jira issue, or an Airtable row):

1. **Find the record.** Ask for its link, or match by feature name and area. One confident match: use it. Zero or several: ask which (one quick confirm) rather than guessing.
2. **Read the record's schema and current value first.** Write to an existing text field for release-note copy (for example `Release Note`). Never create a new property. If there's no matching field, say so and hand over the text.
3. **Save only the entry itself** (title and paragraph, with any placeholders). Never append flags, assumptions, the placeholder count, notes, or card metadata (month, status, product). Leave every other field on the record as it is.
4. **Never overwrite** a non-empty field. Show the existing value and ask before replacing.
5. **Read back, then confirm.** Re-fetch the record and check the field holds the saved entry, then confirm with the record link. If the write fails or the read-back doesn't match, tell the user and hand them the text. A silent failure is not acceptable.

## Related skills (optional)

These are separate skills in https://github.com/pendo-io/Product-Loop. None is required.

- **product-announcement-generator** drafts the internal team announcement for the same launch.
- **feature-creation** calls this skill for a feature record's Release Note field.

If the user asks for one of those jobs and the skill isn't installed, say it exists and offer to install it. In Claude Code, or anywhere you can write files and reach GitHub, offer to do it yourself, and on the user's OK download that skill's folder from the repo into `~/.claude/skills/<skill-name>/`. Elsewhere, tell the user to download the folder and upload it as a skill. Never block on it.

## Principles

- Customer-facing copy, not an engineering recap. Rewrite any sentence that describes mechanism instead of outcome.
- Only what's user-facing, and at publish, only what shipped. Tickets with a done-type resolution (not Won't Do, Duplicate, or similar) define scope for shipped features; pre-drafts use intended scope with placeholders and no lifecycle warnings. Bug fixes and backend-only changes don't get a note.
- The conviction doc or spec is the "why", the tickets are the "what", recent entries for the same product are the length reference, and `references/release-note-standards.md` is the rulebook for wording. Use all four.
- Never invent a name, benefit, availability status, doc title, or URL. Flag it with a placeholder instead.
- Match the length of recent entries, not their wording. One paragraph, no sub-headers or bullets in the entry.
- A brand voice shapes tone and word choice only. The standard and the hard rules win where they conflict.
