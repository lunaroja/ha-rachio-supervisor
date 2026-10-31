<!-- second-brain-repository-bootstrap:v1 -->

# Repository agent guide

This repository extends Abraham's canonical agent contract at
`/Users/abraham/src/second-brain/AGENTS.md`. Repository-specific instructions
below add to that contract; they do not replace it.

## Required task-start synchronization

Before substantive work or a substantive response in a new task:

1. Run `.codex/bin/repository-preflight` from anywhere inside this repository.
2. If it reports a dirty, detached, untracked, or diverged checkout, preserve
   the existing work and report the exact synchronization blocker. Never reset,
   clean, force, or overwrite work to make the preflight pass.
3. After a successful pull, read the current Second Brain `AGENTS.md`, then this
   file, before acting. Start a new task if pulled instruction changes must be
   loaded by the runtime immediately.

The preflight verifies the global Second Brain link and performs only
fast-forward pulls for clean checkouts with tracking branches.

## Repository-specific instructions

Add this repository's purpose, validation commands, deployment workflow,
privacy boundaries, and source-of-truth rules here as they become known. Keep
cross-project identity, preferences, credentials, and delivery policy in Second
Brain.

# Project Agent Notes

This repository is the public upstream seed for `Rachio Supervisor for Home Assistant`.

Repository rules:
- Keep this repo publishable. Do not add house-specific secrets, screenshots with private addresses, exported Home Assistant storage, or raw property data.
- The product is a HACS custom integration first. Do not pivot it into an add-on-first runtime without updating `PRD.md`.
- Treat `PRD.md` as the decision contract for public-facing scope.
- Keep the docs site polished and restrained. The public docs should match the actual integration surface and example dashboard package.
- Do not claim runtime behavior that is not implemented. If scaffolding exists ahead of functionality, say so plainly in docs and release notes.
- Keep `rachio_supervisor.evaluate_now` as a forced fresh Rachio evidence reconciliation, not only a coordinator refresh. Photo import, webhook cutover checks, and operator debugging depend on live evidence.

## Contribution fork scope

This is a temporary contribution fork of david00769/ha-rachio-supervisor,
not a fork of Home Assistant Core. Work is tracked in the private Home Assistant
repository's issue #305. Keep all household evidence in that owning repository.
Authorized scope: Flex discovery, exact per-zone mapping, official upcoming-run
reporting and weather units. Push reviewed changes to this fork's main. Do not
open upstream PRs, publish releases, or deploy to live Home Assistant without
Abraham's further instruction. Preserve upstream licensing and attribution.
Run `python3.13 -m unittest -q tests.test_supervisor_logic`, compileall, and
`node --check custom_components/rachio_supervisor/www/rachio-supervisor-zone-grid-card.js`.

Abraham subsequently authorized only the focused weather-unit upstream PR
on October 31: david00769/ha-rachio-supervisor#1, branch `fix-weather-units`.
That branch starts from upstream main and excludes this personal scaffold and
the other feature work. Further PRs, releases and live deployment remain
unauthorized. Keep external PRs clear, simple and narrowly scoped.
