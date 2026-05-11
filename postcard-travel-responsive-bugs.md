# Postcard.travel — Responsive UI Bug Report

**Date:** 2026-03-12
**Tested URL:** https://www.postcard.travel/
**Breakpoints tested:** 768, 1024, 1280, 1366, 1440, 1536, 1920
**Tool:** Playwright automated testing + visual verification

---

## Who should fix what?

**This is entirely a developer CSS fix.** The design intent is clear — the implementation just doesn't hold up across viewports. Layout decisions for the two ambiguous cases are included below with reasoning.

---

## P0 — Critical

### 1. Cards overflow the viewport at every breakpoint

**What's happening:** Country cards, region cards, theme cards, and diary cards all extend past the right edge of the screen. At 768px, cards overflow by up to 324px. Even at 1920px, they overflow by 71px.

**Affected elements:**
- `.no-scrollbar.css-lw55d0` (flex container, 6 children)
- `.no-scrollbar.css-nlc0sf` (flex container, 6 children)
- `.chakra-stack.css-f6hy0w` (flex container, 3 children)

**Numbers:**

| Breakpoint | Cards overflowing |
|---|---|
| 768px | 70 |
| 1024px | 70 |
| 1280px | 56 |
| 1440px | 56 |
| 1920px | 56 |

**Dev fix:** The scroll containers need `overflow-x: auto` or `scroll` with proper containment. Cards use a fixed ~384px width — either make them responsive with `min-width` + `flex-shrink`, or ensure the parent clips properly and provides a visible scroll affordance (scrollbar or swipe indicator).

---

## P1 — High

### 2. Section headings wrap to 2–3 lines at every breakpoint

**What's happening:** Headings like "Discover Postcard Experiences by popular regions in INDIA." wrap to 3 lines even on 1920px desktop. The text container is too narrow (337–506px) while the font keeps scaling up via `vw` units.

**Affected headings (all breakpoints):**

| Text | Lines at 1366px | Lines at 1920px |
|---|---|---|
| "Discover Postcard Experiences by country." | 2 | 2 |
| "Discover Postcard Experiences by popular regions in INDIA." | 3 | 3 |
| "Discover Postcard Experiences by popular travel themes." | 3 | 3 |
| "The Postcard Travel Library!" | 2 | 2 |
| "Postcard Travel Diary. It's Free!" | 2 | 2 |

**Root cause:** The orange/coral info card on the left has a constrained width. The heading font size scales with `vw` but the container width doesn't grow proportionally. Result: text wraps no matter the screen size.

**Dev fix (two-part):**

1. **Font sizing:** Switch font-size from raw `vw` to `clamp()` — e.g., `clamp(20px, 2.5vw, 36px)`. This prevents the text from outgrowing its container at any viewport.

2. **Layout at ≤1024px — stack the info card above the cards row.**

   **Decision:** At tablet (768–1024px), the orange/coral info card should **stack on top** (full-width) with the horizontal card scroll row below it.

   **Reasoning:** The side-by-side layout allocates only ~200px to the info card at 768px — not enough for any heading to fit on one line regardless of font size. Stacking gives the heading the full viewport width, which eliminates wrapping entirely. This is also the standard responsive pattern users expect (side-by-side on desktop → stacked on tablet/mobile). The card row below benefits too — it gets the full viewport width to scroll through instead of being squeezed next to the info card.

   **Implementation:** Add a media query at `max-width: 1024px` to change the parent flex container from `flex-direction: row` to `flex-direction: column`, and set the info card to `width: 100%`.

---

### 3. Nav bar items wrap to 2 lines at ≤1280px

**What's happening:** At 1280x800 (common laptop resolution, including Lenovo Yoga), the nav items "Signature Experiences", "StarPartner Stays", and "Travel Concierge" each break to 2 lines. The nav bar was clearly designed for single-line items.

**Dev fix (two-tier approach):**

**Decision:** Use **smaller text at 1024–1280px**, then **collapse to hamburger at ≤1024px**.

**Reasoning:**
- A hamburger at 1280px is too aggressive — that's a standard laptop size and hiding nav items behind a menu hurts discoverability on a site where users need to browse categories (Experiences, StarPartner, Concierge). Users on laptops expect visible navigation.
- Truncation is a bad fit here — "Signature Exp..." or "StarPartner S..." loses meaning. These aren't generic labels that users can guess from truncation.
- Reducing font-size from 21px → 14–15px at 1024–1280px keeps all items visible and readable. The items are short enough to fit at 15px.
- At ≤1024px (tablet), there genuinely isn't enough horizontal room even with smaller text. A hamburger is the right call here — it's the expected pattern on tablets.

