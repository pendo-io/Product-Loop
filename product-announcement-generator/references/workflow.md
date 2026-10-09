# Workflow

How a product announcement is drafted, reviewed, and readied for posting. A person posts the final draft to the team's announcement channel when it's time; this skill never posts.

## Overview

1. Gather context and draft the announcement (standard markdown, no Slack codes).
2. Show the draft in chat. Optionally save it to a tracker record the user names (see SKILL Step 3).
3. Get a screenshot or GIF (a visual is expected on every announcement).
4. Iterate on feedback.
5. When ready, a person copies the draft and posts it to the announcement channel, adding the visual and scheduling as needed.

## Save + iterate

- **Save (opt-in).** Only if the user asks: read the target record's schema and current value, write to an existing announcement text field, never overwrite a non-empty value without asking, then re-read and check that the header came through bold. Confirm with the record link and show the draft.
- **Then get the visual.** Offer to capture the screenshot or GIF with a browser tool, if one is available. If the right test or demo environment isn't known, ask the user for its URL or to provide the image.
- **Iterate.** Offer *Regenerate with changes* / *Looks good, I'll post it* / *Question about posting*.

## Pre-handover checklist

Run `references/pre-publish-checklist.md` before handing over:

- [ ] Header on one line and bold, bookend emojis match, no date, "now available", or "now live", no dashes
- [ ] No internal shorthand (for example, a project codename or team acronym); terms match the user's glossary if there is one
- [ ] No ticket ids, keys, or flag names in the announcement (ticket references only inside URLs)
- [ ] Body is 2 to 3 sentences (not 4+)
- [ ] "Who cares most" names the roles that care most, not an internal team
- [ ] Mechanical check (SKILL.md Step 2) passed (or, after two fix passes, remaining hits named at hand-back)
- [ ] Links are complete and correct
- [ ] Channels appear as plain text, such as `#channel-name`, with no Slack `![](#id)` or `<@U…>` codes
- [ ] A visual is sourced or explicitly requested from the user
- [ ] No placeholder text remaining (unless intentional `[TBD]`)
- [ ] Passes the 10-second read test

## Channel reference

| Channel | Purpose |
| --- | --- |
| Your announcement channel (for example `#<your-announcement-channel>`) | Final announcements, posted by a person |
| Your questions channel(s) (for example `#<area>-launch-questions`) | The "Questions:" line |
