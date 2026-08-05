# Book Now Live Price Estimate — Pre-Handoff QA / Accessibility / Usability Audit

**Date:** 2026-08-05
**Auditor role:** QA (functional, WCAG 2.1 AA, Nielsen heuristics)
**Target:** [`index.html`](../index.html) — the top-level clickable prototype, confirmed current (matches `README.md`; supersedes the two earlier drafts in `prototype/`, last touched 2026-08-03)
**Scope confirmed with requester:** prototype only. The Vue/Nuxt scaffold in `code/` (`PriceEstimate.vue`, `BookingCalendar.vue`, `BookNowModal.vue`, `ServiceOption.vue`) was explicitly excluded from this pass — its markup differs and needs its own audit before build.
**Configurations tested:** Mobile web × Web, crossed with Today (no price) × Proposed (estimate) — 4 configurations, all reachable from the same DOM via the prototype's toggles.

## Method

Static markup/CSS/JS review plus live DOM inspection and interaction testing in a rendered browser (computed styles, computed contrast ratios, focus/tabindex enumeration, and live-clicking through both services, the full calendar, and both states). Not performed: testing with actual assistive technology (a screen reader, switch device), and testing at arbitrary real-device viewport widths — the prototype's "phone" is a fixed 404px-wide decorative frame, not a resizable viewport, so true responsive behavior at e.g. 320–375px is not verifiable from this artifact (see Limitations).

Per the requester's constraint, WCAG 2.1 AA is used as the accessibility bar for this review **as an assumed professional default, not a confirmed org standard** — `docs/project-context.md` (Gap 5) states accessibility requirements are "not defined in source material" for this feature. This should be confirmed with whoever owns accessibility standards at Yourgi before engineering treats these as hard requirements.

---

## Executive summary

The design successfully does what it set out to do for sighted mouse/touch users: the estimate updates immediately and correctly as service and dates change, in both layouts, and it's built entirely from existing visual primitives per `docs/component-inventory.md`'s constraint. The problems are concentrated in two places:

1. **Keyboard and screen-reader access to the entire booking flow is absent**, not just degraded. Of 139 elements inside the booking modal, only **4 are keyboard-focusable** (prev/next month, "Log In," "Send Request"). A keyboard-only user cannot pick a service, pick a date, set a time, add a pet, or even close the dialog.
2. **One formatting bug** (missing thousands separator above \$1,000) undercuts the trust story that motivates this whole feature — see `docs/project-context.md` §6, which already flags early-price trust/liability as a live risk.

| Severity | Functional QA | Accessibility (WCAG 2.1 AA) | Usability (Heuristics) |
|---|---|---|---|
| Blocker | 0 | 1 | 0 |
| Critical | 0 | 2 | 1 |
| Major | 1 | 2 | 1 |
| Minor | 4 | 2 | 3 |
| Positive | — | — | 2 |

---

## 1. Functional QA

