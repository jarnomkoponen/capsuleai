# CLAUDE.md: Capsule app build

You are building the Capsule AI application from its design system. This file is the
contract for how to build. `capsule-design-system.md` is the source of truth for what to
build; when they conflict, the design system wins on specification and this file wins on
process.

## What Capsule is (one paragraph so you build the right thing)

Capsule gives a person a team of personal assistants that share one user-owned picture of
their life, coordinate with each other, and act only with scoped, revocable consent. The
UI's job is to make control feel like the product: every data access is visible, every
action is asked, every grant is reversible. Build calm, trustworthy, anti-addictive
interfaces. When in doubt, favor the user's control and clarity over engagement.

## Source files

- `capsule-design-system.md`  The design system. Tokens (Layer 0), module contracts
  (Layer 1), templates (Layer 2), journey configs (Layer 3), build prompts (Layer 4),
  validation set (Layer 5). This is what you build from.
- `CAI.pdf`                    Upstream wireframes. Reference only; the design system
  supersedes it. Do not build from the PDF directly.
- `capsule-wireframes-spec.md` The 15 ASCII frames, now the validation set. Your build is
  correct when it can reproduce these.

## Architecture (non-negotiable)

The system is atomic and composition-based. Respect these boundaries or the whole point
is lost:

- **Tokens -> Modules -> Templates -> Journey configs.** Build in that order. Never
  hardcode a value a token defines. Never build a screen before its modules exist.
- **A screen is a UniversalCanvasModule of ordered slots.** Screens are composed, never
  authored as bespoke layouts.
- **A journey is a configuration, not code.** Adding a journey selects existing modules
  and supplies data. If a journey seems to need a new screen, it needs a new MODULE with a
  full Layer 1 contract, reused, never a one-off screen. This is the test of the whole
  system: resist the one-off.
- **Modules are named by the `...Module` convention** exactly as in the design system, so
  design, prose, and code share one vocabulary.
- **Vocabulary mirrors the PRD behavioral recipes:** a module is a recipe surface, a
  journey config is a recipe instance. Keep names aligned so design and platform stay one
  idea.

## Build order

1. **Tokens.** Implement Layer 0 as the single source of style: color roles, type ramp
   (Dynamic Type compatible), 4pt spacing, 44pt minimum targets, the distinct consent
   treatment, reduced-motion handling. Everything downstream references these.
2. **Module library.** Build each Layer 1 module from its contract: typed props = the Data
   block, state machine = States + Variants, acceptance criteria = the A11y block. Build
   structural and content modules first, chat modules next, entry modules last. Do not
   assemble screens yet.
3. **Templates.** Implement Layer 2 as slot compositions of the library. Slots carry order
   and priority; population rules may reorder by priority; pinned slots (nav) stay put.
4. **Journey configs.** Implement Layer 3 configs as data feeding templates. Start with
   one (Alison Home), then prove generalization by configuring a different journey (Becca
   Chat) with zero new components.
5. **Validate.** Reproduce the Layer 5 frames. A frame you cannot reproduce from modules
   is a spec gap: report it, do not invent a one-off to paper over it.

## Accessibility (acceptance criteria, not a later pass)

- Target WCAG 2.2 AA across the app; AAA for consent-critical surfaces (ConsentRequest,
  ConnectedAppsModule, ChatCanvasModule consent variant, RoomsModule membership changes).
- Every module's A11y block is a definition of done. A module that renders correctly but
  fails its A11y block is not done.
- Consent surfaces specifically: 7:1 contrast; scopes as labelled switches with state
  announced; grant and decline equal in the accessibility tree; focus moves to the request
  on render; the revocable nature is stated in text, never color alone.
- Streaming/status: announce via live region as text, never a silent spinner; honor
  prefers-reduced-motion everywhere.
- This is not compliance theater. Capsule's promise is that the user sees and controls
  their data; if control is not perceivable to assistive tech, the product fails for that
  user.

## Consent is a type invariant, not a UI convention

- ConsentRequest.revocable is always true. The system cannot express an irrevocable grant.
  Do not add a code path that grants access without a scoped, revocable ConsentRequest.
- Scopes are individually toggleable. A grant is per-scope, never all-or-nothing.
- Privacy walls (Rooms, shared threads) are enforced as ConsentRequest scopes, not as
  view-level hiding. A member sees only what the grant admitted, at the data layer.

## Stack guidance (adjust to the team's choice; principles hold regardless)

- Component-driven (the module library is literally a component library). A framework with
  strong typed props and composition (React + TypeScript is the reference assumption) fits
  the contracts directly; the Data blocks become prop types.
- Tokens as a real token layer (CSS variables or a token package), not scattered constants.
- No browser storage assumptions baked into modules; data arrives as props. Modules are
  presentational + interaction; data and recipe logic live above them.
- Keep modules free of journey-specific logic. A module never knows which journey it is in;
  it renders its props. Journey-specific behavior lives in the config layer.

## Things you will be tempted to do. Do not.

- Do not build a bespoke screen because a journey "just needs one." Add a module or a
  config. The one-off is how design systems die.
- Do not hardcode copy, color, spacing, or starter/follow-up text. Copy and generated
  content (conversation starters, follow-ups) arrive as data from the recipe layer.
- Do not merge ConversationStartersModule and FollowUpButtonsModule because they look
  alike. Different trigger, data, lifecycle. Keep them separate.
- Do not weaken a consent surface to match the styling of ordinary controls. Consent is
  meant to look distinct.
- Do not skip the A11y block to "add it later." It is the acceptance criteria now.

## Provenance and change flow

- Design changes originate in `capsule-design-system.md` (which originates from CAI.pdf),
  then flow to code. Do not encode design decisions only in components; upstream them.
- A new UI pattern is a new module contract in the design system first, then built here.
- Report validation gaps (a frame you cannot reproduce, a journey that resists
  configuration) as design-system issues, not code workarounds.
