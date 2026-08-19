---
name: wordpress-webdesign
description: Dale's build rules for his WordPress sites (NextSchool, ThinkGen, Ai4Kids and future sites). Use whenever the task involves WordPress, themes, pages, blocks, patterns, CSS, site content, migrations, forms, WooCommerce, or WordPress Studio — or when Dale mentions "site", "homepage", "theme", "블로그", "홈페이지", even without saying "WordPress".
---

# WordPress Build Rules

Read first: `Assistant/work/Webdesign/Webdesign.md` (project state) and the site's `01_skills/` files (palette, schema).

## Architecture

- Block-native only: all content = Gutenberg blocks/patterns, editable in wp-admin. Never hardcode content, text, or images in theme templates.
- Theme = structure + styling only. Block theme (FSE), `theme.json` first.
- Every section type = registered pattern. Repeated content = synced pattern. Structured repeating data (courses, testimonials, team) = custom post type + Query Loop.
- No Elementor/Divi. If extra blocks needed: GenerateBlocks or Kadence.
- Code → GitHub (one repo per site, code only). Content → database, edited on live after launch. Never edit content in two environments.

## Design

- Apple-like: calm, spacious, smooth. Glass = block style with `backdrop-filter` + solid fallback.
- Motion CSS-first: scroll-driven animations, View Transitions, Interactivity API. Respect `prefers-reduced-motion`. No JS animation libraries unless CSS can't.
- 3D/WebGL: isolated lazy-loaded custom block, max one per page.
- Core Web Vitals green on mobile = hard requirement. Images WebP + compressed + alt text. Video always external embed (YouTube/Vimeo/Bunny), never in media library.
- Design tokens in `theme.json`; identical preset slugs across all sites; child theme changes tokens only.

## CSS

- `theme.json` for everything it can express. Custom properties for tweakables. Mobile-first, `clamp()` type. No CSS frameworks.

## SEO / GEO — every page

- Rank Math + per-page title/meta. schema.org per content type (use shared template `Webdesign/01_skills/json-ld-schema.md`; never duplicate Rank Math's schema output).
- `llms.txt`, XML sitemap, hreflang on ko/en pairs.
- Question-style H2/H3, direct answer in first sentences, FAQ where natural.

## Forms / PIPA

- Forms → WordPress → webhook → Airtable. Airtable = back office only, never queried at page render.
- Required consent (수집·이용) + separate marketing opt-in + 개인정보처리방침 link. Collect minimal fields.
- Airtable records: 수집일 + 보유기한 fields. Disclose 국외 이전 (Airtable, US). Under-14 users → guardian consent in the form flow.

## Workflow

- Local dev: WordPress Studio on local disk (not NAS) → GitHub PR → live. Migrate via WP-CLI (DB) + rsync (media); never browser-upload archives.
- All new/changed content = draft until Dale approves. Bulk content via REST API + application password.
