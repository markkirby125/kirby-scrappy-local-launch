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
*   `@type: "Service"`, `serviceType` from `serviceTypes`, `url` = this page's canonical URL, `areaServed` = this locality only.
*   `provider` references ONE canonical entity: `{"@id": "{site}/#organization"}` — the same `@id` site-wide; never a per-page provider object.
*   Output MUST validate on validator.schema.org; the deploy gate re-asserts uniqueness across the folder (`../../kirby-seo-deployment/SKILL.md` Module 19 §19.3).

### **1.6 Phase 2 Integration Order**
1.  Create the locality data files (one per target area that passes Gate 0 in `../../kirby-local-seo/SKILL.md` Module 7 §7.3 — if you would not dispatch a team there this week, do not scaffold it).
2.  Author per-page substance (housing stock, local water/soil issues, municipal rules) BEFORE wiring components — components evidence differentiation, they cannot substitute for it.
3.  Wire the map, travel-block, and JSON-LD components from this module.
4.  Run the swap-test deploy gate before publishing the folder.
