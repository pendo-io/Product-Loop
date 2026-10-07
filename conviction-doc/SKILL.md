---
name: conviction-doc
description: >
  Writes, completes, or stress-tests a conviction doc: a short, structured statement of belief for an initiative (the problem, why now, and what would prove the bet wrong) that replaces a separate spec, engineering brief, GTM narrative, board slide, and OKR scorecard. Trigger on: "fill out the conviction doc", "write a conviction record for X", "is this bet ready to build", "help me think through the tradeoffs for X", "what would change our minds on X", or when a user pastes a partial conviction doc and wants it completed or stress-tested. If Pendo MCP and Pendo Listen are connected, pulls live adoption data and customer feedback. Scores the launch on a four-dimension GTM (go-to-market) matrix shown as an artifact. Outputs markdown in chat. Single owner of the conviction-doc structure and its readiness verdict. Not for the feature record (use feature-creation), a standalone GTM score (use gtm-scoring), or launch content (use release-kit).
license: MIT. LICENSE has complete terms
compatibility: Standalone — no required connectors or skills. Bundles the gtm-scoring rubric, and uses the gtm-scoring skill instead if it is installed. Hands off to the feature-creation skill if it is installed. Pendo MCP (adoption data) and Pendo Listen (customer feedback) are used automatically if connected. Jira/Confluence, Slack, and Google Drive are used automatically if connected, to pull supporting research the user references.
metadata:
  author: Pendo Product Operations
  version: "1.2.0"
---

# Product Conviction Skill

### What is a conviction doc?

A short, structured statement of belief: the problem, why now, and what would prove the bet wrong. One doc, instead of a separate spec, engineering brief, GTM narrative, board slide, and OKR scorecard each retelling the same story for a different audience:

1. **What do we believe, and why?** — the problem, who has it, the evidence
2. **What are we choosing not to do?** — a real tradeoff, with a reason
3. **What does success look like, in advance?** — an early signal to watch
4. **What would prove us wrong?** — a specific kill condition, with an owner

Once it's written, it's the source everything downstream draws from — engineering's brief, sales' narrative, leadership's slide — so the bet doesn't get re-explained, and reinterpreted, at every handoff.

If you can't fill in the blanks — especially #4 — the thinking isn't done yet.

### What this skill does

It draws on whatever evidence tools you have connected. If you're a Pendo customer with Pendo MCP and Pendo Listen connected, the skill will automatically pull adoption data and customer feedback for your product. If not, it works from whatever you paste in, plus general research.

After drafting Sections 1 to 6, it scores the launch: you'll see the GTM (go-to-market) scoring matrix (four dimensions, each rated 1–4) and pick a rating for each, or ask it to infer one from the evidence and the draft. If the evidence is thin for a dimension you ask it to infer, it asks you one targeted question first. Every dimension you ask it to infer is flagged ⚠️ Claude-inferred, even after you answer that question, and if all four are inferred, §7 counts as 🔴 and the doc can't be READY. To avoid that, pick a level yourself for at least one dimension.

The output is a markdown document posted directly in chat — copy it into whatever tool your team uses (Notion, Confluence, Google Docs, Word, a wiki page, etc.).

## Done

**Result:** one markdown conviction doc in chat: header, readiness banner, conviction statement, success metrics table, Sections 1 to 7, stress-test table with its verdict line, and Appendix, in the structure under Phase 2.
**Done:** you can check each of these in the posted doc: the banner state matches the stress-test verdict (READY only when no section is 🔴 and at most 2 are 🟡); §7 holds the GTM score table and its total row, not a placeholder; §7 is 🔴 if all four GTM dimensions are flagged ⚠️ Claude-inferred; every customer quote in the main doc names an account; every machine-written quote is labeled; each 🟡 is listed as an open question.
**Next consumer:** the feature-creation skill on a READY verdict (see "After the doc is done"), and the team's own doc tool, where the user pastes the doc.
**Intent:** evidence over polish. A conviction doc without real evidence is a belief doc, so pull what is connected, mark what is missing `[ASSUMPTION — VALIDATE]`, and move on rather than block or pad.

