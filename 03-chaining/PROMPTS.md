# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: Improve Action Screens

### Step 1: Expand, build new screens in a strict sequence
```
Build the next phase of this app in a strict sequence:

1. Add a screen **“Sign in to continue” (`/sign-in`)**. Match the existing prototype’s calm, minimal layout, spacing, typography, and visual system. This screen appears when the user clicks the recommended action **“Resend to 1,180 affected signups.”** Explain briefly that authentication is required because the recommended action affects 1,180 customers. Include email/username, password, and a primary **“Sign in and continue”** button. For the prototype, simulate authentication locally—no real authentication backend is required. After sign-in, validate that the user is authorized to perform the resend action. If the user is not authorized, show a clear inline state explaining that they do not have permission to perform this action and do not allow them to proceed.

2. Add a screen **“Confirm recommended action” (`/confirm-action`)**. Match the data density and decision-focused design of the existing **The Answer** screen. Clearly show what will happen before anything is executed: **“Resend the welcome email to 1,180 affected signups?”** Include the reason for the recommendation, expected impact, affected audience, and two choices: **“Confirm resend”** and **“Cancel.”** Do not execute the action until the user explicitly confirms it. Track whether the user confirms or cancels so this becomes part of the session signal.

3. Navigation: write the logic so clicking the recommended action on **The Answer (`/`)** routes unauthenticated users to **Sign in to continue (`/sign-in`)**. Successful authentication and authorization routes the user to **Confirm recommended action (`/confirm-action`)**. Clicking **Confirm resend** routes to the existing **Action Taken (`/action-taken`)** screen. Clicking **Cancel** returns the user to **The Answer (`/`)** without recording the recommended action as completed.

Build these in order so **Sign in to continue** is the anchor for **Confirm recommended action**. Preserve the existing session tracking and extend it to capture authentication attempted, authorization result, confirmation opened, action confirmed, and action cancelled.
```

### Step 2: Behavior, hard-code the states
```
Apply the following logic constraints to the **Sign in → Authorization → Confirm Recommended Action** flow:

* Use a skeleton screen while the **user authentication and authorization state** is being checked. Keep the skeleton brief and match the existing layout so the transition does not cause the page to jump.

* Hard-code an **authorized user** state. When the user enters the prototype credentials and clicks **“Sign in and continue,”** simulate successful authentication and authorization, then route to **Confirm recommended action (`/confirm-action`)**.

* Hard-code an **unauthorized user** state that can be triggered during testing. Show the state: **“You’re signed in, but you don’t have permission to resend emails to customers. Contact an authorized team member to take this action.”** Do not allow the user to continue to the confirmation screen.

* If authentication information is missing, show the empty/validation state: **“Enter your email and password to continue.”** Keep the user on the sign-in screen.

* On simulated authentication or authorization failure, trigger the error state: **“We couldn’t verify your access. Please try again.”** Provide a **“Try again”** action and do not proceed to the confirmation screen.

* On **Confirm recommended action**, hard-code the action details: **1,180 affected signups**, **welcome email resend**, and the expected outcome that most missing trial starts should recover. Do not fetch these values from a backend.

* When the user selects **“Confirm resend,”** simulate a successful action and route to the existing **Action Taken (`/action-taken`)** screen.

* When the user selects **“Cancel,”** return to **The Answer (`/`)** without marking the recommended action as completed.

* Preserve session tracking across the flow. Record whether authentication was attempted, whether authorization succeeded or failed, whether the confirmation screen was reached, and whether the user confirmed or cancelled.

Maintain the same calm, credible design language throughout and tether all behavior strictly to these hard-coded rules. Do not add a real authentication service, API, database, or email integration. This is a behavioral prototype, not a production authentication implementation.
```

### Step 3: Refine, one surgical polish
```
The **Confirm Recommended Action (`/confirm-action`)** screen needs a professional **calm, executive decision-focused** polish.

1. Start by listing the **3 biggest gaps in typography and spacing** compared to the existing **The Answer (`/`)** screen. Focus specifically on heading hierarchy, vertical spacing between decision information, and the visual relationship between the action details and primary CTA.

2. Once you've identified those gaps, resize the headers and update the **confirmation action area** to match the typography, spacing, button proportions, border radius, and visual hierarchy of The Answer screen.

Make **“Confirm resend to 1,180 signups”** feel like the clear primary decision while keeping **Cancel** visually secondary. The affected audience and consequence of the action should be immediately understandable without adding more content.

Don't change anything else in the project or touch the underlying authentication, authorization, navigation, session-tracking, or confirmation logic.
.
```

## Reusable techniques learned

- _____
- _____

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

_____
