# Claude Design runbook: Capsule v0.6

The build sequence for the Capsule prototype in Claude Design, using the v0.6 design
system with the wireframe PDF as the visual acceptance target.

## Session setup (do this first)

1. Start a **fresh Claude Design project** (v0.6 changes Home, retires Rooms, adds nine
   modules; a clean project avoids reconciling against the old structure).
2. **Attach two files** to the project:
   - `design-system.md` (v0.6) as the contract source.
   - `CAI.pdf` (the wireframes and flow) as the visual acceptance target.
3. Run the prompts below in order. Pause at the three checkpoints (marked CHECKPOINT).

The binding rule, repeated in every prompt: **build reusable components from the
contracts, and match the attached wireframe screen for screen. Do not improvise content
or layout the wireframe does not show.** Two correction phrases to reuse whenever output
drifts: "Build this as a reusable component, not a one-off screen" and "Match the attached
wireframe (CAI.pdf); do not improvise."

---

## Prompt 1: Visual language and tokens

> Using the Layer 0 and Layer 0b visual language in the attached design-system.md, build
> the foundational style layer for a mobile-first app. Implement: the defined grey ramp
> (grey-0 through grey-900 exactly as specified), a single swappable `accent` token, the
> `consent` color kept distinct, hairline separators (grey-200, 0.5 to 1pt), the type ramp
> with 17pt body, the 4pt spacing grid, and 44pt minimum touch targets. Enforce the visual
> rules: hierarchy from weight, size, and hairlines, not fills or shadows; no drop shadows;
> greyscale-plus-single-accent only. Honor Dynamic Type and prefers-reduced-motion. Build
> this as a reusable token/style layer every later component references, not a screen.
> Confirm the grey values you used and their contrast ratios on white.

CHECKPOINT 1: verify the greys match the spec and body text (grey-500+) clears 4.5:1. The
whole aesthetic rides on this layer; a wrong ramp poisons everything downstream.

---

## Prompt 2: ConnectionConfirmModule (consent validation)

> Build ConnectionConfirmModule from its full Layer 1 contract in the attached
> design-system.md, and match its wireframe in CAI.pdf (the "Connecting with [app/data
> source]" / "Assistant setup: detail" screen). Render Home helper connecting to Google
> Calendar with the rights "see, edit and modify events", and the assurance text preserved:
> that the data is used only to improve the assistant, kept encrypted and safe, and
> revocable at any time. Requirements: Confirm access and Skip for later are equal visual
> weight (Skip is not a faded secondary); the encryption and revocability are readable
> content, not fine print; use the consent color with a clear heading; meet WCAG 2.2 AAA
> (7:1 contrast, works at 200% text size); follow the minimal visual language (hairlines,
> greyscale plus accent, iOS form). Build it as a reusable component I can reuse with
> different apps and data sources, not a one-off screen. After building, tell me which
> parts of the contract were straightforward and which were ambiguous or underspecified,
> and confirm it matches the wireframe.

CHECKPOINT 2: this is the most important consent surface and its microcopy is
load-bearing. Confirm the assurance text reads clearly, Skip is equal weight to Confirm,
and Claude Design reports no blocking ambiguity. If it flags gaps, bring them back to tune
the contract (as we did with the chat canvas) before proceeding.

---

## Prompt 3: EfficiencyMeterModule (build for iteration)

> Build EfficiencyMeterModule from its Layer 1 contract in the attached design-system.md,
> matching how it appears in the CAI.pdf setup screens (a ring that fills 0 to 100% with
> the percentage shown, per assistant). Build it as a reusable component that accepts these
> props: assistant name, value (0 to 100), optional recent delta, and a variant (setup =
> large hero ring, inline = small beside a name, moment = compact). Requirements: the
> percentage is always shown as text next to the ring (not ring-only); the fill is accent
> or grey, never a red-to-green spectrum; it reads in greyscale alone; changes announce
> politely to assistive tech. Then, so I can iterate it, render a single test board showing
> the SAME component in all three variants and at five values (0, 35, 55, 95, 100) side by
> side, each labelled, so I can compare the visual treatment across states in one view.
> Build the component once; the board just instantiates it repeatedly.

CHECKPOINT 3 (iteration surface): the test board is your efficiency-meter playground.
Iterate here before it is used elsewhere. Things to try by asking Claude Design to adjust
the board: ring thickness, where the percentage sits, how the just-improved delta is
emphasized, the accent-versus-grey fill treatment, the compact variant's minimum legible
size. Lock the treatment you like, then it propagates everywhere the meter is used. To
iterate: "On the efficiency-meter board, try [change] across all five values, keep
everything else." Keep iterating until the meter feels right; it is a central element.

---

## Prompt 4: The core module library

