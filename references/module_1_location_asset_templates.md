# Module 1: Location Asset Templates — Polygon Maps, Travel Blocks & Per-Page Schema

*Source: Sept 24, 2026 Spam Update doctrine (Nico | AI Ranking). Implements Rules 2, 4, 5, and 7 of `../../kirby-local-seo/SKILL.md` Module 7 for the Phase 2 Astro scaffold.*

### **1.1 Purpose & Routing**
Phase 2 scaffolds `/locations/[location].astro` pages; this module supplies the three differentiating components that keep those pages out of doorway classification. Every component is data-driven from per-locality frontmatter — never hand-copied across pages, which is how substance duplication starts. Differentiation substance itself (Rule 1 body content) is authored per page in Phase 2; the swap-test gate at deploy time is `../../kirby-seo-deployment/SKILL.md` Module 19.

### **1.2 Locality Data Model**
One `src/data/locations/[slug].json` per locality (or equivalent Astro content collection with the same schema). Required fields:

```json
{
  "name": "Round Rock",
  "slug": "round-rock",
  "polygonKml": "/kml/round-rock.kml",
  "mapEmbedUrl": "https://www.google.com/maps/embed?pb=...",
  "driveTime": {"minutes": 20, "route": "via I-35", "commitment": "same-day service if called before 2 PM"},
  "neighbors": [{"slug": "georgetown", "note": "15 min north"}],
  "landmarks": ["Dell Diamond", "Old Town"],
  "serviceTypes": ["water-heater-install", "drain-repair"],
  "photos": [{"src": "/img/round-rock-water-heater-install.jpg", "alt": "…describes this actual job photo…"}]
}
```

*   `polygonKml`/`mapEmbedUrl` MUST encode the suburb polygon — never reuse the HQ pin across localities (Rule 4).
*   `neighbors` is capped at 2–4 genuine adjacent localities; `landmarks` at 1–3 (Rule 6/7 limits).

### **1.3 Polygon Map Embed Component**
Lazy-load pattern so the third-party iframe never blocks Astro's static hydration budget:
*   Render a static placeholder (locality outline SVG or a branded frame with the locality name) with `loading="lazy"` on the iframe, or use the facade pattern: placeholder until first `click`/`IntersectionObserver` entry, then swap in the embed.
*   Each locality's placeholder and embed MUST differ (polygon outline + locality label); identical map frames across pages are a Rule 4 failure signal.

### **1.4 Neighbor Travel-Time Block**
For each entry in `neighbors`, render an in-context sentence — not a bare link list: `Also serving Georgetown, 15 min north of Round Rock →` linking to the sibling location page. The block always links upward to the service hub and the parent areas directory once per page (Rule 7 architecture).

### **1.5 Per-Page Service JSON-LD Generator**
A single generator function consumed by every location page — never a copied block:

```astro
<script type="application/ld+json" set:html={serviceJsonLD(loc)} />
```

Generator invariants (Rule 2):
*   `@type: "Service"`, `serviceType` from `serviceTypes` (one `Service` node per page; `serviceType` may be an array when the page covers several), `url` = this page's canonical URL, `areaServed` = this locality only.
*   `provider` references ONE canonical entity: `{"@id": "{site}/#organization"}` — the same `@id` site-wide; never a per-page provider object.
*   Emit ONE `@graph` per page for this generator's output: a `LocalBusiness` node, or the subtype for the trade (`{site}/#organization`, the same canonical entity and `@id` on every page, not a per-page variant) holding the canonical entity (`name`, site-root `url`, `telephone`, `openingHoursSpecification` if the business publishes hours, `address` and `geo` only if it publishes a street address (omit both for a service-area business that hides its address), plus `hasMap` and `sameAs` as set out below; all site-wide constants, never read from per-locality data), and the `Service` node, whose `provider` references that `@id`. Other schema the page needs, such as `FAQPage`, stays in its own block.
*   Never add `AggregateRating`, `Review`, `AggregateOffer` or `Offer` nodes. These are local service pages, so the ecommerce carve-out in `../../kirby-local-seo/SKILL.md` Module 6 §6.10 does not apply (`Offer` markup is for genuine ecommerce offers only, `../../kirby-aiseo-skill/SKILL.md` §2.49). The offer bans are house guardrails, not Google rules. `hasMap` is the Google Maps CID URL (§6.10 item 3), and `sameAs` holds that same URL plus only live, indexed profiles the business owns and can prove it controls.
*   `serviceJsonLD(loc)` returns the `JSON.stringify` output with every `<` replaced by the six characters \u003c, and U+2028 and U+2029 replaced by \u2028 and \u2029 (for example `.replace(/</g, "\\u003c")`). `set:html` skips Astro's escaping, so locality data containing `</script>` would otherwise close the element. This is a JSON string escape applied to the whole serialised string, not HTML `&lt;`.
*   Output MUST validate on validator.schema.org; the deploy gate re-asserts uniqueness across the folder (`../../kirby-seo-deployment/SKILL.md` Module 19 §19.3).

