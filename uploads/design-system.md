# Capsule AI Design System

**Version:** 0.6 (major update: visual language, assistant-efficiency system, Moments Home, activity/trust layer, iOS nav; Rooms retired into Chat participants)
**Source:** CAI.pdf (FigJam wireframes and module description, Jarno Koponen)
**Status:** v0.6 major update. Full design system for review and Claude Design rebuild.
Locks at 1.0 when Layers 1 to 3 are validated live in Claude Design against the wireframes.

---

## How to read this file

This design system drives four surfaces from one source:

- **Claude (chat):** read the prose contracts to reason about journeys and screens.
- **Claude Design:** read the tokens and module contracts to generate components and screens.
- **Claude Code:** read the typed props and states to build real components.
- **The Capsule app:** inherits all of the above; the vocabulary mirrors the PRD so
  design and platform stay one idea.

Each module below is specified once, in layers that nest: purpose (for humans),
data and props (for code), states and variants (for both), composition (where it
sits), and accessibility (non-negotiable). Read only the layer your surface needs.

---

## The architecture in one paragraph

Every screen is a **UniversalCanvasModule** that contains ordered **module slots**.
Base modules are reusable and modifiable; their content changes dynamically based on
placement, purpose, and journey. Additional modules surface, or replace defaults, based
on user intent and relevance (recent user or agent interactions). A **screen** is a
template of slots; a **journey** is a configuration that selects modules and feeds them
data. This is the visual expression of the PRD's behavioral recipes: a module is the
customer-experience surface of a recipe, and a journey configuration is a recipe instance.

Mapping to atomic design (shared vocabulary for anyone who joins): tokens are the base,
modules are organisms, the universal canvas is the template, journeys are pages.

---

## Behavioral-recipe vocabulary (locked)

| Design term | PRD / platform term | Meaning |
|---|---|---|
| Module | Recipe surface | A reusable UI unit that renders one slice of a recipe |
| Module data contract | Recipe data scope | The typed data a module may receive and display |
| Screen template | Recipe layout | Ordered slots a recipe fills for a given context |
| Journey config | Recipe instance | A named selection of modules plus data for one journey |
| Population rule | Recipe personalization | How a module fills and how slots reorder by relevance |
| Consent moment | Consent grant surface | The UI where a scoped, revocable permission is asked |

---

## Layer 0: Design tokens

All values are PROPOSED starting points grounded in Apple HIG and WCAG 2.2, to be
tuned in Claude Design. None are taken from the source wireframes, which are
grayscale and fidelity-neutral. Tune freely; keep the token structure.

### Spacing scale (4pt base grid, HIG-aligned)
```
space-0: 0     space-1: 4    space-2: 8    space-3: 12
space-4: 16    space-5: 24   space-6: 32   space-7: 48   space-8: 64
```
Default module padding: space-4 (16). Default gap between modules: space-3 (12).

### Type ramp (SF Pro / system font first, Dynamic Type compatible)
```
display  34/41 bold        title    28/34 bold
heading  22/28 semibold    body     17/22 regular   <- base, HIG default
callout  16/21 regular     subhead  15/20 regular
footnote 13/18 regular     caption  12/16 regular
```
Line heights shown as size/leading. Body is 17pt, the HIG baseline for readability.
All type must honor the OS Dynamic Type setting; never hardcode pixel sizes in a way
that blocks scaling to 200 percent (WCAG 1.4.4).

### Color (semantic, not raw; proposed)
```
surface           app background
surface-raised    module / card background
content-primary   primary text
content-secondary supporting text
accent-trust      Capsule's single brand accent (actions, active states)
consent           the distinct consent-moment treatment (see below)
success / warning / danger   status
separator         hairlines
```
Consent moments get their own semantic color, deliberately distinct from accent-trust,
so a permission ask never looks like an ordinary button. Because control is the product,
consent is a first-class visual state, not a variant of primary. The distinction must
hold under colorblindness (verify against deuteranopia and protanopia): a consent color
that reads identical to accent-trust for a colorblind user reintroduces the confusion it
exists to prevent. This is why consent surfaces also carry a text heading ("Permission
needed"): color is never the sole signal.

### Contrast obligations (WCAG 2.2)
- Body and UI text: minimum 4.5:1 (AA). Large text (>=24px or >=19px bold): 3:1.
- **Consent-moment text and controls: target 7:1 (AAA)**, per the locked accessibility
  target below. The one place we exceed AA is the one place comprehension is critical.
- Non-text UI (toggles, focus rings, icons carrying meaning): minimum 3:1 (WCAG 1.4.11).

### Motion
Streaming and transition animation must respect prefers-reduced-motion: replace
movement with a static status label. Never convey status by motion alone (WCAG 2.2.2).

### Touch targets
Minimum 44x44pt hit area (HIG), even when the visual glyph is smaller (WCAG 2.5.8
Target Size). Applies to every icon button in the nav and chat box.

---

## Accessibility target (locked)

**WCAG 2.2 AA across the system; AAA for consent-critical moments.**

Rationale: AA is the right bar for a broad consumer app. But Capsule's core promise is
that the user sees and controls every data access. If a consent prompt, a permission
toggle, or a data-sharing confirmation is not perceivable and operable by someone using
a screen reader, low vision, or switch control, the product's central claim fails for
that person. So the consent surfaces (permission prompts, access toggles, share
confirmations, revoke controls) are held to AAA: 7:1 contrast, no reliance on color
alone, explicit screen-reader announcement of what is being granted and that it is
revocable, and focus placed on the decision.

Every module contract below carries an **A11y** block. It is not optional and not a
later pass.

---

## Layer 1: Module inventory

Modules are grouped by role. Each carries: Purpose, Data (typed props), States,
Variants, Composition, A11y. Names use the `...Module` convention matching the source
labels, PascalCase for code.

### Structural

#### UniversalCanvasModule
- **Purpose:** the root container every screen is built from; holds all other modules
  in an ordered, scrollable, responsive column that scales mobile to wide.
- **Data:** `{ slots: ModuleSlot[]; screenContext: ScreenName; density?: 'comfortable'|'compact' }`
  where `ModuleSlot = { module: ModuleName; props: object; priority: number }`.
- **States:** loading (skeleton slots), ready, empty, error.
- **Variants:** mobile (single column), wide (max content width, centered, optional
  two-column for nav + content).
- **Composition:** the template. Everything else is a child slot. Order is set by the
  screen template but may reorder by `priority` under population rules.
- **A11y:** the scroll container is the main landmark (`role="main"`). Logical DOM order
  matches visual order so screen-reader and focus order are correct. One h1 per screen.

#### TopNavModule
- **Purpose:** persistent top bar; orients the user and exposes global actions.
- **Data:** `{ logo: boolean; startChat?: Action; notifications?: { count: number; action: Action }; back?: Action; title?: string }`.
- **States:** default, with-back (chat and detail screens), with-unread (badge).
- **Variants:** home (Start Chat + logo + notifications), chat (back + title + logo),
  section (back + title).
- **Composition:** first slot in most templates.
- **A11y:** `role="banner"` or a `<header>`; icon buttons have text labels; notification
  count announced ("3 unread"); back button reachable first in focus order.

#### BottomNavModule
- **Purpose:** primary section switch: Home, Assistants, You.
- **Data:** `{ items: { label: string; icon: string; screen: ScreenName; current: boolean }[] }`.
- **States:** per-item selected / unselected.
- **Variants:** three-tab default; can extend but keep <=5 (HIG tab-bar guidance).
- **Composition:** last slot in the main app templates (not in Sign in / Onboarding / Chat).
- **A11y:** tab-bar semantics (`role="tablist"`, each `role="tab"`, current has
  `aria-current="page"`); labels always visible (not icon-only), which also aids
  cognitive accessibility; 44pt targets.

### Content

#### LifeTopicsModule
- **Purpose:** the user's chosen life domains as status cards ("My life": Home, Health,
  Family & Friends, ...). The personal mindmap at a glance.
