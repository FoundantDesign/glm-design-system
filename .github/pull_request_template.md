## Jira

UX-

## What changed

<!-- One or two sentences. Which kitchen sink sections, proposals (glm-prototype-foundation.css) or prototypes. -->

## Status impact

<!-- For each kitchen sink section touched: before and after status (SHIPPED / PROPOSAL / PRODUCTION GAP) and whether its reference strip was updated. -->

## Evidence

<!-- Which GLM.Web files and selectors confirm this (wwwroot/css, DESIGN-SYSTEM.md), at which trunk commit. "Confirmed by absence" counts, with the search used. -->

## Open decisions touched

<!-- Any item from the Open decisions table in DESIGN-SYSTEM.md, or "None". If one applies, options go here and work stops. -->

## Corrections recorded

<!-- Any earlier claim that was wrong, and where the correction was written. Or "None". -->

## Checklist

- [ ] No color literals outside token definitions
- [ ] New classes are tagged PROPOSED with a ticket
- [ ] Proposals that shipped in GLM.Web were removed from glm-prototype-foundation.css
- [ ] Rendered result checked in a browser against production CSS from beta
- [ ] drift.py run from GLM.Web; Changed, Production-only and Untagged are empty or explained
- [ ] Intermediate commits carry [skip ci]
