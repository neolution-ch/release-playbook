# release-playbook

The canonical **[Changesets](https://changesets.dev) release guide** for
neolution-ch, plus copy-paste templates for adopting it in any repository.

Targets **Changesets v3** (`@changesets/cli@^3`) with **`changesets/action@v2`**, which needs
Node.js `^22.11 || ^24 || >=26`.

- 📖 **[docs/CHANGESETS.md](docs/CHANGESETS.md)** — the full guide: for contributors (adding
  changesets) and maintainers (releases, prereleases, backports, Dependabot automation, adapting for
  NuGet). Read this.
- 📦 **[templates/](templates/)** — ready-to-copy `ci.yml`, `release.yml`, `dependabot-changeset.yml`,
  `dependabot.yml`, `.changeset/config.json`, the Dependabot changeset script, and `.npmrc`.

This repo is documentation only — it does not itself publish any package.

## Already on Changesets v2?

Follow [Upgrading from Changesets v2](docs/CHANGESETS.md#upgrading-from-changesets-v2). The CLI and
the GitHub Action have to move in the same PR, and Changesets v3 stops versioning private packages
unless you opt back in.

## Adopt it in a new repo

1. Copy [`templates/`](templates/) into your repo root and customize the marked spots
   (see [templates/README.md](templates/README.md)).
2. Set each package's `package.json` `version` to its current published version.
3. Add the `CHANGESETS_BOT_APP_ID` / `CHANGESETS_BOT_PRIVATE_KEY` secrets (Actions **and**
   Dependabot).
4. Turn on branch protection for `main` requiring the **Changeset Check** status check.
5. Open a PR with a changeset and verify the flow end-to-end.

The full walkthrough — including the delta for **public npm vs GitHub Packages** — is in the
[Migration Guide](docs/CHANGESETS.md#migration-guide).