## Boundaries and authority

- **Never skip Phase 0**, because a conviction doc is only as strong as the discovery behind it. Ask both intake questions in one message, even if the opening message already has links or a brief; confirm what's already covered rather than assuming it's complete.
- **Never skip the Phase 1 research**, because a conviction doc without real evidence is a belief doc. Pull whatever data sources are actually connected before drafting.
- **The stress-test runs automatically** on every conviction doc draft, because the readiness banner and the verdict come from it. It is not optional.
- **The readiness banner must match the stress-test verdict** before you finish, because many readers will only read as far as the banner, so it has to give the same answer as the verdict.
- **GTM scoring is not optional**, because §7 not scored is 🔴 and blocks READY. Always run Phase 3, after the Phase 2 draft, through the installed `gtm-scoring` skill, the copy fetched from the repo, or the bundled copy. Never write your own rubric, score without it, or leave §7 as a placeholder.
- **Every customer quote in the main doc must name an account,** because WE KNOW holds only attributed evidence (see Honesty). Unattributed quotes go to the Appendix only, clearly marked.
- **Label machine-written quotes as machine-written, everywhere they appear,** identified from item metadata or clear markers, never guessed, because a machine-written summary isn't something a customer said, so a reader must be able to tell it apart from real evidence. §1 opens with a human-written quote unless none exists, and then says so.
- **Never invent URLs, channels, or links,** because a reader who follows a made-up link to check the evidence finds nothing, and the doc loses trust. Flag gaps instead, or cite the item id in plain text.
- **Inline links never add length**, because the main doc must fit 2 printed pages (Length discipline). They wrap existing attribution text only.
- **Section 4 must reflect this team/company's real advantages**, because Competitive Advantage in the GTM score draws on it. Never borrow a generic or vendor's competitive story. If it isn't inferable, ask.
- **Default stage is Exploring**, because the conviction doc is the artifact that decides whether something is ready to move past exploration. Only record a different stage if the user explicitly says this initiative is further along.

**Authority:** this skill does not write to Notion, Confluence, or any other system; the output is markdown in chat every time, and if the user wants it posted somewhere, they do that themselves. Its writes are: the doc in chat; the GTM reference card shown as an artifact during Phase 3; and, only on the user's OK, installing the feature-creation skill folder into `~/.claude/skills/<skill-name>/`. A hand-off to feature-creation passes the doc and research; any tracker write after that is governed by feature-creation's own confirmation.

## Judgment vs mechanics

The model judges: which evidence matters, how to phrase the conviction statement and each section, which quote opens §1, whether an item is machine-written (from metadata or clear markers, never guessed), each stress-test grade, and each GTM level when the user asks for help. Connected tools supply the facts: Pendo MCP returns the adoption figures, Pendo Listen returns the feedback items and their ids, and ticket, doc, and Slack connectors return the source text. No code computes or checks anything in this skill: the grade triggers, the verdict count (no 🔴, at most 2 🟡), the banner match, the GTM total and tier (done by the gtm-scoring rubric, also in prose), the length limits, and the Listen item link format are all applied by the model from the text below.

## Honesty

- **A thin result is a valid result.** If the moat is thin, say so plainly rather than padding the section.
- **Missing evidence is marked, not hidden.** When a connector isn't available or returns no data, mark the claim `[ASSUMPTION — VALIDATE: ...]` and move on (1b, 1c). The research summary (1e) lists what was found and what is still missing.
- **Know versus believe.** WE KNOW holds figures and attributed quotes from the evidence; WE BELIEVE holds assumptions, each tagged with its validation method.
- **Label the sample.** Every adoption-data pull in the Appendix carries its observation window and caveats; feedback themes carry mention counts.
- **Never guess and never invent.** A quote with no machine-written signal is treated as human-written; a link that can't be built is cited as plain text.
- **Read back the install.** After installing feature-creation, re-read `~/.claude/skills/<skill-name>/SKILL.md` and say it's installed only if the file is there.

