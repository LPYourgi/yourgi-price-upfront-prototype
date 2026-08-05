# Component Inventory — Yourgi customer web (for the price-upfront fix)

**Date:** 2026-08-03
**Method:** Combed the live customer site (`yourgi.com/app/...`): search results, the Book Now / "Contact provider" modal, and the provider profile (About / Reviews / Services / Location).
**Purpose:** Build the price-estimate fix from components that already exist. Introduce nothing new unless it genuinely doesn't exist.

---

## Existing components found (reuse these)

**Price display — `Starting at` / `$XX.XX`.** Lives on the provider profile Services tab. Small gray "Starting at" label above a large price. This is the price typography to reuse for the estimate — do not invent a new price style. Real base rates seen: Dog Walking **$25.00**, House Sitting **$100.00**. (These are pre-fee.)

**Buttons.** Three pill variants already exist:
- Black-filled primary — "Book" (profile widget), "Search Pet Care" nav.
- Gold-filled — "Send Request" (booking modal), "Cancel" (exit dialog).
- Outlined / ghost — "Book Now" (search rows), "Yes, Exit" (exit dialog).
Note: primary is black in some places, gold in others — a pre-existing inconsistency, not something this fix should resolve. Reuse the gold "Send Request" as-is.

**Chips / badges.** Outlined tier chip (Yourgi Pro / Pet Resort / Vet + Pet Resort); "Accepts" chips with icon (Dogs, Cats); light-blue info chip ("Flexible Cancelation").

**Modal.** Centered sheet with header (back chevron, title, paw watermark, ✕), scrolling body, and a **sticky footer that already holds the primary button + fine print**. The estimate goes in this existing footer — no new container needed.

**Calendar.** Month grid with prev/next arrows and a gold-filled selected-day state (in the booking modal).

**Form fields.** Dropdown field ("Start Time"), add-row field ("+ Add a Pet").

**Cards.** Search result row; service card (icon + name + Accepts chips + description + "Starting at" price + attribute rows + illustration); right-rail booking-widget card (name, location, rating, services, Book, Flexible Cancelation).

**Rating.** "5.0" + blue filled stars + "N reviews" link.

**Illustrations.** Hand-drawn dog / house marks, on-brand — reuse, don't commission new.

**Confirmation dialog.** "Are you sure you want to exit…" with Yes, Exit / Cancel.

---

## Where price exists today vs. where it's missing

Price (base "Starting at $X") exists **only on the provider profile Services tab**. It is absent from the search rows and — the point of this ticket — from the **entire Book Now modal** (service, date, time, pets → Send Request). So the parent has to leave the booking flow to learn cost.

The fix moves the *already-existing* price pattern into the booking flow and makes it fee-inclusive and quantity-aware.

---

## The only genuinely new element

There is **no existing "computed total / order summary" component anywhere on the site** — every price shown is a static per-service "Starting at $X." So a **live estimate (service × quantity + 5% fee = total)** is a new *concept*.

Constraint honored: it is built entirely from existing primitives — the "Starting at / $XX.XX" price type, the existing modal footer, a plain divider, and body text. No new colored callout box, no new button, no new iconography. Visually it should read as the existing price component doing a little more math, not as a new module.

---

## Flags for the build
- **Rates are pre-fee** on the Services tab; the fix adds 5% (× 1.05).
- **Add-Ons** exist as a service category on the profile; whether they feed the estimate is undetermined (see project-context.md, OQ 2).
- **Button-color inconsistency** (black vs gold primary) predates this work — reuse "Send Request" gold; don't try to fix the system here.

*Generated from a live site audit. Pairs with docs/project-context.md.*
