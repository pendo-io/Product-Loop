# Asset Format Specs

**Four rules that apply to every spec below:**
- Length ranges are targets, not gates. Structure and completeness win over hitting an exact
  word count — don't pad or amputate to satisfy a number.
- Never fabricate quotes, stats, or customer names. If the brief (the feature input, conviction doc, and uploaded context) doesn't supply one,
  insert a `[TBD: quote/stat, owner: PM]` placeholder and flag it to the PM when delivering.
- **Announcement and release-note copy never uses em or en dashes and never carries dates.**
  This covers the email announcement (spec 5), internal launch email (spec 14), Slack
  announcement (spec 15), release notes, and what's new entries. Use commas, colons, or
  separate sentences instead. State the capability, not the timing: "now" plus a
  capability verb is fine (for example "You can now…" or "now forecasts"); "now
  available", "now live", "as of", "today", and dates are out. The one exception is an
  email subject line, which may say "[Feature] is live" (spec 14).
- **Internal until cleared.** Conviction-doc stats and customer names stay out of
  customer-facing assets until the PM confirms they're cleared; use
  `[TBD: cleared stat, owner: PM]` or `[TBD: cleared customer name, owner: PM]` instead
  (see "Boundaries and authority" in SKILL.md).

---

## 1. Blog post

**Target length:** 600 to 1000 words
**Audience:** Customers and prospects, primarily the buyer and user personas in the brief
**Tone:** Thought leadership with a product reveal. Not a press release. Not a feature dump.

**Structure:**
```
Headline (benefit-led or tension-led, sentence case, kept under 70 characters)
Subheadline (optional: a single sentence that builds on the hook)

Intro (75 to 100 words)
  - Open with the problem or a bold claim, not "Today we're excited to announce"
  - No "we're thrilled" or "we're proud to share"

Main body (400 to 700 words)
  - Section 1: The problem / why this matters now
  - Section 2: How the feature works (plain English, no jargon)
  - Section 3: What it means for the customer (outcomes, not capabilities)
  - Break the body with a subheading every 150 to 200 words so it scans
  - Pull in proof points, customer quotes, or data if available
  - Lists work best in threes, and should be used sparingly

Closing (75 to 100 words)
  - Come back to the key outcome
  - A single CTA: try it, read the docs, or talk to your account team
  - No summary restatement of everything you just said
```

**What not to do:**
- Don't open with "In today's fast-paced environment..."
- Don't bullet-list your way through the entire post
- Don't use headers like "Introducing [Feature Name]" as the first line
- Don't end with "We're excited to see what you build"

---

## 2. Slide copy

**Format:** Headline + 3–5 supporting bullets (or short body paragraph)
**Audience:** Whoever is in the room — calibrate to the PM's context

**Headline rules:**
- Under 10 words
- Outcome-led or tension-led, not feature-name-led
- Sentence case
- No punctuation at end unless it's a question

**Bullet rules:**
- 6–10 words per bullet
- Start with a verb or a noun, not "The ability to..."
- Parallel structure across all bullets
- No periods at end

**Output format:**
```
Slide headline:
[Headline here]

Supporting copy:
- [Bullet 1]
- [Bullet 2]
- [Bullet 3]
- [Bullet 4 — optional]
- [Bullet 5 — optional]

Speaker note (optional):
[1–2 sentences the presenter can say to bring this to life]
```

---

## 3. Social copy

### LinkedIn post

**Target length:** 150 to 300 words
**Structure:**
- Hook (first line — must work as a standalone sentence; this is what shows before "see more")
- Body (story, proof, or context in 3 to 5 short paragraphs)
- A single, clear CTA (a link, a comment, or a tag)
- 3–5 hashtags at the end (not mid-copy)

**Tone:** Professional but human. First person is fine if posting as an individual PM.
Write as if a smart, non-corporate person is sharing something they're genuinely excited about.

**What not to do:**
- No "Thrilled to share..."
- No "Big news!"
- Don't start with the feature name
- No wall of text — break it up

### X (Twitter) post

**Limit:** fewer than 280 characters; aim for 220 so links and media still fit
**Format:** One punchy take. Can be a thread opener if the story needs more space.

**Thread format (if needed):**
```
Tweet 1: The hook — bold claim or surprising stat
Tweet 2: The problem it solves
Tweet 3: What's new
Tweet 4: Proof point or customer outcome
Tweet 5: The CTA
```

---

## 4. Customer FAQ

