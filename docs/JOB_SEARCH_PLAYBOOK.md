# Job Search Playbook — Using career-ops End-to-End

This is your personal walkthrough for how to actually run a job search with this
system, day to day. It assumes the setup already done: `cv.md`,
`config/profile.yml`, `modes/_profile.md`, `article-digest.md`, and
`portals.yml` are all filled in with your real data (full-stack / AI / fintech
platform archetypes, $60K–90K USD target, standard spend tier, remote-preferred).

For install/CLI-specific setup, see [SETUP.md](SETUP.md). For "what does mode X
do internally," see `AGENTS.md` at the repo root — this doc is the workflow,
not the reference.

---

## 1. The core loop

There are really only three loops you'll run, repeated over and over:

```
   ┌─────────────┐     ┌──────────────┐     ┌─────────────┐
   │   DISCOVER   │ --> │   EVALUATE   │ --> │    APPLY    │
   │ scan/pipeline│     │ auto-pipeline│     │apply/email  │
   └─────────────┘     └──────────────┘     └─────────────┘
                                                     │
                              ┌──────────────────────┘
                              v
                     ┌─────────────────┐
                     │  TRACK & FOLLOW  │
                     │ tracker/followup │
                     └─────────────────┘
                              │
                              v
                     ┌─────────────────┐
                     │ INTERVIEW / CLOSE│
                     │interview-prep etc│
                     └─────────────────┘
```

You don't have to type long prompts. In Claude Code, `/career-ops <mode>` (or
just paste a job URL/JD) is enough — the router in `.claude/skills/career-ops/`
figures out the rest and loads your profile automatically.

---

## 2. Discover: finding offers

**When you have no leads yet:**

```
/career-ops scan
```

This scans the ~45 companies preloaded in `portals.yml` via their public
Greenhouse/Ashby/Lever/Workday APIs — zero LLM cost, so run it often. Given
your archetypes, it's filtering for full-stack / AI / fintech-platform roles
already (via `title_filter` in `portals.yml`). New matches get written to
`data/pipeline.md` as an inbox.

**Want a broader, keyword-first sweep** (no company list needed, scans full
public ATS datasets): `/career-ops discover` first to resolve companies into
scannable boards, or ask for the reverse-ATS scanner directly.

**Want more title ideas from your own CV** (adjacent titles you might be
underselling yourself on): `/career-ops titles`.

**To automate this** so you're not manually re-running it: say "scan every 3
days" and I'll set up a recurring `/career-ops scan` via `/loop` or `/schedule`
(see [AUTOMATION.md](AUTOMATION.md) for cron/launchd if you'd rather run it
outside the CLI).

---

## 3. Evaluate: turning a lead into a decision

**The one command you'll use most:** paste a JD or URL directly, or run:

```
/career-ops {paste JD text or URL}
```

This is **auto-pipeline** — it evaluates the offer against your profile (Blocks
A–F: role fit, comp, culture signals, risk, etc., plus Block G posting
legitimacy), writes a full report to `reports/{num}-{company}-{date}.md`,
generates a tailored PDF if the score clears the auto-PDF threshold, and adds
it to your tracker automatically. One command, fully hands-off except your
final read of the report.

**If you have a backlog of URLs** sitting in `data/pipeline.md`:

```
/career-ops pipeline
```

Processes them all, with a fast triage pass first to filter out clear misses
before running the expensive full evaluation.

**If you want evaluation only** (no auto-PDF, just the scored report):
`/career-ops oferta`

**Comparing multiple live offers against each other:** `/career-ops ofertas`

**Read the score honestly.** Per this system's ethics rule: **below 4.0/5, the
system will explicitly recommend against applying.** That's by design — this
tool is built for fewer, better-targeted applications, not spray-and-pray. If
you disagree with a score (e.g., it missed relevant experience), tell me and
I'll update `modes/_profile.md` or `article-digest.md` so future evaluations
get it right — the system is supposed to get smarter about you, not just about
job postings.

---

## 4. Apply: turning a good match into a submission

Once a report scores well and you decide to apply:

| You want... | Command |
|---|---|
| A tailored, ATS-optimized CV PDF | `/career-ops pdf` |
| A LaTeX/Overleaf version of the CV | `/career-ops latex` |
| A cover letter | `/career-ops cover` |
| A formal application email (draft only) | `/career-ops email` |
| Help filling out a live application form | `/career-ops apply` |
| LinkedIn outreach to a recruiter/hiring manager | `/career-ops contacto` |
| Deep company research before applying | `/career-ops deep` |

**Hard rule, non-negotiable:** the system will draft, fill forms, and generate
PDFs — but it will **never click Submit/Send/Apply for you.** You always
review and send it yourself. This is enforced by the system's ethics
guidelines, not a preference you can turn off.

---

## 5. Track: staying on top of the pipeline

```
/career-ops tracker
```

Gives you a status overview from `data/applications.md` — your canonical
tracker. Canonical statuses are: `Evaluated → Applied → Responded → Interview
→ Offer → Hired` (or `Rejected` / `Discarded` / `SKIP` off-ramps at any point).

