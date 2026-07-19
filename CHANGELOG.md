# Changelog

## Mobile rebuild + non-template redesign (2026-07-19)

### fix: mobile responsive layout

The site was effectively broken on phones. Root cause: `.hero-text` and
`.hero-diagram` are CSS grid items inside `.hero-inner`, and grid items
default to `min-width: auto`. The gate-log ticker's `width: max-content`
(needed for the marquee effect) has an intrinsic width of ~4000px, and
that blew the grid item out to ~4100px even though the grid track itself
correctly collapsed to a single column on mobile. Everything below the
fold rendered clipped off the right edge of the viewport.

- `min-width: 0` on the hero, about, and contact grid items.
- Real hamburger nav. The previous mobile nav just set
  `display: none` on every link but the CTA, with no way to reach them.
  New drawer is keyboard accessible (Escape closes it, `aria-expanded`
  is kept in sync, links aren't tab-reachable while the drawer is closed).
- The seven-node loop diagram swaps to a simplified vertical stack below
  600px instead of shrinking the SVG until the labels are illegible.
- Tap targets on the scorer options and specimen tabs bumped to >=44px
  through tablet-portrait widths (900px).
- Footer link row now wraps (previously a fixed-width flex row that
  overflowed under ~480px).
- `overflow-x: clip` on `html`/`body` as a backstop, `prefers-reduced-motion`
  now also covers the hero entrance animations, not just the ticker.

**Verification:** zero horizontal overflow and zero <44px tap targets at
320 / 360 / 375 / 390 / 414 / 768 / 1024px, checked via an iframe-based
viewport harness (the browser extension's own window couldn't be resized
below the host display in this environment, so real viewports were
simulated with sized iframes, which get their own CSS containing block
for media queries). All seven sections plus FAQ and Contact confirmed
rendering correctly at every listed breakpoint, including the scorer
interaction and the specimen table's bounded horizontal scroll.

**Lighthouse mobile (375px, throttled):**

| | Before | After |
|---|---|---|
| Performance | 97 | 96 |
| Accessibility | 94 | 97 |
| Best Practices | 100 | 100 |
| SEO | 100 | 100 |
| CLS | 0 | 0.003 |

### feat: non-template redesign

- **Standing Rule is now the visual centerpiece**: a single warm cream
  panel (`#F4EFE6`) breaks up the dark flow, referencing printed
  technical documentation. Oversized Fraunces headline, two words
  italicized in cobalt as the single accent color, generous negative
  space, a small signal-colored mark.
- **Panel system, not cards-with-shadow**: committed to one radius
  (3px) across every panel/button/input. The featured pricing tier lost
  its drop-shadow-and-lift treatment in favor of a top accent line and a
  mono "Most Engagements Start Here" tag, matching the existing
  `about-card-tag` convention instead of inventing a new one.
- **Asymmetry**: a large ghost "02" numeral bleeds off the top-right
  corner of the Problem section, breaking the grid on purpose.
- **Texture**: a faint cobalt hairline grid plus subtle film grain on
  the hero background (no gradient-blob).
- **Type**: a ~1.25 modular scale via CSS custom properties
  (`--step--1` through `--step-5`), applied to the major headings, with
  tighter tracking on large Fraunces display text.
- **Motion**: Lenis + GSAP ScrollTrigger, loaded from CDN, gated to
  `pointer: fine` + standard DPR + no `prefers-reduced-motion` only.
  Touch devices and low-end mobile keep the original lightweight CSS +
  IntersectionObserver reveal instead of paying for scroll-jacking
  libraries — this was a deliberate choice to protect the mobile fix
  above, not an oversight. Adds staggered section reveals, the 01-07
  numerals counting in as the Method section scrolls into view, and a
  cursor-aware glow on the loop diagram. Guarded with `typeof` checks
  throughout, so a CDN failure falls back to the plain reveal system
  rather than breaking the page.
- **Nav**: added "The Rule" (the Standing Rule section existed but had
  no nav entry) and shortened "The Governed Loop" to "The Loop", per
  the brief's own example of operational nav labels.

**Lighthouse mobile (375px, throttled), after the redesign:**

| | Mobile fix only | + Redesign |
|---|---|---|
| Performance | 96 | 94 |
| Accessibility | 97 | 97 |
| Best Practices | 100 | 100 |
| SEO | 100 | 100 |
| CLS | 0.003 | 0.056 |

Performance dipped slightly (three additional CDN script requests) and
CLS rose from 0.003 to 0.056 — still comfortably inside the "Good" Core
Web Vitals band (<0.1), most likely font-swap timing shifting slightly
later with the added requests. Worth a follow-up look, not urgent.

**Verification:** zero horizontal overflow at every breakpoint after the
redesign (re-ran the same audit that caught the original grid bug), no
console errors, Lenis/GSAP/ScrollTrigger confirmed loaded and the
enhanced-motion path confirmed active on a fine-pointer device, real
wheel-scroll interaction confirmed triggering the GSAP reveals correctly
(a programmatic `scrollTo` doesn't route through Lenis's intercepted
scroll handling, which briefly looked like a bug during testing before
that was clear).

### chore: brand name via siteConfig

- Added `siteConfig` (`BRAND_NAME`, `DOMAIN`, `TAGLINE`) near the top of
  the main script, plus a `hydrateBrand()` function that populates every
  `data-brand="name"` / `"wordmark"` / `"domain"` / `"tagline"` element
  on load. Wired into the nav wordmark, footer wordmark, footer
  copyright line, and the body-copy mentions of the company name (hero
  subhead, Standing Rule body, two FAQ answers).
- The wordmark hydration re-derives the two-tone styling (outer words
  plain, middle word(s) in cobalt) from whatever `BRAND_NAME` is set to,
  so a future rename doesn't need the styling touched separately.
- **`<title>`, meta description, Open Graph, Twitter Card, and JSON-LD
  stay static literal text**, each marked with a `<!-- NAME: change in
  siteConfig -->` comment, not JS-templated. Search engine and social
  crawlers read the static HTML response, not JS-hydrated content, so
  templating these client-side would silently break social link
  previews and SEO. This is a real constraint of a build-less static
  site, not a shortcut — true single-sourcing of the meta surfaces would
  need a build step (e.g. a small template + Netlify build command),
  which is a separate, larger change from what was asked for here.
- Added Twitter Card tags, a JSON-LD `ProfessionalService` block, and a
  canonical link tag — none of these existed before, and the brief
  listed Twitter/JSON-LD as surfaces that should reference the brand
  tokens.
- `TAGLINE`'s default ("Governed AI enablement for revenue teams") is
  used in the *new* Twitter description and JSON-LD description fields
  only. The existing, longer meta/OG description was left as-is rather
  than shortened to the placeholder tagline — that would have been a
  real SEO copy change nobody asked for. Swap `TAGLINE` there too if
  that shorter framing is preferred once the name is finalized.

**To rename:** edit the three values at the top of `siteConfig` in
`index.html`, then update the ~8 lines marked `NAME: change in
siteConfig` in `<head>`. Grep for `Houser Intelligence Advisory` to
confirm nothing was missed (back-end/legal LLC references, if any,
are out of scope).
