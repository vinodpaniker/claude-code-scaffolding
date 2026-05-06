# New Project Scaffold — Claude Code Starter Prompt

> **How to use:** Paste everything below the horizontal rule into Claude Code at the start of any new project. Fill in the `[PROJECT NAME]` and `[DESCRIPTION]` placeholders when prompted, or just paste as-is and answer Claude's follow-up questions.
>
> **Companion doc:** Once the project is set up, see `DAILY_PLAYBOOK.md` for guidance on day-to-day use of the commands and agents.

---

## Assumptions & adaptations

This scaffold bakes in a specific stack and team shape. Read the table before scaffolding — if any row doesn't describe your project, follow the swap-out guidance.

| Assumption | Default | If you don't share this |
|---|---|---|
| **Deploy mechanism** | Vercel auto-deploys on git push | Configure your trigger in `INFRASTRUCTURE.md` Deploy Config (GitHub Actions, manual CI, `kubectl apply`, Fly machines, etc.). The ship commands read from there — gate logic stays the same; only the "now ship it" step changes. |
| **Branch model** | `main` = production, `staging` = dev environment | Set DEV_BRANCH and PROD_BRANCH in Deploy Config. For trunk-based: collapse to a single branch and use `PROMOTION_MODEL: tag` or `trunk-only`. For GitFlow: substitute `develop` for `staging` and add a `release/*` step in your deploy trigger. |
| **Runtime / package manager** | Node + npm (`npm run dev`, `localhost:3000`) | Set LOCAL_DEV_COMMAND and LOCAL_DEV_URL in Deploy Config (`go run ./cmd/server`, `cargo run`, `python manage.py runserver`, `bin/rails s`, etc.). |
| **Hostnames** | `dev.<your-domain>`, `portal.<your-domain>` | Set DEV_URL and PROD_URL in Deploy Config to your actual environment URLs. |
| **Compliance posture** | None assumed by default | If your project handles regulated data (HIPAA/PHI, PCI, SOC 2, GDPR, FedRAMP, etc.), note it in `TECH_ARCHITECTURE.md` Security & Compliance section. The `database` and `qa` agents will pick it up from there. |
| **Database** | Relational with versioned SQL migrations | If using NoSQL, a schemaless store, or ORM-managed schema, rewrite the `database` agent's rules. "Never DROP columns" is Postgres-shaped guidance. |
| **Repo shape** | Single deployable | For monorepos with multiple deploy targets, add a target argument to `/commit` and `/deploy` (e.g., `/deploy api`, `/deploy web`), or maintain one command set per target. |

For most adaptations the change is in two or three places: `INFRASTRUCTURE.md` Deploy Config (one-time setup), the relevant agent (`database.md`), and the playbook examples. The doc set, the agent role split, the gate logic, and the session loop are stack-agnostic and don't need changes.

---

## PROMPT — PASTE THIS INTO CLAUDE CODE

Please scaffold this project with the full standard structure I use for every project. This means three things:

1. A `/docs` folder with 11 template documents
2. A `.claude/agents/` folder with 7 specialist agents
3. A `.claude/commands/` folder with 11 slash commands

Do all of this now, then ask me to describe the project so we can populate the PRD, architecture docs, and Deploy Config together.

---

### PART 1 — Create `/docs` with these 11 files

For each file, create a sensible template with section headers and placeholder comments. Do not leave any file empty.

**`/docs/README.md`**
Sections: Project Overview, Quick Start, Repo Structure, Key Contacts, Links

**`/docs/PRD.md`**
Sections: Problem Statement, Goals & Non-Goals, User Personas, Feature List (with priority: P0/P1/P2), User Stories, Out of Scope, Open Questions

**`/docs/IMPLEMENTATION_PLAN.md`**
Sections: Phase overview table, then one section per phase with: Goals, Tasks (checkbox list), Dependencies, Success Criteria, Estimated Timeline. Then a **Backlog** section at the bottom — a table with columns: ID, Idea, Source, Bucket (Tiny tweak / Feature / Architectural), Notes, Captured. New ideas captured via `/add-idea` are appended here. Completed backlog items move to a "Completed" sub-section or get deleted during periodic backlog hygiene.

