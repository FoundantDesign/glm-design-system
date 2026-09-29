# GLM Design System: instructions for Claude Code

Read this file in full before changing anything in this repo.

## What this repo is

The GLM (Grant Lifecycle Manager) kitchen sink and design contract, deployed on Netlify at https://glm-design-system.netlify.app/.

Three surfaces must converge on one visual language and one implementation: the kitchen sink (this repo), production (the live GLM app), and prototypes built for user testing. Same classes, same variables, same tokens, same icons.

Sync direction depends on what changed:

- Anything that already exists in production: production is the source of truth. Reproduce it faithfully, including its flaws. Do not silently fix production behavior here.
- Anything new that starts in the kitchen sink: it is a proposal that must round-trip into production. A kitchen-sink-only solution is never the finish line. Log it as a GAP.

## Files

- `index.html`: the kitchen sink. Canonical.
- `glm-prototype-foundation.css`: single source of truth for tokens, remaps, and real component classes. `index.html` links it.
- `GLM-design-contract.md`: ratified values, gaps, the remap principle, and the completion backlog.
- `index_preview.html`: throwaway with CSS inlined. Regenerate after foundation CSS changes to check rendering. Never commit it.
- Production evidence (`mdb-customizations.css`, `GLM_Summary.css`, `foundant-tokens.css`, production HTML captures): ground truth for OVERRIDDEN vs GAP. If these are not in the repo, ask for them. Never infer production state from memory.
- `mdb_min.css` is huge. Search it with grep for specific selectors. Never read it whole.

## Non-negotiable rules

1. Style components by remapping MDB `--mdb-*` variables onto Foundant `--colors-*` tokens on the real MDB class. Example: `.btn-primary { --mdb-btn-bg: var(--colors-brand-primary-dark-blue); }`
2. Never write a color literal (hex, rgb, named color) anywhere except inside a token definition.
3. Real production classes only. No invented classes. `.ft-label` and `.ft-footer` are real production classes and stay. Any genuinely new class must be marked GAP and logged in the contract backlog for engineering review.
4. Buttons are always pill-shaped (`--radius-round`).
5. Icons are Font Awesome Pro. `fa-light` by default, `fa-regular` for filled-state controls.
6. Subsection headings under an `h2` are real `<h3>`. Use `.section-eyebrow` only for a true overline label above another heading.
7. Primary is `--colors-brand-primary-dark-blue`, which is dark navy, not blue.

## Cascade verification

A `--mdb-*` override can silently fail when MDB's compiled rule sets the rendered property directly or at higher specificity. After any remap, check the actual winning selector in `mdb_min.css` and confirm the rendered result, not just the variable assignment. Colors baked into SVG data URIs cannot be reached by remapping (see UX-137).

## Status vocabulary

Every section carries exactly one status, derived from evidence:

- OVERRIDDEN: production customizes this via remaps (confirmed in production files).
- DEFAULT: stock MDB, no Foundant override (confirmed by absence).
- GAP: stock or inconsistent today, Foundant treatment drafted or needed.
- WIP: section not yet rebuilt to real markup.

Nothing is described as ratified unless confirmed against real production files.

## Section rebuild template

1. `.usage` table with What, Where, When, Why, How rows.
2. Inline `.code-block` copy-paste markup under each demo group.
3. Real production classes in the demo.
4. A status badge (`.status-overridden` or `.status-default`) derived from evidence.
5. `.gap-flag` or `.gap-note` for anything not yet in production.

## Stop and ask: open decisions

Do not resolve these. If work touches one, describe the options in the PR and stop:

- UX-118 h2 to h6 type ramp
- UX-129 AG Grid theme direction
- UX-130 success, warning, info ramps
- UX-131 badge status-to-semantic map
- UX-132 hex vs rgb in token definitions
- UX-133 disabled button color per variant
- UX-138 remap sweep vs Sass build

General rule: if finishing a ticket requires a design choice that is not already recorded in the contract, it is a decision. Stop and ask.

## Evidence and corrections

- Look before cutting. Read the section, check it against production files, then edit. Never bulk regex across `index.html` or the foundation CSS.
- When an earlier claim in the kitchen sink or contract turns out to be wrong, record the correction in place (what was claimed, what is true, how it was confirmed). Do not quietly overwrite it.
- A ratification is only done when it exists in all three: contract (decision and why), foundation CSS (executable), kitchen sink (rendered demo and usage).

## Git workflow

- Never commit to `main`. Never merge a PR. Never force push.
- Branch per ticket: `UX-123/short-slug`.
- Commit messages start with the Jira key: `UX-123: add usage table to Tabs`.
- Add `[skip ci]` to every commit except the last one intended for deploy. Netlify production deploys cost credits.
- Open PRs as drafts. Fill in `.github/pull_request_template.md` completely.
- Push with git from the local clone. Open and update PRs with the GitHub MCP connector, not the gh CLI.
- Batch work: one PR per ticket, one production deploy per session at most.

## Jira

- Jira is the source of truth for work tracking. Do not mirror Jira tickets into GitHub issues. Use GitHub issues only for repo-internal work (build scripts, tooling).
- Jira writes are permanent: comments and links cannot be deleted through the connector. Post at most one comment per ticket, linking the draft PR.
- Do not transition ticket status, create links, or create tickets without explicit approval.
- Unattended runs (routines) only work tickets labeled claude-ready. A ticket named directly in a user's prompt is authorized by that prompt and needs no label.
- If a ticket's description is too thin to define done, stop and ask rather than guessing at acceptance criteria.
- Atlassian cloudId: 8ac51c62-b131-4177-9bfb-75275e7a7a6b
- Project key: UX
- Board query: project = UX AND statusCategory != Done ORDER BY key ASC
- Work queue: project = UX AND labels = claude-ready AND statusCategory != Done

## Writing style

- Use at most one em dash in any file or document you write.
- Prose over bullets in kitchen sink notes. State what was confirmed and how.
