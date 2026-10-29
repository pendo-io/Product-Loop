# Brand voice lookup

Optional. Finds a brand voice or writing style the user has already specified, so announcements match it without asking the user to paste it every time. Nothing here is required: with no voice, the skill uses its built-in style (direct, concrete, like telling a smart colleague over coffee, no marketing voice).

## What a voice can and can't change

**The voice governs tone and word choice only.** It never overrides this skill's hard rules, which win every time they conflict:

- The three-question format in `references/announcement-template.md`: the one-line header pattern with bookend emoji and a status value, the one-line value hook, the 2 to 3 sentence body, the "Who cares most" roles, the "From here" rules, the regional line, and the 10-second test.
- No em dashes or en dashes anywhere in the announcement, header or body.
- No dates: no launch date, "as of", "today", "now available", or "now live" ("now" plus a capability verb is fine).
- Ticket ids, keys, and flag names only inside URLs, never in the header, body, or link text.
- The character and date checks in `references/pre-publish-checklist.md`, run before every hand-over.

If a voice rule conflicts with one of these (for example "use em dashes freely", "lead with the launch date", or "write long, story-driven posts"), follow the skill's rule, apply the rest of the voice, and tell the user once, when you hand back the draft, which voice rule was set aside and why.

## Lookup order

Run this once per session, before drafting the first announcement.

1. **This skill's saved config.** If `product_announcement.voice` exists, use it without asking again. If it points to a `source_skill`, re-read that skill: where its current rules differ from the saved summary, let the skill win and tell the user.
2. **Installed skills that define a voice.** Look through the skills available in this session for one whose name or description mentions brand voice, voice, tone, style guide, or writing style. Read it and take its voice or style rules as the source.
3. **Memory and standing instructions.** Check Claude's memory and standing instructions for this user (memory files, CLAUDE.md or project instructions, saved user preferences) for voice, tone, or style rules.

If step 1 finds a saved voice, use it and skip steps 2 and 3. Otherwise check both.

## What to do with what you find

- **One found:** summarize it in two to four lines (how it sounds, traits it is and isn't, terms to avoid), name the source (the skill's name, or "your memory" or "your project instructions"), note any rule the hard rules above will set aside, and ask once: "Use this voice for announcements?" with *Yes*, *Edit it*, and *No, use the default style*.
- **Several found:** show each summary with its source and let the user pick one or merge them. Confirm the result once.
- **None found:** use the built-in style. Offer, in one line, to set a voice ("Want to set a brand voice for announcements? Paste a style guide or point me to one."). Never require it, and don't ask again in the same session.
- **User declines:** use the built-in style and don't save a voice.

## Saving

Save only after the user confirms. Write the `product_announcement` section of a shared config file, by default `config.json` in the user's working folder. If the `daily-digest` conductor's shared config (`.daily-digest/config.json` in the workspace) exists, use this skill's section of that file instead. Keep other sections, and the other keys in this section (such as `setup` and `glossary`), untouched. Shape: `config.example.json`.

Save a pointer to the source (`source_skill` for a skill, `source_refs` for memory or instruction files) plus the confirmed summary. Never copy a whole skill's or instruction file's content into config. If there's no writable folder, use the voice for this session only and say so.

To change it later, the user can say "update my announcement voice"; rerun the lookup and confirm again.
