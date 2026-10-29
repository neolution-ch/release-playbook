# Adoption templates

Copy-paste-ready files for adding the [Changesets release setup](../docs/CHANGESETS.md) to a
neolution-ch repository. Copy the contents of this `templates/` folder **into your repository root**,
then customize the few spots marked below.

These templates target **Changesets v3** (`@changesets/cli@^3`, `@changesets/changelog-github@^1`)
and **`changesets/action@v2`**. Changesets v3 requires Node.js `^22.11 || ^24 || >=26` and
npm `>=10.9.0` (pnpm `>=10.0.0`, Yarn `>=4.5.2`).

Your `package-lock.json` must be committed. Changesets v3 recognizes an npm workspace only when the
repo root has both a `workspaces` field and a lockfile; without it the repo is treated as one
package and `changeset` reports "No versionable packages found".

> These files live under `templates/` (not the repo's real `.github/`) so that this playbook repo
> doesn't run them. When you copy them into an adopting repo, they land in their normal locations
> (`.github/workflows/…`, `.changeset/…`, `scripts/…`).

## What to copy

```
templates/.github/workflows/ci.yml                  → .github/workflows/ci.yml
templates/.github/workflows/release.yml             → .github/workflows/release.yml
templates/.github/workflows/dependabot-changeset.yml→ .github/workflows/dependabot-changeset.yml
templates/.github/dependabot.yml                    → .github/dependabot.yml
templates/.changeset/config.json                    → .changeset/config.json
templates/scripts/generate-dependabot-changeset.mjs → scripts/generate-dependabot-changeset.mjs
templates/.npmrc                                     → .npmrc
```

Do **not** copy this `README.md`.

## What you must customize

| File                            | Change                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `.changeset/config.json`        | Set `changelog[1].repo` to `neolution-ch/<your-repo>`. Set the `fixed` array to your package group (or `[]` if you don't group packages — see note). For **public npm**, set `access` to `"public"`. If your packages are `"private": true` (e.g. a NuGet repo), add `"privatePackages": { "version": true, "tag": true }` — Changesets v3 skips private packages otherwise. |
| `.npmrc`                        | Keep for GitHub Packages under the `@neolution-ch` scope. For **public npm**, remove it (or point it at `registry.npmjs.org`) — it overrides the workflow's registry otherwise.                                                                                                                                                                                              |
| `.github/dependabot.yml`        | Adjust `directories`/ecosystems to your layout. Remove `/packages/*` for a single-package repo.                                                                                                                                                                                                                                                                              |
| `.github/workflows/release.yml` | For **public npm**: set `registry-url` to `https://registry.npmjs.org`, `NODE_AUTH_TOKEN` to `${{ secrets.NPM_TOKEN }}`, replace the `packages: write` permission with `id-token: write`, and append `--provenance` to the publish command. For **NuGet**: drop `packages: write`, `registry-url` and `NODE_AUTH_TOKEN`, and add the `version-script` from [Adapting for NuGet Packages](../docs/CHANGESETS.md#version-sync-script). |
| `.github/workflows/ci.yml`      | The **Check for changesets** step is skipped only for Version Packages PRs opened by `csag-changesets-bot[bot]`. If your GitHub App has a different name, use its bot login there.                                                                                                                                                                                          |
| `.github/workflows/dependabot-changeset.yml` | For **NuGet**, replace the detect step so it watches the `.csproj` files, see [Dependabot changesets for NuGet](../docs/CHANGESETS.md#dependabot-changesets-for-nuget).                                                                                                                                                                              |

## Using pnpm

The templates use npm. For a pnpm repo, pin pnpm in the root `package.json`
(`"packageManager": "pnpm@11.22.0"`) and change every workflow that installs dependencies:

- add an **Enable Corepack** step (`run: corepack enable`) right after **Setup Node.js**
- `npm ci` → `pnpm install --frozen-lockfile`
- `npx changeset …` → `pnpm exec changeset …`

pnpm blocks dependency build scripts by default. List the ones you need under `allowBuilds` in
`pnpm-workspace.yaml`. If a package sets `publishConfig.registry` or a scope registry, move it to
`.npmrc` as well (see the pnpm note in
[Publishing to public npm](../docs/CHANGESETS.md#publishing-to-public-npm)).

## Action pins

The templates pin actions to major versions (`actions/checkout@v7`). A repo that pins exact
versions (`actions/checkout@v7.0.1`) can keep doing so: Dependabot keeps the precision of each ref
when it updates it. `changesets/action@v2` is a branch, not a tag, so Dependabot never proposes
updates for it. Pin an exact tag such as `@v2.1.2` if you want Dependabot to track it.

## Required secrets

The Release and Dependabot Changeset workflows authenticate as a GitHub App so their pushes/PRs
re-trigger CI. Add these secrets (an **Actions** secret and a **Dependabot** secret with the same
values — Dependabot has a separate secret store):

- `CHANGESETS_BOT_CLIENT_ID`
- `CHANGESETS_BOT_PRIVATE_KEY`

> These names must match whatever your org actually configured. If neolution-ch already created
> org-level secrets under a different name, either reuse that name here or rename the secrets — the
> workflow references and the secret names have to agree.

## Single-package repos & no fixed group

`scripts/generate-dependabot-changeset.mjs` bumps the configured `fixed` group. If you have a single
package, put it in a one-element `fixed` group (e.g. `[["@neolution-ch/my-pkg"]]`) so Dependabot
updates get a real bump. The script **fails loudly** if a workspace package changed while no `fixed`
group is configured, rather than silently emitting a no-op changeset.