### **1.6 Phase 2 Integration Order & Post-Launch Pruning Governance**
1.  Create the locality data files (one per target area that passes Gate 0 in `../../kirby-local-seo/SKILL.md` Module 7 §7.3 — if you would not dispatch a team there this week, do not scaffold it).
2.  Author per-page substance (housing stock, local water/soil issues, municipal rules, local pricing, permit lead-times) BEFORE wiring components — components evidence differentiation, they cannot substitute for it. Use `<main>`/`<article>` without wrapper-div nesting and name the primary entity and city in the first ~70 words (heuristic only, directional).
3.  Wire the map, travel-block, and JSON-LD components from this module (**no rating, review or offer JSON-LD**, including on third-party/aggregated reviews; schema here is hygiene, not a ranking lever; follow the §1.5 invariants), and verify 1:1 upward linking to the corresponding `src/pages/services/[service].astro` URLs mapped to the business's GBP Primary/Secondary Categories and GBP Services.
4.  Run the swap-test deploy gate (`../../kirby-seo-deployment/SKILL.md` Module 7 §7.3 & Module 19), the **Anti-Rank-and-Rent Gate** (real entity + branded search + authentic job-site photos, zero stock images), and the **Above-the-Fold Viewport-Match Check** (`../../kirby-text-first-cro/SKILL.md` §1.9: booking/calculator, `tel:`, price anchor, and live reviews visible immediately) before publishing the folder.
5.  **Post-Launch 90-Day Zero-Impression Pruning:** At Day 90+ post-indexation, audit location pages in GSC. Avoid churning launched location pages (titles, H1s, slugs) while they accumulate data (directional, house guardrail). Prune pages with zero impressions over a full 90-day trailing window only after clearing all four safety gates (`../../kirby-seo-deployment/SKILL.md` Module 7 §7.12 B): (a) inbound backlink check (`301` redirect linked pages to the nearest intent-parity hub; use `410 Gone` strictly on unlinked zero-impression suburb bloat), (b) trailing 90-day zero GSC impressions & `<90d`/seasonal exemptions (`<90d` new URLs and seasonal services), (c) preserve conversion-enabling MOFU/BOFU pages (pricing, financing, guarantees, `trust.astro`, `facts.astro` SSOT), and (d) active core-update freeze gate (`../../kirby-seo-telemetry/SKILL.md` Module 16 / `../../kirby-seo-deployment/SKILL.md` Module 7 §7.6 — freeze during active rollout + 7-day settle window).

### **1.7 10-Point Service Page Template Checklist & Clean-History / `$4`-Sale Utility Microsite Track (Nov 2026)**
*   **Downstream Astro Service, Location, Flat-Area & Intersection Page Contract (Kasra Dash, Video 12 `HfRNZZIlZ6w`, Nov 2026; `../../kirby-text-first-cro/SKILL.md` §21.1):** Regardless of route shape (`src/pages/services/[service].astro`, `src/pages/locations/[location].astro`, flat `src/pages/[service-area].astro`, or intersection pages), scaffold every commercial page with all 10 points of §21.1: (1) first-screen service + town `<h1>` & 3-to-4-bullet capability/trust block, (2) dual clickable **Call + Book Online** first-screen buttons above the fold, (3) transparent `"From"` pricing floor + next live booking/call-out slot wired from the user's real booking/calendar feed (or omit the slot; never hardcode), (4) crisp `3`-step operational workflow + explicit **What's Included / What Isn't** scope block, (5) intent-split subpage links (`repairs` vs `servicing` vs `new installations`), (6) verified service-specific review proof, currently-held checkable accreditation numbers/registrations (`Gas Safe`, `SRA`, `GDC` where applicable), and authentic job-site photos with verified technician names, (7) interactive **postcode coverage checker / lazy-loaded polygon map** (replacing 50-town footer lists), (8) pre-booking objection FAQ, (9) closing dual **Call / Book Online** CTA band matching the header promise, and (10) service telemetry audit (`../../kirby-seo-telemetry/SKILL.md` Module 5) — strictly preserving `LocalBusiness` hygiene (zero `AggregateRating`, `Review`, `AggregateOffer`, or `Offer` JSON-LD).
*   **Clean-History / `$4`-Sale Utility Microsite Track (single-source directional case metric, not a market guarantee — Edward Sturm Ep. 1,224 / Mike Hawley, `ULvVvU0fWj8`, Nov 10, 2026):** When extending a local launch with standalone free utility calculators or niche microsites (`cheese.help` / `SEO-TV.com` non-spam utility track) instead of main-site clutter, hand off build orchestration, daily Python cron automation, and Cloudflare/SSR templates to [`kirby-linkable-microsite-factory`](../../kirby-linkable-microsite-factory/SKILL.md) (passing `../../kirby-seo-deployment/SKILL.md` Module 7 §7.3 Logo-Swap / standalone entity rules).