> Build the following modules as reusable components from their Layer 1 contracts in the
> attached design-system.md, each implementing its typed props, states, variants, and A11y
> block (WCAG 2.2 AA, AAA for consent surfaces), in the v0.6 visual language. Do not
> assemble screens yet, and keep each component free of journey-specific logic. Build:
> UniversalCanvasModule, TopNavModule (iOS nav-bar form, home/chat/section variants),
> BottomNavModule (iOS tab-bar form, three tabs), MomentModule, TopicVisualizationModule,
> AgentActivityLogModule (scoped all/assistant), AssistantsModule, ConnectedAppsModule
> (AAA), PersonalizeAssistantModule (AAA, placements setup/chat/moment), AssistantSetupModule,
> ProgressIndicatorModule, ChatModule, ChatParticipantIndicatorModule, ChatBoxModule,
> AssistantMessageModule, UserMessageModule, StreamingMessageModule, FollowUpButtonsModule,
> ConversationStartersModule, and ChatCanvasModule. Reuse the EfficiencyMeter and
> ConnectionConfirm you already built. Confirm the full component list when done.

CHECKPOINT: ask Claude Design to list every component it now has, and confirm the library
is complete before composing screens. A missing component here is a missing screen later.

---

## Prompt 5: The entry and setup flow

> Compose these screens from the components you built, adding no new components, matching
> the attached wireframe (CAI.pdf) screen for screen: SignIn (email, phone, "Continue
> without signing in", troubleshoot), Onboarding (choose life topics with the six "How you
> want Capsule to help" framings, plus the progress indicator), OnboardingAssistantSetup
> (the AssistantSetup hero with EfficiencyMeter at 35%, PersonalizeAssistant connect/grant
> nudges, progress), ConnectingToSource (streaming animation), AssistantSetupDetail (the
> ConnectionConfirm consent screen), and AssistantSetupComplete (efficiency raised to 55%,
> "start a conversation"). Use the templates in Layer 2 of the design-system. Confirm each
> matches its wireframe and which components you reused.

---

## Prompt 6: Home (Moments feed) and the primary tabs

> Compose the three primary tab screens from the Layer 2 templates, matching CAI.pdf, no
> new components. Home: a feed of MomentModules (health status, home status, each tappable
> to start a chat with its agent; at least one moment embedding a PersonalizeAssistant
> efficiency nudge, one at max efficiency), TopNav (Start Chat, small logo, bell), bottom
> nav. Assistants: the roster (Home helper, Health coach, Training buddy, Companion,
> Financial coach, each with a settings gear, plus Explore more) and the all-agent activity
> log. You: my life topics plus Connected apps (Google Calendar, Gmail, eBay, bank app,
> with per-source toggles and an Add access button opening an iOS-share-style sheet). All
> three share the same bottom nav (Home, Assistant, You). Confirm each matches its
> wireframe.

---

## Prompt 7: Notifications and the assistant detail screens

> Compose these from the templates, matching CAI.pdf, no new components: Notifications
> (opens from the top-nav bell): the tasks module (including efficiency-improvement tasks
> like "confirm task completion", "give consent for Home Helper", "share medical data")
> plus the all-agent activity log ("assistants are working on these tasks..."). Individual
> assistant view: one assistant's activity log plus its connected apps. Assistant settings:
> the AssistantSetup hero with EfficiencyMeter, PersonalizeAssistant nudges, and the
> assistant's access/connections. Confirm each matches its wireframe.

---

## Prompt 8: The chat surface and a journey

> Compose the Chat template from Layer 2, matching CAI.pdf, no new components: TopNav (chat
> variant: back, small logo, bell), the ChatParticipantIndicator (surfaced when multiplayer),
> the ChatModule streaming conversation starters when empty, assistant and user messages, a
> ChatCanvasModule for rich content, follow-up prompts, and a dismissible PersonalizeAssistant
> nudge, plus the chat box (with upload, voice, send). Populate it with one real journey:
> [Becca's recovery-day chat, or your choice], using its Layer 3 config. This exercises the
> canvas (progress variant), the participant indicator, and the mid-chat nudge together.
> Confirm it matches the wireframe.

---

## Prompt 9: Wire the navigable prototype (static, GitHub-Pages-ready)

> Wire the screens into a single navigable prototype matching the flow in CAI.pdf. Reuse the
> screens and components; build no new screens; add only navigation. Three layers:
> (1) entry flow Sign in -> Onboarding -> Assistant setup -> Home; (2) persistent bottom
> tabs Home / Assistant / You; (3) the top-nav bell opening Notifications, and drill-in from
> Home moments or the Assistants roster into a chat, with the top-nav back returning. For the
> demo, land on Home with the entry flow reachable but not gating. Keep the framework-agnostic,
> dependency-free architecture: HTML files rendered via the vanilla-JS runtime (support.js),
> inline styles, `<sc-*>` and `<dc-import>` tags, navigation state in the runtime, no external
> dependencies. Structure for static hosting on GitHub Pages: a single index.html entry, all
> paths relative (served from a /reponame/ subpath, no leading-slash absolute paths),
> everything runnable by opening index.html. After wiring, give me the file tree, confirm all
> screens and all six journeys are reachable by clicking, and list any absolute paths or
> external dependencies that would break on GitHub Pages.

---

## After the build

- Validate against Layer 5: each built screen should reproduce its wireframe frame. A screen
  you cannot reproduce from modules is a spec gap; report it, do not paper over it with a
  one-off.
- Deploy: new GitHub repo, drop the static files (root or /docs), Settings -> Pages -> deploy
  from that branch/folder, live at username.github.io/reponame/.
- Iterate the efficiency meter anytime via its test board (Prompt 3); locked changes
  propagate to every screen that uses it.
