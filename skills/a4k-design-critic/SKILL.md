---
name: a4k-design-critic
description: Score and gate visual work on Dale's A4K sites (NextSchool, ThinkGen, Ai4Kids) before Dale sees it. Use after building or changing any visible section, page, or component — in a build-graph run (UI lane stage 3) or in live section-by-section review — and before sending Dale any screenshot of new visual work. Also use when Dale asks "score this", "design check", "디자인 점수", or "why does this look off".
---

# A4K Design Critic

Internal QA gate. Work that has not cleared this gate does not reach Dale.
The critic never redesigns — it scores against the spec, names defects,
and the builder fixes them. House rules always beat generic taste.

## Procedure

1. Screenshot the section at three widths: desktop (>1068px), tablet (≤1068px),
   mobile (≤734px) — Apple's breakpoints, which the themes use.
2. Score against the rubric below. Evidence per deduction: what, where, how many
   points, the concrete fix.
3. Builder applies fixes. Re-screenshot, re-score.
4. **Gate: 85/100. Maximum 3 rounds.** Score ≥85 → pass onward (integrate /
   show Dale). Three rounds under 85 → STOP: the spec is the problem, not the
   CSS. Escalate to Dale as a HITL question with the last scorecard.
5. Report the scorecard in chat with the screenshot, and (in a build-graph run)
   paste the summary line into the run's Decision notes so the wp-admin twin
   shows it. Log critic tokens as their own Lane work row: `UI · critic`.

## Rubric — 100 points

**Layout & spacing — 20.** Grid alignment; equivalent rows align vertically
across sibling cards; breathing room between bands (no section touching the
next); nothing overflows horizontally; consistent gaps (multiples of the
spacing scale, no eyeballed values); safe padding at every breakpoint.

**Typography & hierarchy — 20.** Apple type scale exactly (parent theme
v0.7.0+): H1 80/64/48 · H2 48/40/32 · H3 32/28/24 · H4 28/24/21 · H5 24/21/19
· H6 21/19/17, weight 600, body 17/1.47, lead 21, caption 12. Heading LEVELS
form a clean outline (no skips introduced); visual size may deviate only via a
documented component mapping; Korean text `word-break: keep-all`, never broken
mid-word.

**Color & contrast — 15.** Text contrast ≥4.5:1 (large text ≥3:1); **one mint
accent moment per view** — count them; **no gradients anywhere** (Dale's
standing rejection); only palette tokens, no invented hexes outside the
documented set; quiet areas (footer) stay quiet.

**Glass language — 15.** Flat matte glass only: `--ns-glass-bg` white 7% /
strong 12%, 1px flat border white 14%, blur 16 saturate 140%; ink canvas
#0f4f60→#0c3f4d; radius scale consistent (18px cards / 16px panels / 9999
pills); glass readable over its actual background (screenshot judges, not
theory); dropdowns/overlays opaque enough to read (~94% ink).

**Motion — 10.** Policy: motion is allowed (Dale, 2026-08-25) under Emil
Kowalski's rules. Entrances/exits `ease-out`, never `ease-in`; UI durations
<300ms (buttons 100–160, dropdowns 150–250, modals 200–500); transitions not
keyframes for anything triggered rapidly; exit mirrors entrance;
`prefers-reduced-motion` handled or the motion is a defect; things used
constantly (menus, shortcuts) animate near-imperceptibly or not at all.
A static section with no motion defects scores full points — absence of
motion is never a deduction.

**Perception & UX — 10.** Eye path matches intended hierarchy (squint test:
biggest/brightest = most important); related items grouped by proximity;
exactly one primary CTA per view, secondary actions visually quieter (pill vs
text-link pattern); tap targets ≥44px; interactive things look interactive,
decorative things don't.

**Semantics, SEO·GEO & CWV — 10.** Meaningful alt text (H1 wordmark alt = real
title); landmarks sane; no layout shift risk (images sized, fonts with
fallbacks, min-heights where content loads late); no oversized images; nothing
render-blocking added; question-style H2/H3 kept where GEO copy uses them.

## Scorecard format

```
DESIGN CRITIC — <section> · round <n>/3
Layout 18/20 · Type 20/20 · Color 13/15 · Glass 15/15
Motion 10/10 · UX 9/10 · Sem 9/10  →  94/100 PASS
Top fixes: 1) chip rows touch at ≤734 (−2 layout) 2) second mint accent in
footer (−2 color)
```

One line version for the twin: `critic 94/100 PASS r2 — fixed: chip spacing, footer accent`.

## House constraints that override everything

No gradients. Content stays editable in wp-admin (a "fix" may never hardcode
content into templates). Block-native markup only. Vanilla CSS/JS only — a fix
may not introduce React, Tailwind, or a motion library. Korean copy is Dale's;
flag awkward copy, never rewrite it silently.
