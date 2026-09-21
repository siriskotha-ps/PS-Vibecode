# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

_One paragraph an engineer can read in 60 seconds._

This is a clickable prototype that tests one hypothesis: replacing a 12-chart dashboard's first screen with one headline metric, one plain-language takeaway, and one recommended action gets users to act instead of bounce. Baseline being challenged: 12 charts, 60% bounce, 1.3 sessions/user. The app is a TanStack Start (React 19) SPA with six screens — an "Answer" screen (/), a deliberate escape hatch to the full 12-chart dashboard (/full-dashboard), an auth gate (/sign-in), a confirmation step (/confirm-action), a receipt screen (/action-taken), and a session-signal readout (/signal). Everything user-visible is hard-coded, deterministic prototype data; the only "real" data is the client-side event store that tracks how each session ends (acted / escaped / backed out / blocked / no decision). There is no database, no API, no real auth — by design. A build is on the branch; bun install && bun run dev and you're live in under a minute.

## Architecture (plain language)

- **Frontend:** TanStack Start v1 (React 19 + Vite 7 + Tailwind v4, Edge/Cloudflare Workers target). Every URL is a thin route file in src/routes/ that does three things: sets head metadata, imports its screen, renders it. No data loading in routes — loaders are deliberately avoided because everything is static.  Screens & data. Code is grouped by feature under src/features/, one folder per PRD screen name:  src/features/   answer/             → TheAnswerScreen + EvidencePanel + answer.data.ts   recommended-action/ → SignInToContinueScreen, ConfirmRecommendedActionScreen,                         ActionTakenScreen + skeletons + recommended-action.data.ts   full-dashboard/     → FullDashboardScreen + DashboardVisuals + data + model   session-signal/     → SessionSignalScreen + session-store.ts Separation rule: .data.ts / .model.ts hold all copy, numbers, and figures; screens only render. answer.data.ts holds the takeaway, quotes, and baselines; recommended-action.data.ts holds the action ("Resend welcome email to 1,180 affected signups") plus the prototype access-control table (getPrototypeAccess() → authorized / denied / failed); full-dashboard.model.ts builds the 12-chart wall's KPIs, funnel, mix, and heatmap deterministically from the filter selections.
- **Backend / data:** session-store.ts — a module-level store with a useSyncExternalStore hook. It records the real behavioral signal: actionIntentAt, authAttempts / authFailures, authenticatedAs, authorized, confirmationOpenedAt, actionConfirmedAt / actionCancelledAt. From these it derives a verdict per session: acted, escaped, backed out, blocked, no decision (+ variants like "verification failed N×"). /signal renders it. This store is the only part intended to survive into a real product. Hard-coded prototype accounts in recommended-action.data.ts: growth@ssrs.inc / sam@ssrs.inc = authorized; viewer@ssrs.inc = signed in but unauthorized; error@ssrs.inc = verification failure. Any password works. Simulated 900 ms auth check and 500 ms confirm-gate delay with skeleton states.
- **Key flows:** / → click recommended action → /sign-in (skeleton → authorized → /confirm-action; denied → inline permission block, no way forward; failed → "Try again") → /confirm-action (gated: nothing renders without authorized=true; Confirm → /action-taken, Cancel → / with only "opened but didn't commit" recorded) → escape hatch at any point via "Show all 12 charts" → /full-dashboard (records escaped, the kill switch) or "See the signal" → /signal.

## What's solid vs. what's duct tape

| Area | State | Notes |
|---|---|---|
| The event store and verdict logic — clean, testable, and the actual point of the prototype. Feature-folder layout with data/display separation; routes are trivial wrappers; URLs match the PRD screen names. All the edge paths the hypothesis depends on: unauthorized state, verification failure, cancel-without-commit, and the escape switch all render correctly and record correctly. Typecheck clean (tsgo --noEmit); all six routes return 200. | solid | _____ |
| Auth is setTimeout + an if-statement. 900 ms fake latency, hard-coded allow-list, any password accepted. Fine for the prototype; the first real integration is replacing getPrototypeAccess() and the store's auth fields. All numbers are literals. 412 signups, ↓31%, 1,180 affected, ~340 trial starts, "4 min" — copied from resendAction where possible, but the dashboard figures come from a deterministic pseudo-model, not a data source. Session store is in-memory only. Refresh wipes it; no persistence, no cross-session aggregation. /signal shows one session's story. Simulated delays are setTimeout in components, not a loader pattern — they exist purely to prove the skeleton states. No tests, no error boundary, no CI beyond the typecheck. | rough | _____ |

## Risks & assumptions for the team

Assumption: the verdict taxonomy survives contact with real users. If people find escape paths we didn't build (browser back, direct URL entry), the signal undercounts. Direct navigation to /confirm-action while signed out silently redirects to /sign-in — intentional, but it means deep links don't pollute the signal; verify that assumption in research.
Risk: prototype data presented as real. The 1,180-affected signups number is invented. Research facilitators must say so out loud; nothing in the UI currently labels itself a prototype (that's the point, for realism).
Risk: no resend failure path. The happy path ends at a receipt. A participant asking "what if the send fails?" will hit a wall — known gap, not built.
Assumption: one user, one session, one browser tab. No multi-device or concurrency story.
Risk: store not persisted means remote research sessions can't be replayed or shared — the "shareable session signal" is still on the unbuilt list.

## How to run it

```
bun install
bun run dev        # dev server, http://localhost:8080
bunx tsgo --noEmit # typecheck (the only gate)
bun run build     # production build (Cloudflare Workers target)
Sign-in walkthrough: growth@ssrs.inc (any password) → authorized → confirm screen → "Confirm resend to 1,180 signups" → receipt. viewer@ssrs.inc → blocked state. error@ssrs.inc → failure + "Try again". To see the kill-switch verdict, go to /full-dashboard from the Answer screen, then /signal.
```
