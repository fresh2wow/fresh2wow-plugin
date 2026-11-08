---
name: plan
description: Order, read and draft a brand's 4-week content plan in the member's Fresh2Wow workspace — the price in credits first, the order only on the member's yes, then the plan week by week and its posts turned into drafts. Use when the user says "plan my content for the next month in Fresh2Wow", "order the content strategy for my brand", "what does the plan cost", "is my plan ready", "show me week 1", "turn the plan into drafts", "planifica mi contenido", "pide la estrategia de contenido", "¿ya está el plan?", "pasa la semana 1 a borradores".
argument-hint: "[brand] [goal] [posts per week] [networks]"
---

# A 4-week content plan through Fresh2Wow

Fresh2Wow's content strategy is a 4-week calendar made for one brand: a diagnosis, 3–4 pillars, and one slot per post with its day, local time, idea, hook, a caption in the brand's voice, format and destinations. It is the same plan the app makes under *Agente → Entregables*, at the same price in credits. This skill runs the loop from the conversation with the Fresh2Wow connector's tools (bare tool names below).

Answer in the member's language. Show times in the workspace's zone (the `_local` twins).

## 0. Can this connection do it?

`get_workspace`:

- The plan must include the content strategy (Creator and Agency). On another plan the tools answer `PLAN_LIMIT`: say it opens on Creator and stop.
- The level must be `draft` or `schedule`. At `read`, reading a finished plan works (`get_strategy_run`); ordering and drafting do not.

## 1. The setup

Gather, asking only for what the member has not said:

- **The brand** — `list_brands`.
- **The goal** — what the four weeks must achieve, in the member's own words, 3 to 140 characters.
- **The destinations** — 1 to 6 of the brand's own active destinations, from `list_social_accounts` with the `brand_id`.
- **The cadence** — posts per week, 1 to 7 (the app's default is 2).
- Optional: `start_week` (a Monday, `YYYY-MM-DD`; omitted = the next Monday) and `language` (omitted = the brand voice's).

If the brand already has a recent plan, `get_strategy_run` with the `brand_id` shows it and the last ten: mention it before ordering another.

## 2. The quote — always before the order

`generate_content_strategy` with the setup and **without** `confirm`. Nothing is ordered or charged. Show the member, plainly:

- the price in credits and the credits available,
- how many posts over which four weeks,
- the destinations by name.

If the credits available are fewer than the price, say so: the member adds credits in the app first.

## 3. The order — only on a clear yes

When the member says yes to that price, call `generate_content_strategy` again with the same setup, `confirm: true` and a fresh `idempotency_key` (for example `plan-2026-11-09-yoga`). The credits are reserved and settle when the plan is done; a failed run releases them. The same key answers the same run again, so reuse it only to retry this very order — never to order a second plan.

## 4. Wait and read

The run takes a few minutes. Read it with `get_strategy_run` and the `run_id`, waiting about the `poll_after_seconds` the order answered between reads; say that it is in progress rather than polling in silence for long.

- `succeeded` — present the diagnosis and the pillars, then the slots. For a long plan read one week at a time with `week` (1–4).
- `failed` — say so, and that the reserved credits were released. Offer to order again with a **new** key.

Present the plan as the member's to judge: the slots are proposals.

## 5. Drafts

When the member picks slots (or says "all of them"), `create_drafts_from_strategy` with the `run_id` and the `slot_ids` (omitted = every slot not drafted yet). Each slot becomes a draft with its caption, destinations, format and suggested time. The answer counts what was created, what already existed and what was skipped (a destination that left the brand).

The drafts are in the app under *Publicaciones*, marked «vía agente IA», for the member to review, edit, approve and schedule. To send one to review or schedule it from the conversation, follow the `post` skill's last step — scheduling needs the `schedule` level and the member's explicit yes for each post.

## When a tool refuses

- `PLAN_LIMIT` — the plan does not include the strategy.
- `PERMISSION` naming a level — the connection's level is too low.
- `INSUFFICIENT_CREDITS` — not enough credits; nothing was charged.
- `VALIDATION` — the message names the field (a goal too short, a destination that is not the brand's, a start week that is not a Monday).
- `RATE_LIMITED` — wait the seconds it gives.

Follow the `next` line of the answer; do not retry an order with a new key unless the member wants a second plan.
