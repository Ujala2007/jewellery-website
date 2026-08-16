# Ujala — Premium Upgrade Roadmap
### Turning the jewellery board into a world-class luxury e-commerce experience

This is a full audit + build plan for taking Ujala from a beautiful catalog board to a trustworthy, high-converting luxury commerce site — the way Tanishq, Cartier, or Mejuri approach it. Everything is grouped into **what to build, why it matters for a jewellery buyer specifically, and how hard it is to ship**, so you can sequence the work sensibly instead of trying to do all of it at once.

A note on scope before the roadmap: items in Sections 1–2 (real login, real payments, order emails) need a **backend and a database** — no static frontend, including the current single-file demo, can do these for real. I've marked those clearly. Everything else can be built client-side today, and I've implemented a working version of most of it in the updated demo alongside this doc.

---

## Priority tiers

| Tier | Meaning | Timeframe |
|---|---|---|
| **P0 — Trust foundation** | Without these, visitors don't believe the site is real. Do first. | Week 1–2 |
| **P1 — Conversion drivers** | Directly move people from "browsing" to "buying." | Week 3–5 |
| **P2 — Retention & delight** | Bring people back, increase order value. | Week 6–8 |
| **P3 — Signature luxury touches** | What separates "nice store" from "brand." | Ongoing |

---

## 1. Authentication & User Accounts

**Needs a backend:** account creation, password hashing, email verification, session/JWT tokens, OAuth handshakes with Google/Apple/Facebook, order history pulled from a database. A frontend can only render the UI for these — it can't authenticate anyone safely on its own.

| Feature | Tier | Notes |
|---|---|---|
| Login / Signup modal (not a separate page) | P0 | Luxury sites keep you in-context — a full page redirect feels like a downgrade. Use a slide-in panel or centered modal over a blurred backdrop. |
| Social sign-in (Google/Apple/Facebook) | P1 | Apple Sign-In is close to mandatory if you ever ship iOS. Use official button specs — don't reskin the "G" logo. |
| Forgot password + email verification | P0 | Table stakes for trust. Use a magic-link flow if you want to skip password UX entirely — many jewellery DTC brands do this now. |
| Profile dashboard | P1 | Saved addresses, saved sizes (ring/bracelet), order history, wishlist, saved payment methods. |
| Order tracking | P1 | Even a simple 4-stage tracker (Placed → Crafted/Packed → Shipped → Delivered) reduces support tickets massively for jewellery, where delivery anxiety is high. |
| Wishlist synced to account | P0 | Already built client-side in the demo (in-memory). Needs a `wishlists` table keyed to user id to persist across devices. |

**Backend shape (minimal viable):**
```
users (id, email, password_hash, name, created_at)
sessions (id, user_id, token, expires_at)
addresses (id, user_id, line1, city, state, pincode, is_default)
wishlists (user_id, product_id)
orders (id, user_id, status, total, created_at)
order_items (order_id, product_id, qty, price, engraving_text, size)
```
Auth-as-a-service options that skip you writing this from scratch: **Clerk**, **Supabase Auth**, or **Firebase Auth** — all support Google/Apple/Facebook out of the box and issue a session you can trust in ~a day of integration, versus weeks of hand-rolled auth.

---

## 2. Payment System

**Needs a backend + a payment processor.** Never handle raw card numbers on your own server — use **Razorpay** or **Stripe** (Razorpay is the standard for India: native UPI, cards, wallets, Buy-Now-Pay-Later via Simpl/LazyPay). The frontend collects intent, the processor's hosted checkout or SDK handles the sensitive part, your backend just confirms the payment and creates the order.

