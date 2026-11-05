# ADR-003: Real-Time vs Batch - Daily Batch With One Fast Exception

Status: Accepted · System: Marketing Automation with AI & Agents (property seeker side, WhatsApp) · Principles: P1, P2, P4

## Context

A property purchase decision takes weeks, so a message that arrives a few hours later rarely changes the outcome. WhatsApp is an intrusive channel, so message frequency must be capped. A stale price in a message is a factual error, which breaks principle P1. The team is small and prefers simple operations.

## Options considered

| Option | Pros | Cons |
| --- | --- | --- |
| A. Real-time, event-driven | Fastest alerts when a listing matches | Complex; hard to cap frequency; higher cost; more load on databases |
| B. Micro-batch every 15-60 minutes | Good speed with moderate complexity | Still needs always-on jobs and more monitoring than a daily run |
| C. Daily batch | Simple and cheap; easy to cap frequency and to review messages | A hot new listing or a price drop can reach the seeker up to a day late |

## Decision

Use a daily batch for matching, recommendations and re-engagement. Add one fast path, delivered within hours: a price increase on a property the seeker saved. Price drops stay in the daily batch. Every message re-checks the price and key specs against the Listings DB at send time, not at generation time, so a price that changed after the batch ran is never sent.

## Consequences

- Positive: simple, cheap and predictable; frequency caps and human review are easy to apply.
- Negative: new listings and price drops can reach seekers up to a day late; accepted because decisions take weeks.
- Price-increase messages use a pre-approved template with factual wording and no pressure language (P2). New templates go through the Human Approval Queue.
- The send-time check (P1) runs for every message, including the daily batch.

## Assumptions and revisit triggers

Assumptions: weeks-long purchase decisions, WhatsApp frequency caps, a small team. Revisit if data shows that late alerts hurt conversion, or if sellers' listings sell within hours. Moving to micro-batch or real-time would be a new ADR and requires ARB review.
