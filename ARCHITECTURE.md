# Pigment Data Room — Architecture & Editorial Decisions

This is the durable reference for *why* the site is structured the way it is — not chat history, not tribal knowledge. If a future session (human or Claude) needs to understand a decision without re-deriving it, it should be here.

## Site structure: 4 categories, Oliva-parity granularity

Nav is **Hair / Skin / Aesthetics / Body & Wellness** — matching competitor Oliva Clinic's structure, chosen deliberately over the site's original flat Hair/Skin-only nav because the clinic's own Treatment Audit confirmed real offerings (Anti-Aging injectables, Body & Wellness) that had no nav home at all under the old structure.

Full granularity was chosen over a consolidated structure — 64 treatment records, most as standalone pages — per explicit owner decision (see `git log` around "full Oliva-parity structure" commits). The rule for standalone-vs-bundled: a treatment gets its own URL only if people search it by that specific name *and* there's enough unique content to fill a page without repeating its parent; otherwise it's a labeled section within the parent page.

## Standalone-URL vs merged: the actual decisions

These aren't arbitrary — each merge/revert below was a specific, reasoned call. Check `TREATMENTS` in `index.html` for the live `bundledInto`/`note` fields, but the *why* is here:

- **QR678 Hair Treatment** — reverted to standalone after being initially merged into Exosome Hair Treatment (the audit data first said "replaced by Dutexome," but the owner corrected this: both are real, current, separate offerings — not a replacement relationship).
- **Revlite Laser → merged into Laser Toning** — owner's own audit answer: "Have Tribeam (equivalent) - merge with Laser Toning." Same device, same protocol, explicit instruction.
- **Hollywood Facial → merged into Carbon Laser Facial** — both run on the same Tribeam Q-Switch device with an identical protocol. Kept as one page with "also known as Hollywood Facial" built into the copy, specifically to avoid keyword cannibalization — two near-identical thin pages competing for the same search term rank worse than one strong page.
- **Inch Loss Treatment & Express Weight Loss → merged into Medical Weight Loss** — same reasoning: marketing-angle variants of one consultation-led service, not genuinely distinct search terms.
- **GLP-1 Weight Loss — kept standalone**, deliberately not merged with the above — this one *is* a real, currently high-demand, mechanistically distinct search term, unlike the other three.
- **CoolSculpting & generic Medical Weight Loss** — briefly marked "not offered" by an early audit pass, then corrected by the owner: both are real, doctor-credentialed offerings. If either shows as unconfirmed again, that's a regression, not a fresh finding.

## Content rules (apply to every page, every draft, forever)

1. **Never fabricate.** No invented machine/device names, session counts, pricing, testimonials, doctor quotes, or clinical statistics. Where the real answer isn't known, flag it inline ("MACHINE NAME NEEDED FROM CLINIC") rather than guess.
2. **Real machine/delivery-method data exists** — pulled directly from the clinic's own Treatment Audit answers (Tribeam Q-Switch Nd:YAG, Fraxis Duo, Alma Soprano Ice Platinum, Ultracel Q+ HiFu, Conmed Hyfrecator, HydraFacial Allegro, Allergan injectables, Dutexome). Use these, don't re-invent.
3. **Never use a competitor's trademarked program name.** Oliva's "InstaGlow," "Glowtox," "GeneIQ" are off-limits — generic renames exist for each (Signature Glow Facial, Skin-Boosting Micro-Botox Treatment, etc.) in the `TREATMENTS` notes.
4. **Format is short and bulleted, not essay-style** — the owner rejected long-form blog-style treatment copy explicitly and early. The live `/hair-fall/` page is the format reference: short intro, bulleted signs/causes, labeled technique subsections, FAQ accordion. Blog posts are allowed to be longer-form; treatment pages are not.
5. **Images: real, verified URLs only**, prioritizing Indian/South Asian representation in patient-facing shots — and it's fine to honestly flag when no good match exists (several pages do this rather than force a bad fit) rather than misrepresent a stock photo.

