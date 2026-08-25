---
name: wordpress-webdesign
description: Dale's build rules for his WordPress sites (NextSchool, ThinkGen, Ai4Kids and future sites). Use whenever the task involves WordPress, themes, pages, blocks, patterns, CSS, site content, migrations, forms, WooCommerce, or WordPress Studio — or when Dale mentions "site", "homepage", "theme", "블로그", "홈페이지", even without saying "WordPress".
---

# WordPress Build Rules (v2)

Read first: `Assistant/work/Webdesign/Webdesign.md` (project state) and the
site's `01_skills/` files (palette, schema).

## Architecture

- Block-native only: all content = Gutenberg blocks/patterns, editable in wp-admin. Never hardcode content, text, or images in theme templates.
- Theme = structure + styling only. Block theme (FSE), `theme.json` first.
- Every section type = registered pattern. Repeated content = synced pattern. Structured repeating data (courses, testimonials, team) = custom post type + Query Loop.
- No Elementor/Divi. If extra blocks needed: GenerateBlocks or Kadence.
- Code → GitHub (one repo per site, code only). Content → database, edited on live after launch. Never edit content in two environments.
- Plugin/theme source of truth: `C:\Claude\Assistant\work\Webdesign\plugins\` — zips are disposable build outputs.

## Design system

- Glass design language: **flat matte glass, NO gradients** (Dale's standing rejection). Tokens: glass bg white 7% (strong 12%), 1px flat border white 14%, blur 16 saturate 140%; ink canvas #0f4f60 → #0c3f4d wash; one mint #30fcb8 accent moment per view. Dropdowns/overlay panels: ink glass ~94% opacity so they read over anything.
- Type scale = apple.com, exactly, in the PARENT theme (v0.7.0+) so all sites inherit: H1 80/64/48 · H2 48/40/32 · H3 32/28/24 · H4 28/24/21 · H5 24/21/19 · H6 21/19/17, all weight 600; body 17/1.47; lead 21; caption 12. Breakpoints >1068 / ≤1068 / ≤734. Children re-map COMPONENT sizes onto this scale but never change the element scale. Visual size and heading level are independent — demote levels for outline cleanliness, restyle size in CSS.
- Apple interaction patterns: primary action = filled pill, secondary = text link with `›` chevron, no hover underline unless asked. Labels as small mint chips. Quiet Apple-style light footer (12px, tight rows, hairline dividers).
- Korean: `word-break: keep-all` on all display text. "학교" only for accredited schools — use "스쿨" otherwise (무경계 스쿨).

## Reference → Design Spec (new)

When Dale supplies a reference (screenshot, URL, video), extract a spec BEFORE
building — never build from a vibe:

1. Read actual values from the reference: fetch its CSS (browser CSSOM walk for
   same-origin sheets) or measure the screenshot (type sizes, spacing, radii,
   colors, shadows). Never guess "Apple-ish" numbers — we cloned apple.com's
   real tokens this way.
2. Write the extraction as a Design Spec: token table + component mapping +
   what we deliberately do differently (our palette, our constraints).
3. In a build-graph run the spec goes INTO the work order (versioned, binding).
   In live review, state the spec in chat before the first edit.
4. Translate to block-native: spec values land in `theme.json` tokens or child
   `style.css` — never inline styles, never a parallel CSS system.

## Motion (policy opened 2026-08-25)

Motion is a standard tool, adapted from Emil Kowalski's skills (installable
originals: github.com/emilkowalski/skills). Hard rules:

- Vanilla CSS transitions/animations (+ Interactivity API where warranted). No Framer Motion, no JS animation libraries unless CSS provably can't.
- Should it animate at all? Constant-use UI (menus, toggles used dozens of times) → near-imperceptible or nothing. Occasional (modals, drawers, toasts) → standard. Rare (onboarding, success) → the delight budget lives here.
- Easing: entrances/exits `ease-out`; on-screen movement `ease-in-out`; hover/color `ease`; constant motion `linear`. **Never `ease-in` on UI.** Use strong curves: `--ease-out: cubic-bezier(0.23,1,0.32,1)`, `--ease-in-out: cubic-bezier(0.77,0,0.175,1)`, drawer `cubic-bezier(0.32,0.72,0,1)`.
- Durations: button feedback 100–160ms, tooltips 125–200, dropdowns 150–250, modals/drawers 200–500. UI stays under 300ms; marketing sections may run longer.
- Transitions, not keyframes, for anything a user can trigger twice in a second. Exit the way it entered.
- `prefers-reduced-motion` ships WITH the animation, never as a follow-up. Scroll-driven effects must not break reading or CWV; no scroll-jacking.

## CSS

- `theme.json` for everything it can express. Custom properties for tweakables. Mobile-first, `clamp()` type. No CSS frameworks.

## SEO / GEO — every page

- Rank Math + per-page title/meta. schema.org per content type (use shared template `Webdesign/01_skills/json-ld-schema.md`; never duplicate Rank Math's schema output).
- `llms.txt`, XML sitemap, hreflang on ko/en pairs.
- Question-style H2/H3, direct answer in first sentences, FAQ where natural.
- Heading hierarchy is structural: H1 once, sections H2, cards H3. Size tweaks never change levels.

## Forms / PIPA

- Forms → WordPress → webhook → Airtable. Airtable = back office only, never queried at page render.
- Required consent (수집·이용) + separate marketing opt-in + 개인정보처리방침 link. Collect minimal fields.
- Airtable records: 수집일 + 보유기한 fields. Disclose 국외 이전 (Airtable, US). Under-14 users → guardian consent in the form flow.

## Workflow

- Deploys: A4K Deploy plugin — commit to `C:\WebPage\<repo>` → touch `C:\Claude\_setup\sync-now` (watcher pushes) → POST `/wp-json/a4k-deploy/v1/deploy` {repo, branch} → clear WP Super Cache → verify live bytes/DOM. NEVER wp-admin "Upload Theme"; SSH is retired.
- All new/changed visual work passes the **a4k-design-critic** gate (85/100, 3 rounds) BEFORE Dale sees screenshots.
- All new/changed content = draft until Dale approves. Bulk content via REST API + application password.
- Local dev: WordPress Studio on local disk (not NAS) → GitHub PR → live. Migrate via WP-CLI (DB) + rsync (media); never browser-upload archives.
