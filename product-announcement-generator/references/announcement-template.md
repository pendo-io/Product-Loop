# Product Announcement Template

The single, canonical template for internal product announcements. Every announcement uses this one short format.

## Template

```
:[emoji]: **[Product Area] | [Feature name] | [Status]:** **[one-line value hook]** :[emoji]:

[2 to 3 sentences: what changed, why it matters, what it unlocks. Write it like you're telling a smart colleague over coffee.]

**Who cares most:** [One or more roles at a company that care most about this change]

**From here:**
* [External resource, such as a help article or other public link]
* [Internal resource, only if it changes the reader's next step]
* Questions: #<questions-channel>

_Attached: images, gifs, videos if applicable_
```

## Field guidance

### Header line

Keep it on one line. Pattern:

`[emoji] **[Product Area] | [Feature name] | [Status]:** **[value hook]** [emoji]`

The same emoji bookends both ends. The product area, feature name, and status are bold. The value hook is also bold to give the line a strong second beat.

**Status values:** your saved set if you have one (see Setup in `SKILL.md`); otherwise the default `GA`, `Open Beta`, `Closed Beta`, `EA`, `Coming Soon`. The defaults are yours to replace. Use `Coming Soon` for an announcement written before launch; switch it before posting.

**No date.** State the capability, not the timing: "now" plus a capability verb is fine (for example "You can now…" or "now forecasts"); "now available", "now live", "as of", "today", and dates are out. Dates go stale because this copy is usually written before the launch date is final, so the header carries Status only.

**No dashes.** Never use an em dash or en dash anywhere in the announcement, header or body. Use `|` as the header separator and commas, colons, or periods in the body.

### Value hook

One line, concrete, reader-facing. Lead with what's newly possible or what friction has been removed.

Good:
- *"set it once, it repeats every week"*
- *"guests can see it, not only members"*
- *"included on every plan"*
- *"no export step"*

Avoid abstractions like *"improved experience,"* *"enhanced workflow,"* or *"more powerful capabilities."* Those say nothing.

### Body (2 to 3 sentences)

Cover what changed, why it matters, and what it unlocks, in terms of what the customer can now do. Resist a fourth sentence. Tone test: would you say this to a colleague over coffee? If not, rewrite.

Avoid:
- Internal system names that readers outside the product team don't recognize
- Internal shorthand and acronyms (for example, a project codename or a team acronym). Use plain language or the product name from your glossary
- Marketing voice ("excited to announce," "thrilled to bring you")

### Who cares most

One or more roles at a company, the people who care most about this change (for example PM, CPO, CEO, CTO, CSO, sales rep, or support lead). It is not an internal team: **who reads this** is your internal teams, **who cares most** is the roles they should think of when they act on it.

Example: *"Finance leads who reconcile billable hours at month end."*

The test: if the line turns into a long list of roles, cut to the ones who'd be most disappointed to miss the change.

### From here

Three lines is the target. Four is acceptable when the action genuinely splits across audiences (for example, existing customers vs. everyone else).

The first line is the highest-value external resource: a help article, blog post, marketing page, or in-app path. The channel for questions is always the last line.

Skip the internal-resource line unless it genuinely changes what the reader does next.

### Attachments

A visual is expected on every announcement, so source one actively: check the feature record for an existing image first, and if there is none, ask the user for a screenshot or GIF (or offer to capture one) before posting. If you have screenshots, GIFs, or a short video of the feature, attach them to the post and add an italicized line at the bottom:

`_Attached: images, gifs, videos if applicable_`

If there's nothing to attach, leave the line off entirely.

## Special cases

### Supplementary info goes in a thread

The 2 to 3 sentence body is firm. Anything that doesn't fit (caveats, edge cases, FAQ-style detail, deeper rationale) goes in a thread reply on the posted announcement, not in the main post. When you hand back the draft, remind the user they can add more context in the thread once it's posted.

### Regional availability

If the feature is *not* available in all regions, add this line in the body (after the 2 to 3 sentence description):

*"{Feature Name} will be available in {List of Regions}."*

If the feature is available globally, no line is needed.

## Surface formatting

**Markdown (default).** Use standard markdown: `**bold**`, `_italic_`, `* ` for bullets, `[text](url)` for links, and channels as plain text such as `#channel-name`. Never use Slack codes (`![](#id)`, `<@U…>`).

**Notion, Confluence, Google Docs, or another tracker:** the same standard markdown.

When a person later posts the draft to Slack or Teams, the app turns `#channel-name` and `@name` into links by itself, so the draft needs no Slack-specific syntax.
