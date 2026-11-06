# GLM Design Lab: instructions for Claude Code

Read this file in full before changing anything in this repo.

## What this repo is

The GLM (Grant Lifecycle Manager) design lab, deployed on Netlify at https://glm-design-system.netlify.app/. It holds the kitchen sink (`index.html`), the prototypes the team tests internally and with users, and proposed updates to the design system.

It is not the design system's rulebook. That is `GLM.Web/DESIGN-SYSTEM.md` in the Foundant/glm repo, rendered in the app at `/Dev/DesignSystem`. Engineering owns shipped components and their documentation there; design owns design system decisions and this lab. When sources disagree, shipped code wins, then DESIGN-SYSTEM.md, then this repo.

## Files

- `index.html`: the kitchen sink. Loads production CSS live from beta (`https://beta.grantinterface.com`) in production's load order, then the proposals file, then the page furniture. Static files on beta need no login.
- `glm-prototype-foundation.css`: proposals only. Everything here is something production does not ship yet. Every block is tagged `PROPOSED` with its ticket so the GLM.Web drift report (`.claude/skills/design-system/drift.py`) files it correctly. Never copy production CSS into it.
- `kitchen-sink.css`: page furniture for `index.html` (usage tables, code blocks, status tags, reference strips). Not GLM components.
- `GLM-design-contract.md`: retired. A pointer to DESIGN-SYSTEM.md; the full text is in git history.
- Prototype folders (`form-building/`, `form-filling/`, `request-summary/`, `key-interactions/`, `applicant-assistant/`, `application-analysis/`) and the `glm-homepage*` files: self-contained prototypes and user-test findings. Each inlines its own CSS, so changing the files above does not change them.

## Rules

1. Production is the source of truth for anything it ships. Read GLM.Web (`GLM.Web/wwwroot/css/` and `DESIGN-SYSTEM.md`) before calling something shipped, proposed or missing. Never infer production state from memory or from older notes in `index.html`.
2. A proposal lives in `glm-prototype-foundation.css`, tagged `PROPOSED` and with its ticket. When it ships in GLM.Web, remove it here in the same change that updates the kitchen sink section's reference strip.
3. Style by remapping MDB `--mdb-*` variables onto Foundant tokens on the real MDB class. No color literals outside token definitions. Real MDB and production classes only; a new class is a proposal and needs design review.
4. Buttons are pill-shaped. Icons are Font Awesome Pro, `fa-light` by default, `fa-regular` for filled-state controls. Icon-only buttons are `.btn.btn-icon` with an `aria-label`.
5. Primary is `--colors-brand-primary-dark-blue`, dark navy, not blue.

## Kitchen sink sections

Each section heading carries one status and a reference strip directly under it:

- SHIPPED: in production and documented in DESIGN-SYSTEM.md. The demo renders production CSS.
- PROPOSAL (or SHIPPED + PROPOSAL): a block from `glm-prototype-foundation.css` applies in this section.
- PRODUCTION GAP: production ships it with a known defect and no fix has shipped.

The strip links the DESIGN-SYSTEM.md section, the open tickets, what this lab adds, and what changed since the August to October rebuild. Update the strip whenever a section's status changes. The older usage tables and notes inside sections are a historical record; when one turns out wrong, record the correction in place (what was claimed, what is true, how it was confirmed) rather than quietly overwriting it.

## Checking against production

- Drift: from a GLM.Web checkout, `python -I .claude/skills/design-system/drift.py --site-css <path to this repo>/glm-prototype-foundation.css`. Changed, Production-only and Untagged should be empty or explained; everything else belongs under Proposed.
- Rendering: open the page, or a branch copy, in a real browser. MDB can beat a `--mdb-*` remap at higher specificity, so check the rendered result, not just the variable. Colors inside SVG data URIs cannot be remapped (DESIGN-SYSTEM.md, Principles).

## Open decisions: stop and ask

Do not resolve these. If work touches one, describe the options in the PR and stop. The current list is the Open decisions table in DESIGN-SYSTEM.md; as of 6 Nov 2026:

- UX-129 AG Grid theme, pending vendor accessibility conformance
- UX-130 success, warning and info color ramps
- UX-131 domain status to badge mapping
- UX-132 hex vs rgb in token definitions
- UX-133 disabled button color per variant
- UX-146 slide-out vs modal
- UX-147 accordion vs collapsible card
- YOGI-8094 validation message placement
- UX-149 and UX-151 switch "on" color, which blocks the UX-116 proposal

If finishing a ticket needs a design choice that is not recorded in DESIGN-SYSTEM.md or on the ticket, it is a decision. Stop and ask.

## Git workflow

- Never commit to `main` or force push. Merge only when Gabe says to in the session.
- Branch per ticket: `UX-123/short-slug`. Commit messages start with the Jira key.
- Add `[skip ci]` to every commit except the last one intended for deploy. Netlify production deploys cost credits; aim for one per session.
- Open PRs as drafts and fill in `.github/pull_request_template.md`.

## Jira

- Jira is the source of truth for work tracking. Do not mirror tickets into GitHub issues.
- Comments and links cannot be deleted through the connector. At most one comment per ticket; correct a wrong comment by editing it.
- Do not transition status, create links or create tickets without explicit approval.
- Unattended runs only work tickets labeled `claude-ready`. A ticket named in the user's prompt is authorized by that prompt.
- Atlassian cloudId: 8ac51c62-b131-4177-9bfb-75275e7a7a6b. Project key: UX.
- Board query: `project = UX AND statusCategory != Done ORDER BY key ASC`

## Writing style

- At most one em dash in any file or document you write.
- Prose over bullets in kitchen sink notes. State what was confirmed and how.
