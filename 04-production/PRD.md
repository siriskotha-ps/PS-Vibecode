# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem

_What user problem does this solve? Tie to the validated hypothesis._

Decision-makers who rely on the SSRS Inc dashboard to understand performance and decide next steps are not acting on it. Today's default landing view shows 12 charts with 5 filters and no stated priority; sessions end without a decision.

Evidence from user research (shown verbatim in the prototype):

"I open it, see twelve charts, and have no idea which one I'm supposed to act on. So I close it." — Marketing manager
"It tells me what happened but never what to do about it. I still export to a spreadsheet to think." — Product lead
"My exec just wants one slide. The dashboard gives me forty widgets instead." — PMM
Baseline reality: 12 charts on the default landing view, 60% bounce rate (sessions under 15s, no interaction), 1.3 average weekly sessions per active user.

Validated hypothesis (what this PRD builds on): A first screen leading with one headline metric, a plain-language takeaway, and one recommended action causes users to act on the dashboard instead of bouncing. The prototype was built to prove or break this, and its success/kill criteria are explicit: we're right when bounce drops and the recommended action is clicked; the concept is killed if users still reach for the full dashboard or an export before they feel ready to act. This PRD assumes the hypothesis holds and scopes the real product around it.

## Users & jobs

- **Primary user:** A decision-maker at SSRS Inc (marketing manager, product lead, PMM) who opens the dashboard to understand what changed and decide what to do next. They are time-poor, not an analyst, and currently bounce or export to a spreadsheet to think.
- **Job to be done:** When I open the dashboard, tell me the one thing that changed, why it happened, and what to do about it — so I can act in under a minute instead of interpreting twelve charts.  Supporting job: when I do need depth, let me reach the full chart view deliberately, not as the default.  Secondary actor: an authorized team member who can execute customer-facing actions (e.g. resending email to customers); not everyone who can read the answer can act on it.

## Scope

- **In:** A first screen ("The Answer") with one headline metric, its direction and time frame, one plain-language takeaway, one supporting explanation, and one recommended action.
Sign-in and authorization gate before any customer-affecting action, with explicit denied and verification-failure states.
A confirmation screen showing what will happen, why it's recommended, who it affects, and expected impact, with explicit Confirm / Cancel choices.
An "action taken" receipt screen: what was done, what to expect, when to check back.
The existing full dashboard (12 charts, 5 filters) retained as a deliberate secondary view, reachable from the answer screen.
Session-level measurement of the decision funnel: time to action, action clicked, sign-in attempted, authorization result, confirmation reached, confirmed vs cancelled, dashboard/export escapes, evidence views.
A visible kill-switch definition: escapes to the full dashboard or export before acting count as the concept failing.
- **Out (explicitly):** Real authentication, user management, or a permission service (prototype hard-codes accounts; production needs a real one, but building it is a dependency, not this initiative's scope).
Real email sending or any customer-facing execution infrastructure.
Live metric pipelines, anomaly detection, or automated root-cause analysis (the prototype's takeaway is hard-coded).
Export-to-spreadsheet functionality.
Mobile-native apps, notifications, scheduled digests, multi-metric answers.
Historical storage of session signals across users (prototype keeps one in-memory session only).

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| 1 | First screen leads with one headline metric, direction, and time frame | Must | Metric, direction arrow and period visible above the fold; no other chart rendered on load |
| 2 | Plain-language takeaway and cause shown next to the metric | Must | One sentence states what happened and why; a second sentence rules out other causes; copy readable at a glance |

## Data & events

_What gets stored, what gets tracked._

Content data (mocked in prototype, must be real in production):

Headline metric value, week-over-week delta, and time frame — prototype hard-codes 412 signups, ↓31%, week of 1–7 September.
Takeaway and cause sentence — prototype hard-codes the stopped welcome-email explanation.
Recommended action parameters — prototype hard-codes 1,180 affected signups, welcome email resend, ~340 trial starts expected back, ~4 minutes to send.
The 12 dashboard charts and 5 filters render deterministic pseudo-data from filter state; no real series exist.
Identity & access (mocked): prototype authentication accepts any password; authorization is a hard-coded allow-list (growth@ssrs.inc, sam@ssrs.inc authorized; others denied; error@ssrs.inc simulates verification failure). Production needs real auth and a real permission model for customer-facing actions.

Events (real in prototype, in-memory only):

action_intent — recommended action clicked on the answer screen
auth_attempt / auth_failure — sign-in submitted / verification failed
authorization_result — allowed or blocked, with identity
confirmation_opened
action_confirmed / action_cancelled
full_dashboard_opened — kill-signal escape
export_clicked — kill-signal escape
evidence_opened
Derived measures: time to first action; verdict = acted / escaped first / backed out / blocked / no decision. Prototype stores one session in browser memory with a manual reset; production needs persistence, per-user attribution, and aggregation across sessions to compute bounce rate and action rate against the 60% / 1.3 baselines.

## Open questions

Where does the takeaway come from in production? The prototype's root cause ("welcome email stopped sending Tuesday") is hard-coded. Real product needs either analyst-authored answers or automated anomaly + cause detection — this is the single biggest scope risk.
How is the one recommended action chosen? Ranking logic, who owns action quality, and what happens when there is no confident recommendation (show nothing? show monitoring state?) are undecided.
What is the real permission model? Which roles may execute customer-facing actions, and does blocking unauthorized users hurt the action-rate metric enough to warrant a "request approval" path instead of a dead end?
How are sessions persisted and attributed? Verdicts are currently per-browser-session and vanish on reload; production needs durable, per-user, aggregatable event storage to prove bounce-rate and action-rate movement.
Does "Export data" exist in the real product? The prototype routes the export click to the signal screen as a kill-signal counter. Whether production offers export at all — and whether using it counts as failure — needs a product decision.
What are the numeric success thresholds? The hypothesis says bounce drops and action clicks rise, but no target (e.g. bounce 60% → 40%, action rate ≥ X%) has been set, nor a test duration or sample size.
What happens when several things change at once? The answer screen assumes one dominant story per period; behavior under multiple simultaneous anomalies is undefined.