| Feature | Tier | Notes |
|---|---|---|
| Secure checkout flow (cart → address → payment → confirm) | P0 | 3–4 steps max. Every extra step loses buyers, especially at jewellery price points where people are already nervous. |
| Multiple payment methods (Card, UPI, PayPal, Apple/Google Pay) | P0 | Razorpay Checkout gives you all Indian methods in one integration. For international, add PayPal + Stripe. |
| Coupon / discount codes | P1 | Server-side validation only — never trust a discount % sent from the client. |
| Buy Now, Pay Later | P2 | Meaningful for bridal sets at higher price points. Razorpay Affirm/Simpl or Klarna internationally. |
| Saved payment methods | P2 | Processors tokenize this for you (Stripe Customer objects, Razorpay saved cards) — you never store card data yourself. |
| Success / failure pages | P0 | Failure page should explain *why* in plain language and offer a retry — not a dead end. |
| Order confirmation emails | P0 | Transactional email via **Resend**, **Postmark**, or **SendGrid**, triggered by your backend on payment webhook success. |

**What I built in the demo instead:** a fully designed, front-end-only checkout flow — cart drawer, address step, payment method selector with the right method-specific fields, coupon field with a live demo code (`UJALA10`), and a confirmation screen — so you can see and reuse the *exact UI* the moment you wire it to Razorpay/Stripe. It doesn't move real money.

---

## 3. Interactive User Experience

All fully buildable client-side — and now partly built in the demo.

| Feature | Tier | Status in demo |
|---|---|---|
| Micro-interactions (hover states, button presses, card lifts) | P0 | ✅ Built |
| Product image zoom on hover | P1 | ✅ Built (cursor-follow zoom on quick-view) |
| 360° product view | P2 | Needs 24–36 sequential product photos per SKU, swapped on drag. Expensive to shoot but very high-impact for rings specifically, where profile matters. |
| Live search with suggestions | P1 | ✅ Built — typing filters a dropdown of matching pieces by name/category in real time |
| Smart filtering & sorting | P1 | ✅ Built — category filter + price sort |
| Recently viewed | P2 | ✅ Built — a shelf that fills as you open product quick-views |
| AI product recommendations | P2 | Real version needs purchase/view history + a similarity model (even a simple "same category, nearby price" heuristic beats nothing). Demo shows a static "You may also like" using category-matching as a stand-in. |

---

## 4. Trust & Conversion Features

| Feature | Tier | Notes |
|---|---|---|
| Reviews & ratings | P0 | Needs a backend table (`reviews: product_id, user_id, rating, text, verified`). Show rating average on cards, not just detail pages. |
| Verified buyer badge | P0 | Only show on reviews linked to an actual `order_items` row — this is what makes it meaningful instead of decorative. |
| Payment trust badges | P0 | ✅ Built — "100% Secure Checkout," "BIS Hallmark Certified," "7-Day Easy Returns" strip. Real badges only once you actually have SSL + a payment processor + a return policy to back them. |
| Returns & refund policy page | P0 | ✅ Section built. Keep it in plain language — jewellery buyers specifically worry about "what if it doesn't fit" or "what if the stone looks different in person." Answer both explicitly. |
| FAQ | P0 | ✅ Built as an accordion covering sizing, authenticity, shipping, returns, care. |
| Testimonials | P1 | ✅ Built — carousel using a bridal portrait + quote format. |
| Live chat | P1 | Use **Crisp**, **Tidio**, or **Intercom** — a few lines of embed script, no backend needed to start. |

---

## 5. Luxury Jewellery-Specific Features

| Feature | Tier | Notes |
|---|---|---|
| Ring size guide / calculator | P0 | ✅ Built — printable string-measurement chart + a size converter table (India/US/UK). This single feature measurably cuts returns for ring purchases. |
| Jewellery care guide | P1 | ✅ Built as a content section — how to store kundan/polki, avoid perfume contact, cleaning dos and don'ts. |
| Gift packaging | P1 | ✅ Built — toggle in the demo checkout with a preview of the box. |
| Custom engraving | P2 | ✅ Built — text input with live character-count on a ring product's quick-view, capped at a realistic engraving length. |
| Personalized jewellery builder | P3 | The most expensive to build well (needs a config UI + inventory logic for each stone/metal combination). Worth doing only once the catalog and checkout are solid — it's a retention/differentiation feature, not a launch requirement. |
| Virtual try-on (camera) | P3 | Realistically needs a hand/ear/neck landmark model (MediaPipe Hands/FaceMesh in-browser) plus product overlays positioned per SKU. High "wow" factor, high production cost per product — sequence this after the catalog has real revenue. |

