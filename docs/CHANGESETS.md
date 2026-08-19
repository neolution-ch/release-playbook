# Changesets Guide

This is the neolution-ch guide for working with changesets. It covers **every scenario** for both **contributors** (who add changes) and **maintainers** (who manage releases), and how to adopt the setup in a new repository.

---

## Table of Contents

- [Changesets Guide](#changesets-guide)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [For Contributors](#for-contributors)
    - [What Is a Changeset?](#what-is-a-changeset)
    - [Adding a Changeset](#adding-a-changeset)
    - [Choosing the Right Semver Bump](#choosing-the-right-semver-bump)
    - [Empty Changesets (Non-Releasable Changes)](#empty-changesets-non-releasable-changes)
    - [Multiple Changesets per PR](#multiple-changesets-per-pr)
    - [What Happens If You Forget a Changeset](#what-happens-if-you-forget-a-changeset)
    - [Changeset File Format](#changeset-file-format)
    - [Editing or Removing a Changeset](#editing-or-removing-a-changeset)
  - [For Maintainers](#for-maintainers)
    - [Regular Release Flow](#regular-release-flow)
    - [How Fixed Packages Work](#how-fixed-packages-work)
    - [Testing Packages Before Merge (pkg.pr.new)](#testing-packages-before-merge-pkgprnew)
    - [Prereleases / Release Candidates](#prereleases--release-candidates)
      - [How it works](#how-it-works)
      - [Step-by-step walkthrough](#step-by-step-walkthrough)
      - [Summary](#summary)
      - [Switching between prerelease phases](#switching-between-prerelease-phases)
      - [When to use prereleases](#when-to-use-prereleases)
      - [Emergency patches during a prerelease](#emergency-patches-during-a-prerelease)
      - [Pivoting from a minor to a major prerelease](#pivoting-from-a-minor-to-a-major-prerelease)
    - [Backporting / Hotfixes on Older Versions](#backporting--hotfixes-on-older-versions)
      - [Understanding the `release/v1.x` branch](#understanding-the-releasev1x-branch)
      - [Setting up a release branch (manual fallback)](#setting-up-a-release-branch-manual-fallback)
      - [Releasing a backport](#releasing-a-backport)
      - [Timeline example](#timeline-example)
    - [Recovering From a Failed Publish](#recovering-from-a-failed-publish)
    - [Manual Publish Fallback](#manual-publish-fallback)
    - [Reverting a Changeset](#reverting-a-changeset)
    - [Reverting a Release](#reverting-a-release)
    - [Skipping a Package From Release](#skipping-a-package-from-release)
    - [Force-Publishing Without Changes](#force-publishing-without-changes)
    - [Dependabot Automation](#dependabot-automation)
      - [How it works](#how-it-works-1)
      - [What gets generated](#what-gets-generated)
      - [Setup](#setup)
  - [Publishing to public npm](#publishing-to-public-npm)
  - [Adapting for NuGet Packages](#adapting-for-nuget-packages)
    - [Architecture](#architecture)
    - [Repository structure](#repository-structure)
    - [Version sync script](#version-sync-script)
    - [NuGet publish workflow](#nuget-publish-workflow)
    - [What stays the same](#what-stays-the-same)
  - [Upgrading from Changesets v2](#upgrading-from-changesets-v2)
    - [Step 1: Check Node and package manager versions](#step-1-check-node-and-package-manager-versions)
    - [Step 2: Update the packages](#step-2-update-the-packages)
    - [Step 3: Update `.changeset/config.json`](#step-3-update-changesetconfigjson)
    - [Step 4: Update the GitHub Actions workflows](#step-4-update-the-github-actions-workflows)
    - [Step 5: Update CLI commands and flags](#step-5-update-cli-commands-and-flags)
    - [Step 6: Migrate prerelease state](#step-6-migrate-prerelease-state)
    - [Step 7: Review the remaining behavior changes](#step-7-review-the-remaining-behavior-changes)
  - [Migration Guide](#migration-guide)
    - [Prerequisites](#prerequisites)
    - [Step 1: Install changesets](#step-1-install-changesets)
    - [Step 2: Configure changesets](#step-2-configure-changesets)
    - [Step 3: Set the initial version](#step-3-set-the-initial-version)
    - [Step 4: Add workflows](#step-4-add-workflows)
    - [Step 5: Set up the GitHub App](#step-5-set-up-the-github-app)
    - [Step 6: Configure branch protection](#step-6-configure-branch-protection)
    - [Step 7: Remove old release tooling](#step-7-remove-old-release-tooling)
    - [Step 8: First changeset](#step-8-first-changeset)
  - [Reference](#reference)
    - [Configuration File](#configuration-file)
    - [CI Workflows](#ci-workflows)
    - [Branch Protection](#branch-protection)
    - [Useful Commands](#useful-commands)
    - [Further Reading](#further-reading)

---

## Overview

This is the neolution-ch standard for managing versioning and changelogs with [Changesets](https://changesets.dev). It applies to any repo that publishes one or more npm packages. The examples below use two illustrative packages — `@neolution-ch/pkg-a` and `@neolution-ch/pkg-b` — configured as a **fixed** group. Substitute your own package names.

| Package (example)     | Path             |
| --------------------- | ---------------- |
| `@neolution-ch/pkg-a` | `packages/pkg-a` |
| `@neolution-ch/pkg-b` | `packages/pkg-b` |

A **fixed** group means the packages always share the same version number: when one is bumped, the others are bumped to the same version too, even if they have no changes. A repo can have a single package, one fixed group, or several — adjust the `fixed` array in `.changeset/config.json`.

By default packages publish to **GitHub Packages** (private/restricted). GitHub Releases are created automatically on publish. To publish to the **public npm registry** instead, see [Publishing to public npm](#publishing-to-public-npm).

This guide targets **Changesets v3** (`@changesets/cli@^3`, `@changesets/changelog-github@^1`) driven by **`changesets/action@v2`**. Changesets v3 ships as ES modules and requires Node.js `^22.11 || ^24 || >=26`, npm `>=10.9.0`, pnpm `>=10.0.0`, or Yarn `>=4.5.2`. Already running v2? Start at [Upgrading from Changesets v2](#upgrading-from-changesets-v2).

> **Adopting this in your repo?** Copy the files from [`templates/`](../templates/) and follow the [Migration Guide](#migration-guide). The workflows and the Dependabot script are generic (copy as-is); only `.changeset/config.json` (the `repo` field and `fixed` group) and `.npmrc` need per-repo customization.

---

## For Contributors

### What Is a Changeset?

A changeset is a Markdown file committed to the `.changeset/` directory that describes:

1. **Which packages** are affected
2. **What type of version bump** is needed (major, minor, or patch)
3. **A human-readable description** of the change (added to the changelog)

### Adding a Changeset

After making your code changes, run:

```bash
npx changeset
```

You will be prompted to:

1. **Select the package(s)** affected by your change — use arrow keys and space to select
2. **Choose the bump type** for each selected package (major / minor / patch)
3. **Write a summary** of what changed

This creates a new file in `.changeset/` with a random name like `.changeset/brave-dogs-dance.md`. There is no confirmation step — the file is written as soon as you finish the summary. Edit or delete it afterwards if you got it wrong.

> **Tip:** Because our packages are configured as **fixed**, you only need to select one package. The other will be bumped automatically to the same version.

To skip the prompts entirely — handy in scripts — pass the bump and message as flags:

```bash
npx changeset --patch @neolution-ch/pkg-a -m "Fix the off-by-one in the paginator"
```

**Commit this file alongside your code changes.**

### Choosing the Right Semver Bump

| Bump                          | When to use                                                                    | Example                                                                         |
| ----------------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| **patch** (`1.0.0` → `1.0.1`) | Bug fixes, documentation updates, internal refactors that don't change the API | Fix a typo in output, correct a calculation                                     |
| **minor** (`1.0.0` → `1.1.0`) | New features that are backwards-compatible                                     | Add a new exported function, add a new option                                   |
| **major** (`1.0.0` → `2.0.0`) | Breaking changes                                                               | Rename/remove a public function, change return types, drop Node version support |

When in doubt, prefer the **smallest** bump type that is accurate.

### Empty Changesets (Non-Releasable Changes)

The CI check only requires a changeset when your PR **changes files inside a package** (e.g. `packages/pkg-a/`). PRs that touch only non-package files pass automatically with no changeset — for example:

- Updating CI configuration
- Changing test files that live outside a package
- Modifying documentation that isn't in a package
- Refactoring root-level build scripts

You only need an **empty changeset** when you changed a package but intentionally don't want a release (for example, a comment-only or test-only change *inside* a package):

```bash
npx changeset --empty
```

This creates a changeset file that tells CI "I intentionally have no release." It will not trigger any version bumps or changelog entries.

### Multiple Changesets per PR

You can add **multiple changesets** to a single PR. This is useful when:

- You made multiple distinct changes that should have separate changelog entries
- You want to bump different packages at different levels

Just run `npx changeset` multiple times. Each creates a separate file.

### What Happens If You Forget a Changeset

If your PR **changed a package** but has no changeset, the **CI workflow fails**. The `Changeset Check` job runs `changeset status --since=origin/<base>`, which exits with code 1 only when a versionable package changed and no changeset exists. (A PR that changed no package passes without one.)

> **"Versionable"** excludes private (`"private": true`) packages. Changesets v3 neither versions nor tags them unless you opt in with `"privatePackages": { "version": true, "tag": true }` in `.changeset/config.json` — which repos that use changesets purely for versioning, such as [NuGet repos](#adapting-for-nuget-packages), have to do.

To fix it:
1. Run `npx changeset` (or `npx changeset --empty` if the package change intentionally has no release)
2. Commit and push the generated file

The commit is not optional: `--since` compares committed history, so an unstaged or uncommitted `.changeset/*.md` does not satisfy the check.

### Changeset File Format

A changeset file looks like this:

```markdown
---
"@neolution-ch/pkg-a": minor
---

Add support for custom configuration options
```

The YAML frontmatter declares which packages are bumped and at what level. The body is the changelog entry (supports full Markdown).

### Editing or Removing a Changeset

- **Edit**: Simply modify the `.changeset/*.md` file — change the bump type in the YAML frontmatter or update the description
- **Remove**: Delete the `.changeset/*.md` file if the change is no longer needed

---

## For Maintainers

### Regular Release Flow

This is the standard release workflow that happens automatically:

```
Developer opens PR → adds changeset → PR merged to main
                                           ↓
                              Release workflow runs
                                           ↓
                          changesets/action detects changesets
                                           ↓
                    Creates/updates "chore: version packages" PR
                        (bumps versions, updates CHANGELOGs)
                                           ↓
                      Maintainer reviews and merges the PR
                                           ↓
                        Release workflow runs again on main
                                           ↓
                  changesets/action runs `changeset publish`
                                           ↓
              Packages published to GitHub Packages + GitHub Releases created
```

**Step by step:**

1. PRs with changesets get merged into `main`
2. The Release workflow automatically creates (or updates) a PR titled **"chore: version packages"**
3. This PR contains all the version bumps and CHANGELOG updates
4. When you're ready to release, **merge the Version Packages PR**
5. The Release workflow runs again, detects no pending changesets, and runs `changeset publish`
6. Packages are published to GitHub Packages and GitHub Releases are created

> **Batching releases:** You can merge multiple PRs with changesets before merging the Version Packages PR. All changesets will be combined into a single release.

### How Fixed Packages Work

Both `@neolution-ch/pkg-a` and `@neolution-ch/pkg-b` are in a **fixed** group. This means:

- They **always share the same version number**
- When a changeset bumps one package, the other gets bumped to the same version automatically
- The **highest bump type wins** — if `pkg-a` gets a `minor` bump and `pkg-b` gets a `patch`, both end up with a `minor` bump
- As long as **at least one** package in the group has a changeset, **both are bumped and published** to the same version — even if the other has no changes (with no changeset for any group member, nothing is published)

**Example:**

Starting state: both at `0.0.0`

| Changeset says               | Result                      |
| ---------------------------- | --------------------------- |
| `pkg-a: patch`               | Both → `0.0.1`              |
| `pkg-a: minor`               | Both → `0.1.0`              |
| `pkg-a: minor, pkg-b: patch` | Both → `0.1.0` (minor wins) |
| `pkg-b: major`               | Both → `1.0.0`              |

### Testing Packages Before Merge (pkg.pr.new)

To test package changes before merging a PR, we use [pkg.pr.new](https://pkg.pr.new/) instead of snapshot releases. pkg.pr.new provides installable package URLs for every commit — without publishing anything to a registry.

This avoids polluting GitHub Packages with temporary snapshot versions that can never be deleted.

pkg.pr.new automatically comments on PRs with install instructions like:

```bash
npm install https://pkg.pr.new/@neolution-ch/pkg-a@<commit-sha>
```

No per-repo configuration is needed — pkg.pr.new is set up as a GitHub App.

### Prereleases / Release Candidates

For planned major releases or beta periods, use changesets' built-in prerelease mode.

> **Note:** Prerelease mode is stored in `.changeset/pre.json`, created by `changeset pre enter`. A fresh repo should **not** have this file. If you inherit a repo that's stuck in prerelease mode, run `npx changeset pre exit` before cutting a stable release.
>
> While prerelease mode is active, every changeset that has already shipped in a prerelease is moved from `.changeset/` into `.changeset/pre/`. Those files stay there to build the changelog of the eventual stable release; edit or delete them directly if an entry is no longer relevant. `.changeset/pre.json` never has to be hand-edited to keep up.

**Allowed prerelease tags:**

| Tag     | Purpose                                                              | Version example |
| ------- | -------------------------------------------------------------------- | --------------- |
| `alpha` | Early unstable builds for internal testing — APIs may still change   | `2.0.0-alpha.0` |
| `beta`  | Feature-complete preview for broader testing — APIs mostly stable    | `2.0.0-beta.0`  |
| `rc`    | Release candidate, believed ready — only critical fixes before final | `2.0.0-rc.0`    |

#### How it works

Prereleases always happen **on `main`**. While prerelease mode is active, every release from `main` produces prerelease versions (e.g., `2.0.0-beta.0`) instead of stable ones. No stable releases can happen until you exit prerelease mode.

If you need to maintain the previous major version during the beta period, branch it off **before** entering prerelease mode. The Release workflow automatically creates a `release/vX.x` branch when a major version is published, so in most cases this is already done for you (see [Backporting / Hotfixes](#backporting--hotfixes-on-older-versions)).

```
main:              1.4.0 → 2.0.0-beta.0 → 2.0.0-beta.1 → 2.0.0 (stable)
release/v1.x:         └→ 1.4.1 → 1.4.2  (maintenance, stable)
```

#### Step-by-step walkthrough

This example assumes packages are at `1.4.0` and you want to beta-test `2.0.0`.

**1. Enter prerelease mode (via PR):**

```bash
git checkout -b chore/enter-beta
npx changeset pre enter beta
git add .changeset/pre.json
git commit -m "Enter prerelease mode (beta)"
git push -u origin chore/enter-beta
# Open PR → main, merge it
```

**2. The Release workflow takes over:**

Once `pre.json` is on `main`, the existing changesets (with major bumps) are versioned as `2.0.0-beta.0` instead of `2.0.0`. The workflow creates a Version Packages PR → you merge it → packages are published to the `beta` dist-tag automatically.

Users running `npm install @neolution-ch/pkg-a` still get `1.4.0`. Only `npm install @neolution-ch/pkg-a@beta` gets `2.0.0-beta.0`.

**3. More betas (normal development continues):**

Merge more PRs with changesets into `main` as usual. Each time you merge the Version Packages PR, the next beta is published: `2.0.0-beta.1`, `2.0.0-beta.2`, etc.

The version stays `2.0.0-beta.X` — new patch or minor changesets during the beta just increment the beta counter, they don't change the base version.

**4. Exit prerelease mode (via PR):**

```bash
git checkout -b chore/exit-beta
npx changeset pre exit
git add .changeset/pre.json
git commit -m "Exit prerelease mode"
git push -u origin chore/exit-beta
# Open PR → main, merge it
```

The Version Packages PR now shows `2.0.0` (stable). Merge it → published to `latest`.

#### Summary

```
1.4.0 (stable, latest)
  │
  ├── PR: enter prerelease mode (beta)     → merge to main
  ├── Version Packages PR                  → 2.0.0-beta.0 published @beta
  │
  ├── (more PRs with changesets)
  ├── Version Packages PR                  → 2.0.0-beta.1 published @beta
  │
  ├── PR: exit prerelease mode             → merge to main
  └── Version Packages PR                  → 2.0.0 published @latest
```

> **Important:** While prerelease mode is active on `main`, **no stable releases can ship from `main`**. For short beta cycles (days to a couple of weeks) this is fine. For very long betas, consider whether you need stable releases from `main` during that time.

#### Switching between prerelease phases

You can progress forward through prerelease phases: **alpha → beta → rc → stable**. To switch (e.g., from alpha to beta), edit the `tag` value in `.changeset/pre.json` — there is no need to exit and re-enter prerelease mode:

```bash
git checkout -b chore/switch-to-beta
# In .changeset/pre.json, change "tag": "alpha" to "tag": "beta"
git add .changeset/pre.json
git commit -m "Switch from alpha to beta"
git push -u origin chore/switch-to-beta
# Open PR → main, merge it
```

If you already published versions under the previous phase (e.g., `2.0.0-alpha.3`), you need at least one pending changeset for the Release workflow to produce `2.0.0-beta.0` — the changesets already consumed by the alphas now live in `.changeset/pre/` and don't count. If there are no pending changesets, add one:

```bash
npx changeset
# Select any package, choose patch, write "Promote to beta"
```

**Full progression example:**

```
2.0.0-alpha.0 → 2.0.0-alpha.1 → 2.0.0-alpha.2
    ↓ switch to beta (exit + re-enter)
2.0.0-beta.0  → 2.0.0-beta.1
    ↓ switch to rc (exit + re-enter)
2.0.0-rc.0    → 2.0.0-rc.1
    ↓ exit prerelease mode
2.0.0 (stable)
```

> **Never go backwards** (e.g., beta → alpha). Semver sorts `alpha` before `beta`, so `2.0.0-alpha.0` is considered a *lower* version than `2.0.0-beta.0`. Package managers won't upgrade users from beta to alpha because it looks like a downgrade. If you promoted to beta too early, just keep publishing more betas (`beta.1`, `beta.2`, ...) — the counter is unlimited.

#### When to use prereleases

Prereleases block stable releases from `main`. Before entering prerelease mode, consider whether it's actually needed:

| Scenario                                               | Recommendation                                                                   |
| ------------------------------------------------------ | -------------------------------------------------------------------------------- |
| **Major release** (breaking changes)                   | Prereleases make sense — consumers need time to adapt                            |
| **Minor release** (new features, backwards-compatible) | Usually unnecessary — just release stable and ship a patch if something is wrong |
| **Patch release**                                      | Never needs a prerelease                                                         |

For minor releases, **pkg.pr.new** often covers the "test before shipping" need without blocking `main`.

If you do run a prerelease for a minor version and then need an emergency patch on the current stable version, see [Emergency patches during a prerelease](#emergency-patches-during-a-prerelease).

#### Emergency patches during a prerelease

If `main` is in prerelease mode (e.g., shipping `1.3.0-beta.0`) and you need to release an urgent patch on top of the current stable version (`1.2.3` → `1.2.4`), you can't do it from `main` — versions there are already `1.3.0-beta.X`.

Instead, create a release branch from the last stable tag and use the normal backport flow:

**1. Create the release branch (one-time):**

```bash
git checkout -b release/v1.2.x @neolution-ch/pkg-a@1.2.3
git push -u origin release/v1.2.x
```

**2. Fix, changeset, PR — same as any backport:**

```bash
git checkout release/v1.2.x && git pull
git checkout -b fix/urgent-bug
# Apply the fix
npx changeset   # select patch
git push -u origin fix/urgent-bug
# Open PR → release/v1.2.x (not main!)
```

**3. Merge → Version Packages PR → merge → `1.2.4` published.**

**4. Cherry-pick the fix to `main`** so it's also included in the upcoming `1.3.0`:

```bash
git checkout main && git pull
git checkout -b fix/cherry-pick-urgent-bug
git cherry-pick <commit-sha>
npx changeset   # select patch
git push -u origin fix/cherry-pick-urgent-bug
# Open PR → main
```

The timeline looks like this:

```
main:              1.2.3 → 1.3.0-beta.0 → 1.3.0-beta.1 (includes cherry-picked fix) → 1.3.0
                      │
release/v1.2.x:       └→ 1.2.4 (emergency patch)
```

> **Note:** Unlike major version release branches (which are auto-created), you must create this branch manually. This is intentional — minor prereleases are rare, and the branch naming uses the full minor version (`v1.2.x`) to distinguish it from the major version branch (`v1.x`).

#### Pivoting from a minor to a major prerelease

If you entered prerelease mode for a minor version (e.g., `2.1.0-beta.0`) but then realize you need breaking changes, you can add a major changeset. The next `changeset version` will produce `3.0.0-beta.0` — changesets recalculates the base version from all pending changesets each time.

Consumers on the `@beta` tag will jump from `2.1.0-beta.0` to `3.0.0-beta.0`. This is valid semver (3.0.0 > 2.1.0) but may surprise beta users.

**If you still need stable `2.1.0`** (the minor features from the beta, without the breaking changes), you can ship it from a release branch:

1. Create `release/v2.x` from the last beta tag before the major change was added:
   ```bash
   git checkout -b release/v2.x @neolution-ch/pkg-a@2.1.0-beta.0
   git push -u origin release/v2.x
   ```

2. Exit prerelease mode on that branch (via PR targeting `release/v2.x`):
   ```bash
   git checkout -b chore/exit-beta-v2
   npx changeset pre exit
   git add .changeset/pre.json
   git commit -m "Exit prerelease mode"
   git push -u origin chore/exit-beta-v2
   # Open PR → release/v2.x
   ```

3. Merge → Version Packages PR shows `2.1.0` (stable) → merge → published with `v2` dist-tag.

The timeline:

```
main:           2.0.0 → 2.1.0-beta.0 → 3.0.0-beta.0 → 3.0.0 (stable)
                             │
release/v2.x:                └→ exit beta → 2.1.0 (stable) → 2.1.1 (patches)
```

> **No conflict with auto-creation:** When `3.0.0` is published from `main`, the workflow tries to auto-create `release/v2.x` but sees it already exists and skips it.

### Backporting / Hotfixes on Older Versions

When `main` has moved on to a new major version (e.g., v2.x) but you need to release a fix for an older major version (e.g., v1.x), you use a **release branch**. This covers both planned backports and urgent hotfixes — the process is identical.

#### Understanding the `release/v1.x` branch

`release/v1.x` is a **long-lived maintenance branch** for the entire v1 line. The naming convention is:

- `release/v1.x` — maintains all v1 releases
- `release/v2.x` — maintains all v2 releases (when v3 is on `main`)
- etc.

The `.x` is just a naming convention meaning "this branch handles all releases for this major version." It is **not** a git pattern or wildcard — it's a plain branch name.

**Key rules:**
- The Release workflow **automatically creates** this branch when a new major version is published from `main` (e.g., publishing `2.0.0` creates `release/v1.x` from the last v1 tag)
- You **reuse** it for all subsequent patches and minor releases for that major version
- It accumulates releases linearly: v1.5.2 → v1.5.3 → v1.6.0 → v1.6.1 → ...
- The CI and Release workflows are configured to run on `release/**` branches, so the process is fully automated — just like `main`

#### Setting up a release branch (manual fallback)

In most cases the release branch is created automatically. If you need to create one manually (e.g., retroactively):

```bash
# Create the release branch from the last v1 tag
git checkout -b release/v1.x @neolution-ch/pkg-a@1.5.2
git push -u origin release/v1.x
```

> **Tip:** After pushing, configure **branch protection rules** for `release/**` in GitHub Settings (same rules as `main` — require status checks, require PR reviews). This ensures backports go through CI and code review.

#### Releasing a backport

The process is the same as a normal release — just target the release branch instead of `main`:

1. **Create a feature branch** from the release branch:
   ```bash
   git checkout release/v1.x
   git pull
   git checkout -b fix/backport-my-fix
   ```

2. **Cherry-pick or apply the fix:**
   ```bash
   git cherry-pick <commit-sha>
   # or make the fix directly
   ```

3. **Add a changeset:**
   ```bash
   npx changeset
   # Select the affected package(s) and bump type (usually patch)
   ```

4. **Push and open a PR targeting `release/v1.x`** (not `main`):
   ```bash
   git push -u origin fix/backport-my-fix
   # Open PR: fix/backport-my-fix → release/v1.x
   ```

5. **CI runs automatically** — the Changeset Check validates against `release/v1.x`

6. **Merge the PR** — the Release workflow runs on `release/v1.x`, creating a Version Packages PR targeting that branch

7. **Merge the Version Packages PR** — packages are published to GitHub Packages and a GitHub Release is created

8. **Cherry-pick back to `main`** if the fix also applies to the current version

#### Timeline example

```
main:          v1.5.2 ──── v2.0.0 ──── v2.1.0 ──── v2.2.0
                  │
release/v1.x:     └──── v1.5.3 ──── v1.5.4 ──── v1.6.0
                        (backport)  (backport)  (backport)
```

All releases from `release/v1.x` go through the same automated flow: PR → CI check → merge → Version Packages PR → merge → publish.

> **Dist-tags:** Backport releases are published with a version-specific dist-tag (e.g., `release-v1`) instead of `latest`. This means `npm install @neolution-ch/pkg-a` always returns the highest stable version from `main`, while `npm install @neolution-ch/pkg-a@release-v1` returns the latest v1 release.

> **Don't combine a release branch with prerelease mode.** `changeset publish --tag <name>` is rejected outright while `pre.json` is in `pre` mode — "Releasing under custom tag is not allowed in pre mode!", exit 1. Since the Release workflow adds `--tag release-vN` on every `release/**` branch, a prerelease there will fail the publish step. Exit prerelease mode before the Version Packages PR is merged, or drop the `--tag` for that branch. Prereleases on `main` are unaffected: no `--tag` is passed there, and changesets publishes under the prerelease tag by itself.

> **Changelogs:** Each branch maintains its own `CHANGELOG.md`. The changelog on `release/v1.x` only contains v1 entries, while `main` only contains entries from `main`. The **GitHub Release page** is the canonical cross-branch reference for all release notes.

### Recovering From a Failed Publish

If the publish step fails (e.g., network error, registry issue):

1. **Check what was published** — look at the GitHub Packages page for your packages
2. **If nothing was published**: Re-run the Release workflow from the Actions tab, or manually run:
   ```bash
   npx changeset publish
   ```
3. **If some packages were published**: Changesets is idempotent — running `changeset publish` again will skip already-published versions and publish the remaining ones. Git tags are checked both locally and against the remote, so a re-run reports existing tags rather than trying to recreate them
4. **If versions were bumped but not published**: The Version Packages PR was already merged, so versions are bumped in `main`. Just re-run publish.

### Manual Publish Fallback

If the CI publishing is not working and you need an emergency release:

```bash
# 1. Pull the latest main
git checkout main && git pull

# 2. Make sure versions are bumped (if not already)
npx changeset version
git add . && git commit -m "Version packages" && git push

# 3. Authenticate to GitHub Packages
npm login --registry=https://npm.pkg.github.com --scope=@neolution-ch

# 4. Publish
npx changeset publish

# 5. Push tags
git push --follow-tags
```

> **Note:** `changeset version` exits with code **1** when there are no unreleased changesets. Skip step 2 entirely if the versions are already bumped — and if you wrap these commands in a script that runs under `set -e`, guard the call with `npx changeset version || true` so an empty release doesn't abort it.

> **Tags without npm:** `changeset publish` creates the git tags as part of publishing. If you only need the tags (a failed publish that already went through, or a repo that publishes elsewhere), run `npx changeset git-tag`.

### Reverting a Changeset

If a changeset was added by mistake (before it's been versioned):

```bash
# Simply delete the changeset file
rm .changeset/brave-dogs-dance.md
git add . && git commit -m "Remove incorrect changeset"
```

### Reverting a Release

If a version was published that needs to be undone:

1. **You cannot un-publish from GitHub Packages** once published (npm registry limitation)
2. Instead, publish a **new patch version** with the fix or revert:
   ```bash
   git revert <commit-of-bad-change>
   npx changeset  # add a patch changeset describing the revert
   # Merge to main, let the regular release flow publish the fix
   ```
3. If the version is truly broken, you can **deprecate** it:
   ```bash
   npm deprecate @neolution-ch/pkg-a@1.2.0 "This version has a critical bug, use 1.2.1" --registry=https://npm.pkg.github.com
   ```

### Skipping a Package From Release

To temporarily prevent a package from being published (e.g., it's not ready):

Add it to the `ignore` array in `.changeset/config.json`:

```json
{
  "ignore": ["@neolution-ch/pkg-b"]
}
```

> **Warning:** This is for **temporary** use only. If a changeset references both an ignored and non-ignored package, publishing will fail. Remove the package from `ignore` when you're ready to publish it again.

> **Dependents have to be ignored too.** As soon as `ignore` is non-empty (or private packages are excluded from versioning), config validation walks the dependency graph and errors if anything that depends on a skipped package is not itself skipped. `devDependencies` are exempt.

> **Permanent exclusion:** If a package should never be published, set `"private": true` in its `package.json` instead. Changesets v3 skips private packages entirely — no version bump, no git tag — unless `privatePackages` opts them back in.

### Force-Publishing Without Changes

Sometimes you need to publish a new version even with no code changes (e.g., to update metadata, rebuild with a new toolchain):

```bash
npx changeset
# Select the package(s) and choose "patch"
# Write something like "Rebuild with updated toolchain"
```

Since the packages are **fixed**, bumping either one will bump both.

### Dependabot Automation

Dependabot PRs don't include changeset files, so CI will initially fail. A dedicated workflow automatically generates a changeset for every Dependabot PR — no manual intervention needed.

#### How it works

```
Dependabot opens PR
  → CI fails (no changeset)
  → Dependabot Changeset workflow triggers:
      1. Checks if a changeset already exists → skips if yes
      2. Checks if any workspace package.json files changed (git diff)
      3. Parses the PR title and body to extract dependency update info
      4. Generates a changeset file
      5. Commits and pushes it (via GitHub App token)
  → Push triggers CI re-run → CI passes ✓
```

The workflow determines the changeset type based on what changed:

| What changed                                             | Changeset type                            | Effect                                                                                                                     |
| -------------------------------------------------------- | ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Workspace `package.json` (direct dependency update)      | `patch` bump for all fixed-group packages | Triggers a release — consumers' dependency tree changes                                                                    |
| Only lockfile or root `package.json` (transitive update) | Empty changeset (no bump)                 | Passes CI but does **not** trigger a release — consumers are unaffected because published packages don't include lockfiles |

#### What gets generated

The script parses the PR body to build a summary table. Dependabot's phrasing varies by PR type, so the parser accepts a few forms:

- **Single / security PRs**: `Bumps [package](url) from X to Y`
- **Grouped PRs**: one line per dependency reading `Updates <name> from X to Y`, where `<name>` is either a markdown link `[package](url)` or written in backticks

The generated changeset includes the PR title and a table with emoji bump indicators:

**Single dependency update:**

```markdown
---
"@neolution-ch/pkg-a": patch
"@neolution-ch/pkg-b": patch
---

Bump jws from 4.0.0 to 4.0.1

| Package | From  | To    | Bump    |
| ------- | ----- | ----- | ------- |
| jws     | 4.0.0 | 4.0.1 | 🟢 patch |
```

**Grouped dependency update:**

```markdown
---
"@neolution-ch/pkg-a": patch
"@neolution-ch/pkg-b": patch
---

Bump the npm-ecosystem group with 3 updates

| Package    | From   | To     | Bump    |
| ---------- | ------ | ------ | ------- |
| react      | 17.0.2 | 18.0.0 | 🔴 major |
| typescript | 5.3.0  | 5.4.0  | 🟡 minor |
| eslint     | 8.56.0 | 8.57.0 | 🟢 patch |
```

If the PR body can't be parsed (unexpected format), the changeset falls back to using just the PR title — it never fails.

#### Setup

The workflow (`.github/workflows/dependabot-changeset.yml`) and script (`scripts/generate-dependabot-changeset.mjs`) are provided in [`templates/`](../templates/) — copy them into your repo. The only manual step is adding **Dependabot secrets**:

1. Go to **Settings** → **Secrets and variables** → **Dependabot**
2. Add `CHANGESETS_BOT_CLIENT_ID` — same value as the existing Actions secret
3. Add `CHANGESETS_BOT_PRIVATE_KEY` — same value as the existing Actions secret

> **Why Dependabot secrets?** Dependabot PRs cannot access regular Actions secrets for security reasons. GitHub provides a separate secrets store specifically for Dependabot-triggered workflows. This avoids using `pull_request_target` (which runs with elevated privileges and is a security risk).

> **Note:** The workflow uses the GitHub App token (not `GITHUB_TOKEN`) to push commits, so the pushed changeset triggers a CI re-run — same pattern as the Release workflow.

---

## Publishing to public npm

By default this setup publishes to **GitHub Packages** (private/restricted). To publish to the **public npm registry** (`registry.npmjs.org`) instead, apply **all** of the following — changing only some of them silently fails:

| #   | File                            | Change                                                                                              | Why                                                                                                                                                   |
| --- | ------------------------------- | --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `.changeset/config.json`        | `"access": "restricted"` → `"public"`                                                               | Scoped packages default to private on npm; the first publish returns HTTP 403 without this                                                            |
| 2   | `.npmrc`                        | Remove the `@neolution-ch:registry=…github.com` line (or set `registry=https://registry.npmjs.org`) | `.npmrc` overrides the workflow's registry — otherwise publishing silently goes to GitHub Packages                                                    |
| 3   | `packages/*/package.json`       | Set `publishConfig.registry` to `https://registry.npmjs.org` (or remove `publishConfig`)            | Overrides `.npmrc` during publish                                                                                                                     |
| 4   | `.github/workflows/release.yml` | `registry-url: "https://registry.npmjs.org"`                                                        | Points npm auth at the public registry                                                                                                                |
| 5   | `.github/workflows/release.yml` | `NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}`                                                         | `GITHUB_TOKEN` only authenticates to GitHub Packages; add an npm automation token as the `NPM_TOKEN` secret                                           |
| 6   | `.github/workflows/release.yml` | Add `id-token: write` to `permissions` and append `--provenance` to the publish command             | Publishes with [npm provenance](https://docs.npmjs.com/generating-provenance-statements) (supply-chain attestation) — recommended for public packages |

Everything else is registry-agnostic: the CI changeset gate, the Version Packages PR, changelog generation, dist-tags/backport tags, pkg.pr.new, and GitHub Releases all work identically.

> **pnpm repos:** when Changesets checks whether a version is already published, it follows pnpm's registry resolution and ignores `publishConfig.registry` and scope-based registry overrides in `package.json`. Row 3 above still governs where `npm publish` sends the tarball, but the *existence* check reads `.npmrc` — so a pnpm repo needs the registry configured there as well.

> **Trusted publishing:** npm now recommends [trusted publishing](https://docs.npmjs.com/trusted-publishers) (OIDC) over long-lived automation tokens, which expire after 90 days. It replaces `NODE_AUTH_TOKEN` entirely and requires splitting the release job so that `id-token: write` is granted only to the publishing step — see the [changesets automation guide](https://changesets.dev/guide/automating) for the `changesets/action/select-mode`, `/version`, `/pack`, and `/publish` sub-action layout. GitHub Packages does not support trusted publishing, so the default setup in this playbook keeps `NODE_AUTH_TOKEN`.

> **On the `access` field:** `access` is an npm CLI flag (`npm publish --access <value>`). It controls scoped-package visibility on **npmjs.org only** (`restricted` = private, `public` = public). GitHub Packages **ignores** it — there, visibility is governed by the repository/package settings, not this field.

---

## Adapting for NuGet Packages

Changesets is an npm tool, but it can manage versioning and changelogs for .NET (NuGet) packages with a thin bridge. The key insight: let changesets handle what it's good at (changeset files, CI checks, changelogs, GitHub Releases) and use a separate workflow for the actual NuGet publish.

Changesets supports this officially — see [Beyond npm](https://changesets.dev/guide/beyond-npm), which names NuGet explicitly, and the [Publish Git Tags Only](https://changesets.dev/guide/automating) recipe. The setup below is that pattern with the neolution-ch workflows wired in.

### Architecture

```
Changesets handles:                    Separate workflow handles:
├── Changeset files (.changeset/*.md)  ├── Triggers on: GitHub Release published
├── CI changeset check                 ├── Reads version from git tag
├── Version Packages PR                ├── dotnet pack with that version
├── Changelog generation               └── dotnet nuget push to GitHub Packages
├── Git tags
└── GitHub Releases
```

Changesets creates GitHub Releases with tags like `@scope/pkg@1.2.3`. The NuGet publish workflow triggers on `release: published` and extracts the version from the tag.

### Repository structure

Each .NET project needs a minimal `package.json` so changesets can track its version:

```
├── .changeset/
│   └── config.json
├── packages/
│   └── my-dotnet-lib/
│       ├── MyDotnetLib.csproj
│       ├── package.json          ← minimal, for changesets only
│       └── src/
├── package.json                  ← root workspace
└── .github/workflows/
    ├── ci.yml
    ├── release.yml               ← same as npm version
    └── nuget-publish.yml         ← new, triggers on release
```

The per-project `package.json` is minimal:

```json
{
  "name": "@neolution-ch/my-dotnet-lib",
  "version": "1.0.0",
  "private": true
}
```

> **Note:** `"private": true` prevents accidental npm publishing. This file exists solely for changesets version tracking.

Because every package here is private, `.changeset/config.json` **must** opt them back in. Changesets v3 skips private packages by default, and without this the whole setup goes quiet rather than failing:

```json
{
  "privatePackages": { "version": true, "tag": true }
}
```

`version: true` lets `changeset version` bump the `package.json` versions; `tag: true` lets `changeset publish` create the `@scope/pkg@1.2.3` git tags that `nuget-publish.yml` triggers on. Nothing is sent to npm — `changeset publish` skips private packages and only tags them. (`changeset git-tag` does the tagging on its own if you ever need it outside the release flow.)

Both sub-options matter, and each fails differently:

| Missing         | Symptom                                                                                                                                                                                                                                                                                             |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `version: true` | `changeset status` treats every package as unversionable, so the **Changeset Check passes on every PR** — the gate is a silent no-op, not a failure. `changeset` and `changeset --empty` meanwhile exit 1 with an error that never mentions `privatePackages`, so contributors can't add one either |
| `tag: true`     | `changeset publish` prints "No unpublished projects to publish.", exits 0, and creates no git tags — so no GitHub Release, and `nuget-publish.yml` never triggers                                                                                                                                   |

> The upstream migration guide suggests `{ "version": true, "tag": false }` for restoring v2 behavior. That is the right value for a repo whose private packages are internal fixtures; it is the **wrong** value here, because the git tags are the trigger for the NuGet publish.

> **Commit `package-lock.json`.** Changesets v3 only recognizes an npm workspace when the root has both a `workspaces` field **and** a `package-lock.json` (Yarn likewise needs `yarn.lock`). Without the lockfile the repo collapses to a single root package: `changeset` fails with "No versionable packages found", and any tag that does get created is named `v1.2.3` instead of `@scope/pkg@1.2.3` — which the version/package parsing in `nuget-publish.yml` below relies on. A .NET repo that never runs `npm install` locally is the easy way to trip over this.

### Version sync script

The Release workflow needs a custom version command that syncs `package.json` versions into `.csproj` files after `changeset version` runs. Add this as `scripts/sync-versions.mjs`:

```javascript
import fs from "node:fs";
import path from "node:path";
import { execSync } from "node:child_process";

// Run changeset version first
execSync("npx changeset version", { stdio: "inherit" });

// Find all packages with both package.json and .csproj
const packagesDir = path.join(import.meta.dirname, "..", "packages");
for (const dir of fs.readdirSync(packagesDir)) {
  const pkgJsonPath = path.join(packagesDir, dir, "package.json");
  const projDir = path.join(packagesDir, dir);

  if (!fs.existsSync(pkgJsonPath)) continue;

  const pkgJson = JSON.parse(fs.readFileSync(pkgJsonPath, "utf8"));
  const version = pkgJson.version;

  // Find .csproj files in this directory
  const csprojFiles = fs.readdirSync(projDir).filter((f) => f.endsWith(".csproj"));
  for (const csproj of csprojFiles) {
    const csprojPath = path.join(projDir, csproj);
    let content = fs.readFileSync(csprojPath, "utf8");

    if (content.includes("<Version>")) {
      content = content.replace(/<Version>[^<]*<\/Version>/g, `<Version>${version}</Version>`);
    } else {
      // Add Version to the first PropertyGroup
      content = content.replace(
        /<PropertyGroup>/,
        `<PropertyGroup>\n    <Version>${version}</Version>`
      );
    }

    fs.writeFileSync(csprojPath, content);
    console.log(`Updated ${csproj} to version ${version}`);
  }
}
```

Then configure the Release workflow to use it:

```yaml
- name: Create Release Pull Request or Publish
  uses: changesets/action@v2
  with:
      github-token: ${{ steps.app-token.outputs.token }}
      version-script: node scripts/sync-versions.mjs
      publish-script: npx changeset publish
```

This way the Version Packages PR includes both `package.json` and `.csproj` version bumps.

### NuGet publish workflow

Add `.github/workflows/nuget-publish.yml`:

```yaml
name: NuGet Publish

on:
  release:
    types: [published]

jobs:
  publish:
    name: Publish to NuGet
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@v6

      - name: Setup .NET
        uses: actions/setup-dotnet@v4

      - name: Extract version from tag
        id: version
        run: |
          # Tag format: @scope/pkg@1.2.3
          TAG="${GITHUB_REF_NAME}"
          VERSION="${TAG##*@}"
          echo "version=$VERSION" >> "$GITHUB_OUTPUT"

          # Extract package name to find the right .csproj
          PKG_NAME=$(echo "$TAG" | sed 's/@[^/]*\///' | sed 's/@.*//')
          echo "package=$PKG_NAME" >> "$GITHUB_OUTPUT"

      - name: Delete package.json from project
        run: |
          # Remove package.json so it doesn't leak into the NuGet package
          find packages/ -name "package.json" -delete

      - name: Pack
        run: |
          dotnet pack packages/${{ steps.version.outputs.package }} \
            -p:Version=${{ steps.version.outputs.version }} \
            -c Release \
            -o ./nupkgs

      - name: Push to GitHub Packages
        run: |
          dotnet nuget add source "https://nuget.pkg.github.com/${{ github.repository_owner }}/index.json" \
            --name github \
            --username ${{ github.repository_owner }} \
            --password ${{ secrets.GITHUB_TOKEN }} \
            --store-password-in-clear-text
          dotnet nuget push ./nupkgs/*.nupkg --source github
```

> **Key detail:** The `find packages/ -name "package.json" -delete` step removes all `package.json` files before packing so they don't end up inside the `.nupkg`. Leave the generated `CHANGELOG.md` files alone — `changesets/action` reads the entry for the released version out of the `CHANGELOG.md` beside each tagged `package.json` to build the GitHub Release body, and errors if the file exists without a matching entry.

### What stays the same

Everything documented in this guide works identically for NuGet repos:

- Contributors run `npx changeset` to add changeset files
- CI blocks PRs without changesets
- The Version Packages PR shows version bumps and changelog updates
- Prereleases, backporting, and fixed packages all work the same
- The only difference is the additional `nuget-publish.yml` workflow that runs after each GitHub Release

---

## Upgrading from Changesets v2

Existing repos on Changesets v2 need a handful of coordinated changes. Do them in **one PR** — the CLI and the GitHub Action have to move together, because `changesets/action@v1` only works with Changesets v2 and `changesets/action@v2` only works with Changesets v3.

### Step 1: Check Node and package manager versions

Changesets v3 requires:

- **Node.js** `^22.11 || ^24 || >=26`
- **npm** `>=10.9.0`, **pnpm** `>=10.0.0`, or **Yarn** `>=4.5.2` (Yarn Classic is no longer supported)

Bump `node-version` in every workflow that runs the CLI (`ci.yml`, `release.yml`, `dependabot-changeset.yml`) and in any `.nvmrc` or `engines` field. Bolt workspaces are no longer detected at all — migrate to npm, pnpm, or Yarn workspaces first.

> **Check that your lockfile is committed.** Workspace detection got stricter: an npm workspace is now recognized only when the repo root has both a `workspaces` field and a `package-lock.json`, and Yarn likewise needs `yarn.lock`. v2 was happy with the `workspaces` field alone. If the lockfile is missing or gitignored, the repo silently degrades to a single root package — `changeset` reports "No versionable packages found", and git tags come out as `v1.2.3` instead of `@scope/pkg@1.2.3`.

### Step 2: Update the packages

```bash
npm install -D @changesets/cli@^3 @changesets/changelog-github@^1
```

All `@changesets/*` packages now ship as **ES modules** — there is no CommonJS entry point left. If you only invoke `changeset` from npm scripts or CI, that changes nothing. A CommonJS custom changelog generator still loads, because Changesets imports it dynamically, but no file anywhere can `require()` a `@changesets/*` package any more; convert those to ESM.

If you need more time, the v2 line stays available under the `maintenance-v2` dist-tag (`npm install -D @changesets/cli@maintenance-v2`).

### Step 3: Update `.changeset/config.json`

| v2                                                                                                  | v3                                                                                                                                                                                        |
| --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `"$schema": ".../@changesets/config@3.x/schema.json"`                                               | `"$schema": "https://unpkg.com/@changesets/config@4.0.0/schema.json"`                                                                                                                     |
| `"prettier": false`                                                                                 | `"format": false` — the new option also covers Oxfmt, Deno, and dprint, and defaults to `"auto"` detection                                                                                |
| `"access": "private"`                                                                               | `"access": "restricted"`                                                                                                                                                                  |
| `"___experimentalUnsafeOptions_WILL_CHANGE_IN_PATCH": { "useCalculatedVersionForSnapshots": true }` | `"snapshot": { "useCalculatedVersion": true }`                                                                                                                                            |
| `"baseBranch": "main"`                                                                              | Optional — `main` is now the default. Keep an explicit `"master"` if that is your default branch                                                                                          |
| private packages versioned implicitly                                                               | `"privatePackages": { "version": true, "tag": false }` to keep that behavior — but use `"tag": true` if you rely on the git tags, as the [NuGet setup](#adapting-for-nuget-packages) does |

> **The private-package default is the one that bites.** v2 versioned private packages (without tagging them); v3 does neither unless you ask. A repo that uses changesets purely to version things it never publishes to npm — the [NuGet setup](#adapting-for-nuget-packages) is exactly this — will otherwise go quiet: no version bumps, no tags, no GitHub Releases, and no downstream publish. If you worked around the v2 default by listing fixtures in `ignore`, you can drop those entries now.

> **Nothing warns you about a stale config.** Unknown keys are neither rejected by the JSON schema nor flagged at runtime, and v1-format configs are now ignored silently. A leftover `"prettier": false` is therefore dropped on the floor while `format` falls back to `"auto"` — silently re-enabling the formatting you had turned off. Delete the old keys instead of leaving them next to the new ones.

### Step 4: Update the GitHub Actions workflows

`changesets/action@v2` renamed every input and output to kebab-case, and it no longer reads the `GITHUB_TOKEN` environment variable:

| v1                          | v2                                                                                                                       |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `with: publish:`            | `with: publish-script:`                                                                                                  |
| `with: version:`            | `with: version-script:`                                                                                                  |
| `with: commit:`             | `with: commit-message:`                                                                                                  |
| `with: title:`              | `with: pr-title:`                                                                                                        |
| `with: branch:`             | `with: pr-base-branch:`                                                                                                  |
| `with: commitMode:`         | `with: push-with-git-cli:` — a boolean, not an enum                                                                      |
| `with: setupGitUser:`       | Gone. Git identity is handled automatically: an identity you already configured is kept, otherwise `github-actions[bot]` |
| `env: GITHUB_TOKEN:`        | `with: github-token:`                                                                                                    |
| `outputs.publishedPackages` | `outputs.published-packages`                                                                                             |
| `outputs.hasChangesets`     | `outputs.has-changesets`                                                                                                 |
| `outputs.pullRequestNumber` | `outputs.pr-number`                                                                                                      |

You don't have to find them all by reading: v2 **fails the step with a migration error** naming any old input it was handed, and it refuses to run at all against a Changesets v2 CLI. That is why the CLI and the action have to move in the same PR.

Four behavior changes are worth knowing about:

- Release commits and tags are pushed through the **GitHub API** by default, so they are signed with GitHub's GPG key and attributed to whoever owns `github-token`. Set `push-with-git-cli: true` to push with the Git CLI instead.
- Git tags are now controlled separately by `push-git-tags` (default `true`). Setting `create-github-releases: false` no longer suppresses tags as a side effect — turn `push-git-tags` off explicitly if that is what you relied on.
- The action no longer writes an `.npmrc` when `NPM_TOKEN` is set. Registry auth has to come from `actions/setup-node`'s `registry-url` plus `NODE_AUTH_TOKEN`, which is what [`templates/.github/workflows/release.yml`](../templates/.github/workflows/release.yml) does.
- Published packages are no longer detected by scraping `changeset publish` stdout; the CLI writes them to the file named by the `CHANGESETS_OUTPUT` environment variable. A plain `publish-script: npx changeset publish` handles that itself, but a **wrapper script** has to pass `CHANGESETS_OUTPUT` through to the CLI it calls — otherwise the action merely warns, and skips the git tags and GitHub Releases.

The v2 action also exposes sub-actions (`select-mode`, `version`, `pack`, `publish`, `pr-status`, `pr-comment`) so a release workflow can be split across jobs and grant `id-token: write` only where it publishes. The single combined `changesets/action@v2` step used in [`templates/`](../templates/) remains supported. Two notes if you go the sub-action route: pin `@v2` rather than `@v2.0.0` (the `cwd` input was dropped in `v2.0.0` and restored in `v2.1.0`), and remember that `changesets/action/version` calls `changeset version` unconditionally, so it runs into the new exit-code-1 behavior on an empty release unless `select-mode` gates it.

### Step 5: Update CLI commands and flags

`changeset tag` is now `changeset git-tag` — the name makes it clear it creates **git** tags, not npm dist-tags. The old name still resolves, with a deprecation warning, so nothing breaks the moment you upgrade. Rename it anyway.

Stale **flags** do break immediately. v3 parses arguments with a library that rejects anything it doesn't recognize, so `--sinceMaster`, `--updateChangelog`, `--isPublic`, `--skipCI`, and `--commit` now fail the command outright instead of printing a deprecation warning. Grep your workflows and package scripts for them; `changeset status --sinceMaster` becomes `changeset status --since=main`.

`--since` still accepts any git ref rather than only a local branch name, so the `--since=origin/<base>` used by [`templates/.github/workflows/ci.yml`](../templates/.github/workflows/ci.yml) keeps working.

### Step 6: Migrate prerelease state

Only relevant if the branch you're upgrading is currently in prerelease mode. Run:

```bash
npx changeset status
```

Changesets moves the already-versioned changesets from `.changeset/` into `.changeset/pre/`, and drops their IDs and the removed `initialVersions` key from `.changeset/pre.json`. Commit the result. See [Prereleases](#prereleases--release-candidates) for how the directory is used from here on.

> `.changeset/pre/` has to be **committed** — those files are the changelog entries for the eventual stable release. Check that no `.gitignore` rule (a bare `pre/` pattern, say) swallows it, or the stable changelog comes out empty.

> Do this migration deliberately, in a PR. Any command that reads the prerelease state — `status`, `version`, `publish`, `add` — performs the migration as a side effect, so the first CI run on an unmigrated branch will move files in its own checkout and then throw the result away.

### Step 7: Review the remaining behavior changes

- **`changeset version` now exits with code 1** when there are no unreleased changesets; v2 exited `0` silently. Any script that runs it unconditionally under `set -e` will now fail on an empty release. The Release workflow is unaffected — the action only versions when there are changesets to consume.
- **A peer dependency bump now gives dependents a `patch`**, not a `major`. If a peer bump really is breaking for a dependent, add an explicit `major` changeset for it.
- **pnpm repos only:** the "is this version already published" check now follows pnpm's registry resolution and ignores `publishConfig.registry` and scope-based registry overrides in `package.json`. Move those to `.npmrc`.
- **Non-npm registries:** a package that has only ever published prereleases now gets the configured prerelease tag rather than `latest`. npm assigns `latest` itself, so nothing changes there.

---

## Migration Guide

This guide walks through adopting changesets in an existing repository that previously used a different release workflow (manual versioning, semantic-release, Nerdbank.GitVersioning, etc.).

> Already using Changesets v2? You don't need this section — go to [Upgrading from Changesets v2](#upgrading-from-changesets-v2) instead.

### Prerequisites

- Node.js `^22.11 || ^24 || >=26` and npm `>=10.9.0` (pnpm `>=10.0.0` / Yarn `>=4.5.2`) — needed for the changesets CLI, even in .NET repos
- Repository uses GitHub and GitHub Actions
- You have admin access to the repo (for branch protection and secrets)

### Step 1: Install changesets

```bash
npm init -y  # if no package.json exists yet
npm install -D @changesets/cli@^3 @changesets/changelog-github@^1
```

For monorepos, configure workspaces in the root `package.json`:

```json
{
  "private": true,
  "workspaces": ["packages/*"],
  "devDependencies": {
    "@changesets/cli": "^3.0.0",
    "@changesets/changelog-github": "^1.0.0"
  },
  "scripts": {
    "changeset": "changeset",
    "version-packages": "changeset version",
    "release": "changeset publish"
  }
}
```

### Step 2: Configure changesets

```bash
npx changeset init
```

This prompts for the changelog generator, commit behavior, publish access, and base branch, then writes `.changeset/config.json` and `.changeset/README.md`. Because it is interactive it cannot run unattended — in a scripted bootstrap, drop in a config file directly instead. Either way the result should end up looking like this:

```json
{
  "$schema": "https://unpkg.com/@changesets/config@4.0.0/schema.json",
  "changelog": [
    "@changesets/changelog-github",
    { "repo": "neolution-ch/your-repo" }
  ],
  "commit": false,
  "fixed": [],
  "access": "restricted",
  "baseBranch": "main",
  "updateInternalDependencies": "patch",
  "ignore": []
}
```

Adjust as needed:
- **`fixed`**: Add package groups that should always share the same version (e.g., `[["@org/pkg-a", "@org/pkg-b"]]`)
- **`access`**: Use `"restricted"` for GitHub Packages, `"public"` for npm public packages
- **`baseBranch`**: `main` is the default, so this line is optional — set it if your default branch is something else
- **`privatePackages`**: Add `{ "version": true, "tag": true }` if your packages are `"private": true` and you still want them versioned and tagged (see [Adapting for NuGet Packages](#adapting-for-nuget-packages))
- **`format`**: Defaults to `"auto"`, which picks up Prettier, Oxfmt, Deno, or dprint from your project. Set it to `false` to leave generated files unformatted

### Step 3: Set the initial version

Each package's `package.json` must have the **current published version**. If your package is currently at `3.2.1`, make sure `package.json` says `"version": "3.2.1"`. Changesets increments from whatever version is in `package.json`.

For .NET repos, also ensure the `package.json` version matches the `.csproj` `<Version>`.

### Step 4: Add workflows

Copy the CI and Release workflows from [`templates/`](../templates/):

- `.github/workflows/ci.yml` — changeset check on PRs
- `.github/workflows/release.yml` — Version Packages PR + publish

For **public npm** instead of GitHub Packages, follow [Publishing to public npm](#publishing-to-public-npm) (it's more than just the registry URL). For NuGet repos, also add the `nuget-publish.yml` workflow (see [Adapting for NuGet Packages](#adapting-for-nuget-packages)).

Add `.npmrc` if using GitHub Packages:

```
@neolution-ch:registry=https://npm.pkg.github.com
```

> **Important:** `.npmrc` takes precedence over the workflow's `registry-url` and auth token. If you switch to public npm, you must remove or repoint this line — otherwise `changeset publish` silently keeps publishing to GitHub Packages.

### Step 5: Set up the GitHub App

The Release workflow uses a GitHub App token so that Version Packages PRs trigger CI. This is a one-time org-level setup:

1. Create a GitHub App at `https://github.com/organizations/neolution-ch/settings/apps`
   - **Permissions:** Contents (Read & write), Pull requests (Read & write)
   - **Webhook:** Disabled
2. Generate a private key and note the App ID
3. Install the App on the repository (or all org repos)
4. Add secrets (org-level recommended for reuse across repos). The `templates/` workflows reference these names — use the same, or update both to match:
   - `CHANGESETS_BOT_CLIENT_ID`
   - `CHANGESETS_BOT_PRIVATE_KEY`

### Step 6: Configure branch protection

1. Go to **Settings** → **Branches** → **Add rule**
2. Branch name pattern: `main`
3. Enable **Require status checks to pass before merging**
4. Add the **Changeset Check** status check
5. (Recommended) Enable **Require branches to be up to date before merging**

> **Note:** The Changeset Check status won't appear in the list until the CI workflow has run at least once. You can either add it after the first PR, or temporarily skip this step.

### Step 7: Remove old release tooling

Remove whatever you were using before:

| Previous tool                        | What to remove                                                                                                         |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| **semantic-release**                 | `.releaserc`, `@semantic-release/*` packages, release workflow steps                                                   |
| **Nerdbank.GitVersioning**           | `version.json`, `nbgv` tool references, `Nerdbank.GitVersioning` NuGet package                                         |
| **MinVer**                           | `MinVer` NuGet package from `.csproj`                                                                                  |
| **Manual versioning**                | Any scripts or CI steps that bump versions                                                                             |
| **Conventional commits enforcement** | `commitlint`, `husky` commit hooks (optional — you can keep them for commit hygiene, but changesets doesn't need them) |

### Step 8: First changeset

Create a PR with any change and add a changeset:

```bash
npx changeset
# Select your package(s), choose the bump type, write a description
```

Push and verify:
- CI passes with the changeset check
- After merging, the Release workflow creates a Version Packages PR
- Merging the Version Packages PR publishes the package

Once this works end-to-end, the migration is complete. All the scenarios documented in this guide (prereleases, backporting, etc.) are now available.

---

## Reference

### Configuration File

The changesets config lives at `.changeset/config.json`:

| Option                       | Value                                              | Meaning                                                                                                                                                                                                                               |
| ---------------------------- | -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `changelog`                  | `@changesets/changelog-github`                     | Changelogs include PR links and contributor attribution. The `repo` option defaults to `$GITHUB_REPOSITORY`, so it only strictly matters when running `changeset version` outside Actions — keep it set so local runs behave the same |
| `commit`                     | `false`                                            | Changeset commands don't auto-commit                                                                                                                                                                                                  |
| `fixed`                      | `[["@neolution-ch/pkg-a", "@neolution-ch/pkg-b"]]` | Both packages always share the same version                                                                                                                                                                                           |
| `access`                     | `restricted`                                       | npm CLI publish flag. On npmjs.org `restricted` = private, `public` = public; **ignored by GitHub Packages** (visibility is set by repo/package settings). See [Publishing to public npm](#publishing-to-public-npm).                 |
| `baseBranch`                 | `main`                                             | Comparisons are made against `main`. `main` is also the default, so the line can be omitted                                                                                                                                           |
| `updateInternalDependencies` | `patch`                                            | Internal dependency ranges are updated on every release                                                                                                                                                                               |
| `format`                     | _(unset → `"auto"`)_                               | Formatter for generated changesets and changelogs. `"auto"` detects Prettier, Oxfmt, Deno, or dprint; `false` disables formatting                                                                                                     |
| `privatePackages`            | _(unset → `{ "version": false, "tag": false }`)_   | Whether `"private": true` packages are versioned and tagged. Set `{ "version": true, "tag": true }` for repos that version packages they don't publish to npm                                                                         |

### CI Workflows

| Workflow                 | File                                         | Trigger                                  | Purpose                                         |
| ------------------------ | -------------------------------------------- | ---------------------------------------- | ----------------------------------------------- |
| **CI**                   | `.github/workflows/ci.yml`                   | Pull requests to `main` and `release/**` | Blocks PRs without a changeset                  |
| **Release**              | `.github/workflows/release.yml`              | Push to `main` and `release/**`          | Creates Version Packages PR; publishes on merge |
| **Dependabot Changeset** | `.github/workflows/dependabot-changeset.yml` | Pull requests from Dependabot            | Auto-generates changeset for dependency updates |

The Release workflow uses a **GitHub App token** (not `GITHUB_TOKEN`) so that PRs it creates trigger CI checks. It also:

- **Publishes backports with a dist-tag** (e.g., `release-v1`) so the `latest` npm tag stays on the highest stable version from `main`
- **Auto-creates `release/vX.x` branches** when a new major version is published from `main`

### Branch Protection

For blocking CI to work, configure **branch protection rules** for `main` in GitHub Settings:

1. Go to **Settings** → **Branches** → **Add rule**
2. Branch name pattern: `main`
3. Enable **Require status checks to pass before merging**
4. Search for and add the **Changeset Check** status check
5. (Recommended) Enable **Require branches to be up to date before merging**

### Useful Commands

```bash
# Add a changeset interactively
npx changeset

# Add a changeset without prompts
npx changeset --patch @neolution-ch/pkg-a -m "Fix the thing"

# Add an empty changeset (no release)
npx changeset --empty

# Check if changesets exist (used by CI; exits 1 when a changed package has none)
npx changeset status --since=origin/main

# Preview what versions would be bumped
npx changeset status --verbose

# Apply changesets (bump versions + update changelogs; exits 1 when there is nothing to release)
npx changeset version

# Publish packages and create git tags
npx changeset publish

# Create the git tags only, without publishing to a registry
npx changeset git-tag

# Write the publish plan to a file instead of publishing
npx changeset publish-plan --output publish-plan.json

# Enter prerelease mode (allowed tags: alpha, beta, rc)
npx changeset pre enter <alpha|beta|rc>

# Exit prerelease mode
npx changeset pre exit
```

### Further Reading

- [Changesets Documentation](https://changesets.dev)
- [CLI Reference](https://changesets.dev/guide/cli)
- [Config File Options](https://changesets.dev/guide/config)
- [Fixed Packages](https://changesets.dev/guide/fixed-packages)
- [Prereleases](https://changesets.dev/guide/prereleases)
- [Backporting Changes](https://changesets.dev/guide/backporting-changes)
- [Automating Changesets](https://changesets.dev/guide/automating)
- [Beyond npm](https://changesets.dev/guide/beyond-npm) — using changesets for artifacts npm doesn't publish
- [Migrating from v2 to v3](https://changesets.dev/guide/migration)
- [Changesets GitHub Action](https://github.com/changesets/action)
- [GitHub Packages npm Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-npm-registry)
- [GitHub Packages NuGet Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry)
