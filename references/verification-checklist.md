# Verification checklist (Phase H)

Work through this live, in a browser, on the actual draft preview (or the
confirmed target theme). Mark each item pass / fail / not-applicable-with-a-
reason in `BUILD-STATUS.md`. A fail routes back to the owning phase for a
real fix (don't patch ad hoc here), then recheck.

## Navigation
- [ ] Desktop: every menu item resolves to a real, finished page — no 404s,
      no stub pages.
- [ ] Mobile: menu opens, every item resolves the same as desktop.
- [ ] Footer links all resolve.

## Product pages
- [ ] Desktop: all product images present; gallery thumbnails switch the
      main image on click; arrows (if enabled) advance/retreat correctly.
- [ ] Mobile: gallery works via swipe/tap as configured; no layout overflow.
- [ ] Every variant combination is selectable and updates price/image where
      applicable.
- [ ] Any quantity-tier/bundle pricing shown on the PDP matches what lands
      in the cart (see Discounts below).

## Add to cart / cart
- [ ] Add to cart succeeds from the PDP and from any homepage buy-box.
- [ ] Cart quantity updates (+/-, remove) recalculate the total correctly.
- [ ] Any auto-preselected cross-sell/upsell items (e.g. shipping
      protection, skip-the-line) actually appear preselected in the cart on
      a fresh add-to-cart — not just toggleable.
- [ ] Removing an upsell item removes its line item and its cost from the
      total.

## Discounts / bundles
Displayed offer text (a PDP badge, a quantity-selector label) is never
sufficient evidence on its own for any item below — each needs the backend
checked directly.
- [ ] Each bundle's component product/variant IDs and quantities, read from
      the actual discount/bundle configuration in admin, match what
      `STORE-BRIEF.md` specified.
- [ ] Inventory tracking and quantity for each bundle component is correct
      (not oversold, decrements as expected on a test add-to-cart).
- [ ] Each discount's eligibility conditions (quantity threshold, which
      variants it's scoped to, automatic vs. code-based), read from the
      discount's actual admin configuration, match the brief.
- [ ] Each discount tier's displayed price (PDP badge, quantity selector
      copy) matches the subtotal shown in the cart.
- [ ] The cart subtotal matches what checkout shows before shipping/tax.
- [ ] Combining a bundle discount with an upsell item doesn't miscalculate
      either amount.
- [ ] A quantity below any discount threshold does *not* get the discount,
      and a quantity at/above combines correctly with any other active
      discount per the brief's stacking rules (verify the boundary, not
      just the "happy path" quantity).

## Shipping / markets
- [ ] Checkout's country/region list includes every destination
      `STORE-BRIEF.md` requires — confirm via a DOM/JS read of the `<select>`
      options, not a screenshot of the closed dropdown.
- [ ] Each required destination shows at least one valid shipping rate at
      checkout (not just "Active" in Markets settings).
- [ ] Shipping costs shown match what was configured, per zone.

## Checkout
- [ ] Checkout loads without error for a domestic address.
- [ ] Checkout loads without error for an international address (if
      required by the brief).
- [ ] Available payment methods match what's actually been activated — no
      dead/unconfigured payment buttons shown to real customers.
- [ ] If no real payment provider is active: run a full order through
      Shopify's Bogus Gateway test mode and confirm it completes and the
      order appears correctly in admin (correct items, prices, shipping).
- [ ] If a real payment provider is active: verify everything up to (not
      including) submitting real payment; log in `BUILD-STATUS.md` that
      final live-payment checkout remains the user's own action.

## Responsive / cross-cutting
- [ ] Full homepage scroll-through on both a desktop and a mobile viewport
      width — no overlapping elements, no horizontal scroll, no cut-off text.
- [ ] Any sticky/floating elements (announcement bar, sticky add-to-cart,
      cart drawer) behave correctly on both.

## Content accuracy
- [ ] No placeholder/lorem-ipsum text remains anywhere in scope.
- [ ] No invented claims (reviews, guarantees, specs, shipping promises)
      that aren't sourced from `STORE-BRIEF.md` or explicitly flagged to the
      user as needing their input.