## FAQ structure & interlinking

Each FAQ is tagged to exactly one page (`tag` field: `treatment:<id>` / `doctor:<id>` / `location:<id>` / empty for general) via `buildTagSelect`. The Data Room's FAQ tab groups by that tag.

**The linking pattern that avoids feeling robotic** (this was a direct owner concern, resolved deliberately): on the live FAQ hub, group FAQs under their page's heading with **one link to the full page per group**, not a repeated "see our page on X" after every single answer. On the treatment page itself, embed that treatment's own FAQs directly — no link needed, it's already in context. Individual answers get an inline link only when the text naturally references something else, not mechanically.

Each FAQ has a `crossLinks` field (which other FAQs/pages it should reference, with reasoning) and a `linkingVerified` checkbox — separate from the main content-verification checkbox — so linking decisions get their own explicit sign-off rather than being bundled into general content review.

**Do not re-add the old copy-pasted FAQ set.** The `faqs` collection used to have a load-time fallback that silently re-seeded the original 20 scraped-from-the-live-site questions any time fewer than 20 were present — this actively undid the independent FAQ rebuild once already. That fallback has been removed from `index.html`; don't reintroduce anything like it.

## Blog interlinking

`blogPosts` has a `doctorIds` array (zero, one, or more doctors — empty means the clinic's own voice, populated means that doctor's byline) sourced live from the `DOCTORS` list, and an `internalLinksText` field holding the reasoned link plan (which treatments/locations/FAQs/related posts, and why) for whoever builds the real page.

## Google Reviews / "Customer Speak"

Real, verbatim reviews only — sourced directly from the clinic's actual Google listing (Search's knowledge-panel review modal works far better than the Maps place page, which hits a hard pagination wall around 9 reviews; sorting by "Newest" and scrolling deep is what actually surfaces more). Never via the Places API, never paraphrased.

