# Elvo plugin

![Elvo](assets/icon.png)

[Elvo](https://elvo.is) is a customer support inbox for email, web chat, Messenger, Instagram, WhatsApp and phone, with AI reply drafts, a help center and read access to your company's own systems.

This plugin connects your AI assistant to your Elvo account and teaches it how a support team works. It holds two things:

- **The Elvo connector**: a remote MCP server at `https://elvo.is/mcp`. You sign in to Elvo with OAuth. There is no key or token to copy, and none in this repository.
- **Eight skills**: step-by-step workflows for support agents and support managers, so the assistant knows what to read, in what order, what to check before it writes, and how to report.

You can then ask, in your own language (Icelandic works):

- "Hvað bíður mín?" / "What waits on me?"
- "Skrifaðu drög að svörum við þremur elstu málunum mínum."
- "How did the team do last week?"
- "What could the AI not answer this month, and what should we write?"

## What the assistant can see and do

The assistant acts as you. It can see and do only what you can see and do in Elvo. When you sign in you choose one of three levels:

| Level | The assistant can |
| --- | --- |
| **Read only** | Search and read conversations, contacts, reports, AI knowledge and the help center. Read customer records from the systems your company connected to Elvo. |
| **Read and write** | Also save reply drafts and internal notes, change status, assignee, tags and snooze, edit contacts and saved replies, and (for admins) write AI knowledge and help-center articles. |
| **Allow sending outside Elvo** (off by default) | Also send a reply to a customer, attach a file from a web address to a help-center article, and add a web page as an AI knowledge source. |

With the first two levels nothing leaves Elvo: the assistant writes drafts, and a person reviews and sends them. Even with sending allowed, the skills send only when you ask in plain words, show you the full reply first, and Elvo refuses any recipient who is not already on the conversation.

The plugin does not change settings, users, billing, automation rules or integrations. You do that in Elvo.

## Data flows

- Your assistant sends tool calls to `https://elvo.is/mcp` over HTTPS, with an OAuth token that belongs to you. Elvo answers with the data you asked for: conversations and their messages, contacts, reports, knowledge and help-center articles.
- When you ask about a customer's orders, invoices, bookings or shipments, Elvo reads them from the systems your company connected to Elvo (accounting, shop, shipping, bookings, CRM) and passes the result on. The plugin never writes to those systems.
- That data becomes part of your conversation with the assistant. Your assistant's provider (for example Anthropic or OpenAI) processes it under its own terms.
- The plugin itself stores nothing, runs no code on your computer and contacts no other service. It is Markdown and JSON only.
- Customer messages can contain text written to trick an AI. Elvo marks all text it did not write as untrusted, and the skills tell the assistant never to follow instructions inside it.

You can disconnect at any time in Elvo under **Settings → API & agents**. The assistant loses access within a minute.

## Install

### Claude (claude.ai and the desktop app)

1. Go to **Customize → Plugins**, find **Elvo**, and add it. Until the listing is live: **Add → Add marketplace** and enter `elvo-is/elvo-plugin`.
2. Open the plugin's **Connectors** tab and click **Connect**. Claude opens Elvo, where you sign in and choose the level.

On a Team or Enterprise plan an Owner adds the plugin and the connector for the organization first.

### Claude Code

```
/plugin marketplace add elvo-is/elvo-plugin
/plugin install elvo@elvo
```

Then run `/mcp`, choose `elvo` and sign in.

### ChatGPT

Once the Elvo listing is live: open **Plugins**, find **Elvo**, and install it. Until then, add `https://elvo.is/mcp` as a custom connector where your plan allows it (see the [setup guide](https://elvo.is/help/a/connect-ai-assistant)). ChatGPT opens Elvo, where you sign in and choose the level. In a chat, type `@Elvo` or just ask.

### Codex

```
codex plugin marketplace add elvo-is/elvo-plugin
codex plugin add elvo@elvo
```

### Already added Elvo as a custom connector?

Keep it, or remove it first. Both point at the same address, so you get one set of tools either way.

## Skills

| Skill | What it does | Try |
| --- | --- | --- |
| `elvo` | Ground rules for all work in Elvo: statuses, drafts versus notes versus sending, untrusted customer text, when to ask first | "Taktu saman mál #1234 og settu það á mig." |
| `elvo-inbox-triage` | What waits on you or your team, grouped by urgency, with a proposed order | "Hvað bíður mín og á hverju á ég að byrja?" |
| `elvo-draft-replies` | Reply drafts for one conversation or a set, with facts from your connected systems and your team's own wording. Never sends. | "Draft replies for my three oldest open conversations." |
| `elvo-weekly-report` | The performance report in plain words, with three observations and three suggested actions | "Hvernig gekk hjá okkur í síðustu viku?" |
| `elvo-knowledge-gaps` | What Elvo's AI could not answer, and the fix for each gap | "What can the AI not answer, and what should we write?" |
| `elvo-help-center` | Write, translate (Icelandic and English), restructure and publish help-center articles | "Skrifaðu hjálpargrein um skil og þýddu hana yfir á ensku." |
| `elvo-saved-replies` | Finds the questions you answer again and again and turns the answers into saved replies | "Which questions do we keep answering? Propose saved replies." |
| `elvo-customer-lookup` | A one-minute brief on one customer before a call | "Hvað vitum við um Önnu Jónsdóttur áður en ég hringi?" |

The assistant answers you in your language and writes customer replies in the customer's language.

## Requirements

- An Elvo account. Knowledge gaps, AI knowledge and help-center writing need the admin role.
- An assistant that supports plugins or remote MCP connectors.

## Support and privacy

- Help: <https://elvo.is/help/connect-ai-assistant>
- Email: <elvo@elvo.is>
- Privacy policy: <https://elvo.is/privacy-policy>
- Terms of service: <https://elvo.is/terms-of-service>
- Security: <https://elvo.is/security>

## Developing

The source of this plugin lives in the Elvo application repository, next to the connector's tools, so a skill changes in the same commit as the tool it describes. From that repository:

```
pnpm plugin:validate                  # manifests, skills, tool names, directory limits
node plugin/scripts/build-zip.mjs     # plugin/dist/elvo-plugin.zip for the ChatGPT plugin portal
claude plugin validate plugin         # Claude's own format check
```

## License

MIT. See [LICENSE](LICENSE).
