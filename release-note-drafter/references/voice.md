# Brand voice lookup

Optional. Finds a brand voice or writing style the user has already specified, so entries match it without asking the user to paste it every time. Nothing here is required: with no voice, the skill uses its built-in style (flat-affect, descriptive, present tense, second person, active voice).

## What a voice can and can't change

**The voice governs tone and word choice only.** It never overrides this skill's hard rules, which win every time they conflict:

- The release-note standard in `references/release-note-standards.md`: eligibility, the open beta and GA rule, entry structure (bold title, one paragraph, doc pointer), the 50 to 90 word length, approved value patterns, banned phrases, terminology traps, and durability rules.
- No em dashes, en dashes, or ampersands in the entry.
- No dates or time-bound phrasing in the entry.
- Ticket ids and keys only inside URLs, never in the title or prose.
- The mechanical character check, run on every entry before it's presented.

If a voice rule conflicts with one of these (for example "use em dashes freely", "lead with the launch date", "use exclamation points", "call it seamless"), follow the skill's rule, apply the rest of the voice, and tell the user once, in the notes block, which voice rule was set aside and why.

## Lookup order

Run this once per session, before drafting the first entry.

1. **This skill's saved config.** If `release_note_drafter.voice` exists, use it without asking again. If it points to a `source_skill`, re-read that skill: where its current rules differ from the saved summary, let the skill win and tell the user.
2. **Installed skills that define a voice.** Look through the skills available in this session for one whose name or description mentions brand voice, voice, tone, style guide, or writing style. Read it and take its voice or style rules as the source.
3. **Memory and standing instructions.** Check Claude's memory and standing instructions for this user (memory files, CLAUDE.md or project instructions, saved user preferences) for voice, tone, or style rules.

If step 1 finds a saved voice, use it and skip steps 2 and 3. Otherwise check both.

## What to do with what you find

- **One found:** summarize it in two to four lines (how it sounds, traits it is and isn't, terms to avoid), name the source (the skill's name, or "your memory" or "your project instructions"), note any rule the hard rules above will set aside, and ask once: "Use this voice for release notes?" with *Yes*, *Edit it*, and *No, use the default style*.
- **Several found:** show each summary with its source and let the user pick one or merge them. Confirm the result once.
- **None found:** use the built-in style. Offer, in one line, to set a voice ("Want to set a brand voice for release notes? Paste a style guide or point me to one."). Never require it, and don't ask again in the same session.
- **User declines:** use the built-in style and don't save a voice.

## Saving

Save only after the user confirms. Write the `release_note_drafter` section of a shared config file, by default `config.json` in the user's working folder. If the `daily-digest` conductor's shared config (`.daily-digest/config.json` in the workspace) exists, use this skill's section of that file instead. Keep other sections, and the other keys in this section (such as `page` and `glossary`), untouched. Shape: `config.example.json`.

Save a pointer to the source (`source_skill` for a skill, `source_refs` for memory or instruction files) plus the confirmed summary. Never copy a whole skill's or instruction file's content into config. If there's no writable folder, use the voice for this session only and say so.

To change it later, the user can say "update my release-note voice"; rerun the lookup and confirm again.
