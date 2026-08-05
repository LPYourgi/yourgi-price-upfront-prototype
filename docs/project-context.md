# Project Context: Move Price Early in Booking Process
**Date:** 2026-08-03
**Source:** A ticket (title + short description) plus three Figma handoff links, and this session's discussion that the work is being approached design-first — a direction-alignment artifact for stakeholder review before engineering handoff.
**Version:** 1.3
**Changelog:**
- 1.3 (2026-08-05) — **Tech stack resolved** (Gap 3). Fingerprinted the live customer portal: **Nuxt 3 · Vue 3.5 · Pinia · Vuetify 3** (SSR; mounts on `#__nuxt`, Pinia in the payload, Vuetify `.v-application` present). An implementation scaffold matching this stack was added under `code/` (Pinia store + pricing util + Vue components for the Book Now modal and fee-inclusive estimate). Design work also moved into code: this project folder now holds the docs, the clickable prototype, and the component scaffold.
- 1.2 (2026-08-03) — Live-site component audit (`docs/component-inventory.md`). Confirmed real pre-fee base rates (Dog Walking **$25.00**, House Sitting **$100.00**) and that a price pattern ("Starting at / $XX.XX") already exists on the provider profile but is absent from the Book Now flow. Constraint added: build the fix only from existing components; the one new concept (a live computed total) has no existing equivalent and is composed from existing primitives.
- 1.1 (2026-08-03) — target surface clarified as the **booking flow that begins after "Book Now"** on a provider (not the search results list): the estimate should start building on the first step of that flow and update as the parent makes selections, rather than appearing only at the final review. Resolves Open Question 1. (A search-list price exploration was built and set aside as the earlier/broader option.)

---

## 1. Problem Statement
Pet parents can only see pricing after working through the entire booking flow, so they cannot factor cost into the choice of provider while they are still choosing. Selecting a pro who is a good match — on price as well as fit — is therefore inefficient, because the price only appears once most of the decision has already been made. Why this is a priority now (a metric that moved, a competitor change, support volume) is not stated in the ticket.

## 2. ICP (Ideal Customer Profile)
**Side:** Pet parents (primary)

Pet parents actively in the process of booking a service (the Figma shows dog walking at $25/walk) who are comparing providers and want price to inform which pro they request. Today they commit to a flow before the number is visible, so they either book without knowing the cost up front or abandon and restart to compare.

**Cross-side impact:** Providers set the rates that would now be surfaced earlier and more prominently (e.g. "$25/walk"). Making price a front-of-flow comparison point could shift which providers receive requests toward the more price-competitive, and create pricing pressure on the supply side. The ticket is silent on any provider-side consequence — that silence is a gap, not an absence (see Gaps 7).

## 3. Pain Points
Reaching pricing currently requires going through the full booking flow — the price/review step sits at the end. A parent comparing two or three pros on cost has to traverse that flow repeatedly, which the ticket calls out as the core inefficiency ("parents have to go through an entire booking flow to get to review pricing"). Additional texture — support tickets, drop-off data, or parent quotes — is not provided in the source.

## 4. Proposed Solution
- Users can see a running fee-inclusive estimate **inside the Book Now / "Contact provider" flow, above the "Send Request" button**, updating as they pick service and dates (modeled as rate × days × 1.05). Today that flow (service, date, time, pets → Send Request) shows no price at any step.
- Users can see a **fee-inclusive** price — the base rate plus the 5% service fee shown as the total, not the pre-fee rate.
- Users can see this price on **mobile web** as a pricing footer positioned above the "Request to Book" button.
- Users can see this price on **desktop** in the booking widget, beside the Book button (e.g. "$25/walk").
- Users can see an estimate that is "as accurate as possible" — but exactly what inputs the estimate is computed from, and whether an exact total (vs. a "from $X" estimate) can be shown at the chosen step, is not defined in the source (see Open Question 2).