- **Data:** `{ topics: { id: string; label: string; status?: string; icon?: string; action: Action }[]; addAction: Action }`.
- **States:** empty (only Add), populated, per-card status present/absent.
- **Variants:** home (large status cards), you-view (compact chips for management).
- **Composition:** Home view and You view.
- **A11y:** cards are a list; each card a single actionable element with an accessible
  name combining label + status ("Health, recovery progress"); Add is a labelled button.

#### TasksModule
- **Purpose:** recent or pending tasks, including consent and confirmation asks, with
  See more.
- **Data:** `{ tasks: { id: string; label: string; kind: 'task'|'consent'|'confirm'; action: Action }[]; seeMore?: Action }`.
- **States:** empty, populated, has-consent-item (elevated treatment).
- **Variants:** default; consent items render with the consent semantic color and are
  never visually identical to ordinary tasks.
- **Composition:** Home view.
- **A11y:** list semantics; a consent task announces its nature ("Consent needed: share
  medical data") not just its label; consent items meet AAA contrast.

#### RecentChatsModule
- **Purpose:** entry points back into recent assistant conversations.
- **Data:** `{ chats: { id: string; assistant: string; preview: string; action: Action }[]; seeMore?: Action }`.
- **States:** empty, populated.
- **Variants:** home (compact), assistant-view (fuller).
- **Composition:** Home view, Assistant view.
- **A11y:** list; each item named by assistant + preview; unread state not by color alone.

#### AssistantsModule
- **Purpose:** the roster of the user's active assistants, each addressable, each with
  settings; plus Explore more.
- **Data:** `{ assistants: { id: string; name: string; icon?: string; settings: Action; chat: Action }[]; exploreMore?: Action }`.
- **States:** empty, populated.
- **Variants:** assistant-view (list with settings gear), compact (picker).
- **Composition:** Assistant view.
- **A11y:** each row exposes two distinct actions (open chat, open settings) as separate
  labelled controls, not one ambiguous tap target.

#### ConnectedAppsModule
- **Purpose:** the access-and-connections surface: which external apps and data sources
  are connected, with per-source control and an Add flow.
- **Data:** `{ connections: { id: string; name: string; connected: boolean; scope?: string; toggle: Action }[]; addAction: Action }`.
- **States:** per-source connected/disconnected; empty.
- **Variants:** you-view list.
- **Composition:** You view.
- **A11y:** THIS IS A CONSENT SURFACE, AAA. Each toggle is a labelled switch with state
  announced ("Google Calendar, connected"); toggling announces the result; scope text is
  associated with its control; never color-only state.

#### ProgressModule
- **Purpose:** a visualization of progress through a task or plan (milestones + tasks);
  surfaces inside the chat canvas or standalone.
- **Data:** `{ milestones: { id: string; label: string; description?: string; state: 'done'|'current'|'upcoming' }[]; tasks?: { id: string; label: string; action?: Action }[] }`.
- **States:** milestone states; all-complete.
- **Variants:** in-canvas (compact), standalone.
- **Composition:** ChatCanvasModule, or a screen slot.
- **A11y:** milestone state conveyed by text/icon plus color, never color alone; the
  sequence is an ordered list; current milestone announced.

### Chat

#### ChatBoxModule
- **Purpose:** the input bar; content adapts per context (onboarding, per assistant).
- **Data:** `{ placeholder: string; actions: { upload?: boolean; voice?: boolean; send: boolean }; context: 'onboarding'|'assistant'|'search' }`.
- **States:** empty, typing, sending, voice-active, disabled.
- **Variants:** onboarding (voice + send), assistant (upload + voice + send),
  search (search affordance).
- **Composition:** bottom slot of Onboarding and Chat; adapts.
- **A11y:** the text field has a persistent visible label or accessible name; icon
  buttons (upload, voice, send) each 44pt with text labels; voice-active state announced.

#### ChatModule
- **Purpose:** the conversation stream; a container populated by message sub-modules in
  sequence.
- **Data:** `{ items: (ConversationStarters | StreamingMessage | AssistantMessage | UserMessage | FollowUpButtons | ChatCanvas)[] }`.
- **States:** loading, streaming, idle.
- **Variants:** default.
- **Composition:** Chat view, inside UniversalCanvas.
- **A11y:** the message list is a log (`role="log"`, `aria-live="polite"` for incoming);
  each message identifies its sender to assistive tech.

#### StreamingMessageModule
- **Purpose:** shows what the agent is doing during load (fetching data, analysing,
  searching web, preparing content, finalizing).
- **Data:** `{ phase: string; label: string }`.
- **States:** cycling phases.
- **Variants:** default.
- **Composition:** appears in ChatModule during agent/user-triggered loading.
- **A11y:** CRITICAL. Status announced via a live region as text ("Analysing your
  data"), never a silent spinner. Honors prefers-reduced-motion: text updates without
  animation. This is Nielsen "visibility of system status" made accessible.

#### AssistantMessageModule
- **Purpose:** a message from a Capsule assistant, optionally with status + content.
- **Data:** `{ assistant: string; status?: string; body: string; timestamp?: string }`.
- **States:** default, with-status.
- **Variants:** default; can host a canvas (see ChatCanvas).
- **Composition:** inside ChatModule.
- **A11y:** sender named; body is readable text with 4.5:1 contrast.

#### UserMessageModule
- **Purpose:** the user's own message in the stream.
- **Data:** `{ body: string; timestamp?: string }`.
- **States:** sent, sending, failed.
- **Composition:** inside ChatModule.
- **A11y:** identified as "You"; failed state not color-only (icon + text).

#### FollowUpButtonsModule
- **Purpose:** suggested prompts the user can tap to continue (the in-chat equivalent of
  the pitch file's chips).
- **Data:** `{ prompts: { label: string; action: Action }[] }`.
- **States:** default, disabled after choice (optional).
- **Variants:** default (2 to 4 prompts).
- **Composition:** after an AssistantMessage or inside ChatCanvas.
- **A11y:** a group of labelled buttons; group has an accessible name ("Suggested
  replies"); 44pt targets; wrap rather than truncate labels.

#### ConversationStartersModule
- **Purpose:** solves the cold start. When a chat opens with no history to continue from
  (a first-time chat, or a fresh chat on a topic), this module offers a few relevant
  opening prompts generated from the starting point: the assistant, the life topic, and
  whether it is the user's first time here. It is the entry-point cousin of
  FollowUpButtonsModule, which continues an existing thread; this one begins one.
- **Data (typed):**
  ```ts
  type ConversationStartersModule = {
    placement: 'chat' | 'home' | 'assistant';   // where it renders
    launchTarget?: { assistant: string; topic?: string };  // required for home/assistant
    context: {
      assistant: string;          // "Companion", "Financial coach"
      topic?: JourneyId | string; // "relationships", "finance"
      firstTime: boolean;         // no prior chats with this assistant/topic
    };
    starters: { id: string; label: string; action: Action }[];  // 3 to 4, generated
    generatedBy: 'agent';         // starters come from the agent, not hardcoded
  };
  ```
  In the `chat` placement, a tapped starter posts as the first user turn in place. In
  `home` and `assistant` placements, a tapped starter opens the chat named by
  `launchTarget`, pre-seeded with that starter.
- **States:** loading (starters being generated, show skeleton not empty), ready, empty
  (generation failed: fall back to a single generic "Tell me what's on your mind"),
  dismissed (user typed instead, starters clear).
- **Variants:**
  - first-time (warmer, orienting: explains what this assistant can do through the
    starter phrasing itself),
  - returning-fresh-topic (assumes familiarity, gets to the point).
- **Population rule (recipe personalization):** starters are generated by the agent from
  `context`, not a static list. A first-time relationships chat with the Companion yields
  reflective openers; a finance chat yields money-situation openers. The module defines
  the SLOT and the contract; the generation is a recipe behavior, specified as the
  interface it plugs into (in: context; out: 3 to 4 labelled prompts), never invented
  here. Example shape only, not fixed copy: Companion first-time might surface openers
  about what the user wants to think through; Financial coach might surface openers about
  a current money question or a subscription review.
- **Composition:** cross-surface. Appears in three placements, same contract, different
  entry behavior:
  - **empty chat** (inside ChatModule, first item): begins the thread in place; replaced
    by real messages once the conversation starts.
  - **Home view** (a slot in the home template): topic entry points that LAUNCH a chat.
    Tapping a starter opens the relevant assistant's chat with that starter as the first
    user turn ("start a finance chat" pre-filled).
  - **Assistant view** (near the roster): per-assistant entry starters, same launch
    behavior, scoped to a chosen assistant.
  The `launchTarget` field distinguishes in-place (empty chat) from launch (Home,
  Assistant). In-place never navigates; launch opens a chat pre-seeded with the tapped
  starter. Never coexists with an active message stream in the chat placement.
- **A11y:** a labelled group with an accessible name that states the context ("Ways to
  start with your Companion"); 3 to 4 labelled buttons, each a 44pt target; labels wrap
  rather than truncate; the group is announced when focus enters the empty chat so a
  screen-reader user is offered the same starting help as a sighted user. If starters are
  still generating, the loading state is announced ("Preparing suggestions") via live
  region rather than presenting an empty region.
- **Distinction from FollowUpButtonsModule:** starters BEGIN a conversation from context
  (cold start, no history); follow-ups CONTINUE one from the last assistant turn. Same
  visual pattern (tappable prompt chips), different trigger and data source. Keeping them
  separate lets each carry its own generation logic and state.

#### ChatCanvasModule  (LIVE TEST TARGET, fully specified below)
- **Purpose:** a rich surface inside the chat for content that is more than a message:
  a visualization, a progress view, or an action requiring consent or confirmation. It
  can host other modules (ProgressModule, FollowUpButtonsModule, consent controls).
- See the dedicated section below.

### Entry

#### RoomsModule
- **Purpose:** the multiplayer surface: shared spaces where people and assistants
  collaborate. Each room card shows last activity, members (people and agents), and an
  invite affordance. This is the home of the shared-thread journeys (Axel and Marko).
- **Data (typed):**
  ```ts
  type RoomsModule = {
    rooms: {
      id: string;
      name: string;                 // "Summer Trip", "Health & Training"
      lastActivity: { preview: string; timestamp: string };
      members: { id: string; label: string; kind: 'person' | 'agent' }[];
      inviteAction: Action;
      openAction: Action;
    }[];
    newRoomAction: Action;
  };
  ```
- **States:** empty (only New room), populated, per-room has-unread.
- **Variants:** list (default). A room, when opened, is a Chat template whose ChatModule
  includes multiple participants; the privacy boundary is enforced by the ConsentRequest
  that admitted each member with a scoped grant.
- **Composition:** its own section, reachable from navigation; not a default Home slot.
- **A11y:** rooms are a list; each card names the room, its last activity, and its member
  count and kinds to assistive tech ("Health & Training, 2 members, 1 person 1 assistant,
  Records shared with clinic"); Invite and Open are separate labelled controls, not one
  ambiguous target; member avatars are not the sole carrier of who is present (text
  alternative required). Because rooms cross a privacy boundary between people, any
  membership or sharing change is announced, and adding a member routes through a consent
  surface (AAA), never a silent add.
- **Note (privacy wall):** a room's shared context is scoped. A member sees only what the
  room's grant admitted; private assistant context stays outside the room. This is the
  Axel privacy-wall mechanic, enforced as a ConsentRequest scope, not a UI convention.

#### LogoModule
- **Purpose:** the Capsule ( O ) mark.
- **Data:** `{ size: 'sign-in'|'header' }`.
- **Composition:** Sign in (large), TopNav (small).
- **A11y:** if decorative, hidden from screen readers; if it is the sole app title on
  a screen, given an accessible name "Capsule".

#### SignInModule
- **Purpose:** authentication entry: email and phone by default, a "Continue without
  signing in" option (used for demos and low-friction entry), plus a troubleshooting link.
- **Data:** `{ methods: ('email'|'phone')[]; allowContinueWithoutSignIn: boolean; troubleshootAction: Action }`.
- **States:** default, submitting, error.
- **Composition:** Sign in screen.
- **A11y:** each method a labelled button; error messages associated with the control and
  announced; troubleshoot is a real link/button, not just text.

#### ChooseTopicsModule
- **Purpose:** onboarding selection of life domains that matter to the user.
- **Data:** `{ options: { id: string; label: string }[]; selected: string[]; addAction: Action }`.
- **States:** none-selected, some-selected.
- **Composition:** Onboarding.
- **A11y:** a multi-select group; selection state announced per option, not color-only;
  group labelled "Choose topics that are important to you".

#### HelpTopicsModule
- **Purpose:** onboarding presentation of the essence-first use cases (the six "How you
  want Capsule to help" entries), letting the user pick how Capsule helps.
- **Data:** `{ options: { id: string; label: string; journey: JourneyId }[] }` where the
  six labels are: run family life without carrying every detail; get your health on
  track and keep it there; let people understand the real you; find your next direction,
  and own it; make your money fit your actual life; recover safely, with life adjusting
  around you.
- **States:** none-selected, selected.
- **Composition:** Onboarding.
- **A11y:** list of labelled options; each a full-width target; wrap long labels.
- **Note:** these six map one-to-one to the six journeys (Alison, Axel/health, Jo,
  Hetta, Maya, Becca). This module is where the JTBD essence becomes the entry point.

---

## LIVE TEST: ChatCanvasModule, fully specified

This is the hardest module (it hosts other modules), so we specify it completely as the
proof that the contract format produces something Claude Design and Claude Code can build.

### Purpose
A surface inside the chat stream for content richer than a text message: a data
visualization, a progress view, or an action that needs the user's consent or
confirmation. It is the in-conversation home of "control is the product": when an agent
needs a decision, the canvas presents it with the scope visible and the choice explicit.

### Data (typed)
```ts
type ChatCanvasModule = {
  variant: 'visualization' | 'progress' | 'consent' | 'confirmation';
  title?: string;
  // hosted sub-modules, rendered in order inside the canvas:
  content:
    | { kind: 'progress'; data: ProgressModuleData }
    | { kind: 'visualization'; data: { spec: VizSpec } }      // e.g. medication schedule
    | { kind: 'consent'; data: ConsentRequest }
    | { kind: 'confirmation'; data: ConfirmationRequest };
  followUps?: FollowUpButtonsData;   // optional prompts surfaced within the canvas
};

type ConsentRequest = {
  requester: string;            // "Home Helper"
  purpose: string;              // "join this Health chat and give tailored suggestions"
  scopes: {
    id: string;
    label: string;              // "Read activity and nutrition data"
    accessLabel: string;        // REQUIRED plain-language consequence, used by the
                                // screen-reader announcement and any helper text:
                                // "can read your activity and nutrition data"
    granted: boolean;           // ALWAYS defaults false. Deliberate opt-in; never
                                // pre-checked. Pre-checking consent is a dark pattern.
  }[];
  allowEmptyGrant?: boolean;    // default false. When false, Grant is disabled until at
                                // least one scope is on. Set true only for a recipe that
                                // legitimately needs presence without data access
                                // (e.g. a reminder-only assistant).
  revocable: true;              // Capsule never asks for irrevocable grants.
  grantAction: Action;
  declineAction: Action;
};

// Post-decision states (the "acted" UI), specified so they are not per-build guesses:
//   granted           -> summary of what was granted + "Manage permissions" (leads to
//                        per-scope controls). Revoke is ALWAYS per-scope, mirroring
//                        per-scope grant and the singular promise "turn this off anytime".
//   partially-granted -> same as granted; shows which scopes are on and which are off.
//   declined          -> a calm declined state with "Review again" to reopen the request.
//   revoked           -> reflects a scope turned off after granting; announced to
//                        assistive tech as the access that was removed.

type ConfirmationRequest = {
  summary: string;              // "Order placed" / "Cancel StreamMax now?"
  details?: { label: string; value: string }[];
  confirmAction: Action;
  cancelAction: Action;
};
```

### States
- loading (skeleton), ready, acted (post-decision, shows outcome), error.
- consent variant: ungranted, partially-granted (some scopes on), granted, revoked.

### Variants and how they populate
- **visualization:** hosts a VizSpec (e.g. Becca's medication schedule with times). Read
  only; may carry followUps.
- **progress:** hosts a ProgressModule (milestones + tasks). This is the "Chat: progress
  & confirmation" screen from the source.
- **consent:** hosts a ConsentRequest. The scopes are individually toggleable; grant and
  decline are equal-weight; the revocable promise is shown, not implied.
- **confirmation:** hosts a ConfirmationRequest (order placed, cancel this, share that).

### Composition
Appears inside ChatModule as one item in the stream. It can itself contain
ProgressModule and FollowUpButtonsModule. It never contains another ChatCanvasModule
(no recursion).

### A11y (consent variant = AAA; all variants >= AA)
- The canvas is a labelled region (`role="group"` with an accessible name from `title`
  or the consent purpose).
- **Consent variant, AAA:** 7:1 contrast on all text and controls. Each scope is a
  labelled switch defaulting to OFF; toggling announces the new state using the scope's
  `accessLabel` ("can read your activity and nutrition data, now on"). Grant and Decline
  are both reachable, both labelled, equal in the accessibility tree (no "primary" that
  hides Decline from quick access). Grant is disabled while all scopes are off unless
  `allowEmptyGrant` is true. On render, focus moves to the canvas and the request is
  announced in full: who wants what, and that it is revocable. Never rely on the consent
  color alone to signal that this is a permission ask; include an explicit text heading
  ("Permission needed"). Post-decision, revoke is per-scope.
- **confirmation variant:** the outcome (placed, cancelled) announced via live region;
  details are a described list.
- **progress / visualization:** see ProgressModule; a pure visualization has a text
  alternative describing what it shows ("Medication schedule: antibiotic 08:00 with
  food, pain relief 12:00 optional").
- Reduced motion: any reveal animation degrades to instant.

### Why this proves the system
ChatCanvasModule shows the contract format handles the three hard things at once:
a module that **hosts other modules** (progress, follow-ups), that **carries typed data
scopes** (the consent request mirrors the PRD's scoped, revocable grants exactly), and
that **changes the accessibility bar by variant** (AAA for consent). If Claude Design can
build this from the contract, and Claude Code can type it, the rest of the inventory is
easier by construction.

---

## What Layer 1 deliberately does not do

- No exact color, spacing, or type VALUES beyond proposed starting points. Those get
  tuned in Claude Design against a real palette; the token structure is what matters here.
- No relevance ALGORITHM. Population rules define the interface (slots carry priority;
  modules can surface or replace by relevance), but the ranking logic is a product
  algorithm, specified as the interface it plugs into, not invented here.
- No templates or journey configs yet. Those are Layers 2 and 3, built on these contracts.

---


## Layer 0b: Visual language (v0.6)

The aesthetic direction, expressed as enforceable rules and tokens. Adjectives are not
buildable; constraints are. Every rule here is something Claude Design, Claude Code, and
the app can act on directly.

### Intent (for humans)
Calm, minimal, subdued. Near-monochrome: whites and greys carry the interface; a single
accent does the pointing. Hierarchy comes from type weight, size, and thin lines, not
from color blocks, fills, or shadows. Restraint is the brand: the interface recedes so
the user's life is the content. This is the visual expression of "focus on the essential."

### Grey ramp (defined, WCAG-contrast-checked)
Near-monochrome base. All surfaces, text, and separators draw from these. Values chosen
so the required text and separator contrasts pass on a white background; tune hues but
preserve the contrast relationships.
```
grey-0    #FFFFFF   app background, raised surfaces
grey-50   #F7F7F8   subtle raised / hover surface
grey-100  #ECECEE   filled chips, inactive track
grey-200  #D9D9DD   hairline separators, borders        (3.0:1 on white, non-text OK)
grey-300  #B8B8BF   disabled text/icons                  (large/non-text only)
grey-500  #6E6E77   secondary text                       (5.2:1 on white, AA body)
grey-700  #3A3A40   primary-adjacent text
grey-900  #17171A   primary text                         (16.8:1 on white, AAA)
```
Contrast obligations from Layer 0 still bind: body/UI text >=4.5:1 (use grey-500 or
darker on white), large text >=3:1, non-text UI >=3:1 (grey-200 hairlines qualify).
Consent-critical text targets 7:1 (grey-700/900 on white).

### Accent (single, swappable)
```
accent    (one token, user-swappable; e.g. teal, orange, blue)
```
- Exactly one accent. Used sparingly: primary actions, active nav state, key interactive
  affordances, single-accent highlights in visualizations. Nothing decorative uses accent.
- **Hierarchy must survive with the accent removed.** The interface must remain fully
  legible and navigable in greyscale alone; accent is a pointer, never the scaffold that
  carries hierarchy or state on its own. Active states co-signal with weight or a fill
  level so they read without color (also required for colorblind users).
- The accent is a single token: swapping it (teal to orange) must leave the whole UI
  coherent, because nothing structural depends on the specific hue.

### Consent color (the one deliberate exception)
The `consent` semantic color remains separate and distinct from accent (see Layer 0),
because a permission ask must stand apart. It must pass the deuteranopia/protanopia check
and is always paired with a text heading ("Permission needed"): color is never the sole
signal.

### Hierarchy: weight, size, lines, not fills
- Express information hierarchy through the type ramp (weight and size) and hairline
  separators, not through background fills, boxed cards with shadows, or color blocks.
- **Separator token:** a single hairline, 0.5pt to 1pt, `grey-200`. Divide sections with
  lines and whitespace first.
- Rule for Claude Design: "Group and separate with whitespace and hairline rules first.
  Use a bordered or filled container only when a line and spacing cannot carry the
  grouping. No drop shadows; if elevation is needed, use a hairline border, not a shadow."

### Navigation: Apple iOS form
- **Top nav** follows the iOS navigation bar: back chevron left, title centered, actions
  right (bell, small logo), hairline bottom separator, minimal height, no heavy chrome or
  shadow.
- **Bottom nav** follows the iOS tab bar: three tabs, icon over label, hairline top
  separator, active tab in accent plus a weight/fill co-signal, inactive in grey-500,
  safe-area aware, 44pt targets.
- Reference for Claude Design: "Model top and bottom navigation on Apple iOS Human
  Interface Guidelines navigation bar and tab bar: minimal height, hairline separators,
  no shadows or heavy fills, SF-style spacing, 44pt targets."

### Visualizations and imagery: same restraint
- TopicVisualizationModule, MomentModule visuals, ProgressModule, and EfficiencyMeter
  render in the same near-monochrome language: greyscale with single-accent highlights,
  thin strokes, generous whitespace, no gradients, no decorative color.
- State is shown by line and fill-level, not multi-color spectrum: the efficiency ring
  fills in accent-or-grey, not red-to-green; progress milestones use fill-state plus
  icon, not color-coding alone.
- Rule: "Data visualizations and imagery use the minimal palette (greyscale plus the
  single accent), thin strokes, no gradients, no decorative color. A chart's meaning must
  read in greyscale alone."

### Accessibility note on minimalism (important)
A near-monochrome, hairline-heavy aesthetic is the harder path for accessibility:
low-contrast grey-on-grey and thin lines are exactly what fails contrast and low-vision
use. The grey ramp above is chosen against contrast math, not vibes, precisely so
minimal and accessible coexist. Do not lighten body text below grey-500 on white, do not
thin separators below the point where they fail 3:1, and never let the accent be the sole
carrier of an active or selected state. Minimal must never become unusable for someone.

---

## Layer 1 additions (v0.6): new modules

These nine modules join the inventory. RoomsModule is retired (see note at end of Layer
1); multiplayer now lives in Chat via ChatParticipantIndicatorModule. All new visual
modules follow Layer 0b.

### Assistant efficiency and setup

#### EfficiencyMeterModule
- **Purpose:** shows a single assistant's "efficiency" as a ring that fills 0 to 100%.
  Efficiency rises as the user connects apps and grants data access, signalling how much
  the assistant can help. Per-assistant, not global.
- **Data:** `{ assistant: string; value: number /*0-100*/; delta?: number /*recent change*/; label?: string }`.
- **States:** low / mid / high (thresholds are presentation only), just-improved (brief
  emphasis on delta), max (100%).
- **Variants:** setup (large ring, hero), inline (small, next to an assistant name),
  moment (compact, inside a MomentModule).
- **Composition:** AssistantSetupModule, Assistant settings, assistant name rows, Moments.
- **A11y:** the value is text, not just a ring ("Home helper, efficiency 35%"); the ring
  is not the sole carrier (percentage shown numerically); fill is accent-or-grey, never a
  red-to-green spectrum; change announced politely when it updates ("efficiency now 55%").
- **Design note:** efficiency gamifies data-sharing. Keep it motivating *useful*
  connection, never manufacturing consent through progress-bar pressure. It must never
  read as "you are incomplete until you share more." This tension with the anti-hook,
  consent-first thesis is deliberate to flag; treat the meter as informative, not coercive.

#### AssistantSetupModule
- **Purpose:** the setup hero for an assistant: a topic visualization or image, the
  assistant's name, and its EfficiencyMeter. Used in onboarding setup and in assistant
  settings.
- **Data:** `{ assistant: string; visualization: TopicVisualizationData; efficiency: EfficiencyMeterData }`.
- **States:** setup (onboarding), settings (managing an existing assistant).
- **Variants:** onboarding, settings.
- **Composition:** Onboarding-Assistant-setup, Assistant-settings templates. Hosts
  TopicVisualizationModule and EfficiencyMeterModule.
- **A11y:** a labelled region named by the assistant; visualization has a text
  alternative; efficiency value is text.

#### PersonalizeAssistantModule
- **Purpose:** the nudge-and-grant surface: "Connect with [app]" and "Give access to
  [data source]" actions that raise an assistant's efficiency. Cross-surface: appears in
  setup, mid-chat (nudging during a conversation), and inside a Moment.
- **Data (typed):**
  ```ts
  type PersonalizeAssistantModule = {
    placement: 'setup' | 'chat' | 'moment';
    assistant: string;
    options: {
      id: string;
      kind: 'connect-app' | 'grant-data';
      label: string;              // "Connect with Google Calendar"
      accessLabel: string;        // plain-language consequence, for the confirm + a11y
      action: Action;             // opens ConnectionConfirmModule
    }[];
    efficiencyHint?: string;      // "adding this improves Home helper's efficiency"
  };
  ```
- **States:** default, some-connected, all-connected (module recedes or hides).
- **Variants:** setup (prominent), chat (a compact inline nudge, dismissible), moment
  (compact).
- **Composition:** AssistantSetup, Chat stream, MomentModule.
- **A11y:** THIS INITIATES A CONSENT FLOW, AAA. Each option is a labelled action stating
  its consequence via accessLabel; opening it routes to ConnectionConfirmModule (the
  actual grant). In chat placement it is dismissible and never blocks the conversation;
  the nudge must not read as coercive (see EfficiencyMeter design note).

#### ConnectionConfirmModule
- **Purpose:** the data-access consent screen. Explains what is being connected, promises
  encryption and revocability, and offers Confirm access or Skip for later. This is the
  most important new consent surface.
- **Data (typed):**
  ```ts
  type ConnectionConfirmModule = {
    assistant: string;
    target: { name: string; kind: 'app' | 'data-source' };
    rights: string;               // "see, edit and modify events"
    // reference microcopy, preserved from source; recipe may adjust wording but not intent:
    assurance: string;            // "We do not use your data for any purpose but to
                                  //  improve the assistant. We keep your data encrypted
                                  //  and safe, and you can revoke access at any time."
    confirmAction: Action;
    skipAction: Action;
    efficiencyAfter?: number;     // efficiency the assistant reaches once connected
  };
  ```
- **States:** default, connecting (hands to StreamingMessage / Connecting screen),
  connected (success, efficiency raised), skipped.
- **Variants:** onboarding-setup, in-chat (reached from a PersonalizeAssistant nudge).
- **Composition:** Assistant-setup-detail template; reachable from PersonalizeAssistant.
- **A11y:** AAA. 7:1 contrast. The rights being granted and the revocability are stated
  in text and announced; Confirm and Skip are equal weight (Skip is not hidden); focus
  moves to the request; the encryption/revoke assurance is not decorative fine print but
  associated, readable content. Never color-only signalling.

### Home, activity, and moments

#### MomentModule
- **Purpose:** a Home-feed card visualizing the status of one topic (health progress, a
  decision shared with a friend, sports results, an item sold). Tappable to start a chat
  immediately with the relevant agent. Can host an efficiency-improve nudge.
- **Data (typed):**
  ```ts
  type MomentModule = {
    id: string;
    topic: string;                // "Health", "Home"
    assistant: string;            // the agent this moment belongs to
    visualization: TopicVisualizationData;
    summary: string;              // "You have maximized assistant efficiency"
    startChatAction: Action;      // opens a chat with `assistant`
    personalize?: PersonalizeAssistantData;  // optional embedded efficiency nudge
  };
  ```
- **States:** default, has-nudge (embeds PersonalizeAssistant), max-efficiency (celebratory,
  no nudge).
- **Variants:** default feed card.
- **Composition:** Home feed (the Home canvas is a feed of these). Hosts
  TopicVisualizationModule and optionally PersonalizeAssistantModule.
- **A11y:** each moment is a labelled card with an accessible name (topic + summary); the
  whole card's primary action (start chat) is clear; an embedded nudge is a separate
  labelled control, not merged into the card tap; visualization has a text alternative.

#### AgentActivityLogModule
- **Purpose:** a timestamped log of what agents are doing, to build trust and visibility:
  the human stays in control even as agents act autonomously or semi-autonomously.
- **Data (typed):**
  ```ts
  type AgentActivityLogModule = {
    scope: 'all' | 'assistant';   // all agents (Notifications, Assistants view) or one
    assistant?: string;           // required when scope is 'assistant'
    entries: {
      id: string;
      timestamp: string;
      assistant: string;
      description: string;        // "checked your calendar for conflicts"
      interactions?: { label: string; action: Action }[];  // e.g. review, undo
    }[];
  };
  ```
- **States:** empty, populated, live (new entries streaming in).
- **Variants:** all (cross-agent), assistant (single agent's history).
- **Composition:** Notifications/Activity view, Assistants view (all); individual
  assistant view (assistant).
- **A11y:** a log (`role="log"` when live, `aria-live="polite"`); each entry names the
  agent, time, and action in text; any interaction (review, undo) is a labelled control.
  This is a trust surface: an autonomous action the user cannot see or reverse is a trust
  failure, so entries carrying an action expose it plainly.

#### ChatParticipantIndicatorModule
- **Purpose:** an expandable indicator of who is in a chat: agents now, and in future
  other humans. This is how multiplayer lives inside Chat (superseding a separate Rooms
  surface). The privacy wall is enforced by the scope each participant was admitted under.
- **Data (typed):**
  ```ts
  type ChatParticipantIndicatorModule = {
    participants: {
      id: string;
      label: string;
      kind: 'agent' | 'human';
      scope?: string;             // what this participant can see (the privacy wall)
    }[];
    expanded: boolean;
    manageAction?: Action;        // add/remove participants (routes through consent)
  };
  ```
- **States:** collapsed (icons only), expanded (names + scopes).
- **Variants:** default.
- **Composition:** Chat top area, inside the Chat template.
- **A11y:** collapsed icons have a combined accessible name ("3 participants: Health
  coach, you, Marko"); expanded, each participant's scope is stated; adding a human or
  agent routes through a consent surface (AAA), never a silent add; membership changes
  announced. This carries Axel's privacy-wall mechanic: a participant sees only what their
  scope admitted.

#### ProgressIndicatorModule
- **Purpose:** onboarding step progress (step 1 of n, 2 of n).
- **Data:** `{ current: number; total: number; label?: string }`.
- **States:** per-step.
- **Variants:** bar, dots.
- **Composition:** Onboarding, Onboarding-Assistant-setup.
- **A11y:** exposes progress semantics ("step 2 of 4"); not color-only; the current step
  announced on change.

#### TopicVisualizationModule
- **Purpose:** a data-visualization or image representing a topic area (health status,
  home status), used in setup, Moments, and chat canvases. Minimal, greyscale-plus-accent.
- **Data:** `{ topic: string; kind: 'data-viz' | 'image'; spec: VizSpec | ImageRef; caption?: string }`.
- **States:** loading, ready, empty.
- **Variants:** hero (setup), card (moment), inline (canvas).
- **Composition:** AssistantSetup, MomentModule, ChatCanvasModule.
- **A11y:** every visualization has a text alternative describing what it shows; follows
  Layer 0b (greyscale plus single accent, readable in greyscale alone).

---

## Layer 1 note: RoomsModule retired (v0.6)

RoomsModule (added v0.4) is retired. Multiplayer now lives inside the Chat view via
ChatParticipantIndicatorModule: a chat can have multiple participants (agents now, humans
later), and the privacy wall is enforced by each participant's admitted scope rather than
by a separate Rooms surface. The Rooms concept is preserved in history here; do not build
a standalone Rooms screen. If a dedicated multi-room browsing surface is needed later, it
returns as a new module with its own contract.

---

## Layer 2: Screen templates (v0.6)

A template is an ordered list of module slots inside the UniversalCanvasModule. Slots
carry a default order and a `priority` population rules may use to reorder. Notation:
`SlotName -> ModuleName [default|surfaced] {priority}`. Navigation model: three bottom
tabs (Home, Assistant, You); Notifications/Activity opens from the top-nav bell, not a
tab. All chrome follows Layer 0b (iOS nav bar and tab bar, hairlines, no shadows).

### Template: SignIn
```
canvas: UniversalCanvasModule
  logo     -> LogoModule (size: sign-in)         [default] {0}
  sign-in  -> SignInModule                        [default] {1}
```
SignInModule now includes "Continue without signing in" alongside email and phone and the
troubleshoot link. No nav (pre-auth).

### Template: Onboarding
```
canvas: UniversalCanvasModule
  choose-topics -> ChooseTopicsModule             [default] {0}
  progress      -> ProgressIndicatorModule        [default] {1}
```
ChooseTopicsModule presents the six essence framings ("How you want Capsule to help").
Additional views may be added inline during onboarding. Flows into Assistant setup.

### Template: OnboardingAssistantSetup
```
canvas: UniversalCanvasModule
  setup     -> AssistantSetupModule               [default] {0}
  personalize -> PersonalizeAssistantModule (placement: setup) [default] {1}
  progress  -> ProgressIndicatorModule            [default] {2}
```
The setup hero: topic visualization + assistant name + EfficiencyMeter, with connect/grant
nudges. Confirming a connection routes to ConnectionConfirm, then the Connecting screen,
then the complete state. Content updates to show success and prompt "start a conversation".

### Template: ConnectingToSource
```
canvas: UniversalCanvasModule
  streaming -> StreamingMessageModule             [default] {0}
```
Shown during a connection: the streaming animation tells the user what the assistant is
doing while connecting.

### Template: AssistantSetupDetail (the consent screen)
```
canvas: UniversalCanvasModule
  top-nav -> TopNavModule (variant: section)      [default] {0}
  confirm -> ConnectionConfirmModule              [default] {1}
```
AAA consent surface. The encryption + revoke-anytime assurance is shown; Confirm access
and Skip for later are equal weight.

### Template: AssistantSetupComplete
```
canvas: UniversalCanvasModule
  top-nav   -> TopNavModule (variant: home)       [default] {0}
  setup     -> AssistantSetupModule (variant: settings) [default] {1}
  personalize -> PersonalizeAssistantModule (placement: setup) [default] {2}
```
Efficiency raised (e.g. 35% to 55%); prompts "start a conversation" or add more
connections.

### Template: Home
```
canvas: UniversalCanvasModule
  top-nav    -> TopNavModule (variant: home)      [default] {0}
  moments    -> MomentModule (feed)               [default] {1}
  bottom-nav -> BottomNavModule                   [default] {99}
```
The Home canvas is a FEED of MomentModules: each visualizes a topic's status and can start
a chat with the relevant agent; a moment may embed a PersonalizeAssistant nudge. The feed
order may reorder by relevance. This replaces the earlier LifeTopics+Tasks+RecentChats
Home. TopNav bell opens Notifications.

### Template: Notifications (opens from the top-nav bell, not a tab)
```
canvas: UniversalCanvasModule
  top-nav    -> TopNavModule (variant: section)   [default] {0}
  tasks      -> TasksModule                        [default] {1}
  activity   -> AgentActivityLogModule (scope: all) [default] {2}
  bottom-nav -> BottomNavModule                   [default] {99}
```
Tasks (including efficiency-improvement tasks) plus the all-agent activity log: what agents
are doing now, for trust and visibility.

### Template: Assistants
```
canvas: UniversalCanvasModule
  top-nav    -> TopNavModule (variant: home)      [default] {0}
  assistants -> AssistantsModule                   [default] {1}
  activity   -> AgentActivityLogModule (scope: all) [default] {2}
  bottom-nav -> BottomNavModule                   [default] {99}
```
Roster (Home helper, Health coach, Training buddy, Companion, Financial coach, each with a
settings gear, plus Explore more) and the all-agent activity log.

### Template: AssistantIndividual
```
canvas: UniversalCanvasModule
  top-nav        -> TopNavModule (variant: section) [default] {0}
  activity       -> AgentActivityLogModule (scope: assistant) [default] {1}
  connections    -> ConnectedAppsModule (variant: assistant-scope) [default] {2}
```
One assistant's activity log and its connected apps/data sources.

### Template: AssistantSettings
```
canvas: UniversalCanvasModule
  top-nav      -> TopNavModule (variant: section) [default] {0}
  setup        -> AssistantSetupModule (variant: settings) [default] {1}
  personalize  -> PersonalizeAssistantModule (placement: setup) [default] {2}
  connections  -> ConnectedAppsModule (variant: assistant-scope) [default] {3}
```

### Template: You
```
canvas: UniversalCanvasModule
  top-nav        -> TopNavModule (variant: home)  [default] {0}
  life-topics    -> LifeTopicsModule (variant: you-view) [default] {1}
  connected-apps -> ConnectedAppsModule           [default] {2}
  bottom-nav     -> BottomNavModule               [default] {99}
```
My life topics plus Connected apps (with the iOS-share-style Add sheet). ConnectedApps is
an AAA consent surface.

### Template: Chat
```
canvas: UniversalCanvasModule
  top-nav      -> TopNavModule (variant: chat)    [default] {0}
  participants -> ChatParticipantIndicatorModule  [surfaced] {1}
  chat         -> ChatModule                        [default] {2}
              // ChatModule streams, in order, any of:
              //   ConversationStartersModule (placement: chat)  when empty
              //   StreamingMessageModule       during load
              //   AssistantMessageModule / UserMessageModule (media-capable)
              //   FollowUpButtonsModule         after assistant turns
              //   ChatCanvasModule              rich / consent / progress content
              //   PersonalizeAssistantModule (placement: chat)  dismissible nudge
  chat-box     -> ChatBoxModule (context: assistant) [default] {3}
```
TopNav (chat variant): back, small logo, bell. Participants indicator surfaces when a chat
has more than the user + one agent (multiplayer). Covers the "progress & confirmation" and
"chat canvas" screens via ChatCanvasModule variants.

### Responsive rule (all templates)
Mobile: single column. Wide: content max-width centered; nav may become a persistent side
rail (BottomNav promoted to a left column) while content keeps its module order. Modules
scale, they do not re-author.

---

## Layer 3: Journey configurations

A journey config is a recipe instance: a named path through templates with modules
selected and fed data. The point of the system: a new journey is a new config, not new
components. Below, each of the six journeys is expressed as its key screen(s) and the
module data that makes it that journey. Data shown is illustrative shape, not fixed copy.

### Config: Alison (Family, cross-agent shared context)
- **Home:** LifeTopics = [Home, Health, Family & Friends]; Tasks includes a cross-agent
  item ("Home Helper adjusted Thursday dinner"); RecentChats = Health coach, Home helper.
- **Chat (Health coach):** AssistantMessage proposes the interval session; ChatCanvas
  (variant: confirmation) surfaces the dinner + pickup adjustment with confirm/keep.
- **Recipe note:** the differentiator (agents comparing notes) renders as a ChatCanvas
  confirmation generated from combined Health + Home context.

### Config: Jo (Relationships, data sovereignty)
- **Chat (Companion), first time:** ConversationStarters (placement: chat, firstTime:
  true, topic: relationships) offers reflective openers.
- **Chat later:** ChatCanvas (variant: consent) with a ConsentRequest scoped to "what I
  need and why", revocable: true, granting a slice to a partner conversation.
- **Recipe note:** scoped + revocable grant is the ConsentRequest type doing exactly what
  the PRD promises.

### Config: Axel (Health, multiplayer via chat participants)
- **Chat (Training buddy, shared):** ChatParticipantIndicator shows Axel, Marko, and the
  Training buddy agent; AssistantMessage proposes two slots for two calendars;
  FollowUpButtons = [A, B, Neither]. The privacy wall is each participant's admitted
  scope: Marko's participant scope covers joint sessions only, Axel's private health
  context withheld. This is multiplayer via ChatParticipantIndicatorModule, not a
  separate Rooms surface.

### Config: Hetta (Career, longitudinal memory + portable asset)
- **Chat (Career coach):** across sessions, memory accumulates; by week six a ChatCanvas
  (variant: visualization) presents the structured career profile with an Export action.
- **Recipe note:** Export is the portable-asset promise; the canvas hosts the profile viz.

### Config: Maya (Finance, cross-domain insight + consented action)
- **Chat (Financial coach), first time:** ConversationStarters (topic: finance) offers
  money-situation openers.
- **Chat:** ChatCanvas (variant: consent) with a ConfirmationRequest to cancel one
  subscription now, scoped to that action, plus a reminder toggle.

### Config: Becca (Health & recovery, shared context under high stakes)
- **Chat (Health coach):** ChatCanvas (variant: progress) shows the recovery-day plan:
  ProgressModule with milestones, medication schedule as a visualization, the care-team
  boundary line as AssistantMessage. Home Helper adjustments surface as a second canvas
  or message. AAA throughout (health + consent sensitivity).

### Adding a NEW journey (the test of universality)
A future journey (say, Travel) is authored as: pick templates (Home + Chat), select
modules (LifeTopics gains a Travel card, a Travel assistant in AssistantsModule,
ConversationStarters with topic: travel, ChatCanvas variant per need), supply data. No
new components. If a genuinely new UI pattern is needed, it becomes a new MODULE added to
Layer 1 with a full contract, then reused, never a one-off screen.

---

## Layer 4: Claude Design build prompts

Use these in order. Each assumes the previous produced reusable components. Paste the
relevant module contracts from Layer 1 alongside the prompt.

**Prompt 1, tokens and primitives:**
"Using these design tokens [paste Layer 0], create the primitive styles for a mobile-first
app: color roles, type ramp, spacing, a 44pt minimum touch target, and a distinct consent
treatment. Honor Dynamic Type and prefers-reduced-motion. Output reusable style
definitions, not a screen."

**Prompt 2, module library:**
"Build these as reusable components from their contracts [paste module contracts]. Each
component takes the typed props shown, implements the listed states and variants, and
meets the A11y block (WCAG 2.2 AA, and AAA for any consent surface). Start with the
structural and content modules; do not assemble a screen yet."

**Prompt 3, one screen from a template:**
"Compose the Home template [paste Layer 2 Home] from the components you built, as a
UniversalCanvasModule with the listed slots in order. Use Alison's config data [paste
Layer 3 Alison Home]. This should look like a real Capsule home screen."

**Prompt 4, prove generalization:**
"Without building new components, recompose the same modules into the Chat template using
Becca's config [paste]. Show that the identical component library produces a different
journey by configuration alone."

**Prompt 5, the consent canvas (hardest):**
"Build ChatCanvasModule, consent variant, from its full contract [paste the ChatCanvas
section]. The ConsentRequest scopes are individually toggleable; grant and decline are
equal weight; the revocable promise is visible; meet AAA. This is the component that
proves the system."

---

## Layer 5: Validation set

The system is correct when it can reproduce known-good screens. The 15 ASCII frames in
capsule-wireframes-spec.md (the earlier concrete wireframes) are the test set. Each frame
maps to a template + config; building it from modules should reproduce it.

| Frame | Template | Key modules exercised |
|---|---|---|
| 1 Permission prompt | Chat | ChatCanvas (consent), ConsentRequest scopes |
| 2 Home Helper joined | Chat | AssistantMessage, FollowUpButtons |
| 3 Order confirmed | Chat | ChatCanvas (confirmation) |
| 4 Doctor prep | Chat | AssistantMessage, FollowUpButtons, share-method choice |
| 5 WhatsApp reminder | (external surface) | messenger chrome variant, not a Capsule template |
| 6 Home: My Life | Home | TopNav, LifeTopics, ChatBox |
| 7 My Assistants | Assistant | AssistantsModule |
| 8 Access controls | You | ConnectedApps (AAA consent) |
| 9 Rooms | Chat | ChatParticipantIndicatorModule (multiplayer via participants) |
| 10 Alison moment | Chat | ChatCanvas (confirmation) |
| 11 Jo share sheet | Chat | ChatCanvas (consent), scoped ConsentRequest |
| 12 Axel shared thread | Chat | AssistantMessage, FollowUpButtons, privacy-wall status |
| 13 Hetta profile | Chat | ChatCanvas (visualization), Export action |
| 14 Maya consent card | Chat | ChatCanvas (confirmation), reminder toggle |
| 15 Becca recovery day | Chat | ChatCanvas (progress), ProgressModule, care-team line |

**One gap remains (honest):** Frame 5 (WhatsApp) is an external-surface variant, not a
native template: a distribution-channel styling (messenger chrome) speccable separately.
Multiplayer (Frame 9) is now handled inside Chat via ChatParticipantIndicatorModule
rather than a Rooms surface. Everything else maps to existing modules.

---

## Using this across Claude surfaces

- **Claude (chat):** discuss journeys and screens in module vocabulary; ask "what modules
  does journey X need?" and get a config.
- **Claude Design:** feed Layer 0 + relevant Layer 1 contracts + a Layer 2 template +
  a Layer 3 config. Build library first, then screens, then prove generalization.
- **Claude Code:** the typed Data blocks are the prop interfaces; states and variants are
  the component's state machine; the A11y blocks are acceptance criteria. Name components
  by the `...Module` convention so design, prose, and code share one vocabulary.
- **The app:** because the vocabulary mirrors the PRD's behavioral recipes, a module maps
  to a recipe surface and a journey config to a recipe instance. The design system and the
  platform architecture are one description at two altitudes.

## Provenance and maintenance

- **Upstream source:** CAI.pdf (FigJam wireframes + module description). If the FigJam
  changes, update Layer 1/2 here; do not fork drawings elsewhere.
- **Relationship to other files:** supersedes capsule-wireframes-spec.md, which becomes
  the Layer 5 validation set. Independent of the pitch file (capsule-ai-intro.md); shares
  only the journey names and the behavioral-recipe vocabulary.
- **Versioning:** this file versions on its own (0.x during Layer buildout). Lock at 1.0
  when Layers 1 to 3 are validated in Claude Design.
