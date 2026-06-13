# Frozen decisions

Durable decisions that must not silently change. Paired with the `frozen-decisions`
skill and the `frozen_decisions`-style review.

**Same-change rule:** when a frozen decision genuinely changes, update BOTH this
file AND the check/verifier that enforces it, in the same change — never let the
doc and the guard drift apart.

## Canonical files
- TODO — the file(s) that are the source of truth for the project.

## Frozen contracts
| Decision | Why frozen | Enforced by |
| --- | --- | --- |
| TODO | TODO | TODO (test / verifier / review) |

## Known drift
- TODO — known-but-not-yet-fixed deviations, recorded so they are not
  "rediscovered" as bugs.

## Do-not-touch
- TODO — generated regions, vendored files, secrets.
