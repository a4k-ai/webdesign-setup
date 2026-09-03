# A4K Webdesign — collaborator setup

> **To the collaborator:** Dale has invited you to the `a4k-ai` GitHub organization (accept the email first). Then open Claude (desktop app, Cowork mode), connect GitHub (Settings → Connectors → GitHub, your own account) and type
> `setup https://github.com/a4k-ai/webdesign-setup` — Claude reads this file and walks you through it, one step at a time.
>
> **To Claude reading this:** you are onboarding a new collaborator into Dale's A4K WordPress workflow. Go step by step, confirm each step is done before moving on, never ask for or store passwords/tokens in chat (they go into the app fields the step points at), and stop at any step marked **Dale** — that one needs Dale to act. Speak Korean or English to match the collaborator.

## What you're joining
Three WordPress sites on one multisite (a4k.ai): NextSchool, ThinkGen, Ai4Kids. Code lives on GitHub (`a4k-ai` org, one repo per theme). Content lives in WordPress. Work runs through the **A4K build graph**: PRD → plan (work order) → Dale approves → build → stage → Dale reviews → merge → deploy. Rules come with the `a4k-webdesign` plugin; state lives in Airtable + a wp-admin page called *Build pipeline*; the team handbook lives in the `webdesign` repo (`a4k-ai/webdesign` — README, `ONBOARDING.md`, `handbook/`, `nextschool/`). That repo is the source of truth; when this page and the handbook disagree, the handbook wins.

## Step 0 — what you need on your computer
- Windows or Mac · Chrome · the **Claude desktop app** (Cowork) signed in to your own Claude account.
- **GitHub Desktop** (github.com/desktop). Nothing else to install.
- Using **Claude Code** (CLI / desktop app terminal) instead of Cowork? Then `git` + `gh` (GitHub CLI, `gh auth login`) replace GitHub Desktop and the folder-connect step — Claude clones and reads folders directly.

## Step 1 — GitHub account → org invite  *(Dale)*  — done before you got here
You already have a GitHub account with 2FA on, sent Dale your username, and accepted the `a4k-ai` invite (that's how you can read this). If any of that is missing, do it now: github.com → Sign up → Settings → Password and authentication → 2FA → send username to Dale → accept invite.

## Step 2 — install the a4k-webdesign plugin (one file, one click)
Dale sends you **`a4k-webdesign.plugin`** (KakaoTalk/email). Drag the file into any Claude (Cowork) chat window → a plugin card appears → press **Install**. That adds both team skills at once (`wordpress-webdesign` build rules + `a4k-build-graph` workflow).
Check: ask Claude "which skills do you have?" — both should be listed.
(Fallback: the same skills are in this repo under `skills/` — each folder's `SKILL.md` can be added via Claude → Settings → Skills. Claude Code: copy each `skills/<name>/` folder to `~/.claude/skills/<name>/`.)
Note: the skills' "read first" paths (`Assistant/work/Webdesign/...`, `C:\Claude\...`) are Dale's machine — on your computer read `webdesign/handbook/` instead.

## Step 3 — check GitHub in Claude
You connected it to read this file; make sure it's your own account. Test: ask Claude "list the repos in a4k-ai". You should see `webdesign`, `a4k-parent-theme`, `nextschool-theme`, `thinkgen-theme`, `ai4kids-theme` (plus this `webdesign-setup` repo).

## Step 4 — clone the working folders
Create `C:\WebPage\` (Mac: `~/WebPage/`). In GitHub Desktop, clone into it:
- `a4k-ai/webdesign` → the team handbook: build rules, design system, deploy runbook, infra, decisions, NextSchool status/PRD.
- the theme repos you'll work on (at least `a4k-parent-theme` + one site theme). Check out the **working branch**: `nextschool-theme` → `stage`; `thinkgen-theme` / `ai4kids-theme` → `dev` (see `webdesign/ONBOARDING.md` for the current table). Never work on `main`.
- Also create two folders **outside git**: `WebPage/_media/<site>/` (images to upload) and `WebPage/_secrets/` (the `wp-<site>.txt` app-password file Dale sends you — never paste its contents into chat).
Then in the Claude desktop app connect the folder `C:\WebPage` (Cowork → folders) so Claude can read/write it.

## Step 5 — WordPress account  *(Dale)*
**Dale** creates your user on a4k.ai (Network Admin → Users), role Editor/Administrator per site, and you set a password + **2FA** at first login (never share accounts). Bookmark:
- `a4k.ai/wp-admin/network` (network) · `a4k.ai/nextschool/wp-admin` (site) · `a4k.ai/nextschool/wp-admin/admin.php?page=a4k-pipeline` (**Build pipeline** — the live board)
- `a4k.ai/nextschool/design/` — the design language (login required)

## Step 6 — read before you touch anything (15 min)
1. `webdesign/handbook/build-rules.md` — how we build, and the 6 forbidden things (this is the code-review bar).
2. `webdesign/handbook/deploy.md` — push → A4K Deploy → Delete Cache → verify.
3. `webdesign/handbook/infra.md` + `decisions.md` — servers, multisite facts, what never to do (e.g. never "Upload Theme" in wp-admin), and why.
4. `webdesign/nextschool/STATUS.md` — where NextSchool stands right now.
5. The design language page.

## Step 7 — how you work (the loop)
1. Get a task from Dale (a PRD or a review note). Ask Claude to "run the graph" for it — Claude writes the work order, you and Dale approve on the Build pipeline page.
2. Claude commits to the theme repo's **working branch** (`stage` for NextSchool) and pushes. `stage` deploys to the stage site (`a4k.ai/stage-nextschool/`) via **A4K Deploy** in wp-admin, then **Delete Cache**, then verify in a private window. Never push to `main` — `main` is the live code.
3. Dale reviews the staged page → merges `stage` → `main` → deploys live (no SSH, no "Upload Theme" zip). New content is created as **draft**; Dale publishes.
4. Content (pages, menus) is edited in wp-admin, never in code. Secrets never in chat.

## Step 8 — say hello
Tell Dale you're set up. He'll assign your first task and add you as **Owner** on a run in Airtable (Dale keeps the Airtable seat; your Claude reads/writes run state through the Build pipeline page).

### If something fails
- Can't see repos → org invite not accepted (Step 1). · Claude can't read files → folder not connected (Step 4). · Build pipeline page 403 → WordPress role (Step 5). · Plugin card doesn't appear → make sure you dropped the `.plugin` file into the chat itself, and your org allows plugin installs.