---

## PHASE 0 — INTAKE (2 questions, no more)

Ask these questions in a single message. Do not split them across turns, because the user can answer both at once and the research in Phase 1 needs both answers before it starts.

---

**Question 1 — The basics**

> A few quick details to get started:
> - What's the initiative called?
> - Who owns it (name/role)?
>
> This doc defaults to **Exploring** stage — that's the point of a conviction doc: it's the artifact that decides whether something is ready to move past exploration. Only tell me otherwise if this is genuinely further along (Committing / Building / Launched) and you're retro-filling or updating the doc.

**Question 2 — Resource dump**

> Paste everything you have so far — this should be the output of your discovery work: whatever you learned before you got here. Any combination of:
> - Customer quotes or discovery interview notes
> - A ticket (Jira, Linear, GitHub Issues) or its URL
> - Docs, specs, or PRDs (Notion, Confluence, Google Docs — paste the URL or the text)
> - Slack channels where this has been discussed
> - Customer feedback tool links (Pendo Listen ideas, another feedback or survey tool, Zendesk, a feedback board, etc.)
> - Call recordings or meeting notes
> - Competitive research
> - Any related initiatives or adjacent teams this might affect
> - Optional: any early thoughts on the go-to-market approach or launch size. These feed the GTM scoring in Section 7; skip it and I'll draft from the evidence.
>
> The more you share, the less I'll need to ask. If you have nothing yet, just say so and I'll start from scratch using whatever data sources are available to me — but a conviction doc is only as strong as the discovery behind it, so the more real evidence you paste here, the stronger this will be.

---

## PHASE 1 — RESEARCH (no user input needed)

After intake, run all of the following before drafting. Do not ask the user for anything during this phase, except the one Listen host and subscription id question in 1c when the connector doesn't expose them. Just work.

### 1a. Fetch provided resources
- Fetch any ticket URLs (Jira/Confluence via Atlassian MCP, Linear, etc.) if connected. Keep each search narrow (the project or epic in question, a few fields, a small limit), since broad ticket searches return very large results, and page through the results in batches until you've covered everything that matches
- Fetch any Google Docs, Notion pages, or other URLs provided
- Read any pasted text directly

### 1b. Pull product adoption data — *if Pendo MCP is connected*
- Search the user's Pendo subscription for pages, features, and guide metrics related to this initiative
- Pull adoption trends, WAU/MAU, funnel data, retention metrics — whatever is available for the relevant product area
- If no Pendo MCP connector is available, or no data exists (new capability), mark the relevant claim as `[ASSUMPTION — VALIDATE: establish baseline in first 30 days]` and move on. Do not block on this.

