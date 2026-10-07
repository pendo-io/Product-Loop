# Product Announcement Examples

Worked examples in the short format. Every "Who cares most" line names the roles at a company that care most about the change, not an internal team; the readers are still your internal teams. All are invented for illustration, with placeholder links, and set in a fictional work-tracking product (Workload View, Timesheets, Plan & Track, Calendar Sync, Recurring tasks). Product areas, channels, plans, and statuses here are yours to replace with your own. Study them for tone, length, and structure. **When examples differ from the template, follow the template in `announcement-template.md`.**

## Contents

- [Open Beta, action splits by role](#open-beta-action-splits-by-role)
- [GA, one audience](#ga-one-audience)
- [Closed Beta, nominate customers](#closed-beta-nominate-customers)
- [Plan-limited rollout](#plan-limited-rollout)
- [Formatting notes](#formatting-notes)

## Open Beta, action splits by role

```
:calendar: **Calendar Sync | Two-way sync with Outlook | Open Beta:** **move a task by moving the meeting** :calendar:

Calendar Sync used to copy task due dates into Outlook one way. In this beta it also works the other way: drag the calendar entry to Thursday and the task's due date follows. Teams that live in their calendar stop keeping two schedules by hand.

**Who cares most:** Operations managers and team leads whose people plan their week in Outlook.

**From here:**
* *Workspace admins*: switch on two-way sync under Settings, Integrations
* *Members*: reconnect your calendar once to grant edit access
* [Help: Two-way calendar sync](https://example.com/help/PLACEHOLDER)
* Questions: #calendar-launch-questions
```

**Notes:**
- The value hook names the new behavior in plain words rather than the feature name.
- Four lines in "From here" because the first step differs by role: admins turn it on, members reconnect.
- The body explains the old behavior in half a sentence so readers see what changed.

## GA, one audience

```
:hourglass: **Timesheets | Approval reminders | GA:** **no more chasing timesheets on Friday afternoon** :hourglass:

Timesheets now reminds approvers when a submitted timesheet has waited two working days, and nudges anyone who hasn't submitted by the end of their work week. Managers pick the reminder day per team, so a team that closes hours on Monday isn't nagged on Friday.

**Who cares most:** Finance leads and project managers who close out billable hours each week.

**From here:**
* [Help: Set up approval reminders](https://example.com/help/PLACEHOLDER)
* Questions: #timesheets-launch-questions
```

**Notes:**
- Two lines in "From here" is fine: the help article covers setup, so no internal resource is needed.
- The roles are narrowed by what they do ("who close out billable hours each week"), not by an internal team.

## Closed Beta, nominate customers

```
:repeat: **Recurring tasks | Skip holidays automatically | Closed Beta:** **recurring work that respects the team's days off** :repeat:

Recurring tasks can read the workspace holiday calendar and move an occurrence that lands on a day off to the next working day. We want a handful of customers with offices in several countries to test it before it opens to everyone.

**Who cares most:** Operations managers who run weekly or monthly checklists across more than one office.

**From here:**
* Nominate a customer in #recurring-launch-questions with the countries their offices are in
* [Help: Holiday-aware recurrence (beta)](https://example.com/help/PLACEHOLDER)
* Questions: #recurring-launch-questions
```

**Notes:**
- Nominating a customer is the action, so it comes first; the help article is background.
- Repeating the channel is fine when the action itself happens in that channel.
- The nomination line says what to include, which saves a round of follow-up questions.

## Plan-limited rollout

```
:chart_with_upwards_trend: **Workload View | Capacity forecast | GA:** **see next month's overload before it happens** :chart_with_upwards_trend:

Workload View projects each person's booked hours four weeks ahead from their assigned and recurring tasks, and shades anyone heading past their capacity. Team leads can move work around before the crunch instead of after it.

Capacity forecast is included on the Business and Enterprise plans.

**Who cares most:** Resource managers and team leads who staff projects a month at a time.

**From here:**
* [Help: Capacity forecast](https://example.com/help/PLACEHOLDER)
* [Pricing: Compare plans](https://example.com/pricing/PLACEHOLDER)
* Questions: #workload-launch-questions

_Attached: images, gifs, videos if applicable_
```

**Notes:**
- The feature is limited to two plans, so one plain line in the body says which, the same way a regional line would.
- The second link is the public plan comparison, so a reader can answer "which plan do I need?" without an internal document.
- The attachments line is there because a screenshot of the shaded forecast shows the idea faster than a sentence can.

## Formatting notes

1. **Bold**: `**text**` (standard Markdown) for the header line, the value hook, and the "Who cares most:" and "From here:" labels.
2. **Bullets**: start the line with `* ` (an asterisk and a space).
3. **Italic**: sparingly. The audience labels in a split action (*"Workspace admins,"* *"Members"*) are a good use.
4. **Links**: standard Markdown, `[display text](URL)`. Two related links can share one line, separated by `·`, to keep "From here" short.
5. **Channels**: plain text such as `#channel-name`, never Slack `<@U…>` or `![](#id)` codes.
6. **Tone**: direct and concrete, with no marketing voice. Drop openers like "we're excited to" or "we're thrilled to".
7. **Customer names**: since the post is internal, you can name a customer who drove the work; keep quotes short and rare.
8. **Length test**: read it top to bottom; it should take under 10 seconds.