**Never hand-edit the tracker table directly.** Two rules that keep it
consistent:
- New rows come from evaluations automatically (via `merge-tracker.mjs`) —
  never manually add a row.
- Status changes go through `node set-status.mjs <report#|company> <State>
  [--note]` — this is the only path that's validated and safe against typos or
  duplicate rows. Just ask me to update a status by company name or report
  number and I'll run it.

**Follow-ups:** `/career-ops followup` tells you who's overdue for a nudge and
can draft the follow-up message. A cadence gets seeded automatically the
moment something turns `Applied`.

**Replies:** if you get an email back, `/career-ops reply-watch` classifies it
and suggests the tracker update. No Gmail connected? Paste the email content
and I'll log it via `paste-reply.mjs`.

**Outcome (when you land something or get rejected):** `/career-ops outcome`
records the final result and archives the artifacts.

---

## 6. Interview & close

| Stage | Command |
|---|---|
| Company-specific prep doc | `/career-ops interview-prep` |
| Red-flag check before committing time | `/career-ops interview-redflag` |
| Time-blocked prep plan | `/career-ops interview/plan` |
| Practice Q&A with feedback | `/career-ops interview/practice` |
| Post-interview debrief + predict next round | `/career-ops interview/debrief` |
| Reviewing a received offer/contract | `/career-ops offer-prep` |

`interview/practice` and `interview-prep` both draw on
`interview-prep/story-bank.md` — your accumulated STAR stories. Build this up
as you go; it compounds. After each real interview, run `debrief` so gaps get
captured and closed before the next round instead of repeating them.

---

## 7. Learning loop: making the system smarter about you

This is the part most people skip and shouldn't. After **every** evaluation:

- **"This score is too high/low"** → tell me why, and I'll update
  `modes/_profile.md` (targeting/narrative) or `config/profile.yml` (facts).
- **"You missed my experience in X"** → same fix, or if X isn't in `cv.md` yet,
  run `/career-ops add` to get it added with a source-grounded preview before
  it's written (never auto-added).
- **Rejection patterns** → periodically run `/career-ops patterns` to see if
  there's a systemic issue (wrong seniority level, comp mismatch, a specific
  ATS vendor rejecting you more, etc.) — it also matches your interview
  answers to best-fit roles.
- **Skill gaps** → `/career-ops upskill` aggregates skill gaps across every
  report you've generated and excludes anything already on your CV, so it only
  ever tells you what's actually missing.

Your personalization always lives in `cv.md`, `config/profile.yml`,
`modes/_profile.md`, `article-digest.md`, `modes/_custom.md` — never in
`modes/_shared.md`. That boundary is what lets you run `/career-ops update`
later to pull in system improvements without ever touching your own data.

---

## 8. What's already set up for you

- **Archetypes** (`config/profile.yml` + `modes/_profile.md`): Senior/Staff
  Full-Stack Engineer (primary), AI/ML Engineer, Fintech Platform Engineer
  (secondary) — the system adapts framing per JD based on which axis it hits.
- **Proof points** (`article-digest.md`): Unit Master (KES 42B+ AUA, 12,400+
  investors, 5 markets), AcentRe (live AI reinsurance platform), SumoReads.
- **Comp target:** $60K–90K USD, remote preferred, open to relocation for the
  right offer.
- **Spend tier:** `standard` — balanced cost/quality model for every
  evaluation. Bump a specific offer to `premium` by asking, if it's one you
  really care about.
- **Portals:** default ~45-company scan list in `portals.yml`.
- **Playwright MCP:** configured globally, so offer-verification (checking a
  posting is actually still live before you waste time on it) uses a real
  browser, not a guess.

## 9. Quick reference: full mode menu

Run `/career-ops` with no arguments any time to see the live menu. The
short version, grouped by what you're trying to do:

- **Find offers:** `scan`, `discover`, `pipeline`, `titles`
- **Evaluate:** paste JD/URL (auto-pipeline), `oferta`, `ofertas`, `deep`,
  `training`, `project`
- **Apply:** `pdf`, `latex`, `cover`, `email`, `apply`, `contacto`
- **Track:** `tracker`, `followup`, `reply-watch`, `outcome`, `agent-inbox`
- **Interview:** `interview-prep`, `interview-redflag`, `interview/plan`,
  `interview/practice`, `interview/debrief`, `offer-prep`
- **Improve targeting:** `patterns`, `upskill`, `add`, `expand`
- **Housekeeping:** `update`, `batch`

## 10. When something feels off

- Health check the tracker: `node verify-pipeline.mjs`
- Normalize statuses: `node normalize-statuses.mjs`
- Dedupe: `node dedup-tracker.mjs`
- Re-run setup checks any time: `node doctor.mjs --json`
- Full command/scripts reference: [SCRIPTS.md](SCRIPTS.md)
- Common questions: [FAQ.md](FAQ.md)
