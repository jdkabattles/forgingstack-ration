# ForgingStack Ration

One quota pool, many agents, no overruns.

mcp-name: io.github.jdkabattles/forgingstack-ration

This is the **public client** for Ration. The service itself — the
token bucket, the receipt chain, the key store, and billing — runs
on ForgingStack's private service and is **not** in this repo.
Copying this repo does not give you the service: you still need an
API key and credits, from signup below.

Live API: https://forgingstack-ration.jkbattles.workers.dev

## What it does

A principal creates a pool with a capacity, a refill rate, a
per-agent cap, and a burst limit. Agents draw units from it. The
pool enforces the limits and issues a hash-chained receipt for
every draw attempt — allowed **and** denied.

A draw is denied when:

- the units exceed the burst limit (`draw_exceeds_burst`)
- the pool does not have enough available units
  (`pool_insufficient_units`)
- the agent's allowed draws inside the rolling 60-minute window
  would exceed the per-agent cap (`per_agent_cap_exceeded`)

The per-agent cap window is a rolling 60 minutes — a documented
design choice, not a calendar window.

## Signup (v1)

```bash
curl -X POST https://forgingstack-ration.jkbattles.workers.dev/v1/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com"}'
```

Returns an API key **once** and 100 starter credits. Save the key
immediately — it is stored only as a hash and cannot be shown again.
Click the verification link in the signup email before the key can
create pools or draw: until then, pool creation and draws return
403 `email_not_verified`.

## Buying more credits

1 credit per pool creation, 1 credit per draw call (allowed or
denied). Top-ups, pool reads, pool lists, and deletes are free.

```bash
curl -X POST https://forgingstack-ration.jkbattles.workers.dev/v1/billing/checkout \
  -H "X-API-Key: YOUR_PRIMARY_KEY"
```

Returns the Stripe Payment Link: **$10 for 1,000 credits**. Pay on
Stripe's hosted page with the same email you signed up with.
Credits land once the payment is verified (within the hour).

## Use it (MCP, stdio)

Set the key, then run the MCP server in this repo:

```bash
export FORGINGSTACK_API_KEY="your-key-here"
python -m app.mcp_server
```

Tools: `create_pool`, `draw`, `get_pool`, `list_pools`,
`topup_pool`, `delete_pool`, `get_credits`, `get_receipt`.
See `SKILL.md` for when to use it and when not to.

## Price

1 credit per pool creation. 1 credit per draw call — a denied draw
still costs 1 credit, the draw was performed. An idempotent replay
(same `Idempotency-Key`) is free and returns the original receipt.

## Files in this repo

- `app/mcp_server.py` — MCP wrapper client (stdio, stdlib only)
- `SKILL.md` — when to use it, how to read allowed/denied
- `agent-card.json` — A2A-style discovery card
- `server.json` — MCP Registry metadata
- `pyproject.toml` — package metadata for the MCP client
