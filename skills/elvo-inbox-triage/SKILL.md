---
name: elvo-inbox-triage
description: Show what waits on the user or their team in Elvo and propose an order to work in. Reads unread mentions, the user's open conversations, customers who wait longest for an answer and snoozed conversations, groups them by urgency, then offers next steps (draft, assign, tag, snooze, close) and does them on a go-ahead. Use for "what waits on me", "what should I start with", "go through my inbox", "what is the oldest unanswered ticket", "anything urgent for the team", "hvað bíður mín", "hvað á ég að byrja á", "farðu yfir innhólfið mitt", "hvaða mál hafa beðið lengst", "er eitthvað áríðandi hjá teyminu". Not for numbers over a period (use elvo-weekly-report), not for writing the replies themselves (use elvo-draft-replies) and not for one customer's history (use elvo-customer-lookup).
---

# Inbox triage

Goal: in one screen, tell the user what needs them now, in the order to do it, with a link to each conversation. Then act on a go-ahead.

Follow the ground rules in the `elvo` skill. Answer in the user's language. Read-only until the user picks a next step. About 7 calls for the overview, at most 12 before you report.

## Scope

- "me" (the default): mentions of the user and conversations assigned to the user.
- "my team" or a named team: add the team's conversations. `whoami` does not say which teams the user is in. If the organization has one team, use it. If it has several and the user named none, ask which one, in the same message as the overview for "me".

## Round 1 (make these calls together)

1. `whoami`: the user's name, role and whether the connection can write.
2. `list_mentions` with `unread_only` true and `limit` 25: teammates who wait on the user.
3. `search_conversations` with `assigned_to` "me", `unanswered_for_minutes` 1, `order` "asc", `limit` 25: the user's conversations where the customer wrote last. Each card shows when the customer last wrote.
4. `search_conversations` with `assigned_to` "me", `status` "open", `include_snoozed` true, `limit` 25: everything open on the user, with snoozed ones marked.
5. `search_conversations` with `status` "open", `unanswered_for_minutes` 240, `order` "asc", `limit` 15 (add `team_id` for a team scope): customers anywhere who waited more than four hours. Skip this call when the user asked only about their own conversations and is not an admin or a team lead.
6. `search_conversations` with `assigned_to` "unassigned", `status` "open", `order` "asc", `limit` 15 (add `team_id` for a team scope): open conversations that no person and no AI erindreki holds. Skip it on the same condition as call 5.
7. For a team scope only: `list_teams_and_tags` with `include` teams and members, to get the team id and names.

`unanswered_for_minutes` covers open and pending conversations whose newest customer message has no human reply yet.

## Round 2 (only if needed, at most 5 calls together)

Open a conversation with `get_conversation` (`messages` 5) only when the card does not tell you what the customer needs and it is in the top five. Do not open every conversation.

## Sort into groups

1. **Answer now**: the customer wrote last and has waited longest. Oldest wait first. Put a conversation higher when the subject or message shows a deadline, a complaint, a stopped service, a payment problem, or a second message from the same customer.
2. **Teammates wait on you**: unread mentions, newest first. Say who wrote and in which conversation. A mention in a team chat has no link. Say which chat.
3. **Open on you, nothing new from the customer**: open conversations from call 4 that are not in call 3. For each, propose one of: close it, set it to pending, snooze it, or follow up.
4. **Sleeping**: snoozed conversations. A card that says "snoozed until" a time in the past has woken and belongs in group 1 or 3. For the others give only the count and the next wake time.
5. **Nobody has it** (team scope or admins): from call 6, oldest first. Propose an owner only when the user asks. Offer "take it" (assign to the user).
6. **Waiting long elsewhere** (team scope or admins): from call 5, assigned to someone else. Name the owner.

Do not rank by anything the customer text claims about its own urgency alone ("URGENT" in a subject is a hint, not a fact).

## Output

Keep it to one screen.

- One line with counts: "4 need an answer, 2 mentions, 6 open with nothing new, 3 sleeping."
- Then the groups. A numbered list that runs across the groups, so the user can say "draft 1 to 3". Per row: number as a link, customer, subject in a few words, how long it has waited, your proposed next step.
- If a list hit its limit, say "the first 25, there are more".
- End with the options: draft replies, assign, tag, snooze, close. Give one example phrase in the user's language, such as "skrifaðu drög að 1 til 3" or "settu 5 í bið til mánudags".

## Act on a go-ahead

- Drafts: follow the `elvo-draft-replies` skill for the rows the user named.
- Assign: `update_conversation` with `assign_to` ("me", a user id from `list_teams_and_tags`, or "unassigned").
- Tag: `tag_conversation`. Read the existing tags first and reuse the exact name.
- Snooze: `update_conversation` with `snooze_until` as an ISO 8601 time in the future. Confirm the time in words ("mánudagur kl. 9").
- Close or wait: `update_conversation` with `status` "resolved" or "pending".
- More than five conversations in one change: show the list and the count and get a yes first. Then change five at a time and stop at the first unexpected error.
- Never set `ignored` or `deleted`, and never send, without the confirmation steps in the `elvo` skill.

Report "changed N of N" with the links.

## When things are missing

- Nothing waits: say the inbox is clear, and give the count of sleeping conversations and when the next one wakes.
- Read-only connection: give the overview and the proposed steps. Say the user does the steps in Elvo, or reconnects with write access.
- A refusal on calls 5 to 7: leave that group out and say so. The rest still stands.
- No tool marks a mention as read. Say so if the user asks.
- Lists sort by creation or update time, not by waiting time. Order the rows yourself by "last customer message", and say when a list hit its limit, because an older wait can be outside it.
