---
name: elvo-weekly-report
description: Explain the support team's performance in Elvo for a week or another period, in plain words for a manager. Covers volume, time to first reply, resolution, backlog, each person and team, what customers wrote about, and what Elvo's AI erindrekar did, compared with the period before. Ends with three observations, three suggested actions and a short summary to paste to the team. Use for "weekly report", "how did we do last week", "how fast do we answer", "what are customers writing about", "how much does the AI handle", "vikuskýrsla", "hvernig gekk í síðustu viku", "hvað erum við lengi að svara", "um hvað eru viðskiptavinir að skrifa", "hvað gerði gervigreindin". Not for what to work on right now (use elvo-inbox-triage) and not for what the AI could not answer (use elvo-knowledge-gaps).
---

# Weekly report

Goal: a manager reads it in two minutes and knows how the week went, what changed, and what to do about it.

Follow the ground rules in the `elvo` skill. Answer in the user's language. Read-only. One call gives almost everything. At most 8 calls.

## Pick the period

| The user says | `period` |
| --- | --- |
| "weekly report", "last week", "síðasta vika" (the default) | `last_week` (the previous whole Monday to Sunday) |
| "this week", "so far", "þessi vika" | `this_week` |
| "last 7 days" | `last_7_days` |
| "this month", "last 30 days", "mánuðurinn" | `last_30_days` |
| "the quarter" | `last_90_days` |
| Named dates | `custom` with `from` and `to` as YYYY-MM-DD |

The previous values compare with the period of equal length just before. Pass `comparison` `previous_year` only when the user asks for a year-on-year view.

## Round 1 (together)

- `performance_report` with the period. Add `team_id` when the user named a team.
- `list_teams_and_tags` with `include` teams and members: names for the ids in the report.

The output has three parts: one line per AI erindreki (replies, resolutions, handoffs to a person, drafts), conversation volume per tag, and the performance numbers as JSON.

## Read the numbers correctly

- All times are **minutes** and rates are percentages. `tfr` is time to first reply and `tfr_business` is the same counted in business hours only. Convert the minutes: under 90 minutes say minutes, under 48 hours say hours, else days.
- `current` and `previous` are **medians**: the typical case. `previous` is the comparison period. `p90` is the slow tail: nine of ten were faster than this.
- Time to first reply has two forms: wall clock and business hours. Lead with the business-hours value when the report has one, and say it counts opening hours only.
- First-contact resolution is the share of resolved conversations closed with one reply.
- Created and closed are counts in the window. More created than closed means the backlog grew.
- The backlog (open, pending, age of the oldest open conversation) is a snapshot from now, not from the period. Say so.
- The reopen rate is the share of resolved conversations that came back.
- AI numbers: drafts offered, drafts used, and the share of closes a machine handled. An erindreki's handoff is a conversation it passed to a person, which is the design, not a failure.
- A peer median, when present, is the typical value across Elvo customers. Mention it once, as context.
- The tag section has no previous period. Give counts and shares, never a trend you cannot see. "No tag" is its own line: a high share means the tags say little.
- A field that is null or missing means "not available". Do not turn it into zero.
- Quote numbers exactly as the report gives them, rounded to a sensible unit. Never estimate a number that is not there.

## Per person and per team

- Present people neutrally: a table sorted by name, not by speed. No "best" and "worst".
- Roles differ. A person with few conversations may handle the hard ones, work part time or lead the team. Say what the numbers show and nothing about the person.
- Fewer than ten conversations in the period is a small sample. Mark it and draw no conclusion from it.
- One week never shows a pattern about a person. If the user asks to judge someone, give the numbers, add this caveat, and suggest a longer period.

## Round 2 (optional, at most 5 calls)

Only when an observation needs an example, or the user asked what customers wrote about in detail:

- `search_conversations` with `tag` set to the largest or most surprising tag, `created_after` at the start of the period, `limit` 5.
- `get_conversation` on two of them, to say in one line each what the customers asked.

## Output

1. **Title**: the period with its dates, and the team if filtered.
2. **The week in numbers**: five to seven lines in plain words, each with the change from the period before. Example shape: "Fyrsta svar barst að jafnaði eftir 42 mínútur á opnunartíma (var 1,1 klst.)."
3. **People and teams**: a small table, only when there is more than one person or team.
4. **What customers wrote about**: the top five tags with counts, and how many of each are still open.
5. **What the AI did**: one line per erindreki, and how many drafts the team used.
6. **Three observations**: each ties to a number in the report. One of them should be something that went well.
7. **Three suggested actions**: each concrete, each tied to an observation. Examples: a tag that grew gets a saved reply or a help-center article (`elvo-saved-replies`, `elvo-help-center`); many handoffs on one topic means a knowledge gap (`elvo-knowledge-gaps`); a slow tail on first reply means a look at the oldest unanswered (`elvo-inbox-triage`).
8. **Offer**: a summary of at most 120 words to paste to the team, in the team's language, with no names of individual people unless the user asks for them.

## When things are missing

- `custom` without both dates: ask for the missing date.
- A refusal: the user's role cannot read reports. Say an admin can.
- An empty period (nothing created, nothing closed): say so. Offer a longer period.
- No AI section or no tag section: the organization does not use them, or the part failed. Say "not available" for that part and report the rest.
- The JSON was cut at the size limit: filter by `team_id`, or use a shorter period, and say what you left out.
- Numbers match the Frammistaða page in Elvo. Point the user there for charts.