| ID | Severity | Area / Config | Finding |
|---|---|---|---|
| F1 | **Major** | Estimate summary — both layouts, Proposed, House Sitting ≥10 nights | Totals ≥ \$1,000 render with no thousands separator, e.g. **`$2310.00 est.`** instead of `$2,310.00 est.`. Root cause: `money = n => '$' + n.toFixed(2)` (`index.html`, script block) has no locale/grouping. Reproduced live: House Sitting, 22 selected nights → `$2310.00 est.`. This reads as a typo on a feature whose whole premise is building price trust. |
| F2 | Minor | Calendar helper line — Web (desktop), any state | The empty-state copy is device-blind: `"Tap the days you need…"` renders identically on desktop/mouse and mobile/touch (one hardcoded string in `render()`). Desktop users click; they don't tap. |
| F3 | Minor | Modal header — both layouts | The **"‹" back chevron** has no click handler at all in either header (`m-head`/`d-mhead`) — confirmed via DOM (`onclick: null`). Whether this is intentionally inert for the prototype or simply unfinished isn't stated anywhere; worth an explicit call-out to engineering so it isn't silently carried forward as "already decided." |
| F4 | Minor | Desktop popup dismissal | The only way to close the desktop popup is the small **"✕"**; clicking the dimmed backdrop does nothing (no handler on `.d-overlay` itself, confirmed in source). See also A1/H4 below — this is also a keyboard-access gap. |
| F5 | Minor | "Send Request" — both layouts | No loading/disabled/confirmation state on click (expected — there's no backend). Flagging only so engineering handoff explicitly owns this state, which is tied to the still-open "binding vs. estimate" question (`project-context.md` OQ 3). |
| F6 | Info | Phone Number / Message fields | Both render as static empty boxes (`<div>`, not `<input>`/`<textarea>`) with no placeholder text — there is nothing to functionally test here (no typing, no validation). Not a defect; a boundary of what this artifact can be QA'd for. |
| F7 | Minor | Mobile layout | The "phone" is a **fixed 404×812 CSS-px decorative frame**, not a resizable viewport — real responsive behavior at common small widths (320–375px) is untested by this prototype. Recommend engineering verify wrapping/overflow of the estimate line at 320px once built in code, since `.est-tot` uses `white-space: nowrap`. |

---

## 2. Accessibility — WCAG 2.1 AA (assumed baseline, see Method)

All findings below were confirmed by direct DOM/CSSOM inspection in a live render, not inferred from markup alone.

| ID | Severity | WCAG 2.1 SC | Finding |
|---|---|---|---|
| A1 | **Blocker** | [2.1.1 Keyboard](https://www.w3.org/WAI/WCAG21/quickref/?showtechniques=211#keyboard) — Level A | Measured directly: `document.querySelectorAll('a[href], button, input, select, textarea, [tabindex]')` inside the booking modal returns **4 elements out of 139 total** — the month-arrows, "Log In," and "Send Request." Both service cards, every calendar day, the "Start Time" control, "+ Add a Pet," the phone field, the message field, and the close control are plain `<div>`/`<span>` with no `tabindex`. A keyboard-only user cannot complete or exit this flow. This is a **Level A** failure (stricter than the AA baseline), so it blocks AA conformance outright regardless of anything else in this report. |
| A2 | **Critical** | [4.1.2 Name, Role, Value](https://www.w3.org/WAI/WCAG21/quickref/?showtechniques=412#name-role-value) — Level A | The same elements from A1 also carry no `role` and no accessible state (no `aria-selected`/`aria-pressed`/`aria-expanded`). Independent of keyboard reach, assistive tech has no way to perceive the service cards, calendar days, "Start Time," or "+ Add a Pet" as interactive controls at all — confirmed `role: null` on each. |
| A3 | **Critical** | 4.1.2 + dialog pattern ([WAI-ARIA APG: Dialog](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)) | The booking popup has no `role="dialog"`, no `aria-modal="true"`, no `aria-labelledby` pointing at its own heading — confirmed via DOM query. The search-results page behind it is not `inert`/`aria-hidden` while the popup is "open." A screen reader user gets no signal that a dialog opened at all. |
| A4 | **Major** | [4.1.3 Status Messages](https://www.w3.org/WAI/WCAG21/quickref/?showtechniques=413#status-messages) — Level AA | The live-updating estimate (`"$131.25 est."`) and the calendar helper line have no `aria-live` region (confirmed `null` on both). This is the feature's entire value proposition — a number that updates as you choose — and it is completely silent for screen reader users. |
| A5 | **Major** | [1.4.3 Contrast (Minimum)](https://www.w3.org/WAI/WCAG21/quickref/?showtechniques=143#contrast-minimum) — Level AA | Measured contrast (WCAG relative-luminance formula) of the standard secondary text color `#757575` against the modal body background `#fcfbf9` (`surface-50`) is **4.46:1** — just under the required 4.5:1 for normal text. Affects the calendar helper line, the message helper text, and "Don't see your pets?" — all sit directly on `surface-50` in the modal body. The *same* color against pure white (e.g., in the sticky footer) measures 4.61:1 and passes — this is specific to the body background, not the color token generally. |
| A6 | Minor | 2.5.5 Target Size (AAA in 2.1 — informational only) | Calendar day hit targets are 34×34 CSS px. **WCAG 2.1 AA itself has no numeric target-size minimum** — 2.5.5 is Level AAA in 2.1 (44×44). It only becomes an AA requirement under **WCAG 2.2**'s SC 2.5.8 (24×24 minimum), which 34px already clears. Noted so it isn't mis-cited as a 2.1 AA violation — the real blocker here is A1 (unreachable), not size. |
| A7 | Minor | [1.1.1 Non-text Content](https://www.w3.org/WAI/WCAG21/quickref/?showtechniques=111#non-text-content) — Level A | Service icons (🐕/🏠) have no `aria-hidden="true"` and aren't coordinated with the adjacent visible label ("Dog Walking"/"House Sitting") — screen readers may announce the emoji's Unicode name redundantly alongside the label. Low severity: no information is lost, just duplicated. |

---

## 3. Usability — Nielsen's 10 heuristics

| ID | Severity | Heuristic | Finding |
|---|---|---|---|
| H1 | **Critical** | [#1 Visibility of System Status](https://www.nngroup.com/articles/ten-usability-heuristics/) | For sighted mouse/touch users the live price update is immediate and clear. For assistive-tech users it's invisible (same root cause as A4) — the feature's core promise fails silently for that audience rather than degrading gracefully. |
| H2 | **Major** | [#3 User Control and Freedom](https://www.nngroup.com/articles/ten-usability-heuristics/) | The desktop popup has exactly one dismiss affordance (a small "✕"; no backdrop-click, no Escape handling in source) — and per A1, keyboard users have **no** way to close it. `docs/component-inventory.md` notes the live site already has a proper "Are you sure you want to exit" confirm-dialog pattern elsewhere; worth checking whether that pattern is meant to gate this modal too, since it isn't modeled here. |
| H3 | Minor | [#2 Match Between System and the Real World](https://www.nngroup.com/articles/ten-usability-heuristics/) | Two instances: unformatted four-digit totals (F1) and touch-specific copy shown on desktop (F2) both work against the input mode/format a user actually expects. |
| H4 | Minor | [#6 Recognition Rather Than Recall](https://www.nngroup.com/articles/ten-usability-heuristics/) | Untested directly (needs the fields to be real inputs), but no error states exist yet to evaluate — flagged as an open item for whenever the fields become functional, not a current defect. |
| H5 | Minor | [#2 Match Between System and the Real World](https://www.nngroup.com/articles/ten-usability-heuristics/) | The total is labeled `"$131.25 est."` — good, it's honestly framed as an estimate — but nothing explains *why* it's an estimate or what could change it. That ambiguity is exactly the trust gap `project-context.md` OQ3 already flags as unresolved (binding vs. estimate); the copy doesn't yet resolve it either way. |
| — | **Positive** | [#4 Consistency and Standards](https://www.nngroup.com/articles/ten-usability-heuristics/) | The estimate reuses the site's existing "Starting at / \$XX.XX" price typography and the modal's existing sticky footer rather than inventing a new pattern — exactly the constraint `docs/component-inventory.md` set, and it shows. |
| — | **Positive** | [#8 Aesthetic and Minimalist Design](https://www.nngroup.com/articles/ten-usability-heuristics/) | No new colored callout, box, or icon was introduced for the estimate — it reads as the existing components "doing more math," not as a bolted-on module. |

---

## 4. Best-practice framing (external research)

Two sources were checked for how the industry treats early price disclosure in booking/checkout flows:

- **Booking UX Best Practices to Boost Conversions in 2025** (ralabs.org): *"A sudden fee at the end can break trust."* Their own case study — adding an expandable price breakdown just before checkout, with the total unchanged — measured a **decrease in drop-offs at the payment step**. This directly supports the ticket's underlying premise: surfacing price earlier is a trust move, not just a convenience one. ([ralabs.org](https://ralabs.org/blog/booking-ux-best-practices/))
- **Seamless Checkout Experience: Best Practices for 2026** (cs-cart.com): recommends surfacing a cost summary (products, fees, taxes) and displaying it *consistently* through the journey, noting that "transparent taxes" and similar detail reduce purchase anxiety. ([cs-cart.com](https://www.cs-cart.com/blog/seamless-checkout/))

Read together with `project-context.md`'s open trust/liability question (OQ3: is the early number binding or an estimate?), the design direction is aligned with the industry pattern — but the payoff depends on resolving that question before build, since both sources tie the trust benefit specifically to the number *not* surprising the user later.

---

## Limitations of this pass

- No test with actual assistive technology (screen reader, switch/voice control) — findings above are derived from DOM/ARIA state and the WCAG success criteria those APIs are built on, which is a reliable proxy but not a substitute for an AT pass.
- No test at real device viewport widths — the mobile frame in this prototype is a fixed-width decorative element, not a resizable viewport.
- Phone Number and Message fields are non-functional placeholders; no input/validation behavior exists yet to test.
- `code/` (Nuxt/Vue implementation scaffold) was explicitly excluded from this pass by request — it has different markup and needs its own audit before it's treated as validated.

## Sources

- [WCAG 2.1 (W3C Recommendation)](https://www.w3.org/TR/WCAG21/) / [Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/)
- [WAI-ARIA Authoring Practices — Dialog (Modal) Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)
- [WCAG 2.2 — SC 2.5.8 Target Size (Minimum)](https://www.w3.org/TR/WCAG22/#target-size-minimum) — cited only to correctly distinguish it from the 2.1 AAA criterion (A6)
- [Nielsen Norman Group — 10 Usability Heuristics for User Interface Design](https://www.nngroup.com/articles/ten-usability-heuristics/)
- [ralabs.org — Booking UX Best Practices to Boost Conversions in 2025](https://ralabs.org/blog/booking-ux-best-practices/)
- [cs-cart.com — Seamless Checkout Experience: Best Practices for 2026](https://www.cs-cart.com/blog/seamless-checkout/)
- `docs/project-context.md`, `docs/component-inventory.md` (internal, this project)