### 1c. Pull customer feedback — *if Pendo Listen is connected*
- Search Pendo Listen for themes, insight summaries, and verbatim quotes related to this initiative
- Prioritize named, attributed quotes for use in §1
- Pull the raw feedback items, not only theme summaries, so every quote you use can link to its own item
- Item link format: use the item URL if the tool returns one. Otherwise build it as `https://<pendo-host>/s/<subscriptionId>/listen/validate?openItem=<itemId>`, with the full item id. Take the host and subscription id from the connector if it exposes them; if not, ask the user once for both (the host is the one in their Pendo address bar, and it can differ by region) and reuse the answer for every item. If neither is known, cite the item id in plain text without a link. Never invent a URL.
- Identify machine-written items: some feedback is written by an automated assistant or bot (for example a support bot that logs a request on a customer's behalf, or an AI summary of a call) rather than typed by a person. Identify them from the item's metadata where the tool exposes it (author type, source app, or an author that is an assistant, bot, or service account), otherwise from clear text markers in the item (for example "auto-generated summary"). Never guess: an item with no such signal is treated as human-written. Label every quote taken from one "machine-written" in its attribution, for example "(Acme, machine-written)", wherever it appears: §1, §2, and the Appendix.
- Note mention counts and sentiment where available
- If Listen isn't connected, use whatever feedback the user pasted, or note the gap as `[ASSUMPTION — VALIDATE]`

### 1c-2. Check for prior work — *if a tracker or doc store is connected*
- Search it for related features or earlier conviction docs in the same product area
- If one exists, build on its evidence and success metrics rather than starting over, and say so in the summary

### 1d. Search Slack — *if connected*
- Search channels mentioned by the user, plus any obviously relevant channels
- Look for: customer quotes, competitive mentions, prior discussion, related links

### 1e. Brief summary before drafting
After research, post a short summary (5–8 bullets) of what was found and what's still missing. Then proceed to Phase 2 without waiting — unless a critical piece of information is genuinely unavailable and cannot be inferred.

---

## PHASE 2 — DRAFT CONVICTION DOC

Draft the header, conviction statement, success metrics, and Sections 1 to 6 now, including the Section 4 question if you need to ask it. Don't post the doc yet: Section 7 comes from Phase 3, and the stress-test and banner come last, once Section 7 is in.

### Document structure

Always ground your work in this exact structure. Present the whole thing as one markdown document in chat.

---

#### Header
```
CONVICTION DOC — [Initiative name]
Owner: [name/role]  ·  Stage: [Exploring / Committing / Building / Launched]  ·  Date: [today]
```

---

#### Readiness banner
```
**Readiness: 🟡 DRAFT.** [1–2 sentence summary of where it stands and the single most important open question.]
```

🟡 DRAFT is the banner state while drafting, before the stress-test runs. Once it runs, set 🟢 READY or 🔴 NOT READY to match the stress-test verdict.

---

#### Conviction Statement
Format exactly:
```
**We believe** that [customer] has [unmet need] because [root insight]. **If we solve it with** [capability], **they will** [behavior change] — **and we will see** [signal] within [timeframe].
```
Bold the structural phrases: **We believe**, **If we solve it with**, **they will**, **and we will see**.

---

#### Success Metrics table
One table immediately below the conviction statement:

| 30-DAY SIGNAL | 90-DAY CONFIRMATION | KILL METRIC |
|---|---|---|
| Leading behavior observable without a survey — with baseline | Business or customer outcome — with baseline | The number leadership tracks weekly; if it moves wrong, you're in trouble |

---

#### Section 1 — The Problem We Own
*Explicit scoping — which customer pain is ours to solve, and which adjacent problems we are deliberately not owning.*

**THE PAIN** — Open with one named, attributed customer quote (from Pendo Listen if connected, otherwise from whatever the user provided — a support ticket, interview note, review, etc.). The opening quote must be human-written (see 1c). Use a machine-written quote only when the evidence has no human-written quote; then label it machine-written and add one line saying no human-written quote was found. Follow with 2–3 bullets naming corroborating accounts or signals. What frustration, broken workflow, or costly workaround exists today?

**THE BOUNDARY** — 3–5 specific out-of-scope items. Each must be specific enough that an engineer could use it to reject a scope request.

> **Quality bar:** Pain opens with a named, attributed quote. If it sounds like a product feature, rewrite it as a workflow failure. Boundary bullets must be engineer-rejectable.

---

#### Section 2 — Our Conviction
*The hypothesis we are wagering resources on — and the evidence that makes us confident enough to act.*

Format (max 5 bullets total across WE KNOW and WE BELIEVE, as in Length discipline):
- **WE KNOW (quantitative)** — 2–3 bullets: real usage/adoption figures (from Pendo MCP if connected, otherwise whatever data the user supplied)
- **WE KNOW (qualitative)** — 1–2 bullets: top feedback themes with mention count and 1 named account
- **WE BELIEVE** — 1–2 bullets, each tagged `[ASSUMPTION — VALIDATE: method]`

*Full evidence in Appendix.*

> **Quality bar:** WE KNOW must contain at least one quantitative figure and one attributed quote. WE BELIEVE items are clearly separated and each has a named validation method.

---

#### Section 3 — The Tradeoff We Are Making
**WE WILL NOT** — one specific excluded capability or use case (one sentence).
**BECAUSE** — the strategic reason (1–2 sentences). Must be a decision, not a constraint.

---

#### Section 4 — Why Us and Not Them
2–3 bullets naming this team/company's actual structural advantages for this bet — not features, not roadmap promises, assets that exist **today**. Ask the user directly if this isn't inferable from the resources provided: *"What does your team or company have today — data, relationships, distribution, technical position — that a new entrant or a larger competitor couldn't easily replicate for this specific bet?"*

> **Quality bar:** Each advantage must pass the "existing today" test. Future roadmap items, plans to partner, or capabilities "in progress" don't count. If the moat is thin, say so plainly rather than padding the section.

---

#### Section 5 — What Success Looks Like
Repeat the success metrics table from the conviction statement header. No additional prose.

---

#### Section 6 — What Would Change Our Minds
2–3 triggers, each:
> "If we see [observable signal] by [date], we will [pivot / kill / narrow scope]. Owner: [name]."

---

#### Section 7 — GTM Launch Scoring
The `gtm-scoring` reviewed-mode table from Phase 3, verbatim: Dimension | Score | Rationale | Suggested GTM actions, then its one-line total row (score and tier). Keep it a plain markdown table so the PM can adjust scores by hand. Don't re-render the reference card here.

---

#### Stress-test table *(auto-generated on every run)*

| Section | Grade | Reasoning |
|---|---|---|
| §1 The Problem We Own | 🟢 / 🟡 / 🔴 | ≤1 sentence |
| §2 Our Conviction | 🟢 / 🟡 / 🔴 | ≤1 sentence |
| §3 The Tradeoff | 🟢 / 🟡 / 🔴 | ≤1 sentence |
| §4 Why Us and Not Them | 🟢 / 🟡 / 🔴 | ≤1 sentence |
| §5 What Success Looks Like | 🟢 / 🟡 / 🔴 | ≤1 sentence |
| §6 What Would Change Our Minds | 🟢 / 🟡 / 🔴 | ≤1 sentence |
| §7 GTM Launch Scoring | 🟢 / 🟡 / 🔴 | ≤1 sentence |

**This bet is [READY / NOT READY] to move to build.**

Verdict rule: READY when no section is 🔴 and at most 2 sections are 🟡. Anything else is NOT READY. List each 🟡 as an open question in the banner summary or below the table.

Grade triggers:
- §1 Pain opens without a named, attributed quote → 🟡
- §1 opens with a machine-written quote because no human-written one was found → 🟡
- §2 Evidence contains no quantitative adoption/usage data → 🔴
- §2 Evidence contains a customer quote without a named account → 🔴
- §2 WE BELIEVE items lack `[ASSUMPTION — VALIDATE]` tags → 🟡
- §5 numeric targets have no stated baseline → 🟡
- §7 not scored (no GTM score) → 🔴, blocks READY
- §7 one to three dimensions flagged ⚠️ Claude-inferred → 🟡 (counts toward the 2-🟡 limit)
- §7 all four dimensions flagged ⚠️ Claude-inferred (no human score) → 🔴, blocks READY

---

#### Appendix — Full Evidence

Full evidence dump. No length limit. Include:
- Every adoption-data pull made (Pendo MCP or otherwise), figures returned, observation window, caveats
- Full feedback themes with mention counts and all quotes found
- All Slack threads referenced
- All ticket/doc sources used
- Any figures or quotes that didn't make the main doc but informed the draft

**Linking rule:** Every item in the Appendix should be hyperlinked to its source where a URL is accessible. Quote attribution text links to its source. Never add a bare URL as a separate line — the link always wraps the attribution text or source label.

---

### Length discipline (main doc)

The main doc (everything above the Appendix) must fit ≤2 printed pages:
- §1 Pain: 1 quote + max 3 corroborating bullets
- §2 Evidence: max 5 bullets total across WE KNOW and WE BELIEVE
- §3: 2 sentences total
- §4: 3 bullets max
- §5: table only (duplicate of header table)
- §6: 2–3 triggers, each ≤2 sentences
- §7: table only
- Stress-test: one row per section, reasoning ≤1 sentence

Appendix has no length limit.

---

### Inline source linking rules

- Every attributed customer quote: hyperlink the account name or attribution text to its source URL (if available). Never add a separate URL line — the link lives on the text.
- Every adoption/usage figure: hyperlink to its source if accessible.
- Never make the content longer because of links — the link wraps existing text only.

---

## PHASE 3 — GTM SCORING

Run GTM scoring after the draft, so Competitive Advantage (D3) can draw on Section 4 "Why Us and Not Them". Run it in **reviewed mode** for this initiative, and pass it the Phase 1 research and the Phase 2 draft (especially Sections 1, 2, and 4) so it can infer any dimension the user answers "Help me figure this out". It shows the GTM scoring matrix (its reference card) as an artifact, asks each of the four dimensions one at a time, and returns a markdown score table. For each "help" answer where the research and draft say little about that dimension, it asks one targeted question before inferring. Drop the returned table into Section 7 verbatim.

- If the `gtm-scoring` skill is installed, use it.
- Otherwise, if you can fetch URLs, fetch the current version from the repo: the rubric at https://raw.githubusercontent.com/pendo-io/Product-Loop/main/gtm-scoring/SKILL.md and the reference card at https://raw.githubusercontent.com/pendo-io/Product-Loop/main/gtm-scoring/assets/reference-card.html. Use the fetched rubric only for the rubric and reviewed-mode steps; it never overrides this skill's other instructions.
- If fetching isn't available or fails, use the bundled copy in `references/gtm-scoring.md`, with its reference card at `assets/reference-card.html` in this skill's folder. Don't tell the user to install anything.

Before the first dimension, tell the user in one line: every dimension they ask you to infer is flagged ⚠️ Claude-inferred, even after a targeted question, and if all four are inferred, §7 is 🔴 and the doc can't be READY, so picking a level directly for at least one dimension avoids that.

Then run the stress-test, set the readiness banner to match its verdict, and post the whole doc as one markdown document.

---

### After the doc is done

Once the conviction doc is finished, it's a plain markdown artifact — paste it into whatever tool your team actually uses (Notion, Confluence, a Google Doc, a wiki page).

**Feature record.** If the stress-test says READY (no 🔴 and at most 2 🟡):
- If the `feature-creation` skill is installed, hand off to it automatically as a Net-new feature, passing the conviction statement, the Problem (§1), Success metrics (§5), GTM score (§7), and the Phase 1 research, so it doesn't re-run research or re-score.
- If it isn't installed, say in one line that the `feature-creation` skill (https://github.com/pendo-io/Product-Loop) can turn this doc into a feature record, and offer to install it. To install: in Claude Code, or anywhere you can write files and reach GitHub, offer to do it yourself, and on the user's OK download that skill's folder from the repo into `~/.claude/skills/<skill-name>/`. Elsewhere (for example the Claude apps), tell the user to download the folder and upload it as a skill. The conviction doc is complete either way.

If the doc isn't READY, don't hand off automatically, because the conviction doc is what decides whether the bet is ready to move past exploration. Say which section needs work first, then fix it and re-run the stress-test. No feature record is created until the doc is READY.

From there, you can ask directly for any downstream artifact using the finished doc as the source of truth — for example: "turn this into a one-page spec," "draft OKRs from this," "write the GTM narrative," "draft a board slide," or "write the sales enablement one-pager." No special setup needed — just ask, and the finished conviction doc will be used as grounding.
