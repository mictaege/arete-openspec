# Proposal: Per-step timeouts

Backlog idea #7a from the arete improvement discussion (split into two changes). Proposal-only.

## Why

Journey/e2e steps (UI flows, HTTP calls, waits) can hang forever; today only whole-test JUnit
`@Timeout` exists. A step that hangs should fail fast with a clear timeout message instead of
blocking the whole suite on CI.

## What Changes

- Add a `timeoutSeconds` (long, default 0 = no timeout) attribute to `@Step` (and consistently to
  `@Given`/`@When`/`@Then`/`@ItShould` if it fits the execution model).
- arete core enforces it during ordered step execution (e.g. wrapping the step invocation in
  `assertTimeoutPreemptively`-style execution), failing the step with a timeout message naming the
  step and the limit.
- Delta spec on `journeys`/`lifecycle` in the `arete` repo when designed: `### Requirement: A step
  exceeding its timeout fails`.

## Out of Scope

- Whole-scenario/journey timeouts (JUnit `@Timeout` already covers that).
- Report changes in arete-gradle (a timed-out step is just a FAILED step).
- Retry-on-timeout semantics (separate idea, rejected).

## Affected Repos

- `arete` (annotation attribute + enforcement)
