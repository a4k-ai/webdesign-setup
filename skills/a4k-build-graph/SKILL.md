---
name: a4k-build-graph
description: Run Dale's A4K build graph — the multi-agent development workflow for his WordPress sites (NextSchool, ThinkGen, Ai4Kids, parent theme). Use whenever Dale hands over a PRD, asks to "run the graph", "start a run", "빌드 그래프", "그래프 돌려", asks to build/change a site feature through the pipeline, or asks about a run's state, work order, or HITL decision. Also use when resuming after Dale records a verdict in Airtable or the wp-admin Build pipeline page.
---

# A4K build graph (v4)

Claude is the development engine; Dale is management. Dale writes the PRD and
decides at two HITL gates. Claude does everything else through five subagent
lanes. **Lanes never talk to each other** — they coordinate only through the
work order (hub-and-spoke). The work order (작업지시서 — software engineers
call this the interface contract) pins the interfaces no lane may renegotiate.

## The graph

```
PRD → Plan(work order v1 + Design Spec) → HITL:plan → Fan-out(5 lanes)
    → integrate → Stage deploy → HITL:review → merge to main (live)
Loops: gate reject → Plan · lane conflict → work order revision → re-run
       affected lanes · review fixes → Fan-out
```

## Lane precedence — every conflict has one legal resolution

| # | Lane | Yields to | Owns |
|---|------|-----------|------|
| 1 | Security / PIPA | nothing | consent, retention, escaping, nonces, capability checks, no secrets in code |
| 2 | Structure | 1 | block-native templates/patterns, tagName correctness, parent/child split |
| 3 | Data | 1–2 | wp options, Airtable back-office writes, field naming |
| 4 | SEO · GEO | 1–3 | heading hierarchy, schema.org JSON-LD, llms.txt, CWV budgets |
| 5 | UI | everyone | tokens in theme.json, glass design language, Apple type scale, motion policy |

Deep references live in the claude.ai project docs (`skills/architecture.md`,
`skills/php.md`, `skills/css.md`, `skills/database.md`, `skills/json-ld-schema.md`)
and the `wordpress-webdesign` skill. Hand each lane ONLY the work order plus its
own lane brief — never the whole conversation. Log each lane's reported token
count to Lane work.Tokens.

## Design Spec (v4)

When the PRD or Dale's references include visual work, Plan produces a Design
Spec section INSIDE the work order: tokens extracted from the actual reference
(type sizes, spacing, radii, colors — measured, never guessed; see
`wordpress-webdesign` → Reference → Design Spec), component mapping, and
deliberate deviations. It is versioned with the work order and binds all lanes.

## UI lane = 3-stage pipeline (v4)

1. **Build** — block patterns + child CSS per the Design Spec.
2. **Spatial correct** — screenshot at >1068 / ≤1068 / ≤734, fix alignment and
   spacing math (sibling rows align, no overflow, spacing-scale gaps).
3. **Critic gate** — run `a4k-design-critic`: score /100, fix, re-score.
   **Pass ≥85, max 3 rounds.** 3 rounds under 85 = spec problem → HITL question
   to Dale with the last scorecard. Nothing reaches integrate — and no
   screenshot reaches Dale — without a passing scorecard.

The critic runs INSIDE the UI lane (hub-and-spoke intact). Its one-line result
(`critic 94/100 PASS r2 — fixed: …`) goes into the build-review Decision notes
so the twin shows it; its tokens get their own Lane work row `UI · critic`.

## Loop guards — hard stops