**Count and format:** 5–8 questions customers will actually ask
**Audience:** Existing customers (direct tone, assume product familiarity)
**Tone:** Clear and direct. No marketing spin in the answers. Be honest about limitations.

**Structure per FAQ entry:**
```
Q: [Plain-English question a customer would type into a support chat]
A: [2–4 sentences. Lead with the direct answer. Then explain. Then note any caveats.]
```

**Good question types to cover:**
- What does this feature do?
- Who has access / what plan is it on?
- How do I turn it on / get started?
- What's changing from what I have today?
- Will this affect my existing setup?
- What's the timeline / when is it available?
- Where do I go if I have questions?

---

## 5. Email announcement

**Audience:** Varies by BOM line: prospect, installed base, or community (see variants below)
**Format:** Laid out as an HTML email

```
Subject line: (sentence case, 20 to 70 characters, no closing punctuation)
Preview text: (adds to the subject, 40 to 100 characters, finishes with punctuation)
Headline: (short, punchy, outcome-led)
Body: (three short paragraphs covering problem, solution, outcome; 50 to 100 words in all)
Primary CTA: (action-oriented, 2 to 5 words)
Secondary CTA (optional): (quieter, for example "Read the docs")
```

**Email voice rules:**
- Say "you/your" more often than "we/our"
- Only one primary CTA, so the email never competes with itself
- Don't bury the announcement, state it in paragraph 1
- No exclamation points unless very intentional
- No "We're excited to share" or "We're proud to announce"
- Keep paragraphs to 1 to 3 sentences each

**Audience variants** (write one per audience the BOM calls for):
- **Prospect:** Lead with the problem and the category, not the feature name. Assume zero
  product familiarity. The CTA is about seeing it: demo, tour, landing page.
- **Installed base:** Warmer, assume familiarity. Lead with what just changed for them and
  where to find it. The CTA is about using it: try it, turn it on.
- **Community:** Peer-to-peer, conversational. Frame it as news worth discussing, not a pitch.
  The CTA is the community thread or docs, never a sales motion.

---

## 6. Landing page copy

**Audience:** Prospects and customers landing from search, ads, or linked from product
**Reference:** Use a hero → capabilities → social proof → CTA page structure as the format model
If setup saved your own product page's section order (`release_kit.landing_page`), follow that
order instead, applying the rules below to each matching section.

**Structure:**

```
[EYEBROW]
2–4 words in all caps, category label (e.g., "WORKLOAD PLANNING", "TEAM REPORTING")

[HERO HEADLINE]
6–10 words. Verb-led or tension-led. The bold claim in sentence case.
Accent word(s): 1–3 words that carry the emotional or category weight — call these out.

[HERO SUBHEAD]
2–3 sentences. What the product does, who it's for, and what changes for them.
Plain English, with no jargon and no feature names.

[HERO CTAs]
Primary: Follow the release stage and the CTA in the brief — beta → "Join the beta"; GA → "Get a demo" (or "Start free trial" if you sell self-serve)
Secondary: One contextual option (e.g., "How does it work?", "See it in action")

---

[SECTION: Capabilities] (repeat for each major capability, 2–4 total)

Section eyebrow: 2 to 3 words (e.g., "CAPABILITIES")
Section headline: outcome-led, 8 to 14 words. Can use accent color on 2–3 words.
Section subhead: 2–3 sentences expanding on the headline.

Capability block headline: 3–6 words (e.g., "Spot overloaded teammates", "Rebalance in one drag")
Capability block body: 1–2 sentences. What it does, why it matters.

---

[SOCIAL PROOF — quote block]
Pull quote: 2–4 sentences from a real customer. First person. No corporate-speak.
Attribution: Name, title, company

---

[AUDIENCE TOGGLE — if applicable]
If the feature serves two distinct audiences (e.g., admins vs. end users),
give each tab or toggle view its own capability headline and benefit list.

---

[SECONDARY CTA SECTION]
Headline: framed as tension or a promise, 6 to 10 words.
Subhead: one or two sentences.
CTA button: 2 to 5 words

---

[TRY IT SECTION — if you offer a self-guided demo, sandbox, or free trial]
Eyebrow: 2–3 words (e.g., "TRY IT")
Headline: "Take [Product Name] for a spin." (or similar low-pressure invitation)
Subhead: 1 sentence. Low pressure, just exploration (e.g., "Click through it at your own pace.")

---

[FAQ SECTION]
5–8 questions. Use the Customer FAQ format from asset 4. Same rules apply.

---

[RELATED FEATURES — footer section, if applicable]
Short copy block for each related feature or product the reader may also want (2–3 sentences each).
Anchor: "Explore more from [Company]"
```

