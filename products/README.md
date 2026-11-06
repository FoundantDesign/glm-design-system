# Product shells for prototyping

Each Foundant product line that we prototype gets a folder here. The `glm-prototyping` skill reads these folders, so adding a product line is adding a folder, not rewriting the skill.

```
products/
  <product>/
    manifest.json     where the product's live CSS, JS and icons come from, where its rules live, which shells exist
    shells/*.html     starting pages: the product's real header, layout and footer, with one marked content area
```

## manifest.json

| Key | What it holds |
|---|---|
| `product`, `name` | Short id used in folder names and the prototype's meta tag, and the display name. |
| `shellVersion` | Date the shells were last checked against production. Each prototype records it. |
| `environment.origin` | The non-production environment whose static files the prototype loads live. They must load without a login. |
| `rules` | Where the product's design rules live (its DESIGN-SYSTEM.md or equivalent) and its rendered reference. |
| `fonts`, `styles`, `scripts`, `icons` | The product's own stack, in production's load order, as paths on `environment.origin`. |
| `proposals` | Optional stylesheet of design-lab proposals, loaded only when a prototype tests one. |
| `shells` | Each starting page: id, file, what it's for, and the persona it represents. |
| `behaviors` | Production scripts a prototype can call (toasts, panels), with their signatures. |

## Adding a product line

1. Confirm the product has a non-production environment whose CSS, JS and icons load without a login, and find its layout file for the load order.
2. Copy `glm/manifest.json` to `<product>/manifest.json` and replace every value. Leave out what the product doesn't have.
3. Build at least one shell from the product's real layout markup, not from memory. Mark the replaceable area with `PROTOTYPE CONTENT START` and `PROTOTYPE CONTENT END` comments.
4. Open each shell in a browser and check that the stylesheets load, the scripts run without console errors and the icons render.
5. Add the product to the table below.

| Product | Folder | Shells | Rules |
|---|---|---|---|
| Grant Lifecycle Manager | `glm/` | admin-page, entity-summary, applicant-page | GLM.Web/DESIGN-SYSTEM.md |