- 3 work order revisions in one run → stop, show Dale the conflict. (Usually an ambiguous PRD — his call.)
- 3 failed attempts at the SAME verification in one lane → stop that lane, ask. (The critic's 3-round gate is this rule applied to design.)
- A gate with Verdict=Pending → nothing downstream runs. No exceptions.

## Standing rules (from Dale, non-negotiable)

Block-native only; ALL content editable in wp-admin, never hardcoded in
templates. Code in git, content in WordPress. Airtable is back office only —
never in the page-render path. PIPA on anything collecting personal data
(consent split 필수/마케팅, retention fields, 국외이전 disclosure; under-14
guardian consent for Ai4Kids). Secrets in wp options / network options only —
never in chat. Drafts and staging first — Dale reviews before anything goes
live. Parent-theme runs affect all four sites: staging is mandatory, never
optional. No gradients. Plugin source of truth:
`C:\Claude\Assistant\work\Webdesign\plugins\` — zips are disposable outputs.

## Airtable protocol — base `apppHExbsGvtJxuOK` · 09_Build_Graph (Claude)

Sites → Projects → Runs. A run links a Project; the site comes through it.
Before starting: check the project's **Open runs** rollup — if ≥1, stop and ask
Dale which run survives. One graph per project at a time.

Write at every phase boundary (the wp-admin twin polls this):

1. **Run start** — Runs row: Status=Planning, Current node=Plan, Opened, Target.
2. **Work order** — Work orders row: Body (field names, block names, file
   paths, hook names, budgets, Design Spec — short), Version, link Run. NEVER
   edit a work order in place: revision = new row + Why revised + tick
   Superseded on the old + increment Runs.Revisions.
3. **Gates** — Decisions row: Type, Verdict=Pending, link Run+Work order; put
   the critic's one-line scorecard in Notes at build review. Set
   Runs.Status=Awaiting approval / Awaiting review. Tell Dale. STOP. Resume only
   on a verdict. Changes requested → apply Notes; Rejected → back to Plan.
4. **Lanes** — Lane work row per lane per work order version: State, Findings,
   Files touched, Overrode, Retries, Tokens. UI adds a separate `UI · critic` row.
5. **Events** — append one row per boundary (lane start/finish, revision, gate,
   deploy). Keep Runs.Current node accurate at all times.
6. **Close** — Status=Merged (or Abandoned), update the site's Theme version.

**Token rows are mandatory on EVERY run, including compressed ones** (single
integrator, presentation-only scope): at minimum Plan / Integrate / Fixes rows,
marked "estimate" when estimated, so Runs "Tokens (lanes)" and Projects
"Tokens (all runs)" never lie.

## PRD — what Claude accepts

Half a page: outcome in visitor terms; which site (parent = all-sites blast
radius); **testable acceptance criteria** (no criteria → push back before
planning, the loop guards are meaningless without them); personal data +
retention if any; who writes the Korean copy and when; out of scope.
No implementation choices — plugins, CSS, table layouts are lane output.

## Build & deploy path (v4 — SSH retired)

Write code into the local clones `C:\WebPage\<repo>` via the device bridge,
then touch `C:\Claude\_setup\sync-now` — Dale's watcher commits and pushes.
**From run 3: work on branch `run/<n>-<slug>`, open a PR, never push to main;**
merge happens at HITL:review. Deploy: POST `/wp-json/a4k-deploy/v1/deploy`
{repo, branch} (A4K Deploy plugin pulls from GitHub over HTTPS). NEVER deploy
via wp-admin "Upload Theme". Content changes = WP REST as drafts. After
deploy: clear WP Super Cache, then verify live with real requests — check
bytes/headers/DOM, never trust a plugin status page. Media never goes through
git: optimize in the sandbox, upload to the WP Media Library, reference
attachment IDs, never hardcoded URLs.

## Efficiency review (Dale asks "how efficient")

Lane cost: sort Lane work by Tokens. Rework cost: token rows on superseded
work order versions — >30% of a run means the PRD was ambiguous. Critic cost:
`UI · critic` rows as % of run — rising critic + falling Fixes is the system
working. Trend: Projects.Tokens (all runs) ÷ shipped runs, falling over time
or the graph is ceremony. Claude's own orchestration tokens are not in these
columns; use explain-usage for the session view.
