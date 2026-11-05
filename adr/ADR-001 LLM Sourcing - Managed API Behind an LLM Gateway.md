# ADR-001: LLM Sourcing - Managed API Behind an LLM Gateway

Status: Accepted · System: Marketing Automation with AI & Agents (property seeker side, WhatsApp) · Principles: P1, P3, P4

## Context

The Content Agent writes short text per listing to fill pre-approved WhatsApp templates. The prompt contains listing data only (area, price, rooms); no buyer data. Personalization such as the seeker's name is added after generation, outside the LLM. Because generation is per listing and not per user, cost scales with the number of listings, not users (assumption: about 2,000 LLM calls per day). No legal, contractual or internal constraint forces self-hosting. Message language is not final; Arabic and English are assumed.

## Options considered

| Option | Pros | Cons |
| --- | --- | --- |
| A. Managed API | High quality, fast start, no GPU operations, absorbs load spikes | Vendor dependency; data leaves the company; price and model changes are outside our control |
| B. Self-hosted open-weights model | Data stays in-house; fixed cost at very high volume | Needs GPUs and MLOps effort; Arabic quality must be proven; hard to handle spikes; overkill at about 2,000 calls per day |
| C. Hybrid (API now, self-hosted later for specific tasks) | Flexible | Needs a routing layer, which the Gateway already provides |

## Decision

Use a managed API behind an LLM Gateway. The Gateway provides PII redaction (defense in depth), a daily budget cap with alerts, model routing, logging and tracing, and automatic fallback to a second provider on an outage or a price or model change. The final model is chosen after an eval on Arabic and English message samples. The Gateway keeps option C open for later.

## Consequences

- Positive: fast start with no GPU operations; the provider can be swapped without touching agents; cost is capped and visible.
- Negative: dependency on external providers, mitigated by the Gateway and the second-provider fallback.
- Negative: data leaves the company, mitigated because prompts contain listing data only (P3).
- The LLM is never a source of facts; prices and specs come from the Listings DB and are verified before sending (P1).

## Assumptions and revisit triggers

Assumptions: about 2,000 calls per day, Arabic and English, listing-only prompts. Revisit if sensitive data ever needs to enter prompts, if monthly LLM cost makes self-hosting cheaper, if no provider meets the Arabic quality bar, or if the law requires in-country processing. A new model or vendor requires ARB review.
