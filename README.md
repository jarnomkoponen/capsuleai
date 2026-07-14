# Capsule AI — Interactive Prototype

A component-based mobile prototype for Capsule AI, built as Design Components (`.dc.html`).
Near-monochrome iOS-form UI: whites and greys carry the interface, a single swappable
accent for actions and active states, hairline separators, no drop shadows.

## Run it

Everything is static — no build step. Just open `index.html`, which redirects to the
wired prototype (`Capsule.dc.html`). The prototype starts on **Home**; append a hash to
jump anywhere: `#signin`, `#onboarding`, `#chathub`, `#chat`, `#you`, `#notifications`, etc.

## Publish on GitHub Pages

1. Create a new GitHub repository and upload **all files in this folder** (keep the
   structure flat — every file sits at the repo root, exactly as provided).
2. In the repo: **Settings → Pages**.
3. Under **Build and deployment**, set **Source: Deploy from a branch**, pick your
   branch (e.g. `main`) and folder **/ (root)**, then **Save**.
4. Wait ~1 minute. Your prototype is live at
   `https://<your-username>.github.io/<repo-name>/`.

`.nojekyll` is included so GitHub Pages serves the files as-is (no Jekyll processing).

## What's inside

- `index.html` — entry point (redirects to the prototype)
- `Capsule.dc.html` — the wired, navigable prototype (router over all screens)
- `support.js` — Design Component runtime (required; do not edit)
- **Screens:** `SignIn`, `Onboarding`, `OnboardingAssistantSetup`, `AssistantSetupDetail`,
  `Home`, `Assistants`, `You`, `ChatHub`, `Chat`, `Notifications`, `AssistantIndividual`,
  `AssistantSettings` (`.dc.html`)
- **Modules:** the reusable component library (`TopNavModule`, `BottomNavModule`,
  `MomentModule`, `EfficiencyMeterModule`, `ConnectedAppsModule`, `ChatCanvasModule`, …)
- **Reference:** `Capsule Style.dc.html` (tokens + visual language),
  `Component Library.dc.html`, `EfficiencyMeter Board.dc.html`

## Notes

- Each `.dc.html` also opens standalone in a browser for inspection.
- All files must stay in the same folder — screens load their modules and `support.js`
  by relative path.