**Voice rules for landing pages:**
- Accent color words are a design decision, but mark them in copy output as [ACCENT: word] so the PM knows
- Headlines should work without the subhead — lead with the payoff
- Give each section one CTA and only one. Two buttons side by side are fine only when one is clearly secondary.
- No "best-in-class," "robust," "seamless," or "powerful"
- Numbers and outcomes beat adjectives every time

---

## 7. In-app guide copy

**Audience:** Users already inside the product — they're mid-task, not reading
**Tone:** Conversational, short, action-oriented. Every word earns its place.

### Step 1: Recommend a guide format

Before writing copy, recommend the right guide type based on the feature:

| Guide type | When to use | Length |
|------------|-------------|--------|
| **Tooltip** | Single UI element needs labeling or a one-line explanation | 1–2 sentences |
| **Lightbox / modal** | A new feature, an opt-in, or an onboarding step where the user has to decide | Headline, 2 to 3 sentences, 1 or 2 CTAs |
| **Walkthrough (multi-step)** | Complex workflows the user needs walking through step by step | 3 to 7 steps, fewer than 30 words per step |
| **Banner** | Light announcements such as a new release, limited-time access, or a reminder | One sentence and a CTA |
| **Empty state prompt** | A user's first visit to a blank screen | Headline, one sentence, and a CTA |

Give your recommendation and its rationale in 1–2 sentences before writing copy.

### Step 2: Write the copy

**Tooltip:**
```
[One sentence on what this element does or why it matters, below 20 words.]
Optional: [Learn more] link
```

**Lightbox / modal:**
```
Headline: [Under 8 words. Outcome or change-framing. Not "Introducing X."]
Body: [2 to 3 sentences going from the problem to what's new to the next step.]
Primary CTA: [Starts with an action verb, 2 to 4 words.]
Secondary CTA (optional): [Lets the user dismiss or defer, 2 to 4 words.]
```

**Walkthrough step:**
```
Step N of N
[Step title: 3–5 words]
[Body: 1–2 sentences. Tell the user exactly what to do. Present tense imperative.]
[Optional: "Next" button label — default is "Next" unless context calls for something specific]
```

**Banner:**
```
[One sentence pairing the news with its benefit, below 15 words.]
[CTA: 2 to 4 words]
```

**Empty state prompt:**
```
Headline: [Under 8 words. Name the possibility, not the void.]
Body: [One sentence on what setting this up will let them do.]
CTA: [2 to 4 words]
```

**Guide copy rules (all types):**
- Never start with "Welcome to" or "Introducing"
- No passive voice — tell users what to do
- Second person ("you/your") always
- If there's a dismiss option, label it "Maybe later" not "Close" or "X" unless space is truly constrained
- Walkthroughs: each step should move the user forward, not just explain — every step should have an action
- Don't write copy that assumes the user has read prior steps — each step should make sense in isolation

---

## 8. Positioning doc + message house

**Length:** 1 page max — this is the source of truth downstream assets pull from, no filler
**Audience:** Internal — PMM, PM, and anyone writing launch copy
**Tone:** Precise. Every word here gets repeated a hundred times downstream.

**Structure:**
```
Positioning statement:
For [target persona] who [need/pain], [Feature] is a [category] that [key benefit].
Unlike [alternative or status quo], it [primary differentiator].

Message house:
Roof: a single umbrella message, fewer than 15 words
Pillars (3), and for each one:
  - A pillar statement, fewer than 12 words
  - 2–3 proof points (capabilities, data, or customer evidence)
```

**Rules:**
- Open from where the feature stands against the alternatives in the brief: if it creates
  a category, lead with the category; if competitors already offer something similar, lead
  with what yours does differently; if it improves an existing approach, lead with the gap
  it closes. If the brief doesn't say which, ask the PM
- No claim a competitor could paste onto their own homepage
- The "Unlike" slot may name competitors — this is an internal doc. Copy written FROM it
  for external assets describes the alternative without naming names
- Use the same proof points here that spec 11 (Customer proof points) later formalizes.
  Positioning comes first and spec 11 inherits from it, never the other way round

---

## 9. Feature demo script

**Target length:** 2 to 3 minutes when spoken, about 300 to 450 words
**Audience:** Prospects and customers watching a recorded or live feature demo
**Tone:** Show, don't narrate the UI. Every beat pairs an on-screen action with a payoff.

