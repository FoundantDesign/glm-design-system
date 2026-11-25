# GLM Design Lab: instructions for Claude Code

Read this file in full before changing anything in this repo.

## What this repo is

The GLM (Grant Lifecycle Manager) design lab, deployed on Netlify at https://glm-design-system.netlify.app/. It holds the kitchen sink (`index.html`), the prototypes the team tests internally and with users, and proposed updates to the design system.

It is not the design system's rulebook. That is `GLM.Web/DESIGN-SYSTEM.md` in the Foundant/glm repo, rendered in the app at `/Dev/DesignSystem`. Engineering owns shipped components and their documentation there; design owns design system decisions and this lab. When sources disagree, shipped code wins, then DESIGN-SYSTEM.md, then this repo.

## Files

- `index.html`: the kitchen sink. Loads production CSS live from beta (`https://beta.grantinterface.com`) in production's load order, then the proposals file, then the page furniture. Static files on beta need no login.
- `glm-prototype-foundation.css`: proposals only. Everything here is something production does not ship yet. Every block is tagged `PROPOSED` with its ticket so the GLM.Web drift report (`.claude/skills/design-system/drift.py`) files it correctly. Never copy production CSS into it.
- `kitchen-sink.css`: page furniture for `index.html` (usage tables, code blocks, status tags, reference strips). Not GLM components.
- `README.md`: a short summary of the repo for GitHub visitors. This file is the full guide.
- `GLM-design-contract.md`: retired. A pointer to DESIGN-SYSTEM.md; the full text is in git history.
- `products/<product>/`: what the `glm-prototyping` skill builds from. `manifest.json` lists the product's live CSS, scripts and icons on its non-production environment, where its rules live, and its shells; `shells/*.html` are starting pages with the product's real header, layout and footer. When production's layout changes, update the shells and bump `shellVersion`. `products/README.md` explains adding a product line.
- Prototype folders (`form-building/`, `form-filling/`, `request-summary/`, `key-interactions/`, `applicant-assistant/`, `application-analysis/`), the `glm-homepage*` files and `key-interactions-timeline-v4.html`: self-contained prototypes and user-test findings. Each inlines its own CSS, so changing the files above does not change them.

## Rules

1. Production is the source of truth for anything it ships. Read GLM.Web (`GLM.Web/wwwroot/css/` and `DESIGN-SYSTEM.md`) before calling something shipped, proposed or missing. Never infer production state from memory or from older notes in `index.html`.
2. A proposal lives in `glm-prototype-foundation.css`, tagged `PROPOSED` and with its ticket. When it ships in GLM.Web, remove it here in the same change that updates the kitchen sink section's reference strip.
3. Style by remapping MDB `--mdb-*` variables onto Foundant tokens on the real MDB class. No color literals outside token definitions. Real MDB and production classes only; a new class is a proposal and needs design review.
4. Buttons are pill-shaped. Icons are Font Awesome Pro, `fa-light` by default, `fa-regular` for filled-state controls. Icon-only buttons are `.btn.btn-icon` with an `aria-label`.
5. Primary is `--colors-brand-primary-dark-blue`, dark navy, not blue.

## Design process

This is design's current process, open to feedback from the rest of the team (UX-179).

1. Design proposes in this lab: a PROPOSED kitchen sink section or prototype, with its UX ticket and a draft PR.
2. The team discusses it async in Teams or live in the weekly design sync.
3. The team decides.
4. Engineering owns DESIGN-SYSTEM.md. When they are available, they open a PR in Foundant/glm to update it and the shipped code. Design does not edit DESIGN-SYSTEM.md.
5. Design closes its own tickets and PRs, and removes the shipped proposal from `glm-prototype-foundation.css` (Rule 2).
6. The kitchen sink loads CSS from beta, so it shows the shipped change as soon as beta updates. No lab deploy is needed for that.

## Pattern governance

This is design's current process, open to feedback from the rest of the team (UX-179). Apply it before proposing a new pattern, class or token.

### Default: use what exists
Most UI needs are solved with existing MDB5 components or existing design system patterns. A need only becomes a pattern candidate when nothing existing fits.

### What counts as a use case
A pattern requires at least two use cases. A use case is a distinct workflow or a distinct user role. Repeated use of the same UI within one workflow counts as one use case: inventing patterns in isolation hides whether the need is truly reusable.

### One-off solutions
When a need has one use case, it is solved as a one-off: a new combination of MDB5 components, or an established plugin already in production (for example AG Grid for data tables). One-offs introduce no new tokens and no new design system classes.

### Candidates
One-offs are logged in the Candidate swim lane on the UX board. When a second workflow or role surfaces the same need, the candidate moves to a pattern proposal in the lab. (Lane format open for team feedback.)

## Kitchen sink sections

Each section shows, in order: a heading with one status, a link to its DESIGN-SYSTEM.md rule, a one-line description, the What / Where / When / Why / How usage table, the live demo, and copy-paste markup. Nothing else. No history, correction notes or evidence trails on the page; those belong in the PR and in Jira.

- SHIPPED: in production. Any open defect ticket is linked next to the badge as "Defect: UX-123".
- PROPOSED: a proposed change or an open decision. Its UX ticket is linked next to the badge. A subsection can carry its own PROPOSED badge and ticket. Every proposal needs a UX ticket; if one is missing, say so and ask rather than inventing one.

Keep the usage tables current: they describe what production does today, in plain language, grounded in DESIGN-SYSTEM.md and the production CSS. Code blocks hold markup someone would copy, not copies of production CSS; production CSS already loads live. Proposal CSS can appear in a code block under its PROPOSED subsection.

## Checking against production

- Drift: from a GLM.Web checkout, `python -I .claude/skills/design-system/drift.py --site-css <path to this repo>/glm-prototype-foundation.css`. Changed, Production-only and Untagged should be empty or explained; everything else belongs under Proposed.
- Rendering: open the page, or a branch copy, in a real browser. MDB can beat a `--mdb-*` remap at higher specificity, so check the rendered result, not just the variable. Colors inside SVG data URIs cannot be remapped (DESIGN-SYSTEM.md, Principles).

## Open decisions: stop and ask

Do not resolve these. If work touches one, describe the options in the PR and stop.

The list of open decisions is the Open decisions table in `GLM.Web/DESIGN-SYSTEM.md`. Read it there each time; this file keeps no copy, because a copy goes stale. If the table and a ticket's Jira status disagree, the table wins, and say so in the PR. If you can't read GLM.Web, ask which decisions are open rather than guessing.

If finishing a ticket needs a design choice that is not recorded in DESIGN-SYSTEM.md or on the ticket, it is a decision. Stop and ask.

## Git workflow

- Never commit to `main` or force push. Merge only when Gabe says to in the session.
- Branch per ticket when there is one: `UX-123/short-slug` (a YOGI key works too), with commit messages starting with the key. Kitchen sink and proposal changes always have a ticket.
- A prototype doesn't need a ticket. Without one, use `proto/<short-slug>` and start commit messages with `PROTO:`. Create a UX ticket for a prototype only when the person asks for one.
- Add `[skip ci]` to every commit except the last one intended for deploy. Netlify production deploys cost credits; aim for one per session.
- Cloud Claude Code sessions are assigned a `claude/...` branch by the harness. Use it when it's the only branch the session may push to, keep one ticket or prototype per branch, and still start commit messages with the key or `PROTO:`.
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
