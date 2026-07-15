# Capsule design system

The single source of truth for Capsule's UI, driving four surfaces: Claude (reasoning),
Claude Design (prototype), Claude Code (real app), and the app itself.

## Files

| File | What it is |
|---|---|
| `design-system.md` | The design system (v0.6). Tokens, visual language, 31 module contracts, 13 templates, journey configs, validation set. The source of truth. |
| `CHANGELOG.md` | Version history. Changes originate here, then propagate. |
| `lint.py` | Completeness check: every module has all contract sections, file em-dash-free, consent surfaces AAA. Run before shipping any change. |
| `claude-design-runbook.md` | The 9-prompt sequence to build the prototype in Claude Design from v0.6, wireframe-bound. |
| `build-tracker.md` | Screen-by-screen fidelity tracker for the rebuild. |
| `CAI.pdf` | Upstream wireframes and flow (kept alongside; attach to Claude Design as the visual target). |

## The flow (one direction only)

```
CAI.pdf (wireframes)  ->  design-system.md  ->  Claude Design (prototype)
                                            ->  Claude Code (real app)
```

Changes originate in the wireframes and the spec. Never edit the design decisions inside
Claude Design or Claude Code and expect them to persist; carry them back to the spec. The
spec is upstream; everything else is downstream.

## To build the prototype

1. Fresh Claude Design project.
2. Attach `design-system.md` and `CAI.pdf`.
3. Follow `claude-design-runbook.md`, prompts 1 to 9.
4. Track progress in `build-tracker.md`.
5. Pause at the checkpoints (tokens, ConnectionConfirm, efficiency meter).

## To change the design system

1. Update `CAI.pdf` and/or describe the change.
2. Update `design-system.md`, bump the version, add a `CHANGELOG.md` entry.
3. Run `python3 lint.py`. Must pass.
4. Re-attach to Claude Design; reconcile with a prompt naming the deltas.

## Vocabulary

Mirrors the PRD's behavioral recipes: a module is a recipe surface, a journey config is a
recipe instance. This keeps design language and platform language unified from prototype
through to the shipped app.
