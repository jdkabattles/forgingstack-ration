# ForgingStack Ration — skill for agents

Use this tool when many agents share one quota and the caller wants
proof the pool enforced its limits.

## When to use it

- Before an agent consumes shared quota: draw from the pool first,
  and act only on `allowed`.
- When the quota is explicit: a pool with a capacity, a refill rate,
  a per-agent cap, and a burst limit.
- To audit a draw afterwards: every draw, allowed or denied, has a
  hash-chained receipt — fetch it with `get_receipt`.

## When NOT to use it

- To decide if a draw is wise or fair. This tool enforces quota only.
- When there is no pool. Create one first with `create_pool`; do not
  draw from a pool id you invented.

## How to call it (MCP)

Tool: `create_pool`
Arguments: `name`, `capacity_units`, `refill_units_per_minute`,
`per_agent_cap_units`, `burst_units` (burst must be 1..capacity).

Tool: `draw`
Arguments: `pool_id`, `agent_id`, `units`
(optional: `idempotency_key` — replaying the same key returns the
original receipt free).

Tool: `topup_pool`
Arguments: `pool_id`, `units` — adds units, clamped at capacity.
Free, no billing credit charged.

## How to read the result

- `decision: "allowed"` → the units were drawn. Keep the receipt id.
- `decision: "denied"` → do not act as if you drew. The `reason` is
  the whole explanation: `draw_exceeds_burst`,
  `pool_insufficient_units`, or `per_agent_cap_exceeded`.
- A denied draw still costs 1 credit. An idempotency-key replay is
  free and returns the original receipt.
- `remaining_units` is what is left in the pool after this draw.
  `reset_in_seconds` is the seconds until the pool refills to full
  (null when the refill rate is 0 or the pool is already full).
- The per-agent cap counts an agent's allowed draws inside a rolling
  60-minute window, not a calendar hour.

## Status

Live, v0.1.0. API: https://forgingstack-ration.jkbattles.workers.dev
Signup: POST /v1/signup with an email returns an API key once and
100 starter credits; click the verification link before the key can
create pools or draw. Billing: $10 for 1,000 credits via POST
/v1/billing/checkout, which returns the Stripe Payment Link.