---

## 6. Mobile Experience

| Feature | Tier | Notes |
|---|---|---|
| Fully responsive layout | P0 | ✅ Built across the whole demo — grid collapses 3→2→1 column, nav becomes a slide-out drawer. |
| Sticky "Add to Cart" bar on mobile product view | P0 | ✅ Built — appears once you scroll past the main product image. |
| Mobile-first nav | P0 | ✅ Built. |
| Performance (image sizes, lazy loading) | P0 | Use `loading="lazy"` on all below-the-fold images (done in demo) and serve real product photography as WebP at 2–3 sizes (thumbnail/detail/zoom) via a CDN like Cloudinary or an `<img srcset>` — this alone is often the biggest speed win on jewellery sites, which tend to be image-heavy. |

---

## 7. Design Improvements

| Element | Recommendation |
|---|---|
| **Palette** | Keep the maroon/gold/ivory/emerald system already established — it's specific to Ujala and reads as Indian bridal jewellery rather than generic luxury. Resist the urge to switch to black-and-white minimalism; that's a Western fine-jewellery convention, not yours. |
| **Typography** | Cormorant Garamond (display) + Jost (UI/body) is a good pairing — keep it. Add a numeric/tabular font (or `font-variant-numeric: tabular-nums`) for prices so they align cleanly in cart/checkout tables. |
| **Glassmorphism** | Use sparingly — a frosted panel behind the login modal and the mobile nav works; don't apply it to product cards, where it competes with the jewellery photography for attention. |
| **Hero section** | ✅ Improved in demo with a signature animated diya + gold sheen headline — keep this as the one "bold" moment per the design-restraint principle, and keep everything else (cards, checkout) quiet by comparison. |
| **Product cards** | ✅ Rebuilt — image-first, shimmer sweep on hover, quick-view + wishlist actions surfaced without needing a click into detail. |
| **Product detail page** | Needs: multiple angle thumbnails, zoom, size/metal selectors, delivery estimate by pincode, "complete the look" cross-sell, reviews block. Demo's quick-view modal is a compressed version of this — worth expanding into a full page once you have multi-angle photography per SKU. |

---

## Suggested build sequence

1. **Weeks 1–2:** Auth (via Clerk/Supabase) + payment integration (Razorpay/Stripe) + trust badges/FAQ/returns copy. This is the unglamorous but non-negotiable foundation.
2. **Weeks 3–5:** Full checkout flow wired to real payment, order confirmation emails, reviews system, live chat embed.
3. **Weeks 6–8:** Recommendations, recently viewed persistence (needs accounts), gift packaging + engraving wired to real order data, ring size guide refinement.
4. **Ongoing:** 360° photography rollout per SKU, virtual try-on, personalization builder — sequence these after the store is actually taking orders, since they're the most expensive to build and least urgent to prove the business works.

---

## What's already built vs. what needs a backend

**Built and working in the updated demo (`ujala.html`):** hero + signature moment, live search, category filter + sort, quick-view with hover-zoom, wishlist, recently viewed, cart drawer with quantity/coupon, full checkout UI (address → payment method UI → gift/engraving → confirmation), login/signup modal UI, ring size guide, trust badges, testimonials, FAQ accordion, sticky mobile add-to-cart, responsive layout throughout.

**Needs a real backend to function, not just look right:** actual authentication, actual payment processing, real order storage/history/tracking, real email sending, real reviews, real recommendations, persistent wishlist across devices/sessions, live chat backend, virtual try-on model, personalization inventory logic.

If you want, I can help you pick a stack for the backend half (Supabase is a strong default — Postgres + auth + storage in one place, pairs well with Razorpay for India) and scaffold the actual API routes next.
