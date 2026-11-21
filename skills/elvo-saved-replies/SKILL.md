---
name: elvo-saved-replies
description: Find the questions an Elvo support team answers again and again and turn the answers into saved replies (canned responses). Looks at recent conversations by tag or topic, reads how the team actually answered, compares with the saved replies that exist, proposes new or improved ones with the evidence, and creates or updates them on a go-ahead. Use for "what do we keep answering", "make saved replies for our common questions", "create a canned response for delivery times", "review our saved replies", "hvaða spurningum svörum við aftur og aftur", "búðu til stöðluð svör", "gerðu staðlað svar um afhendingartíma", "farðu yfir stöðluðu svörin okkar". Not for a reply to one conversation (use elvo-draft-replies), not for a public article (use elvo-help-center) and not for facts the AI lacks (use elvo-knowledge-gaps).
---

# Saved replies

Goal: the five or so answers the team types most often exist as saved replies in the team's own voice, with no duplicates. Agents insert them in the reply box, and Elvo's AI may reuse them in its drafts.

Follow the ground rules in the `elvo` skill. Answer in the user's language. Write each saved reply in the language the customers use. Budget: about 25 calls. Nothing is created before a go-ahead.

## Round 1 (together)

- `list_teams_and_tags` with `include` teams, tags and canned_responses: the existing saved replies (title, id, category, the start of the text), the tags with their sizes, and the team ids.
- `performance_report` with `period` `last_30_days`: conversation volume per tag, which shows where the repeated questions are. Skip it when the user named the topic.

If the user named a topic ("delivery times"), go straight to round 2 with that topic.

## Round 2: find the repeated questions (together, at most 5 searches)

For each of the three to five largest tags, or for the topic:

- By tag: `search_conversations` with `tag` (exact name, case-sensitive), `status` "resolved", `created_after` 30 days back, `limit` 10.
- By words: `search_conversations` with `query`, `status` "resolved", `created_after` 30 days back, `limit` 10. Try the word in the customers' language, and one synonym.

The subjects on the cards show which questions repeat. Group them. A candidate is a question that appears at least three times in the period.

## Round 3: read how the team answers (at most 10 reads, 5 together)

For each candidate, `get_conversation` on two or three conversations. Look at the replies the team sent, not the customer messages, and not Elvo's AI.

A candidate is real when the team's replies say the same thing each time. If the answers differ between agents, that is a finding of its own: report the difference and ask the user which answer is the house answer. Do not average them.

## Compare with what exists

For each candidate, check the saved replies from round 1:

| Situation | Proposal |
| --- | --- |
| No saved reply on the topic | New |
| One exists and matches what the team sends | Leave it. Say it exists. |
| One exists but the team now answers differently | Improve it |
| Two cover the same question | Keep one. Tell the user. No tool deletes a saved reply. |

The list shows only the start of each text. For every saved reply that could cover a candidate, read the whole text with `get_saved_reply` (together, at most 8) before you judge it. It also shows the teams that may use it and whether Elvo's AI may reuse it.

## Propose

Show a table: question, how often in the period, proposal (new, improve, leave), and example conversations as links. Under it, the full text of each new or improved reply, as the customer would see it.

How to write one:

- General, not personal. No customer name, order number, date or amount from a real conversation.
- The team's own wording, greeting and sign-off, taken from the replies you read. No exclamation marks, no em-dashes.
- Short. The answer first, then the one detail people ask next.
- Elvo supports variables in saved replies (such as the customer's name). Use a variable only in the exact form you see in the full text of an existing saved reply (`get_saved_reply`). If you see none, write the text without variables and say the user can add them in Elvo.
- A clear title the agent will search for ("Afhendingartími", "Skil og endurgreiðsla"). A `category` when the existing ones use categories.
- `team_ids` when the answer belongs to one team. Leave it out for everyone.
- `exclude_from_ai` true only for an internal template the AI must not reuse.

Wait for a go-ahead. The user may accept some and change others.

## Create or update

- New: `create_saved_reply` with `title` and `content`. The content is simple HTML: `<p>` paragraphs, `<br>`, `<strong>`, `<a>`. One `<p>` per paragraph. Nothing else.
- Improve: `update_saved_reply` with the `saved_reply_id`. A new `content` replaces the whole old text, so always read the current text with `get_saved_reply` first, start from it, keep its variables and links, and change only what must change. Show the user the old and the new text before the call. To change only the title, category or teams, send only those fields.
- Create at most five in a row, then report.

## Report

What was created or updated (title, category, team), what was left as is, and what waits on the user (answers that differ between agents, duplicates to remove in Elvo).

Offer the next step where it fits: a public help-center article for a question customers could answer themselves (`elvo-help-center`).

## When things are missing

- Read-only connection: do the analysis and give the full texts in the chat. Say the user adds them in Elvo under Settings → Canned responses.
- The plan's limit on saved replies is reached (the create call is refused): say so, and propose which existing ones to improve instead.
- No tags, or most conversations have no tag: search by words. Ask the user for the three questions they are most tired of answering.
- Few conversations in 30 days: use 90 days (`last_90_days`, and `created_after` 90 days back) and say so.
- Nothing repeats three times: say so. Do not invent saved replies to fill a list.