**Implementation:**
```css
/* Laptop — tighter nav */
@media (max-width: 1280px) {
  .nav-link { font-size: 14px; white-space: nowrap; }
}

/* Tablet — hamburger */
@media (max-width: 1024px) {
  .nav-links { display: none; }
  .hamburger-menu { display: flex; }
}
```

---

### 4. Zero-size images (invisible) at every breakpoint

**What's happening:** At least 10 images render at 0×0 pixels across all breakpoints. They're in the DOM but invisible.

**Affected images:**
- `p_stamp.png`
- `p_stamp_trans.png` (transparent stamp)
- Multiple images from `images.postcard.travel` CDN

**Dev fix:** Check if these are meant to be displayed. If yes, fix the CSS sizing. If decorative/hidden, remove from DOM or add `aria-hidden="true"` and `display: none` to avoid unnecessary network requests.

---

## P2 — Medium

### 5. Button text wraps at all breakpoints

**What's happening:** Several buttons have text breaking to 2+ lines because their containers are too narrow for the scaled font size.

| Button | Expected height | Actual height (1366px) |
|---|---|---|
| "Try Postcard Search" | ~17px | 42px |
| "SIGN UP!" | ~15px | 34px |
| "Subscribe to the Newsletter" | ~26px | 47px |
| "Become a Member" | ~19px (at 1024px) | 40px |

**Dev fix:** Add `white-space: nowrap` to all button text. If the button becomes too wide, use `clamp()` on font-size or set `min-width` on the button container.

---

### 6. Marquee/ticker lacks overflow containment

**What's happening:** The `.rfm-marquee` animated ticker extends past the viewport edge at every breakpoint. The container doesn't consistently apply `overflow: hidden`, which can cause horizontal scrollbar flicker on some browsers.

**Dev fix:** Add `overflow: hidden` to `.rfm-marquee-container` (the outermost marquee wrapper).

---

### 7. "Sign In" button too small at 768px

**What's happening:** At 768px tablet, the "Sign In" button is 40×14px — the tap target is well below the 44px minimum recommended by WCAG.

**Dev fix:** Set `min-height: 44px; min-width: 44px` on all interactive elements, or increase the clickable area with padding.

---

### 8. "Try Postcard Search" CTA missing at 768px

**What's happening:** This button exists at 1024px+ but disappears at 768px. If it's a key action, it should be accessible on smaller tablets too.

**Dev fix:** Check the media query that hides it. Either lower the breakpoint threshold or place it elsewhere in the mobile layout.

---

## P3 — Low

### 9. H1 font size uncapped — 76.8px at 1920px

**What's happening:** The hero heading uses ~4vw, which results in 76.8px text at 1920px and 30.72px at 768px. No min/max cap.

**Dev fix:** `font-size: clamp(28px, 4vw, 64px);`

---

### 10. "POSTCARD TRAVEL CLUB" footer link wraps to 3.3 lines at every breakpoint

**What's happening:** This link in the footer wraps awkwardly at every screen size tested.

**Dev fix:** Widen the container or reduce font-size for this element.

---

### 11. Multiple redundant fixed-position elements

**What's happening:** 5 elements with `position: fixed; z-index: 10` (`.css-1wrg53t`) are in the DOM at every breakpoint. If these are alternate states (modals, popups), they should be conditionally rendered.

**Dev fix:** Audit these elements. If they're hidden UI states, render them only when active to reduce DOM weight.

---

## Root Cause

Most of these issues stem from one underlying problem: **`vw`-based font sizing without container-width coordination**. Font sizes scale linearly with viewport, but the containers holding the text don't scale at the same rate. This causes text to wrap at breakpoints where the ratio tips over.

**The single highest-impact fix** is switching all `vw` font declarations to `clamp()` with sensible min/max values. This alone would resolve bugs 2, 5, 9, and 10, and reduce the severity of bug 3.

---

## Screenshots

All screenshots in `./screenshots/`:
- `{breakpoint}-full.png` — full page
- `{breakpoint}-viewport.png` — above the fold
- `{breakpoint}-mid.png` — card sections
- `{breakpoint}-bottom.png` — footer area
- `{breakpoint}-hero-wrap.png` — hero text wrapping
- `{breakpoint}-sections-wrap.png` — section heading wrapping

Breakpoints captured: `tablet-768`, `tablet-1024`, `laptop-1280`, `laptop-1440`, `desktop-1920`, `yoga-1280x800`, `yoga-1366x768`, `yoga-1536x864`