**`/docs/TECH_ARCHITECTURE.md`**
Sections: System Diagram (text/ASCII), Tech Stack, Component Structure, Data Flow, Key Architectural Decisions, Security & Compliance Considerations (note any regulatory posture: HIPAA, PCI, SOC 2, GDPR, etc., or "none" if not applicable), Known Constraints

**`/docs/INFRASTRUCTURE.md`**
Sections: Local Dev Setup (step by step), Environment Variables, Services & Ports, **Deploy Config** (see below), Production Environment, Deployment Process, Backup & Recovery.

The Deploy Config section is the single source of truth for branch names, URLs, and deploy mechanics — the `/commit`, `/deploy`, `/rollback`, and `/check-drift` commands all read from it. Filled in once at scaffold time. Required keys:

- **DEV_BRANCH** (e.g., `staging`, `develop`) — branch that auto-deploys to dev
- **PROD_BRANCH** (e.g., `main`, `production`) — branch that auto-deploys to prod
- **DEV_URL** — full URL of the dev environment
- **PROD_URL** — full URL of the production environment
- **LOCAL_DEV_COMMAND** (e.g., `npm run dev`, `cargo run`, `python manage.py runserver`)
- **LOCAL_DEV_URL** (e.g., `http://localhost:3000`)
- **DEV_DEPLOY_TRIGGER** — one of: `auto-on-push` | `workflow-dispatch` | `tag-release` | `manual`
- **DEV_DEPLOY_DETAILS** — platform specifics (e.g., "Vercel project: project-dev. Dashboard: <url>")
- **PROD_DEPLOY_TRIGGER** — same options as DEV_DEPLOY_TRIGGER
- **PROD_DEPLOY_DETAILS** — platform specifics
- **PROMOTION_MODEL** — one of: `merge` (DEV_BRANCH merges into PROD_BRANCH) | `tag` (tag a commit on PROD_BRANCH) | `trunk-only` (single branch, release via workflow)

