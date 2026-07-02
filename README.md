# release-playbook

The canonical **[Changesets](https://github.com/changesets/changesets) release guide** for
neolution-ch, plus copy-paste templates for adopting it in any repository.

- 📖 **[docs/CHANGESETS.md](docs/CHANGESETS.md)** — the full guide: for contributors (adding
  changesets) and maintainers (releases, prereleases, backports, Dependabot automation, adapting for
  NuGet). Read this.
- 📦 **[templates/](templates/)** — ready-to-copy `ci.yml`, `release.yml`, `dependabot-changeset.yml`,
  `dependabot.yml`, `.changeset/config.json`, the Dependabot changeset script, and `.npmrc`.

This repo is documentation only — it does not itself publish any package.

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