## 5. Success Metrics
The ticket frames the goal qualitatively — parents can "more efficiently select a pro who's a great match" — but sets no target, metric definition, or baseline. Success Metrics: Not defined in source material. Plausible axes if defined later would be demand-side (time-to-book, flow completion/drop-off, requests per active parent) and match-quality (rebooking, post-booking satisfaction), but none is stated (see Gaps 1).

## 6. Design Constraints
**Platform:** Both — mobile web and desktop (both appear in the Figma handoff).
**Geography:** Not defined in source material.
**Accessibility:** Not defined in source material.
**Technical:** The displayed price must include the 5% service fee and should be as accurate as the step's inputs allow. Front-end stack is **Nuxt 3 · Vue 3.5 · Pinia · Vuetify 3** (resolved 2026-08-05; see Gap 3 and `code/`). The estimate's computation — which variables feed it and which service produces the fee-inclusive number — is still Not defined in source material.
**Brand:** Follows the `yourgi-brand` skill (available in this session). Any surfaced copy or price formatting defers to it.
**Trust & liability:** Surfacing a price early sets an expectation before checkout. If the early estimate can differ from the final charged amount, that gap is a trust risk; whether the early number is binding or an estimate that may change is undetermined and needs a decision before design. This project does not appear to touch the Yourgi Guarantee, provider vetting, or pet-safety/incident handling based on the input.
**Other:** Must match the Figma handoff for placement, layout, and styling:
- Mobile web — pricing footer above "Request to Book": https://www.figma.com/design/EeAyvbn1bogb50v2u70DjQ/Handoff---Yourgi-Customer-Portal?node-id=18594-34189
- Footer close-up: https://www.figma.com/design/EeAyvbn1bogb50v2u70DjQ/Handoff---Yourgi-Customer-Portal?node-id=18594-34578
- Desktop — booking widget with $25/walk by the Book button: https://www.figma.com/design/EeAyvbn1bogb50v2u70DjQ/Handoff---Yourgi-Customer-Portal?node-id=17998-85781

## 7. Open Questions
1. **How early is "as early as possible"? — RESOLVED (2026-08-03):** the estimate surfaces inside the Book Now flow (the "Contact provider" modal), as a fee-inclusive footer above "Send Request," building live as the parent selects service and dates. This is the Figma "pricing footer above Request to Book." A search-list price was explored and set aside as a broader, separate option.
2. **What feeds the estimate, and can it be exact?** Which inputs drive the price (walk duration, number of walks, number of pets, dates, add-ons), and can a precise fee-inclusive total be shown before those are chosen, or should it be a "from $X" estimate? The ticket asks for maximum accuracy but doesn't define the calculation.
3. **Is the early price binding or an estimate?** Does the surfaced number commit to the final charge, or is it explicitly an estimate subject to change at checkout? This drives both copy and trust exposure.

## 8. Gaps
1. **Success Metrics** — no target, definition, or baseline. Matters because whether this is a quality-of-life fix or a conversion priority changes the scope worth investing. Ask the PM who owns booking conversion.
2. **Estimate computation & data source** — the variables and the service that produces the fee-inclusive price aren't specified. Matters because "the more accurate the better" can't be designed without knowing what's knowable at each step. Ask engineering / the PM.
3. **Tech stack** — RESOLVED (2026-08-05): the customer portal is **Nuxt 3 · Vue 3.5 · Pinia · Vuetify 3** (fingerprinted from the live app). A matching implementation scaffold lives in `code/`. Still open: the actual repo location and how this feature slots into it (routing, where provider rates are fetched). Confirm with engineering.
4. **Geography** — launch markets / regional restrictions not stated. Owner unknown.
5. **Accessibility** — no stated requirements for the pricing display (an interactive/priced element). Owner unknown.
6. **Why now** — no trigger given for prioritizing this. Matters for sequencing and metric choice. Ask the ticket author / PM.
7. **Provider-side impact** — the effect of front-of-flow price comparison on which providers get requests and on pricing pressure isn't addressed. Matters because a demand-side change with a real supply-side consequence is usually under-specified. Owner unknown.

---
*Generated by yourgi-project-context. Update as decisions are made and questions are resolved.*
