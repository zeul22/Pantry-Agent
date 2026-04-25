# RationPilot

**An AI copilot for reordering your monthly household ration on Swiggy Instamart — with one sentence.**

> Built for the Swiggy Builders Club MCP program.

---

## The problem

Every month, most Indian households reorder roughly the same 25–35 grocery staples: atta, rice, dal, oil, sugar, tea, toothpaste, detergent, milk, eggs, and so on. On Instamart, rebuilding that cart takes 15–20 minutes of scrolling, searching, and tapping — every single month, forever.

It's the same cart. It's almost always the same brands. And yet there's no "just reorder my monthly ration" button, because what counts as a "ration" is different for every household and changes slowly over time.

## The solution

RationPilot is an AI copilot that:

1. **Learns** your recurring grocery patterns from your Swiggy Instamart order history.
2. **Understands** what's a staple vs. a one-off (your monthly atta is different from the cake you ordered for a birthday).
3. **Rebuilds** your monthly cart on demand when you say *"order my monthly ration."*
4. **Shows** you the cart before anything is purchased — edit, remove, swap — and only checks out after you confirm.

The product is a thin conversational layer over Swiggy Instamart, not a replacement for it. Swiggy fulfils the order, Swiggy owns the checkout, Swiggy owns the user relationship. RationPilot just removes the repetitive work of rebuilding the same cart every month.

## How it works

```
  User                       RationPilot                     Swiggy MCP
   │                              │                              │
   │   "Order my monthly ration"  │                              │
   ├─────────────────────────────▶│                              │
   │                              │  Fetch 4–6 mo order history  │
   │                              ├─────────────────────────────▶│
   │                              │◀─────────────────────────────┤
   │                              │                              │
   │                              │  Score items by frequency    │
   │                              │  × recency → identify        │
   │                              │  staples, group by category  │
   │                              │                              │
   │                              │  Check current stock/prices  │
   │                              ├─────────────────────────────▶│
   │                              │◀─────────────────────────────┤
   │                              │                              │
   │   Cart preview (₹3,240)      │                              │
   │◀─────────────────────────────┤                              │
   │                              │                              │
   │   "Remove the sugar,         │                              │
   │    confirm the rest"         │                              │
   ├─────────────────────────────▶│                              │
   │                              │  Submit checkout             │
   │                              ├─────────────────────────────▶│
   │   Order placed ✓             │◀─────────────────────────────┤
   │◀─────────────────────────────┤                              │
```

## Staple detection (the interesting part)

Not everything a user has ordered is a staple. The heuristic:

```
score(item) = frequency(last_6_months) × recency_weight(last_order)
             × category_boost(pantry/dairy/cleaning/personal_care)
```

An item qualifies as a staple if it's been ordered **3+ times in the last 4 months** and falls into a household-goods category. Impulse buys (ice cream, one-off snacks, party items) get filtered out. Seasonal items (mangoes, specific festival goods) are flagged as "optional additions" rather than defaults.

Users can manually promote or demote items — those preferences persist.

## Architecture

| Layer | Choice |
|---|---|
| Agent orchestration | Anthropic Claude via MCP client (Python SDK) |
| Auth | OAuth 2.0 against Swiggy Instamart endpoints |
| Secrets | AWS Secrets Manager |
| Data store | PostgreSQL on RDS (ap-south-1) |
| Compute | AWS Fargate, stateless |
| Egress | NAT gateway with static IP (shareable for whitelisting) |
| Conversational surface | Web app + WhatsApp (Phase 2) |

All user data stays in ap-south-1. Order history is cached only as long as needed to compute staples and is re-fetched on each session.

## Guardrails

The Swiggy Builders guidelines are explicit about what's allowed, and RationPilot is designed to stay well inside those lines:

- **No autonomous ordering.** Every cart is shown to the user for confirmation before checkout. The agent is a copilot, not an autopilot.
- **No hidden branding.** The UI clearly shows items are being ordered on Swiggy Instamart. Prices, availability, and delivery times come directly from Swiggy and are displayed as-is.
- **No aggregation.** RationPilot only works with Swiggy. It does not compare against competing platforms or act as a meta-layer.
- **No data harvesting.** Only the authenticated user's own order history is read. No scraping, no competitor benchmarking, no extraction beyond the MCP API surface.
- **Rate limit respect.** Caching and batching at the application layer to minimize API calls.
- **User data boundaries.** Order data is treated under Swiggy's platform terms and never shared with third parties.

## Roadmap

**Phase 1 — Sandbox** (first 4 weeks after access)
Read-only order history analysis. Staple scoring. Cart preview UI. No checkout.

**Phase 2 — Guided checkout** (weeks 5–8)
Full cart → confirm → checkout flow with explicit user confirmation.

**Phase 3 — Smart substitutions** (weeks 9–12)
User-defined rules for out-of-stock handling ("swap brand, keep category" vs. "skip it").

**Phase 4 — Household mode** (post-launch)
Multiple users sharing a single ration list, role-based approval for orders.

## Current status

Early stage. Core scoring logic is drafted and tested locally against synthetic order data. Conversational flow is prototyped against a mocked MCP server. Production build begins on receiving sandbox access.

## Why this fits the Builders Club

Swiggy's program explicitly encourages *"AI-powered assistants and copilots that use MCP to automate commerce workflows"* and *"integrations that make ordering better for users."* RationPilot is exactly that — a workflow automation that reduces a painful, repetitive task while keeping Swiggy's brand, experience, and relationship with the user fully intact.

## Team

*Solo developer / small team — update with your details.*

- **Name:** Rahul Anand
- **Background:** Solo Developer with experience in Flutter, React, Node, Java & Python

## Contact

anandrahul044@gmail.com ·

---

*Applying to Swiggy Builders Club, [month] 2026. Happy to walk through the design, share the prototype, or answer any questions during the review.*