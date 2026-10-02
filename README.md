# Medium-inspired site — source archive

This is a source-only package of the latest local workspace version, including the dark Figma-inspired interface and membership page. The page's HTML, CSS, and browser JavaScript are in `index.html`; the CSS and JavaScript are embedded in that file. Membership is a local design preview only: it does not purchase a membership or start a payment or subscription.

## Contents

- `index.html` — current reading feed and membership UI, with inline styles and scripts.
- `assets/` — the six original JPEG image assets used by the page and Worker.
- `src/worker.js` — Cloudflare Worker API implementation.
- `migrations/` — D1 migrations, including the retained `owner_state` table and the editor CMS tables.
- `test/worker.test.js` — Worker API tests using an in-memory D1 test double.
- `scripts/build-assets.mjs` — generates the deployable `public/` asset directory from `index.html` and `assets/`.
- `scripts/hash-editor-password.mjs` — reads a password from stdin and prints a salted PBKDF2-SHA256 hash; it does not save the password.
- `package.json`, `wrangler.jsonc`, and `.gitignore` — project commands and configuration.

The package omits generated `public/` files; `npm run build:assets` recreates them. It also omits `.dev.vars`, `.env` files, `.wrangler/`, `node_modules/`, screenshots/output captures, and the older backend/reference notes whose deployment statements no longer match the current state. Fixed fake login strings in the tests were replaced in this package copy with random per-run test values. No production editor password or deployment token is included.

Image pixel content is retained. EXIF/XMP, IPTC/Photoshop, and JPEG comment metadata were stripped from the package copies only to avoid carrying embedded location or device metadata; the working project images were not changed.

## Requirements and local checks

Use Node.js 22 or newer with npm. The current package declares no runtime npm dependencies; the test suite uses Node's built-in test runner and Web APIs.

```sh
npm test
npm run build:assets
```

`npm test` runs the Worker tests. `npm run build:assets` creates `public/index.html` and `public/assets/` from the source page and images. The build command and tests were run against a staged copy of this archive, not against the working project.

For a local D1 database, Wrangler can apply both migrations locally:

```sh
npx wrangler d1 migrations apply medium-inspired-site-state --local
```

For local editor sign-in, run `node scripts/hash-editor-password.mjs` and provide the desired password through stdin. Put only the resulting hash in a local, ignored `.dev.vars` file as `EDITOR_PASSWORD_HASH=...`; never put the raw password in a command argument or source file. `.dev.vars` is intentionally not included.

## Deployment notes

The current `wrangler.jsonc` targets the existing D1 database binding named `medium-inspired-site-state`. Before any remote migration or deploy, verify that Wrangler is authenticated to the intended Cloudflare account and that this is the correct existing database. Configure `EDITOR_PASSWORD_HASH` as a Worker secret using a securely generated hash, then apply the migrations to the intended remote database and deploy only when ready. The package does not contain Cloudflare credentials or secret values.

`npm run deploy` runs the asset build and then invokes `npx wrangler deploy`. It is included for a future, explicitly authorized deployment; it was not run while preparing this package.

**Actual deployment state:** the Worker at `https://medium-inspired-site.andregsman.workers.dev` exists from the prior deployment, but its live smoke test returned Cloudflare **403/1010**. The latest dark-theme and membership UI edits are local-only and were not redeployed. This packaging task performed no remote URL checks and no deployment.
