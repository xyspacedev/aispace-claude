# AI Space for Claude

Use [AI Space](https://ai.xyspace.dev) from the Claude app. Once it's installed, you can ask
Claude things like:

- "Compare Claude and GLM on writing a friendly reminder about an unpaid invoice"
- "Ask Kimi to review this paragraph for clarity"
- "Which of my models is cheapest for summarising a long email?"

Claude asks the other models through AI Space and shows each answer with what it cost.

The plugin adds two things to Claude:

- **The AI Space connector**, which lets Claude reach the models your AI Space
  organization can use.
- **The `aispace` skill**, which teaches Claude when to compare models, how to keep a
  conversation with another model going, and what to do when AI Space says no.

You need an AI Space account in an organization. Claude plugins need a paid Claude plan
(Pro, Max, Team or Enterprise). On the Free plan, add the connector by hand instead:
[ai.xyspace.dev/connect](https://ai.xyspace.dev/connect).

## Install it in the Claude app

This works the same in the desktop app and on claude.ai, and the plugin follows your Claude
account into Chat, Cowork and Claude Code.

1. In Claude, open **Customize** in the left sidebar, then the **Plugins** tab.
2. Click **Add**, then **Add marketplace**, then **Add from a repository**.
3. Paste `xyspacedev/aispace-claude` and confirm.
4. Find **AI Space** in the list and click **Add**.
5. Sign in to AI Space: go to **Customize → Connectors → AI Space → Connect**, sign in
   with the email you use for AI Space, and click **Allow**.
6. Start a new chat and ask: **What can AI Space do?**

If Claude doesn't use AI Space in a chat, click **+** in the chat box, choose
**Connectors** and switch **AI Space** on.

## Install it for a whole team

On a Claude Team or Enterprise plan, an owner can add this repository as a marketplace for
the organization and make AI Space installed by default, so members only sign in. See
Anthropic's [Use plugins in Claude](https://support.claude.com/en/articles/13837440-use-plugins-in-claude).

## Install it in Claude Code

```
/plugin marketplace add xyspacedev/aispace-claude
/plugin install aispace@xyspace
```

Then type `/mcp`, choose **aispace** and sign in.

## What it costs

Every answer from another model is billed to your AI Space organization, at the price AI
Space shows. Listing your models is free. Claude shows the cost of each answer and the
total for a comparison, and every comparison and conversation also appears in your AI Space
account at [ai.xyspace.dev](https://ai.xyspace.dev).

## Troubleshooting

| What you see | What to do |
|---|---|
| Claude answers without using AI Space | Click **+** in the chat box → **Connectors** → switch **AI Space** on. |
| "No organization is associated with this account" | Accept your AI Space invitation, or reconnect with the email your organization invited. |
| "An active AI Space subscription is required" | Ask your organization's owner or admin to check billing at [ai.xyspace.dev/billing](https://ai.xyspace.dev/billing). |
| A limit has been reached | Weekly and monthly limits reset at the start of the next period. Your AI Space admin can raise them. |

## What's in this repository

```
.claude-plugin/marketplace.json     the "xyspace" marketplace, listing one plugin
plugins/aispace/
  .claude-plugin/plugin.json        the plugin
  .mcp.json                         the AI Space connector (https://ai.xyspace.dev/mcp)
  skills/aispace/SKILL.md           the aispace skill
  skills/aispace/evals/             the checks the skill was tested against
```

MIT licensed. Made by [XY Space](https://xyspace.dev).
