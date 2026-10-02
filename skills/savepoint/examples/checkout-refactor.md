---
id: savepoint-2026-01-15-checkout-refactor
title: Checkout refactor
date: 2026-01-15
type: savepoint
status: active
tags: [checkout, refactor]
generated_at: 2026-01-15T18:20:00Z
---

## State

Payment step extracted into `CheckoutService`; the legacy route still uses the old inline logic. Tests are green on `main`, but the feature branch is 2 commits ahead.

## Done
- `src/checkout/service.ts:1` — new service handling card and pix flows
- `tests/checkout.test.ts` — 12 cases passing
- Decision recorded: retries handled by the queue, not the service

## Pending / Blocked
- Remove the legacy branch in `routes/checkout.ts:88` (blocked on QA window)
- Confirm webhook signature header with the payments team `(to confirm)`

## Bottlenecks
- Sandbox webhook is flaky; needs a local tunnel to reproduce

## Sources of truth
- `docs/adr/014-payment-retries.md`
- Branch `feat/checkout-service`