Standing curation rules from the owner: **5-star only** (deliberate choice for the public-facing site, not a completeness requirement); **one clean personal name per review** — exclude business-account-style names (e.g. "Ma Laxmi Aaya Centre") and don't publish a review under a name that's actually clinic staff (a "Malina Barman" review was excluded because "Malina" is referenced as a staff member/CRM across other genuine reviews — that's an insider post, not an independent patient).

Known real gap, not a sourcing failure: **zero real 5-star reviews exist for Body & Wellness or Aesthetics** (Botox, fillers, HIFU, weight loss) after multiple deep passes through the full review history. This clinic's public review footprint is genuinely Skin/Hair-dominated. Don't force a tag to fill that gap.

## Publishing status — read this before assuming anything is live

**The Data Room is a staging/CMS tool. Nothing in it is automatically on pigmentclinics.in.** The live WordPress site still runs the old ~9-page flat structure. None of the 55 new treatment pages, the new 4-category nav, the FAQ rebuild, the Google Reviews section, or the blog interlinking exist on the real site yet. Building that — new WordPress/Elementor pages, the new nav, a "Customer Speak" template that doesn't exist as a feature yet — is the single largest remaining body of work in this project, and it hasn't been started.

## SEO auditing

A repeatable audit checklist exists as a Claude Code skill (`~/.claude/skills/pigment-seo-audit/`, user-level — available in any session via `/pigment-seo-audit`). It covers 9 categories: image SEO, PageSpeed/Core Web Vitals, on-page content, technical SEO/schema, local SEO, keyword strategy, backlinks, analytics, accessibility. Intended to run against the real site once it's live, not the current placeholder state.

## Known open gaps (not yet resolved, don't assume they're fixed)

- **Neither doctor has a photo on file** in the `doctors` collection, despite the real photos existing on the live `/about-us/` page (confirmed via screenshot: Dr. Namitha's photo is genuinely there). Never resolved to a definitive doctor→photo mapping — flagged twice in this project, still open.
- **Before/after gallery has only 2 real pairs** against 64 treatment pages. Cannot be fabricated — needs real, consented patient photos from the clinic.
- **Google Reviews has zero coverage for Body & Wellness / Aesthetics** (see above) — a real content gap, not a task left undone.
- **7 treatments still marked "Not offered"** in the live Audit as of last check: Hair Transplant, Hair Thread Treatment, DermaFrac, Fire & Ice Facial, Foaming Enzyme Facial, HydraGeneo Facial, plus whichever else — check the Audit tab for the current live list, this drifts as the owner answers more.
- **Verification status:** as of this writing, almost nothing has been formally Verified in the Data Room (5 of 76 blog posts; 0 elsewhere) even though content is drafted and real. Verification is real review work, not a formality — don't treat a high draft-count as equivalent to launch-readiness.

## Image SEO — required before any page goes live

Every image currently in the Data Room is a **hotlinked Pexels URL**, not self-hosted. This was the right call for fast, honest drafting (real, verified photos, zero fabrication risk) but is **not launch-ready**. Before publishing, every image needs: (1) downloading and re-hosting on the clinic's own domain/media library — hotlinking ties page speed to Pexels' server and gives Pexels the Google Images search credit, not the clinic; (2) a descriptive, keyword-relevant filename (not `pexels-photo-19239114.jpeg`); (3) real alt text naming the treatment/context. This is explicit, standing owner instruction — "naming of the images also has to be done by you before it goes live on WordPress." Not yet started as of this writing.

Recommended tooling for ongoing compression once past this one-time re-hosting pass: a WordPress plugin (ShortPixel or Imagify) rather than a third-party image CDN service (ImageKit/Cloudinary/imgix) — the site's scale (a few hundred images, Bangalore-local traffic) doesn't justify a dedicated image CDN's cost/complexity.

## Publishing pipeline — the plan going forward

Decided approach: **one real Elementor template first, then scale**, not 55 individually-built pages. The owner will grant **WordPress REST API access** (an Application Password, generated from WP Admin → Users → Profile → Application Passwords) so content can be pushed programmatically and reliably, rather than by clicking through Elementor's visual builder 55 times — the same "script it, don't click it" pattern already used for all the Firestore migrations in this project.

Sequencing: (1) design and get sign-off on one treatment-page template — content structure is already proven (Meta description / Intro / Signs / Causes / Treatment options / What to expect / FAQ / Internal links / Images), the open question is visual layout; (2) build that template in Elementor by hand once, so it exists as a real, reusable page structure; (3) use the REST API to populate new pages against that template; (4) supervised rollout, not fully autonomous bulk publishing — this is live, public, customer-facing content, a different risk tier from everything built in the Data Room sandbox so far.

Design direction default (pending explicit confirmation): **extend the existing Pigment brand** (cream/gold palette, "Glowing Confidence" identity) rather than a full visual overhaul — the brand already has equity, and consistency across 55+ near-identical page types matters more than novelty per-page.

## Pre-publish QA gate

Decided approach: a real **dev/staging environment** (a WordPress host's staging-site feature, or equivalent) separate from the live `pigmentclinics.in`, with nothing going to prod without passing a check first. The check is a Claude Code skill, `~/.claude/skills/pigment-pre-publish-check/` — invoke via `/pigment-pre-publish-check` — covering four things: rules compliance against this file, the SEO checklist (delegates to `/pigment-seo-audit`, doesn't duplicate it), a functional check (links/images/forms/CTAs actually work), and a **regression check** comparing dev against the current live version to catch anything that used to work and silently broke. Regression is the check most teams skip and the one that matters most given how interlinked this site is — always the first section of the report, "none found" is a fine and expected outcome, don't pad it. The dev/staging URL itself doesn't exist yet as of this writing — needs to be set up (most WP hosts have a one-click staging feature) before this skill can actually run.
