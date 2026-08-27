---
stepsCompleted: [1, 2, 3]
inputDocuments:
  - docs/prds/prd-stellar-intents-gateway-2026-08-25/prd.md
  - docs/prds/prd-stellar-intents-gateway-2026-08-25/addendum.md
  - docs/architecture.md
project_name: 'stellar-intents-gateway'
user_name: 'JafetCHVDev'
date: '2026-08-26'
---

# UX Design Specification Zephyroute

**Author:** JafetCHVDev
**Date:** 2026-08-26

---

<!-- UX design content will be appended sequentially through collaborative workflow steps -->

## Executive Summary

### Project Vision

Zephyroute lets someone holding stablecoins or BTC on Ethereum, Arbitrum, or Bitcoin put that capital to work earning yield on Stellar — in two signatures, without learning Stellar's account model, without a bridge the product operates itself, and without the product ever touching their funds. It fuses two already-live, already-audited systems (NEAR Intents for cross-chain acquisition, DeFindex for yield) that nobody has chained together as a single flow before. The product's job is to be the thinnest possible layer between them.

### Target Users

**Primary persona — The Cross-Chain Yield Seeker:** a self-custody crypto user, moderately technical (already operates a wallet, has done a cross-chain swap before), holding idle USDC or BTC on Ethereum, Arbitrum, or Bitcoin. Knows Stellar has cheap, audited yield infrastructure but has no reason to learn its account/trustline model until the two-signature version exists.

Two sub-profiles need materially different UX depth within the same flow:
- **Returning Stellar user** (UJ-1) — already has a funded G-account and trustline; wants the fastest possible path from quote to earning.
- **Brand-new-to-Stellar user** (UJ-2) — needs trustlines, reserves, and the account model itself hidden entirely, onboarding through an embedded wallet (pending Open Question 1) or a documented manual fallback.

**Explicit non-users (v1):** people with no crypto on any chain yet (needs a fiat ramp, deferred); regulated/institutional capital needing compliance guarantees this flow doesn't provide; anyone wanting yield-strategy choice beyond DeFindex's existing vault menu.

`[ASSUMPTION — to confirm during Visual Foundation/Responsive steps]` Device and usage-pattern context wasn't fully pinned down with the user at this step. Working assumption, consistent with the product's dual distribution surface (FR-12/FR-13): design **responsive-first, not desktop-first** — the standalone app must work fully on mobile web, since the embeddable widget will very plausibly run inside a partner's mobile app (e.g. THORWallet). Usage pattern is assumed to be primarily a deliberate, one-off action per session ("move capital to yield now"), with a lighter secondary "check my position" pattern layered on top via FR-9's resumable state — not a habitual daily-check product.

### Key Design Challenges

- **Signature-window countdown UX** — the ~1–5 minute Soroban authorization window (FR-8) is a hard NFR, not a nice-to-have; expiry must never surface as an opaque error, only as a graceful, automatic rebuild.
- **Variable settlement wait** — 40s on Arbitrum vs. ~14 min on Bitcoin (§E, addendum); the user must stay informed without the product feeling stalled, especially on the BTC route.
- **Two onboarding depths, one flow** — hiding trustlines/reserves entirely for UJ-2 while never over-explaining to UJ-1.
- **One experience, two containers** — the same UX must hold up as a full standalone app and as a narrow, partner-branded embedded widget (THORWallet).
- **Cross-session, cross-device resumability** (UJ-3) — a user who settled funds but never deposited must be able to resume from any device, any wallet, at any later time, with zero funds ever having left their control in the meantime.

### Design Opportunities

- Surfacing the quote (fee, ETA, slippage) **verbatim, never editorialized** is already a functional requirement (FR-1) — this can double as a genuine trust differentiator against competitors that abstract these numbers away.
- Framing flow status as a narrative (quoted → submitted → settled → depositing → earning) turns a multi-minute technical wait into a guided journey instead of a black box — a direct differentiator against swap-only aggregators like WOWMAX that stop at the bridge step.
- The moment the dfToken balance updates is the product's real emotional payoff ("you're now earning yield on Stellar") — worth designing as its own confirming beat, not just another line of data on screen.

## Core User Experience

### Defining Experience

The single most important interaction isn't only the second signature (the deposit), though that one stays the most technically fragile because of the ~1-5 minute Soroban authorization window. The first signature (confirming the swap) is just as critical from a trust standpoint: it's the moment the user releases funds into a cross-chain flow that hasn't proven anything to them yet. If we nail one interaction, it's the full chain from "confirm swap" to "see funds arrive" to "deposit signed": each link needs its own level of confidence, not just the last one.

### Platform Strategy

Web, multi-container: standalone app plus an embedded widget inside partner apps (THORWallet, mobile and web). No native app (explicit non-goal, §6). No offline functionality needed, since every step depends on live chain state. Device capability to leverage: browser wallet-extension detection/connection on desktop (Freighter via Stellar Wallets Kit) and embedded-wallet flows on mobile, inside the partner's app.

### Effortless Interactions

- Reconnecting after closing the browser (UJ-3) should feel like nothing happened: same address, resumes exactly where it left off.
- The transition from "settlement detected" to "deposit ready to sign" happens automatically (Horizon polling): the unsigned XDR simply appears.
- For the returning user (UJ-1): quote, sign, sign should feel fast, with no extra exposition. The literal "two taps" framing is dropped here; the goal is to feel like two steps, not to literally be two steps.
- Trustline validation (FR-3) is the real fork point between the returning-user path (UJ-1) and the new-user path (UJ-2), and should be designed as a visible, explicit moment, not an invisible check that happens before the quote screen appears. A new user needs to see why they're being routed differently, not just end up on a different path.
- Eliminates the manual hunt for a yield destination that aggregators like WOWMAX leave the user to do.

### Critical Success Moments

- Confidence in the first signature. The moment a user confirms the swap needs its own clear success signal (not just a spinner): it's the first real vote of confidence they give the product.
- The settlement wait as its own moment, not only a technical challenge. For a first-time user (UJ-2), the 40 seconds to 14 minutes between confirming and seeing funds arrive is the highest-anxiety point in the whole flow, higher than the Soroban signing window, because it's the first time they're watching something cross-chain actually work. It needs active reassurance (live status, not a generic progress bar), handled separately from the "variable settlement wait" design challenge already listed above.
- "This is better": watching the deposit XDR appear automatically the instant funds land.
- Feels successful: the dfToken balance updates and is reflected as "earning."
- The interaction that would ruin it if it fails: the signature window expiring as a dead-end error instead of a graceful rebuild.
- First-time success: UJ-2 completes the whole loop without ever needing to understand what a trustline is.

### Experience Principles

1. Never let a technical constraint look like a product failure.
2. Show, don't abstract.
3. One flow, two depths, with the fork (trustline) designed explicitly, not hidden.
4. State lives on-chain, not in a session.
5. Container fidelity: inside a partner's widget (THORWallet), the experience should feel native to that app, not like an iframe bolted on top; the partner's brand and visual rhythm take priority over Zephyroute's own identity in that context.
