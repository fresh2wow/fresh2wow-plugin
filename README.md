# Fresh2Wow for Claude

Publish to your social networks from Claude, through your own [Fresh2Wow](https://fresh2wow.com/en/#ai-agents) workspace. Claude reads your brands, their voice and kit, your media, your connected accounts, your posts and your metrics; with your permission it uploads what you made, leaves drafts with each network's options, sends them to review and schedules them. Your approvals, calendar and analytics stay in Fresh2Wow, and Claude never holds the keys to your networks.

The plugin bundles two things:

- **The Fresh2Wow connector** — the hosted MCP server at `https://mcp.fresh2wow.com/mcp` (OAuth; nothing runs on your machine).
- **Two skills** that teach Claude the flows:
  - `/fresh2wow:post` — a finished file or an idea becomes a post: caption in the brand's voice, upload, a draft with per-network options, then review or scheduling.
  - `/fresh2wow:plan` — the brand's 4-week content plan: the price in credits first, the order on your yes, the plan week by week, its posts turned into drafts.

You need a Fresh2Wow account on the Starter plan or above (the content plan needs Creator or Agency).

## Install

In Claude Code:

```
/plugin install fresh2wow --marketplace orney21d/fresh2wow-plugin
```

or in two steps:

```
/plugin marketplace add orney21d/fresh2wow-plugin
/plugin install fresh2wow@fresh2wow
```

In Claude (web and desktop): **Customize → Plugins → Add → Add marketplace**, and enter `orney21d/fresh2wow-plugin`.

## Connect

1. In Claude Code run `/mcp`, choose **fresh2wow** and **Authenticate**. In Claude, open the plugin's **Connectors** tab and press **Connect**.
2. Fresh2Wow's permission page opens in your browser. Sign in, choose **the workspace** and **the level**, and press **Allow**:
   - **Leer (Read)** — Claude reads; it changes nothing.
   - **Borradores (Drafts)** — also uploads media and leaves drafts you approve and schedule in the app. An admin grants it.
   - **Programar (Schedule)** — also schedules or publishes a post, each time only after you confirm the post, the networks and the time in the conversation. An admin grants it.
3. Ask for what you need.

One connection reads one workspace. To work in another, connect again and choose it. See and revoke your connections in Fresh2Wow under **Configuración → Agentes IA**.

If you already added Fresh2Wow as a custom connector by hand, you can keep it: the skills work with either.

## Try it

- "What are my last five posts in Fresh2Wow and which networks did they go to?"
- "Summarise the voice of my brand in Fresh2Wow: tone, audience and banned words."
- "Upload final.mp4 to Fresh2Wow and leave it as a draft for Instagram and TikTok, proposed for Thursday at 10:00."
- "Send that draft to review and tell me who can approve it."
- "Order the content plan for my brand for the next four weeks, three posts a week on Instagram. Tell me the price first."
- "Schedule that draft for Thursday at 10:00." (Schedule level; Claude shows you the post, the networks and the local time and waits for your yes.)

## What the plugin sends, and where

The skills are instructions only: they run no code and collect nothing. The connector sends your requests to Fresh2Wow's own server (`mcp.fresh2wow.com`) under the permission you granted, and nowhere else. A file you upload goes from your machine straight to your workspace's media library through a one-time upload link. Fresh2Wow does not receive your conversation: only the arguments of the tools Claude calls. Calls are capped per connection (60 a minute, and a daily cap by plan).

- Guide: https://oems.gitbook.io/fresh2wow-docs/help-center/en/account-and-team/ai-agents (Spanish: https://oems.gitbook.io/fresh2wow-docs/cuenta-y-equipo/agentes-ia)
- Privacy: https://fresh2wow.com/en/privacy
- Terms: https://fresh2wow.com/en/terms
- Support: support@fresh2wow.com

## Licence

MIT, see [LICENSE](LICENSE). Fresh2Wow™ is a trademark of DMGO LLC; the licence covers this plugin's files, not the Fresh2Wow service.
