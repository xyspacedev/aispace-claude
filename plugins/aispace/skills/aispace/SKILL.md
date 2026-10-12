---
name: aispace
description: Uses AI Space, XY Space's multi-model service, from Claude. Runs one prompt across several AI models side by side with what each answer cost, asks another model (GPT, GLM, Kimi, DeepSeek, Grok, Qwen, Nemotron or another Claude) for a second opinion and keeps the follow-up going, lists the models a person's organization can use, and exports retained prompts for organization admins. Use when the user mentions AI Space or aispace, asks which AI model is best or cheapest for a task, wants to compare models or see how different models answer the same thing, wants a second opinion from another model, names a non-Claude model they want to ask, asks what models they can use, or types /aispace. Also use when an aispace tool returns an error the user needs explained.
version: 1.0.0
license: MIT
---

# AI Space

AI Space (https://ai.xyspace.dev) lets this Claude account ask other AI models through the
**AI Space connector**. Each person belongs to an AI Space organization, and every answer
another model gives is billed to that organization's budget. That is why this skill is
mostly about asking the right model once, with exact model keys, and showing the cost.

## The tools

The connector adds these tools. Your tool list may show them with a prefix in front of the
name; match on the ending.

| Tool | What it does | Spends money |
|---|---|---|
| `aispace_list_models` | The models this person's organization can use, as `key` and `label` | No |
| `aispace_compare_models` | Runs one prompt on 2 to 5 models at once; returns each answer and its cost, a total, and a link to the comparison | Yes, per model |
| `aispace_start_session` | Asks one model; returns its answer, a `session_id`, and a `[run: …]` line with the model and cost | Yes |
| `aispace_continue_session` | Sends a follow-up to an existing session | Yes |
| `aispace_export_prompts` | Organization owners and admins: the prompts the organization keeps for audit | No |

## 1. Check that AI Space is connected

If none of these tools is in your tool list, AI Space is not connected in this chat. Do not
answer as if you had asked another model. Tell the user how to switch it on, then stop:

- **The connector is installed but off in this chat:** click **+** in the chat box, choose
  **Connectors**, and switch **AI Space** on.
- **It says it needs signing in:** in Claude go to **Customize → Connectors → AI Space →
  Connect**, sign in with the email they use for AI Space, and click **Allow**.
- **AI Space is not listed at all:** follow the steps at https://ai.xyspace.dev/connect
  (pick the Claude tile).

## 2. Get the exact model keys first

Before you name a model in any other call, call `aispace_list_models` once in the
conversation. It is free. Map what the user said ("GPT", "Kimi", "Opus", "the cheap one") to
an exact `key` from that list.

Why this matters:
- `aispace_compare_models` refuses any key that is not in the list.
- `aispace_start_session` does **not** refuse an unknown key. It silently answers with the
  default model instead, so a guessed key gives the user an answer from the wrong model,
  presented as the one they asked for.

If the model the user wants is not in the list, say so and offer the closest models that
are. After every `aispace_start_session`, check that the model named in the `[run: …]` line
is the key you asked for. If it is not, tell the user which model actually answered.

## 3. Compare models

Use this when the user asks which model is best or cheapest for something, asks to compare
models, or wants to see how different models answer.

1. **Choose the models.** Use the ones the user named. If they named none, choose **3** that
   differ: one Claude model, one strong model from another maker, and one fast, cheap model
   (keys with "flash" or "mini" in them are usually the cheap ones). Use at most 5, and only
   more than 3 when the user asks, because every model is billed.
2. **Write the prompt for the models, not for yourself.** The other models cannot see this
   conversation. Put the user's actual task in `prompt`, with any text, facts or tone the
   answer needs. Do not add your own opinions about which model should win.
3. **Leave `system` out** unless the user wants a particular persona. Supplying one stops the
   models from reading web pages.
4. **Report the result:**
   - for each model, its answer (in full if short, otherwise the key part) and its cost;
   - which answer best fits what the user is trying to do, and why, in a sentence or two;
   - the cheapest answer you would still accept, if that differs;
   - the total cost line and the comparison link from the tool's result, so they can open
     it in AI Space.
5. If a model failed, the result says `[model-key] failed: reason`. Name it and give the
   reason in plain words. Do not re-run the whole comparison for one failed model.

## 4. Ask another model (a second opinion)

Use this when the user asks a particular model something ("ask GPT…", "what would Kimi
say", "get a second opinion on this", "have GLM review my paragraph").

1. Call `aispace_start_session` with the exact `model` key and a **self-contained**
   `prompt`: paste in the text, code or question it needs, because the other model cannot
   see this chat.
2. **Keep the `session_id`.** When the user follows up on the same thing ("ok ask it for a
   shorter version", "what about the second option"), call `aispace_continue_session` with
   that `session_id`. Do not start a new session for a follow-up: the session already holds
   the context, so continuing is cheaper, and the model remembers what it said. Start a new
   session only for a new topic or a different model.
3. Show the other model's answer clearly as **its** answer (quote it or label it with the
   model's name), then the cost from the `[run: …]` line. Add your own view afterwards only
   if it helps, for example where you agree or disagree.
4. If the `[run: …]` line says `stopped at step cap`, tell the user the answer may be cut
   short and offer to continue the session.
5. If a follow-up returns `No such session` (sessions end after 7 days idle), start a new
   session and tell the user the earlier context was lost.

Every session and comparison also appears in the person's AI Space account at
https://ai.xyspace.dev, where they can carry on in the browser.

## 5. Getting started

When the user types `/aispace`, asks "what can AI Space do?", or has just set AI Space up:

1. Do step 1. If AI Space is not connected, stop there.
2. Call `aispace_list_models` and show the models as a short list of labels.
3. Offer three things to try, written in plain words with real model names from the list,
   for example:
   - "Compare [a Claude model] and [another model] on writing a friendly reminder about an unpaid invoice"
   - "Ask [another model] to review this paragraph for clarity"
   - "Which of my models is cheapest for summarising a long email?"
4. Mention in one line that each answer from another model is billed to their AI Space
   organization, and that comparisons show the cost.

## 6. Spending rules

- Call a tool that spends money only when the user asked for another model's answer or a
  comparison, or plainly wants one. Do not route ordinary requests through AI Space; answer
  those yourself.
- Do not retry a failed call more than once, and never loop over models one by one when a
  single comparison would do.
- Quote costs exactly as the tool returned them. Never estimate prices or per-token rates
  from memory. If the user asks which model is cheapest, base the answer on costs AI Space
  returned in this conversation, and say so.

## 7. Export prompts (organization admins)

Only when an owner or admin asks for it. `aispace_export_prompts` returns up to 92 days per
call (the last 7 days by default), newest first; pass `nextCursor` back as `cursor` for the
next page. Offer to summarise the export or save it as a file.

## 8. When a tool refuses

Explain the refusal in plain words and say who can fix it. Do not retry calls that cannot
succeed.

| The tool says | Tell the user |
|---|---|
| No organization is associated with this account | They signed in to AI Space with an account that is not in an organization. They should accept their invitation email, or reconnect with the email their organization invited: **Customize → Connectors → AI Space**, disconnect, then connect again. |
| An active AI Space subscription is required | Their organization has no active plan or credit. An owner or admin can fix it at https://ai.xyspace.dev/billing. |
| Chat is turned off / Compare is turned off for your organization | An admin of their AI Space organization switched that feature off. Only that admin can switch it back on. |
| A weekly, monthly or organization limit has been reached | Say which limit, that a weekly or monthly limit resets at the start of the next period, and that their AI Space admin can raise it. |
| Unknown model key | Call `aispace_list_models` and retry once with the right key. |
| Comparing needs at least 2 distinct models / Too many models | Fix the list (2 to 5 different keys) and retry once. |
| No such session | Start a new session; tell the user the earlier context was lost. |
| Only an organization owner or admin can export prompts / Prompt retention is off | Exporting is for admins of an organization that keeps prompts. Nothing to retry. |
| An error about signing in or authorization | Do step 1: they need to connect AI Space again. |