**`/docs/QA_GOLDEN_PATHS.md`**
Defines the critical user journeys this project must protect — the workflows that, if broken, mean the product is broken regardless of what passing tests say. Each golden path is numbered (GP-1, GP-2, ...) and gets its own section with: Path Name, User Role, Trigger, Step-by-Step Flow, Success Criteria, Common Failure Modes, Compliance Considerations (only if the project's compliance posture in `TECH_ARCHITECTURE.md` requires them — e.g., HIPAA, PCI, SOC 2). The `/qa` command tests features against these golden paths and produces manual test plans. For simple projects this might be one or two paths; for regulated software it might be a dozen. Note: this doc is optional for very simple projects — `/qa` and the `qa` agent can be skipped if golden paths aren't formalized.

**`/docs/SESSION_TRANSITION.md`**
Sections: Last Session Summary, Completed This Session, Currently In Progress, Blocked / Needs Decision, Next Recommended Actions, Open Questions for Next Session
Note: This file is overwritten at the end of every session by the doc-writer agent. The `/session-end` command also appends any unusual git drift findings (e.g., PROD_BRANCH ahead of DEV_BRANCH) to "Next Recommended Actions" so they surface in the next `/session-start` briefing.

**`/docs/LESSONS_LEARNED.md`**
Sections: How to use this file, then a template entry with: Date, Discovery, Why It Matters, How to Apply
Note: This file grows by appending — never overwrite existing entries.

**`/docs/DECISION_LOG.md`**
Sections: How to use this file, then a template entry with: Date, Decision, Alternatives Considered, Rationale, Owner
Note: Append only — never overwrite existing entries. Deploys, rollbacks, and P0 overrides are auto-logged here by the `/deploy` and `/rollback` commands.

**`/docs/KNOWN_ISSUES.md`**
Sections: Active Issues table (ID, Title, Severity, Status, Owner, Notes), Resolved Issues table
Severity levels: P0 Critical / P1 High / P2 Medium / P3 Low
Note: The `/deploy` command reads this file and blocks production deploys when active P0s are present (override requires a typed reason that gets logged to `DECISION_LOG.md`). The `/rollback` command auto-appends a P0 entry capturing whatever caused the rollback.

**`/docs/DOCS_INDEX.md`**
A table listing all 11 docs with: Filename, Purpose, Who Updates It, Update Frequency

---

### PART 2 — Create `.claude/agents/` with these 7 agent files

Each file must have valid YAML frontmatter (`name`, `description`, `tools`) followed by a detailed system prompt. The agents should reference the `/docs` folder for context.

**`.claude/agents/architect.md`**

- tools: `read_file`
- Role: Evaluates proposed features and structural changes against `/docs/TECH_ARCHITECTURE.md`, `/docs/PRD.md`, and `/docs/DECISION_LOG.md`
- Always reads those three docs before evaluating
- Returns: `GO`, `NOGO`, or `NEEDS MORE INFO` with reasoning, risks, and suggested approach
- Flags any security, compliance, or dependency conflicts explicitly

**`.claude/agents/developer.md`**

- tools: `read_file`, `write_file`, `execute_bash`
- Role: Implements features following established patterns in the codebase
- Always reads the relevant existing code and `/docs/TECH_ARCHITECTURE.md` before writing
- Summarizes what changed and why after each implementation
- Flags any security or compliance surface area that was modified

**`.claude/agents/reviewer.md`**

- tools: `read_file` ONLY — never writes or modifies files
- Role: Audits code and returns a prioritized issue list
- Severity: P0 Critical (security/data loss), P1 High (broken logic), P2 Medium (pattern violations), P3 Low (style)
- Ends every review with: `READY TO COMMIT` or `FIX REQUIRED (N blocking issues)`

**`.claude/agents/explorer.md`**

- tools: `read_file`, `execute_bash`
- Role: Scans the codebase and docs cheaply to answer orientation questions — never writes
- Session start: reads `SESSION_TRANSITION.md`, `KNOWN_ISSUES.md`, and `IMPLEMENTATION_PLAN.md` (including the Backlog section) and returns a concise briefing (under 20 lines)
- Briefing covers: completed last session, in progress, blocked, recommended next action — and surfaces any **recently captured or high-priority backlog items** worth considering for this session (does not list the full backlog)
- Output style: bullets, concise, no unnecessary explanation

**`.claude/agents/doc-writer.md`**

- tools: `read_file`, `write_file`
- Role: Maintains the `/docs` folder
  - Updates `SESSION_TRANSITION.md` after every session
  - Appends to `LESSONS_LEARNED.md` and `DECISION_LOG.md`
  - Updates `KNOWN_ISSUES.md`
  - Manages the **Backlog** table in `IMPLEMENTATION_PLAN.md` (appending new entries from `/add-idea`, marking items completed)
- Rule: Never deletes existing content — always appends or updates in place
- `DECISION_LOG.md` entries require: date, decision, alternatives considered, rationale
- `LESSONS_LEARNED.md` entries require: what was discovered, why it matters, how to apply it
- Backlog entries require: unique ID (B-NNN), one-line description, source, bucket, notes, captured date

**`.claude/agents/database.md`**

- tools: `read_file`, `write_file`, `execute_bash`
- Role: Handles all database schema, migration, and query work
- Always reads existing migration files before proposing changes
- Always reads `/docs/TECH_ARCHITECTURE.md` Security & Compliance section before proposing schema changes
- For relational/SQL projects: never DROP columns — use nullable ALTER or new tables. Propose migrations as versioned SQL files (e.g., `003_add_auth_fields.sql`). Always include rollback SQL with every migration.
- For NoSQL/schemaless projects: document schema expectations in code or migration scripts; flag breaking changes explicitly
- Sensitive or regulated fields (per the project's compliance posture in `TECH_ARCHITECTURE.md` — PII, PHI, payment data, credentials, etc.) must be flagged in comments

**`.claude/agents/qa.md`**

- tools: `read_file` ONLY — never writes or modifies files
- Role: Produces manual test plans for features against the project's golden paths
- Always reads `/docs/QA_GOLDEN_PATHS.md` and `/docs/TECH_ARCHITECTURE.md` Security & Compliance section before evaluating
- For a specific feature: identifies which golden paths the feature touches, produces a step-by-step test plan covering happy path → edge cases → regression risks → compliance checks (only if the project's compliance posture requires them), flags untested or fragile scenarios
- For a full review: assesses coverage across all golden paths and identifies the highest-risk untested scenarios
- Returns a verdict: `PASS`, `NEEDS ATTENTION`, or `BLOCKED`
- Note: This agent produces test plans only — it does not execute tests. The human runs the manual tests.

---

### PART 3 — Create `.claude/commands/` with these 11 command files

The command set is organized in three groups: **session management** (`session-start`, `session-end`, `ask-architect`, `add-idea`), **build** (`implement`, `qa`, `pre-commit`), and **ship** (`commit`, `deploy`, `rollback`, `check-drift`).

**Branch model and deploy mechanics:** All ship commands read from the Deploy Config section of `INFRASTRUCTURE.md` (DEV_BRANCH, PROD_BRANCH, deploy triggers, PROMOTION_MODEL). Commands are stack-agnostic — they push to git and hand off to whatever mechanism the project configured. If Deploy Config is missing or incomplete, ship commands refuse to run and prompt the user to fill it in.

#### Session management commands

**`.claude/commands/session-start.md`**
Instructs the `explorer` agent to read `SESSION_TRANSITION.md`, `KNOWN_ISSUES.md`, and `IMPLEMENTATION_PLAN.md`, then return a briefing covering: what was completed last session, what's in progress, what's blocked, the recommended next action, and any recently captured or high-priority backlog items worth surfacing. Briefing stays under 20 lines.

**`.claude/commands/session-end.md`**
Runs:

1. `reviewer` — quick scan of anything modified this session
2. `doc-writer` — update `SESSION_TRANSITION.md` with what was completed and what's next
3. `doc-writer` — append new discoveries to `LESSONS_LEARNED.md`
4. `doc-writer` — log any architectural decisions to `DECISION_LOG.md`
5. `doc-writer` — mark completed tasks in `IMPLEMENTATION_PLAN.md`
6. **Light git drift check:** Run `git log PROD_BRANCH..DEV_BRANCH --oneline` and `git log DEV_BRANCH..PROD_BRANCH --oneline` (using values from Deploy Config). If PROD_BRANCH is ahead of DEV_BRANCH (i.e., `DEV_BRANCH..PROD_BRANCH` is non-empty), this is unusual — append a note to `SESSION_TRANSITION.md` under "Next Recommended Actions" recommending a back-merge of PROD_BRANCH into DEV_BRANCH before the next `/deploy`. Routine drift (DEV_BRANCH ahead of PROD_BRANCH with WIP) gets a one-line mention in "What's Next" and is not flagged as an action.

Returns a one-paragraph session summary ready to use as a commit message. Stays advisory — does not auto-commit. This preserves the option to end a session with uncommitted WIP.

**`.claude/commands/ask-architect.md`**
Accepts `$ARGUMENTS` as a design question or proposal.
Passes it directly to the `architect` agent along with the relevant context from `/docs`.
Use for quick architecture questions without running a full `/implement` workflow.
Returns: `GO`, `NOGO`, or `NEEDS MORE INFO` with reasoning. Does not invoke developer, reviewer, or doc-writer — purely advisory.

**`.claude/commands/add-idea.md`**
Accepts `$ARGUMENTS` as a one-line idea description.
Workflow:

1. If `$ARGUMENTS` is empty, prompt the user for a description.
2. Prompt for source (e.g., "stakeholder call 4/22", "own idea", "team feedback") — defaults to `unspecified` if skipped.
3. Prompt for bucket: `Tiny tweak` (under 30 min, no architectural impact) / `Feature` (default — an hour+ or touches multiple files) / `Architectural` (changes how the system works, needs `/ask-architect` before scheduling).
4. Prompt for optional notes (blockers, context, related items).
5. `doc-writer` agent reads `IMPLEMENTATION_PLAN.md`, finds the highest existing `B-NNN` ID in the Backlog table, increments by 1, and appends the new entry with today's date.
6. If the Backlog section doesn't exist yet, create it.

Returns confirmation with the new ID. Bucket-specific reminders: Architectural ideas should run through `/ask-architect` before scheduling; Tiny tweaks are good batch material.

This command does **not** evaluate the idea, schedule it, or write to `DECISION_LOG.md` or `SESSION_TRANSITION.md`. Pure capture.

#### Build commands

**`.claude/commands/implement.md`**
Accepts `$ARGUMENTS` as the feature/task to implement.
Workflow in order:

1. `explorer` — find relevant existing code
2. `architect` — validate approach (must return GO to proceed)
3. `database` — review schema changes if applicable
4. `developer` — implement
5. `reviewer` — audit (P0/P1 must be resolved before continuing)
6. `qa` — produce a manual test plan against golden paths
7. `doc-writer` — update `SESSION_TRANSITION.md` and any relevant docs

After `/implement` completes, the developer is expected to test locally (using LOCAL_DEV_COMMAND from Deploy Config) before running `/commit`.

**`.claude/commands/qa.md`**
Accepts `$ARGUMENTS` as a feature name or description (optional).
Workflow:

1. `explorer` — identify all files involved in the feature being tested
2. `qa` — produce a test plan against the project's golden paths

If a feature is named: identifies which golden paths (GP-1 through GP-N) the feature touches and produces a specific test plan covering happy path → edge cases → regression risks → compliance checks (only if the project's compliance posture requires them).

If no feature is named: runs a full golden path review across all paths and lists the highest-risk untested scenarios.

Returns a verdict: `PASS`, `NEEDS ATTENTION`, or `BLOCKED`. The human is responsible for executing the manual test steps — `qa` produces the plan, not the test results.

**`.claude/commands/pre-commit.md`**
Runs:

1. `reviewer` — full audit of recently modified files
2. `doc-writer` — verify `SESSION_TRANSITION.md` is current

Returns either `✅ READY TO COMMIT` or `❌ FIX REQUIRED` with a list of blocking issues. Pure audit — no side effects. Used standalone, and also called internally by `/commit` and `/deploy`.

#### Ship commands

**`.claude/commands/commit.md`**

Purpose: Get vetted code from the local machine to DEV_BRANCH (or a feature branch), where the configured DEV_DEPLOY_TRIGGER ships it to DEV_URL.

Workflow:

1. Read Deploy Config from `INFRASTRUCTURE.md`. If missing or incomplete, refuse and ask the user to fill it in.
2. Run `/pre-commit` internally (`reviewer` audit + `doc-writer` SESSION_TRANSITION check). If `❌ FIX REQUIRED` → stop, show blockers, exit. Do not allow override.
3. If `✅ READY TO COMMIT` → continue.
4. Check current git branch. **Refuse to proceed if on PROD_BRANCH** — print an error explaining that production deploys go through `/deploy`, not `/commit`. If on DEV_BRANCH or a feature branch, continue.
5. Run `git status` and show the user the list of files about to be staged.
6. Run `git add -A`.
7. Generate a commit message from the doc-writer's session context, OR accept one passed as `$ARGUMENTS`.
8. Show the proposed commit message to the user and require explicit confirmation before committing.
9. On confirmation: run `git commit -m "<message>"` followed by `git push origin <current-branch>`.
10. Return: commit SHA, branch name, and a deploy-trigger-aware reminder:
    - If DEV_DEPLOY_TRIGGER is `auto-on-push`: "New build will appear at DEV_URL shortly. Monitor at DEV_DEPLOY_DETAILS."
    - If `workflow-dispatch`: print the workflow invocation command and remind the user to run it.
    - If `tag-release`: note that no deploy fires until a tag is pushed.
    - If `manual`: print the operator command from DEV_DEPLOY_DETAILS.

Guardrails:

- Never commits directly to PROD_BRANCH — refuses outright with a pointer to `/deploy`.
- Never force-pushes under any circumstance.
- Stops cold if `/pre-commit` audit fails — does not allow override.
- Never bypasses the confirmation step.

**`.claude/commands/deploy.md`**

Purpose: Promote tested code from DEV_BRANCH to PROD_BRANCH (or equivalent, per PROMOTION_MODEL), then trigger or hand off to the configured production deploy mechanism. This is the highest-risk command and the workflow reflects that.

Pre-flight checks (all must pass before any state-changing action):

1. Read Deploy Config. If missing, refuse.
2. Confirm the user is on the correct branch:
   - `merge` model: must be on DEV_BRANCH.
   - `tag` / `trunk-only` models: must be on PROD_BRANCH.
3. Confirm there's something to ship:
   - `merge` model: `git log PROD_BRANCH..DEV_BRANCH` must be non-empty. If empty, exit — nothing to deploy.
   - `tag` / `trunk-only`: confirm there are unreleased commits since the last tag/release.
4. **Reverse drift check** (merge model only): Run `git log DEV_BRANCH..PROD_BRANCH`. If PROD_BRANCH has commits DEV_BRANCH doesn't, **block the deploy with no override.** Print a message instructing the user to back-merge PROD_BRANCH into DEV_BRANCH first (`git checkout DEV_BRANCH && git merge PROD_BRANCH && git push origin DEV_BRANCH`), verify the build still works, then re-run `/deploy`. (Skipped for `tag` and `trunk-only` — there's no second branch to drift.)
5. Read `KNOWN_ISSUES.md` and parse the Active Issues table.
6. **P0 gate:** If any active P0 issues exist, display them and block the deploy. To override, the user must type the exact phrase: `override P0: <reason>`. Without this exact format, exit. If override is provided, capture the reason for logging.
7. Run `/pre-commit` audit one final time as a last gate. If `❌ FIX REQUIRED`, exit.

Confirmation step:

8. Display a summary to the user containing:
   - The list of commits being promoted
   - The list of files changed
   - Any P0 overrides captured in step 6
   - The target environment: PROD_BRANCH → PROD_URL via PROD_DEPLOY_TRIGGER
9. Require explicit typed confirmation. The user must type `yes deploy to production` exactly. Anything else exits without action.

Promotion (varies by PROMOTION_MODEL):

**If `merge`:**
10. `git checkout PROD_BRANCH && git pull origin PROD_BRANCH`
11. `git merge DEV_BRANCH --no-ff`
12. `git push origin PROD_BRANCH`

**If `tag`:**
10. Prompt for version (suggest semver bump from latest tag).
11. `git tag -a vX.Y.Z -m "<commit message>"`
12. `git push origin vX.Y.Z`

**If `trunk-only`:**
10. Already on PROD_BRANCH. Push pending commits if not already pushed.
11. (No tag/merge needed — release is workflow-driven.)

Trigger handoff (after promotion):

13. Based on PROD_DEPLOY_TRIGGER:
    - `auto-on-push`: nothing more to do; tell the user where to watch the build (PROD_DEPLOY_DETAILS dashboard URL).
    - `workflow-dispatch`: invoke (e.g., `gh workflow run deploy-prod.yml -f sha=<sha>`) or print the command and wait for the user.
    - `tag-release`: the tag push already triggered it; point to the build URL.
    - `manual`: print the operator command and wait for the user to confirm it ran. Capture their confirmation in the log.
14. Append a new entry to `DECISION_LOG.md` with: date/time, deploy SHA, list of commits shipped, owner, P0 override reason (if any), promotion model used, trigger mechanism.
15. Return user to DEV_BRANCH (or stay on PROD_BRANCH for `trunk-only`).

Guardrails:

- Never force-pushes. Uses `--no-ff` merge (where applicable) to preserve history.
- **Reverse drift check has no override** — PROD_BRANCH ahead of DEV_BRANCH is always a back-merge requirement (merge model only).
- Never proceeds past the P0 gate without a typed reason.
- Never proceeds past the typed confirmation step without the exact phrase.
- All deploys (including overrides) are logged to `DECISION_LOG.md` automatically.

**`.claude/commands/rollback.md`**

Purpose: Quickly revert the most recent production deploy when something is wrong. Scope is intentionally limited to the last deploy only.

Workflow:

1. Read Deploy Config. If missing, refuse.
2. Identify the most recent production change (varies by PROMOTION_MODEL):
   - `merge` model: most recent merge commit on PROD_BRANCH (`git log PROD_BRANCH --merges -1`).
   - `tag` model: most recent tag.
   - `trunk-only`: most recent commit (or batch since last release marker).
3. Display: the SHA/tag, the commits included, and roughly when it shipped.
4. Require explicit typed confirmation. User must type `yes rollback production` exactly.
5. Prompt the user for a rollback reason (required, not optional).
6. Execute the rollback (varies by PROMOTION_MODEL):
   - **`merge` model:** `git checkout PROD_BRANCH` → `git pull origin PROD_BRANCH` → `git revert -m 1 <merge-sha> --no-edit` → `git push origin PROD_BRANCH`
   - **`tag` model:** prompt for the previous tag to redeploy; push that tag (or use platform-specific "redeploy previous tag" feature).
   - **`trunk-only`:** `git revert <sha>` for the offending commit(s), then `git push`.
7. **Platform-specific instant-rollback note:** If PROD_DEPLOY_DETAILS indicates a platform with instant-rollback (Vercel, Netlify, Cloudflare Pages, etc.), inform the user that for fastest recovery they can also use the platform's native instant-rollback feature in addition to the git revert.
8. Append an entry to `DECISION_LOG.md` with the rollback details.
9. Append a new P0 entry to `KNOWN_ISSUES.md` capturing whatever caused the rollback.
10. **Back-merge prompt** (merge model only): Prompt the user to back-merge PROD_BRANCH into DEV_BRANCH so the next `/deploy` doesn't get blocked by the reverse-drift check. If user declines, warn that next `/deploy` will be blocked.
11. Return user to DEV_BRANCH (or PROD_BRANCH for `trunk-only`) with a summary.

Guardrails:

- Never force-pushes. Uses `git revert` to preserve full history.
- Only ever rolls back the most recent change.
- Auto-logging to both `DECISION_LOG.md` and `KNOWN_ISSUES.md` is non-optional.
- Back-merge prompt is non-optional (merge model), but the user can decline (with warning).

**`.claude/commands/check-drift.md`**

Purpose: Diagnostic, on-demand check for git drift between DEV_BRANCH and PROD_BRANCH. **Report-only** — does not write to any doc, does not block any action.

Workflow:

1. Read Deploy Config. If PROMOTION_MODEL is `tag` or `trunk-only` (single-branch), report "single-branch model — no branch drift to check" and exit.
2. `git fetch origin`
3. Run `git log PROD_BRANCH..DEV_BRANCH --oneline` and `git log DEV_BRANCH..PROD_BRANCH --oneline`
4. Build a report showing:
   - DEV_BRANCH ahead of PROD_BRANCH (normal if WIP present)
   - PROD_BRANCH ahead of DEV_BRANCH (unusual — needs back-merge)
   - Severity indicator: ✅ in sync / ℹ️ dev-ahead only / ⚠️ prod-ahead detected
5. If PROD_BRANCH is ahead, suggest the back-merge command.

Scope: git drift only. Does NOT check schema drift between dev and prod databases, environment variable parity between deploy targets, or dependency version drift. Those are separate concerns.

Guardrails:

- Read-only. Never modifies branches, never commits, never pushes.
- Never blocks any subsequent action — purely diagnostic.
- The actual gates against drift live in `/deploy` (which blocks on prod-ahead).

---

### Typical end-to-end workflow

A normal feature shipping from idea to production looks like this (using your Deploy Config values):

```
/session-start                    → briefing on where you left off + relevant backlog items
/implement <feature>              → explorer → architect → database (if schema) → developer
                                    → reviewer → qa → doc-writer
[LOCAL_DEV_COMMAND — test locally] → fastest feedback loop, iterate until satisfied
/commit                           → audits, then commits + pushes to DEV_BRANCH
                                    DEV_DEPLOY_TRIGGER fires → new build at DEV_URL
[manual QA on DEV_URL]            → verify in real deploy environment with dev DB
/deploy                           → branch + drift + P0 + audit gates → typed confirmation
                                    → promotes per PROMOTION_MODEL → PROD_DEPLOY_TRIGGER fires
                                    → new build at PROD_URL. Logged to DECISION_LOG.md
/session-end                      → updates docs, light drift check, generates summary
```

**Anytime callouts** (run whenever, doesn't interrupt the main flow):
- `/add-idea <description>` — capture a new idea or feature request to the Backlog
- `/ask-architect <question>` — quick design Q&A without running `/implement`
- `/check-drift` — diagnostic git drift report between DEV_BRANCH and PROD_BRANCH

If something breaks in production: `/rollback` reverts the last deploy, logs a P0 to `KNOWN_ISSUES.md`, and (for merge model) prompts a back-merge of PROD_BRANCH into DEV_BRANCH. Fix forward in the next session.

The three test environments, in order of speed and isolation:

1. **LOCAL_DEV_URL** — fastest feedback loop. No deploy required. 90% of testing happens here.
2. **DEV_URL** — dev deploy after `/commit`. Catches what local can't (env vars, build failures, edge runtime quirks).
3. **PROD_URL** — production. Reached only via `/deploy`.

---

### Final command set summary

| Group | Command | Purpose |
|---|---|---|
| Session | `/session-start` | Briefing on where you left off |
| Session | `/session-end` | Wrap-up, light drift check, generate commit message |
| Session | `/ask-architect` | Quick design Q&A — advisory only |
| Session | `/add-idea` | Capture new ideas to the Backlog |
| Build | `/implement` | Full feature implementation workflow |
| Build | `/qa` | Manual test plan against golden paths |
| Build | `/pre-commit` | Audit only — no side effects |
| Ship | `/commit` | Audit + push to DEV_BRANCH |
| Ship | `/deploy` | Promote DEV_BRANCH → PROD_BRANCH (per PROMOTION_MODEL) with full safety gates |
| Ship | `/rollback` | Revert the last production deploy + back-merge prompt |
| Ship | `/check-drift` | Diagnostic git drift report (read-only) |

**11 commands, 7 agents, 11 docs.** Each command does one thing well. Each has clear guardrails. Together they cover the full lifecycle of an idea from "thought I just had" to "running in production" — and back, if something goes wrong.

For day-to-day usage patterns, scenarios, and worked examples, see the companion **DAILY_PLAYBOOK.md**.

---

### After scaffolding is complete

Once all files are created, confirm the structure and then ask me:

1. What is the project name and one-sentence description?
2. What problem does it solve and who is the user?
3. What's the tech stack (or should we decide together)?
4. What's the deploy setup? Specifically: branch names (DEV_BRANCH, PROD_BRANCH), prod and dev URLs, what triggers deploys (Vercel auto-deploy, GitHub Actions workflow, manual command, tag-based release), and which PROMOTION_MODEL fits (`merge`, `tag`, or `trunk-only`)? (Used to populate INFRASTRUCTURE.md Deploy Config.)
5. Are there any compliance, security, or regulatory constraints (HIPAA, PCI, SOC 2, GDPR, etc.) I should know about upfront? (Used to populate TECH_ARCHITECTURE.md Security & Compliance section, which the database and qa agents read from.)
6. What are the critical user journeys (golden paths) that must never break? (Used to populate `QA_GOLDEN_PATHS.md` — if the project is simple, we can skip this and the `/qa` workflow.)

Use my answers to populate `/docs/PRD.md`, `/docs/TECH_ARCHITECTURE.md`, `/docs/INFRASTRUCTURE.md`, and `/docs/QA_GOLDEN_PATHS.md` with real content, then update `/docs/DOCS_INDEX.md` and `/docs/README.md` accordingly.
