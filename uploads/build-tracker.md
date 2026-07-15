# v0.6 build tracker: fidelity against wireframes

Track the rebuild screen by screen. A screen is "done" only when it is built from modules
(no one-offs) AND matches its CAI.pdf wireframe. Fill Built and Matches as you go.

## Components (build before screens)

| Component | Prompt | Built | Notes |
|---|---|---|---|
| Tokens + visual language | 1 | [ ] | grey ramp values confirmed, body >=4.5:1 |
| ConnectionConfirmModule | 2 | [ ] | AAA, Skip = Confirm weight, assurance readable |
| EfficiencyMeterModule | 3 | [ ] | iterate on the test board, then lock |
| Core library (20 modules) | 4 | [ ] | ask for the full component list to confirm |

## Screens (compose from components, match wireframe)

| # | Screen | Prompt | Built | Matches wireframe | Notes |
|---|---|---|---|---|---|
| 1 | SignIn | 5 | [ ] | [ ] | incl. "Continue without signing in" |
| 2 | Onboarding | 5 | [ ] | [ ] | six essence framings + progress |
| 3 | OnboardingAssistantSetup | 5 | [ ] | [ ] | efficiency 35%, connect/grant nudges |
| 4 | ConnectingToSource | 5 | [ ] | [ ] | streaming animation |
| 5 | AssistantSetupDetail | 5 | [ ] | [ ] | the consent screen (AAA) |
| 6 | AssistantSetupComplete | 5 | [ ] | [ ] | efficiency 55%, start a conversation |
| 7 | Home (Moments feed) | 6 | [ ] | [ ] | feed of moments, one with nudge, one maxed |
| 8 | Assistants | 6 | [ ] | [ ] | 5 assistants incl. Training buddy, activity log |
| 9 | You | 6 | [ ] | [ ] | life topics + connected apps + Add sheet |
| 10 | Notifications | 7 | [ ] | [ ] | tasks + all-agent activity, from bell |
| 11 | AssistantIndividual | 7 | [ ] | [ ] | one agent's log + connections |
| 12 | AssistantSettings | 7 | [ ] | [ ] | setup hero + personalize + connections |
| 13 | Chat + journey | 8 | [ ] | [ ] | participants + canvas + nudge together |
| - | Wired prototype | 9 | [ ] | [ ] | all reachable, relative paths, static |

## The rule for a "Matches" check

Open the CAI.pdf frame beside the built screen. It matches when the same modules are
present in the same arrangement with the wireframe's content (the named assistants, the
connected apps, the efficiency values, the task labels). Different styling is fine (the
wireframe is grayscale-neutral); different structure or invented content is not. A screen
that does not match is a prompting gap (re-prompt with the exact content) or a spec gap
(a module is missing; report it, do not one-off it).
