---
name: a4k-build-graph
description: Run Dale's A4K build graph — the multi-agent development workflow for his WordPress sites (NextSchool, ThinkGen, Ai4Kids, parent theme). Use whenever Dale hands over a PRD, asks to "run the graph", "start a run", "빌드 그래프", "그래프 돌려", asks to build/change a site feature through the pipeline, or asks about a run's state, work order, or HITL decision. Also use when resuming after Dale records a verdict in Airtable or the wp-admin Build pipeline page.
---

# A4K build graph

Claude is the development engine; Dale is management. Dale writes the PRD and
decides at two HITL gates. Claude does everything else through five subagent
lanes. **Lanes never talk to each other** — they coordinate only through the
work order (hub-and-spoke). The work order (작업지시서 — software engineers
call this the interface contract) pins the interfaces no lane may renegotiate.

## The graph

```
PRD → Plan(work order v1) → HITL:plan → Fan-out(5 lanes) → integrate
    → Stage deploy → HITL:review → merge to main (live)
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
| 5 | UI | everyone | tokens in theme.json, A4K/Apple layout rules, no hover-zoom |

Deep references live in the claude.ai project docs (`skills/architecture.md`,
`skills/php.md`, `skills/css.md`, `skills/database.md`, `skills/json-ld-schema.md`)
and the `wordpress-webdesign` skill. Hand each lane ONLY the work order plus its
own lane brief — never the whole conversation. Log each lane's reported token
count to Lane work.Tokens.

## Loop guards — hard stops

- 3 work order revisions in one run → stop, show Dale the conflict. (Usually an ambiguous PRD — his call.)
- 3 failed attempts at the SAME verification in one lane → stop that lane, ask.
- A gate with Verdict=Pending → nothing downstream runs. No exceptions.

## Standing rules (from Dale, non-negotiable)

Block-native only; ALL content editable in wp-admin, never hardcoded in
templates. Code in git, content in WordPress. Airtable is back office only —
never in the page-render path. PIPA on anything collecting personal data
(consent split 필수/마케팅, retention fields, 국외이전 disclosure; under-14
guardian consent for Ai4Kids). Secrets in wp options / network options only.
Drafts and staging first — Dale reviews before anything goes live. Parent-theme
runs affect all four sites: staging is mandatory, never optional.

## Airtable protocol — base `apppHExbsGvtJxuOK` · 09_Build_Graph (Claude)

Sites → Projects → Runs. A run links a Project; the site comes through it.
Before starting: check the project's **Open runs** rollup — if ≥1, stop and ask
Dale which run survives. One graph per project at a time.

Write at every phase boundary (the wp-admin twin polls this):

1. **Run start** — Runs row: Status=Planning, Current node=Plan, Opened, Target.
2. **Work order** — Work orders row: Body (field names, block names, file
   paths, hook names, budgets — short), Version, link Run. NEVER edit a work
   order in place: revision = new row + Why revised (which lane forced it,
   on what grounds) + tick Superseded on the old + increment Runs.Revisions.
3. **Gates** — Decisions row: Type, Verdict=Pending, link Run+Work order. Set
   Runs.Status=Awaiting approval / Awaiting review. Tell Dale. STOP. Resume only
   on a verdict (Airtable or the wp-admin page). Changes requested → apply
   Notes; Rejected → back to Plan.
4. **Lanes** — Lane work row per lane per work order version: State, Findings,
   Files touched, Overrode, Retries, Tokens.
5. **Events** — append one row per boundary (lane start/finish, revision, gate,
   deploy). Keep Runs.Current node accurate at all times.
6. **Human hold (not a verdict)** — whenever the run cannot move without a
   human action (SSH pull, install/activate, DNS, supply asset, credential):
   set Runs.`Waiting on Dale` = exact instructions (commands verbatim) and
   `Hold kind`. The twin shows amber ON HOLD + a Done button. Dale ticks
   `Hold done` (button or Airtable). On the next poll: log an Event, clear all
   three fields, continue. Never leave a human step implicit — if Dale must act,
   the twin must say so on the current node.
7. **Close** — Status=Merged (or Abandoned), update the site's Theme version.

## PRD — what Claude accepts

Half a page: outcome in visitor terms; which site (parent = all-sites blast
radius); **testable acceptance criteria** (no criteria → push back before
planning, the loop guards are meaningless without them); personal data +
retention if any; who writes the Korean copy and when; out of scope.
No implementation choices — plugins, CSS, table layouts are lane output.

## Build & deploy path

Write code into the local clones `C:\WebPage\<repo>` via the device bridge,
then touch `C:\Claude\_setup\sync-now` — Dale's watcher commits and pushes.
Server: git pull (SSH by Dale, or A4K Deploy plugin when installed). NEVER
deploy via wp-admin "Upload Theme" — it deletes the server's .git clone.
Stage on the staging subsite (dev theme folder, search engines discouraged);
preview content changes as WP drafts. After deploy: clear WP Super Cache,
then verify live with real requests — check bytes/headers/DOM, never trust a
plugin status page. Media never goes through git: optimize in the sandbox,
upload to the WP Media Library, reference attachment IDs, never hardcoded URLs.

## Efficiency review (Dale asks "how efficient")

Lane cost: sort Lane work by Tokens. Rework cost: token rows on superseded
work order versions — >30% of a run means the PRD was ambiguous. Trend:
Projects.Tokens (all runs) ÷ shipped runs, falling over time or the graph is
ceremony. Claude's own orchestration tokens are not in these columns; use
explain-usage for the session view.