**Structure:**
```
Cold open (1–2 sentences): the problem, in the viewer's words
Demo beats (3–5): [What you do on screen] → [what the viewer should notice] → [why it matters]
Close (1 or 2 sentences): come back to the outcome and end on one CTA the viewer can act on
```

**Rules:**
- Never "as you can see," "simply," or "just click"
- Name the user's job, not the menu items

---

## 10. Overview video script

**Target length:** 60 to 90 seconds of voiceover, about 150 to 220 words
**Audience:** Top-of-funnel — prospects who don't know the feature exists yet
**Tone:** Bold claim up front, proof in the middle, one ask at the end.

**Structure:**
```
Hook (0 to 10s): the problem or a bold claim, strong enough to hold the viewer for the next 60 seconds
What it is (10 to 30s): the feature in plain English plus a category-context sentence
What changes (30 to 70s): 2 or 3 outcomes, each one linked to something the viewer sees on screen
CTA (last 10s): a single action
```

Mark visual direction inline as [VISUAL: ...] so the PM can hand it to a producer.
Only spoken words count toward the voiceover word range; [VISUAL] lines are excluded.
For the CTA, use the landing page rule from spec 6: follow the release stage and the CTA in the brief.

---

## 11. Customer proof points

**Count and format:** 3 to 5 proof points, each made of a claim, its evidence, and a source
**Audience:** Internal — feeds the blog, landing page, deck, and social copy
**Tone:** Specific and checkable. A proof point without evidence is just an adjective.

**Structure per proof point:**
```
Claim: [One sentence, outcome-led]
Evidence: [The stat, quote, or behavior that backs it — from the brief or uploaded context]
Source: [Customer name/title, study, or internal data — or [TBD: evidence, owner: PM]]
```

Never invent evidence — the global fabrication rule applies hardest here. Mark each proof
point "cleared" or "internal only"; only cleared ones go into customer-facing assets.

---

## 12. Community post

**Target length:** 150 to 250 words
**Audience:** Your community members — existing users, practitioner-to-practitioner
**Tone:** Peer sharing news, not marketing making an announcement.

**Structure:**
```
Title: (plain, useful — what shipped, not a slogan)
What's new: 1–2 sentences
Why it matters: 2–3 sentences grounded in a workflow they already run
How to try it: where it lives, what plan, any setup
Discussion prompt: a real question that invites replies
```

Skip hashtags and CTA buttons. Link to the docs and pose a question in their place.

---

## 13. Webinar abstract + promotion copy

**Who it's for:** Prospects and customers deciding whether an hour is worth it

**Abstract:**
```
Title: (outcome- or tension-led, under 10 words, never "An Introduction to X")
Abstract: 75 to 125 words covering the problem, what attendees will learn (3 concrete takeaways),
  who should attend
Speakers: name, title
```

**Promotion copy:**
```
Invite email blurb: 40 to 60 words and a single CTA ("Save your seat")
Social blurb: one or two sentences, works standalone, no hashtag pileup
```

Lead with what attendees take away, not with the agenda.

---

## 14. Internal launch email

**Audience:** Internal: sales, post-sale (customer success or solutions), and support teams
**Tone:** Peer-to-peer and fast. This is announcement, not training. Talk tracks and
objection handling belong to a sales-enablement skill or your enablement team.

**Structure:**
```
Subject: [Feature] is live: what to know (or similar; plain, scannable, no dashes, no dates, no "now available")
What shipped: 1 or 2 sentences
Why customers care: 2 or 3 bullets, outcome-led
What to do now: 1 or 2 actions (for example, raise it in quarterly business reviews or send customers the blog)
Where to learn more: links to the blog, the docs, and any enablement material
```

Keep it below 200 words. Customer-facing teams only skim it.

---

## 15. Slack announcement

**Limit:** below 150 words
**Audience:** Internal: your launch channel or the channel your customer-facing teams read
**Tone:** Energy without exclamation-point inflation. One emoji in the headline is plenty.

**Structure:**
```
Headline line: emoji + what shipped, one line
What it does: 1 or 2 sentences
Why it matters: 1 sentence
Links: blog / docs / demo
Who to ping: the PM or launch owner
```

No thread-length essays. If it needs more, it's a doc, and this links to it.
Links that don't exist yet get `[link TBD, owner]` placeholders. Beta stats the brief
hasn't cleared for external use are fine here (this is internal), but say so inline
("internal only until GA") so nobody pastes them into customer-facing copy.
