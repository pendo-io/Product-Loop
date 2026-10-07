---
name: conviction-doc
description: >
  A conviction doc is a short, structured statement of belief for an initiative — the problem, why now, and what would prove the bet wrong. One doc replaces the separate spec, engineering brief, GTM narrative, board slide, and OKR scorecard that would otherwise retell the same story for different audiences. Use this skill to write, complete, or stress-test one. Trigger on: "fill out the conviction doc", "write a conviction record for X", "is this bet ready to build", "help me think through the tradeoffs for X", "what would change our minds on X", or when a user pastes a partial conviction doc and wants it completed or stress-tested. If Pendo MCP and Pendo Listen are connected, pulls live adoption data and customer feedback to ground the doc in evidence. Outputs the finished doc as markdown in chat — no Notion or other external system required.
compatibility: Standalone — no required connectors. Pendo MCP (adoption data) and Pendo Listen (customer feedback) are used automatically if connected. Jira/Confluence, Slack, and Google Drive are used automatically if connected, to pull supporting research the user references.
metadata:
  author: Pendo Product Operations
  version: "1.1.0"
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

The output is a markdown document posted directly in chat — copy it into whatever tool your team uses (Notion, Confluence, Google Docs, Word, a wiki page, etc.).

---

## PHASE 0 — INTAKE (2 questions, no more)

Ask these questions in a single message. Do not split them across turns.

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
>
> The more you share, the less I'll need to ask. If you have nothing yet, just say so and I'll start from scratch using whatever data sources are available to me — but a conviction doc is only as strong as the discovery behind it, so the more real evidence you paste here, the stronger this will be.

---

## PHASE 1 — RESEARCH (no user input needed)

After intake, run all of the following before drafting. Do not ask the user for anything during this phase — just work.

### 1a. Fetch provided resources
- Fetch any ticket URLs (Jira/Confluence via Atlassian MCP, Linear, etc.) if connected
- Fetch any Google Docs, Notion pages, or other URLs provided
- Read any pasted text directly

### 1b. Pull product adoption data — *if Pendo MCP is connected*
- Search the user's Pendo subscription for pages, features, and guide metrics related to this initiative
- Pull adoption trends, WAU/MAU, funnel data, retention metrics — whatever is available for the relevant product area
- If no Pendo MCP connector is available, or no data exists (new capability), mark the relevant claim as `[ASSUMPTION — VALIDATE: establish baseline in first 30 days]` and move on. Do not block on this.

### 1c. Pull customer feedback — *if Pendo Listen is connected*
- Search Pendo Listen for themes, insight summaries, and verbatim quotes related to this initiative
- Prioritize named, attributed quotes for use in §1
- Note mention counts and sentiment where available
- If Listen isn't connected, use whatever feedback the user pasted, or note the gap as `[ASSUMPTION — VALIDATE]`

### 1d. Search Slack — *if connected*
- Search channels mentioned by the user, plus any obviously relevant channels
- Look for: customer quotes, competitive mentions, prior discussion, related links

### 1e. Brief summary before drafting
After research, post a short summary (5–8 bullets) of what was found and what's still missing. Then proceed to Phase 2 without waiting — unless a critical piece of information is genuinely unavailable and cannot be inferred.

---

## PHASE 2 — DRAFT CONVICTION DOC

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

Use 🟢 READY, 🟡 DRAFT, or 🔴 NOT READY based on stress-test result.

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

**THE PAIN** — Open with one named, attributed customer quote (from Pendo Listen if connected, otherwise from whatever the user provided — a support ticket, interview note, review, etc.). Follow with 2–3 bullets naming corroborating accounts or signals. What frustration, broken workflow, or costly workaround exists today?

**THE BOUNDARY** — 3–5 specific out-of-scope items. Each must be specific enough that an engineer could use it to reject a scope request.

> **Quality bar:** Pain opens with a named, attributed quote. If it sounds like a product feature, rewrite it as a workflow failure. Boundary bullets must be engineer-rejectable.

---

#### Section 2 — Our Conviction
*The hypothesis we are wagering resources on — and the evidence that makes us confident enough to act.*

Format:
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

#### Stress-test table *(auto-generated on every run)*

| Section | Grade | Reasoning |
|---|---|---|
| §1 The Problem We Own | 🟢 / 🟡 / 🔴 | ≤1 sentence |
| §2 Our Conviction | 🟢 / 🟡 / 🔴 | ≤1 sentence |
| §3 The Tradeoff | 🟢 / 🟡 / 🔴 | ≤1 sentence |
| §4 Why Us and Not Them | 🟢 / 🟡 / 🔴 | ≤1 sentence |
| §5 What Success Looks Like | 🟢 / 🟡 / 🔴 | ≤1 sentence |
| §6 What Would Change Our Minds | 🟢 / 🟡 / 🔴 | ≤1 sentence |

**This bet is [READY / NOT READY] to move to build.**

Grade triggers:
- §1 Pain opens without a named, attributed quote → 🟡
- §2 Evidence contains no quantitative adoption/usage data → 🔴
- §2 Evidence contains a customer quote without a named account → 🔴
- §2 WE BELIEVE items lack `[ASSUMPTION — VALIDATE]` tags → 🟡
- §5 numeric targets have no stated baseline → 🟡

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
- Stress-test: one row per section, reasoning ≤1 sentence

Appendix has no length limit.

---

### Inline source linking rules

- Every attributed customer quote: hyperlink the account name or attribution text to its source URL (if available). Never add a separate URL line — the link lives on the text.
- Every adoption/usage figure: hyperlink to its source if accessible.
- Never make the content longer because of links — the link wraps existing text only.

---

### After the doc is done

Once the conviction doc is finished, it's a plain markdown artifact — paste it into whatever tool your team actually uses (Notion, Confluence, a Google Doc, a wiki page).

From there, you can ask directly for any downstream artifact using the finished doc as the source of truth — for example: "turn this into a one-page spec," "draft OKRs from this," "write the GTM narrative," "draft a board slide," or "write the sales enablement one-pager." No special setup needed — just ask, and the finished conviction doc will be used as grounding.

---

## GUARDRAILS

- **Never skip the Phase 1 research.** A conviction doc without real evidence is a belief doc — pull whatever data sources are actually connected before drafting.
- **Every customer quote in the main doc must name an account.** Unattributed quotes go to the Appendix only, clearly marked.
- **Never invent URLs, channels, or links.** Flag gaps instead.
- **The stress-test runs automatically** on every conviction doc draft. It is not optional.
- **Inline links never add length** — they wrap existing attribution text only.
- **Section 4 must reflect this team/company's real advantages** — never borrow a generic or vendor's competitive story. If it isn't inferable, ask.
- **This skill does not write to Notion, Confluence, or any other system.** The output is markdown in chat, every time. If the user wants it posted somewhere, they do that themselves (or ask for a skill that integrates with that specific tool).
- **Default stage is Exploring.** Only record a different stage if the user explicitly says this initiative is further along.
