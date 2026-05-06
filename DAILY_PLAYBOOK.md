# Daily Playbook — How to Actually Use the Workflow

> **Companion doc:** This is the daily-use guide for projects scaffolded with the standard 11-command, 7-agent, 11-doc structure. The scaffold doc tells you *what exists*. This doc tells you *what to do on a Tuesday morning*.
>
> Generic across all projects. Substitute your domain, project name, and team conventions where relevant.

---

## What this doc is

A reference for the patterns that come up over and over. Not exhaustive — opinionated. The goal is that future-you, six months from now, can open this and remember how the system is supposed to work without re-deriving it.

It's organized so you can jump to the section you need:

1. [The mental model](#the-mental-model)
2. [The standard daily loop](#the-standard-daily-loop)
3. [The minimum viable session](#the-minimum-viable-session)
4. [Capturing ideas mid-session](#capturing-ideas-mid-session)
5. [Common scenarios](#common-scenarios)
6. [Asking the architect](#asking-the-architect)
7. [Shipping](#shipping)
8. [When something breaks](#when-something-breaks)
9. [Diagnostics](#diagnostics)
10. [Backlog hygiene](#backlog-hygiene)
11. [Anti-patterns](#anti-patterns)
12. [A worked example](#a-worked-example)

---
> **Defaults used in examples below:** Vercel auto-deploy, `main` = production, 
> `staging` = dev, Node + npm, subdomain-per-env URLs. If your project's 
> `INFRASTRUCTURE.md` Deploy Config differs, mentally substitute your DEV_BRANCH, 
> PROD_BRANCH, DEV_URL, PROD_URL, and LOCAL_DEV_COMMAND throughout. The patterns 
> are the same; only the names change.

---
## The mental model

Eleven commands, three groups. Memorize the groups, look up the commands when you need them.

| Group | What it's for | Commands |
|---|---|---|
| **Session** | Orient, capture, ask, wrap up | `/session-start` `/session-end` `/ask-architect` `/add-idea` |
| **Build** | Make the change | `/implement` `/qa` `/pre-commit` |
| **Ship** | Move it through environments | `/commit` `/deploy` `/rollback` `/check-drift` |

Within each group there's a typical order, but you don't always need every command. A minor CSS tweak might skip `/qa` and `/ask-architect` entirely. A schema change might ping `/ask-architect` three times before you ever run `/implement`.

Three test environments, in order of speed and isolation:

1. **`localhost`** — your fastest feedback loop. 90% of testing happens here.
2. **`your dev URL (DEV_URL)`** — staging deploy. Catches build failures, env var issues, edge runtime quirks.
3. **`your prod URL (PROD_URL)`** — production. Real users.

The whole workflow is built around the principle that local is cheap, staging is medium, prod is expensive. Test cheaply first, escalate only when needed.

---

## The standard daily loop

The 80% case: open the project, work on a feature, ship it, close out. Roughly 1–3 hours.

```bash
# 1. Open the project
cd ~/path/to/project
git status                         # confirm clean working tree
git pull origin staging            # get any changes from previous session
claude                             # launch Claude Code
```

Inside Claude Code:

```
/session-start                     → 30 sec: briefing on where you left off
                                   → reads SESSION_TRANSITION, KNOWN_ISSUES,
                                     IMPLEMENTATION_PLAN (incl. Backlog)
                                   → tells you: completed last session,
                                     in progress, blocked, next action

[decide what to work on]           → usually: continue the recommended next
                                     action, OR pick a backlog item, OR
                                     address a blocker

[optionally] /ask-architect <Q>    → for anything with structural impact
                                   → cheap insurance; skip for routine work

git checkout -b feat/<name>        → for anything bigger than a small fix

/implement <description>           → explorer → architect → database (if schema)
                                     → developer → reviewer → qa → doc-writer
                                   → each step has a gate; failure stops the chain

[npm run dev]                      → test on localhost, iterate
                                   → fix bugs, re-implement as needed

/commit                            → audit + push to current branch
                                   → Vercel auto-deploys to dev.<your-domain>

[QA on dev.<your-domain>]          → verify in real deploy env

/deploy                            → reverse-drift + P0 + audit + typed confirm
                                   → merges staging → main
                                   → Vercel auto-deploys to prod
                                   → logged to DECISION_LOG

/session-end                       → 90 sec: updates docs, drift check,
                                     session summary
```

If you're working on a feature branch (recommended for anything non-trivial), merge it back to `staging` either via the GitHub UI or:
```bash
git checkout staging && git merge feat/<name> && git push origin staging
```
…before running `/deploy`.

---

## The minimum viable session

Sometimes you only have 30 minutes. The minimum that still maintains forward motion and keeps the docs honest:

```
/session-start                     → 30 sec: orient
[do one focused thing]             → 25 minutes: implement, test
/commit                            → push to staging
/session-end                       → 90 sec: capture what you did
```

Skip `/deploy` entirely. Staging is fine — you can deploy in the next session, or batch up multiple commits and deploy once.

The discipline of running `/session-end` even on short sessions is what keeps SESSION_TRANSITION current. Without it, two short sessions of "I'll update the docs later" turns into "I forget what I was doing" three weeks from now.

---

## Capturing ideas mid-session

Ideas appear at exactly the wrong moment — when you're focused on something else. The system is designed to capture them in 20 seconds and get out of your way.

### The three buckets

When an idea hits, decide its bucket in 10 seconds:

- **Tiny tweak** — under 30 minutes, no architectural impact. (e.g., "Change pill color", "Fix date format")
- **Feature** — an hour or more, or touches multiple files. (e.g., "Add bulk export", "New dashboard widget")
- **Architectural** — changes how the system works. (e.g., "Switch from polling to realtime", "Multi-tenant support")

If unsure, default to Feature — it'll get sorted properly when you revisit.

### Capture, don't decide

For tiny tweaks, jot them on a sticky note or a personal todo list. Don't formalize — you'll batch them later.

For features and architectural items, use `/add-idea`:

```
/add-idea <one-line description>
```

You'll be prompted for source, bucket, and notes (all optional except description). Takes 20 seconds. The doc-writer agent appends a row to the Backlog table in `IMPLEMENTATION_PLAN.md` with a unique `B-NNN` ID and today's date.

Then go back to what you were doing.

### Don't:

- Don't run `/ask-architect` on a new idea mid-session. You're context-switching for nothing — capture, decide later.
- Don't try to estimate or schedule the idea. The Backlog is just a list.
- Don't skip the bucket prompt. Even a wrong guess is better than no signal.

---

## Common scenarios

### Continuing where you left off

This is the default `/session-start` case. The previous `/session-end` left SESSION_TRANSITION in a state that tells you exactly what to pick up. Trust it.

If "Next Recommended Actions" mentions a back-merge from a recent rollback or hotfix, do that first — otherwise your next `/deploy` will be blocked.

### Starting fresh with no context

`/session-start` will tell you that SESSION_TRANSITION is empty (or stale), and recommend picking a task from `IMPLEMENTATION_PLAN.md` Phase 1.

If `/ask-architect` for "what's the recommended starting point?" doesn't help (it might not, with limited context), just start with the lowest-risk, highest-clarity task in the current phase.

### Picking up after a teammate

Run `/check-drift` first thing. If a teammate has been working in the repo, you want to know whether they've shipped anything to production while you were away.

Then run `/session-start` — your teammate's `/session-end` should have captured what they did in SESSION_TRANSITION.

If something looks unfamiliar, `git log staging --oneline` (and `main --oneline`) tells you what's happened since you last looked.

### Picking up after vacation

Same as above, but more deliberate:

```
/check-drift                       → full picture of git state
/session-start                     → context from last session
[skim DECISION_LOG.md]             → any major decisions while you were out?
[skim KNOWN_ISSUES.md]             → any new P0s/P1s?
```

Don't try to ship anything in the first hour back. Get oriented first.

### Batching tiny tweaks

End-of-week or whenever you have a clear 30 minutes:

```
/session-start
[implement tweak 1]   /commit       → pushed to staging
[implement tweak 2]   /commit
[implement tweak 3]   /commit
[QA on dev]
/deploy                             → ship all three at once
/session-end
```

Tiny tweaks usually don't need `/ask-architect` or `/qa`. Just implement, commit, repeat.

### Shipping a hotfix

A hotfix is a bug that needs to ship immediately, not wait for the next session's normal flow. Two paths:

**Path A — through staging (preferred):**
1. Fix on `staging` branch
2. Test on localhost
3. `/commit`
4. Verify on `dev.<your-domain>`
5. `/deploy`

This takes 15–30 minutes and is safest.

**Path B — straight to main (rare, for true emergencies):**
This intentionally isn't supported by `/commit` (which refuses to commit to main). For true emergencies where staging would slow you down, do it manually:

```bash
git checkout main
git pull origin main
# fix the bug
git add -A
git commit -m "Hotfix: <description>"
git push origin main                # Vercel deploys to prod
git checkout staging
git merge main                      # back-merge so /deploy isn't blocked next time
git push origin staging
```

Then go to KNOWN_ISSUES.md and DECISION_LOG.md and document what happened, manually. The automated logging only happens through `/deploy`.

This should be **rare** — most "emergencies" are 30 minutes through staging, not 5 minutes direct to prod. Use Path A unless you have a reason that survives saying out loud.

### Ending a session mid-task

It happens. You ran out of time, the kid is crying, whatever. The work is half-done.

```
/session-end                       → captures what's incomplete
                                   → "What's Next" makes the half-done state clear
```

Don't `/commit` half-done work to staging unless it actually builds. Uncommitted WIP on your local machine is fine — that's what the local working tree is for. The next `/session-start` will pick it up.

If the WIP needs to live somewhere durable (you're switching machines, or it's a feature branch you want backed up), commit it to a feature branch — never to `staging` if it doesn't work.

---

## Asking the architect

`/ask-architect` is for quick design Q&A. It doesn't write code, doesn't update docs, doesn't kick off `/implement`. It just gives you an opinion grounded in the project's existing architecture.

### Use it for:

- Questions where the wrong answer is expensive to undo: schema choices, auth model, multi-tenancy, caching strategy
- Proposed structural changes: "split this into a service", "move this to a new endpoint"
- Anything you'd want a second opinion on before spending an hour building

### Don't use it for:

- Routine implementation questions ("how do I add a column" — just do it)
- Style preferences (architect doesn't have opinions on these)
- Speculation about future features that aren't being built yet

### The verdicts

- **GO** — proceed. The proposal is consistent with documented decisions. Run `/implement` next.
- **NOGO** — there are specific concerns tied to documented decisions or constraints. Address them and re-ask. Maximum 2 NOGO rounds before escalating to a human conversation.
- **NEEDS MORE INFO** — the proposal is ambiguous in ways that affect the answer. Provide what's missing and re-ask.

### A useful mental anchor

If your proposal is more than three sentences long, the architect probably needs more info even if it doesn't say so. Vague proposals get vague answers. "Add search" is too thin. "Add full-text search across client name, MRN, and diagnosis, returning client cards filtered by user's program access" is enough to evaluate.

---

## Shipping

The local → staging → prod flow is the spine of the workflow. Each step has a job.

### Local (`npm run dev`)

Where you live. Fast feedback, no risk. Iterate freely.

What this catches: most logic bugs, UI issues, obvious crashes.
What this misses: env var problems, build failures, edge runtime issues, CDN/caching behavior.

### Staging (`/commit` → `dev.<your-domain>`)

Where you verify before production. Same code as prod, different DB and env.

What this catches: build failures, env var problems, "it works on my machine" issues.
What this misses: anything that depends on production data, real user behavior, scale.

`/commit` runs the audit before pushing. If the audit fails, fix the issues — don't override.

### Production (`/deploy` → `portal.<your-domain>`)

Where users live. Highest risk. The friction in `/deploy` is intentional.

The full gate sequence:
1. Branch must be `staging`
2. `staging` must be ahead of `main`
3. `main` must NOT be ahead of `staging` (no override)
4. No active P0s in KNOWN_ISSUES (override requires typed reason)
5. Final pre-commit audit must pass
6. Typed confirmation: `yes deploy to production`

Each gate exists because someone (probably you) screwed something up that way at some point. Don't try to remove gates without thinking through the failure mode they're catching.

### When you don't deploy

Not every session ends with a `/deploy`. A perfectly normal session is local → `/commit` → done. Multiple commits to staging accumulate, then you `/deploy` once when you're ready.

Common reasons not to deploy at session end:
- Work-in-progress on staging that isn't ready for users
- You want a teammate to review the dev environment first
- It's late and you don't want to be on call for issues
- The change touches a sensitive area and deserves a fresh-eyes review tomorrow

There's no rule that says "deploy what you commit." Use judgment.

---

## When something breaks

Something is broken in production. The fastest path back depends on what kind of broken.

### "It's broken — I need to revert NOW"

```
/rollback                          → reverts the last merge to main
                                   → triggers Vercel rebuild from reverted state
                                   → for fastest possible recovery, ALSO use
                                     Vercel's "Instant Rollback" UI
                                   → logs to DECISION_LOG and KNOWN_ISSUES (P0)
                                   → prompts to back-merge main into staging
```

Default to **yes** on the back-merge prompt. The cost is 10 seconds; the cost of declining is that next session's `/deploy` will be blocked until you remember.

### "It's broken — I think I know how to fix it"

If you can fix forward in under 15 minutes, do that instead of rolling back:

```
[fix the bug]
/commit                            → push fix to staging
[verify on dev]
/deploy                            → ship the fix
```

If the fix is going to take longer than 15 minutes and users are actively affected, roll back first, then fix forward in the next session.

### "It's broken but I'm not sure how"

Roll back. Diagnose offline. Don't debug in production.

### After any rollback

The rollback automatically logs a P0 in KNOWN_ISSUES.md. The next `/deploy` will be blocked by it until you resolve the underlying issue (or explicitly override with a typed reason).

This is intentional. The P0 isn't punishment — it's a forcing function so the bug doesn't slip back into production unfixed.

---

## Diagnostics

`/check-drift` is the "what's the state of the world?" command. Run it whenever you want a full picture.

```
/check-drift                       → fetches remote refs
                                   → reports staging-vs-main commit counts
                                   → suggests back-merge if main is ahead
                                   → READ-ONLY — never modifies anything
```

When to run it:

- Before a big deploy where you want to know exactly what's shipping
- After a teammate has been in the repo
- When something feels off
- Periodically (monthly) as hygiene
- When you're thinking "wait, did I actually deploy that?"

What it doesn't check:

- Schema drift between dev and prod databases
- Environment variable parity between Vercel projects
- Dependency version drift

Those are separate concerns. For small teams, manual quarterly review is usually enough. If you need automated checking later, those can be added as separate commands.

---

## Backlog hygiene

Without occasional pruning, the Backlog turns into a graveyard. About once a month, spend 15 minutes:

1. **Delete dead ideas.** If something's been in the backlog for 3 months and nobody's mentioned it again, it's probably not worth doing. Delete it. (You can always re-capture if it comes back.)

2. **Promote scheduled items.** Move things from Backlog into a specific Phase task in `IMPLEMENTATION_PLAN.md` once you've decided they're happening. Reference the backlog ID in the task ("Add bulk export — promoted from B-014"). Move the row to a Completed sub-section with note "Promoted to Phase X task."

3. **Re-bucket as needed.** Sometimes a "feature" turns out to be architectural once you understand it. Or a "tiny tweak" reveals more depth. Update buckets so the labels stay honest.

4. **Update notes on blocked items.** If a blocker resolved (e.g., "stakeholder confirmed requirements"), update the entry. If a blocker is still real but you have new info, add a note.

5. **Look for patterns.** After 6 months of `/add-idea` use, the Source column tells you something: are most ideas coming from one stakeholder? Is one area of the product generating disproportionate requests? That's product signal worth acting on.

This is also a good time to skim `DECISION_LOG.md` (any decisions you forgot you made?) and `LESSONS_LEARNED.md` (anything worth re-reading?).

---

## Anti-patterns

Don't:

**Edit `KNOWN_ISSUES.md` to make a P0 disappear.** The `/deploy` gate exists for a reason. Either fix the issue or use the override (which logs the reason). Don't quietly downgrade or delete.

**Use `/add-idea` for every micro-thought.** The Backlog is for things you might actually do. If everything goes in, nothing gets prioritized. Sticky notes exist for a reason.

**Force-push.** Anywhere. The commands refuse to do this — don't bypass them manually. Force-pushing on `main` after a deploy is how you lose the audit trail.

**Commit half-built work to `staging`.** Staging auto-deploys to `dev.<your-domain>`, which means broken commits = broken dev environment. Use feature branches for WIP.

**Deploy without testing on `dev.<your-domain>` first.** "It worked on localhost" has been the famous last words of many deploys. Staging is fast — use it.

**Skip `/session-end` because the session was short.** The 90 seconds it takes to update SESSION_TRANSITION saves you 20 minutes of context-rebuilding next session. Math.

**Answer your own architecture questions when you could ask the architect.** It's free. It's grounded in the project's documented decisions. It catches blind spots. The cost of asking is 30 seconds; the cost of not asking can be hours of rework.

**Pile up unscheduled commits on staging.** Staging is supposed to be a snapshot of "what's about to ship." If 15 commits accumulate without a deploy, you've lost the connection between staging and production. Deploy in batches that you can mentally hold.

**Treat `/rollback` as failure.** It's a tool. Using it is a sign that the gates worked (the bug got caught) and the fallback worked (production recovered). The failure would be NOT having the tool when you need it.

---

## A worked example

Tuesday morning. You're picking up after a weekend off.

```bash
cd ~/path/to/project
git status                          → clean
git pull origin staging             → 2 new commits, your teammate has been working
claude
```

```
/check-drift
```

> Staging ahead of main (2 commits) — NORMAL.
> Main ahead of staging — none.
> ✅ In sync direction-wise. Two commits awaiting deploy.

```
/session-start
```

> **Last session:** Teammate added the bulk export feature for the team.
> **In progress:** Bulk export tested locally, awaiting review on dev.
> **Blocked:** None.
> **Active issues:** P1: Slow load on the dashboard view (KI-012)
> **Recommended next action:** Verify bulk export on dev.<your-domain>, then deploy. After that, address KI-012.
> **Recent backlog:** B-014 (Feature) — Add saved filters to the dashboard. Captured Friday from stakeholder feedback.

OK, plan: verify the existing staging work first, deploy it, then look at the dashboard issue.

You open `dev.<your-domain>`, click around, the bulk export works. Good.

```
/deploy
```

> Reverse drift check: ✅
> P0 gate: ✅ (KI-012 is P1, doesn't block)
> Audit: ✅
>
> Commits to ship (2 total):
>   ab12345 Add bulk export endpoint
>   cd67890 Add bulk export UI button
>
> Type `yes deploy to production` to proceed.

```
yes deploy to production
```

> ✅ DEPLOYED. SHA: ef34567. Vercel rebuilding now.

You check `portal.<your-domain>` in a minute. Bulk export works in production. Good.

Now to the dashboard issue. You open `KNOWN_ISSUES.md`, find KI-012. It says "Dashboard takes 4+ seconds to load on first paint." You think this might be an N+1 query in the census fetch.

```
/ask-architect the slow dashboard load (KI-012) — I think it's an N+1 in the census query. Proposal: rewrite the query to use a single join instead of fetching beds and clients separately.
```

> **Verdict: GO** with one note.
>
> Approach is consistent with existing service-layer patterns (DEC-005). The single-query approach will need to preserve the explicit `Client.status` field (don't derive from join state).

```
git checkout -b fix/dashboard-n1
/implement fix the N+1 query in the census fetch — rewrite as a single join while preserving Client.status as the source of truth
```

The implement chain runs. Architect: GO. Database agent: confirms the query change is fine. Developer: rewrites the query in `lib/census.ts`. Reviewer: clean, no P0/P1. QA: produces a test plan covering the census view and confirming the auth-expiry highlighting still works.

You run `npm run dev`, click around the dashboard. It loads fast now. Auth expiry coloring still works. Good.

```
/commit
```

> Audit: ✅. Branch: fix/dashboard-n1.
>
> Proposed commit: "Fix N+1 in census query. KI-012 resolved. Single-query approach preserves Client.status."
>
> Confirm? (yes/no/edit)

```
yes
```

> ✅ Committed and pushed.

You merge to staging via the GitHub UI, then verify on `dev.<your-domain>`. The dashboard is fast.

```
/deploy
```

Through the gates. Production deploy succeeds.

You also want to update KI-012 to Resolved. The `doc-writer` agent will handle that automatically when you run `/session-end`, but you double-check it's getting flagged.

```
/session-end
```

> **Code scan:** No new issues.
> **Docs updated:** SESSION_TRANSITION ✅, KI-012 marked Resolved.
> **No new lessons or decisions this session.**
> **Implementation plan:** 1 task marked done.
> **Drift check:** ✅ in sync.
> **Summary:** Deployed teammate's bulk export feature. Fixed dashboard N+1 (KI-012). Production stable.

You close the laptop. The session took about 90 minutes. You shipped two distinct improvements to production with full audit trail. The next session will start with SESSION_TRANSITION reflecting today's work, and B-014 (saved filters) is still in the backlog waiting for when you decide to tackle it.

---

## When this doc gets updated

Add to it whenever you discover a pattern that's coming up repeatedly and isn't covered. Refactor when sections start contradicting each other. Delete sections that stop being true.

This is your playbook. Make it useful for you.
