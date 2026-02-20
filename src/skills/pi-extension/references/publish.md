# Publishing

Extensions are published to npm and installed with `pi install`.

## Package Setup

The `package.json` must have the `pi` key declaring extension resources. See `references/structure.md` for the full template.

Key fields for publishing:

```json
{
  "name": "@scope/pi-my-extension",
  "version": "0.1.0",
  "type": "module",
  "license": "MIT",
  "private": false,
  "publishConfig": { "access": "public" },
  "files": ["src", "README.md"],
  "pi": {
    "extensions": ["./src/index.ts"]
  },
  "peerDependencies": {
    "@mariozechner/pi-coding-agent": ">=0.51.0"
  }
}
```

## Versioning with Changesets

Use [changesets](https://github.com/changesets/changesets) for versioning and changelogs.

### `.changeset/config.json`

```json
{
  "$schema": "https://unpkg.com/@changesets/config@3.1.1/schema.json",
  "changelog": "@changesets/cli/changelog",
  "commit": false,
  "fixed": [],
  "linked": [],
  "access": "public",
  "baseBranch": "main",
  "updateInternalDependencies": "patch",
  "ignore": []
}
```

### Creating a changeset

The interactive CLI (`pnpm changeset`) is not available in headless environments. Write the file directly instead. Create `.changeset/<descriptive-name>.md`:

```md
---
"@scope/pi-my-extension": patch
---

Description of what changed and why it matters to users.
```

Bump types: `patch` for fixes and internal changes, `minor` for new user-facing features, `major` for breaking changes.

Commit the changeset file alongside the changes it describes. Multiple changeset files can coexist — they are all consumed together on the next release.

### Releasing manually (no CI)

```bash
pnpm changeset version   # consumes .changeset/*.md, bumps version, updates CHANGELOG.md
pnpm changeset publish   # publishes to npm
```

## GitHub Actions Automation

The recommended setup uses a single `publish.yml` workflow that runs on every push to `main`. It handles two cases automatically:

- **Pending changesets present**: opens (or updates) a version PR titled `Updating @scope/pi-my-extension to version X.Y.Z`.
- **Version PR merged**: publishes the package to npm and creates a GitHub release with a matching git tag.

Copy `.github/workflows/publish.yml` from `pi-extension-template` into the new repo. It uses `changesets/action@v1` under the hood.

The workflow requires two secrets, configured in the repo's GitHub settings under **Settings → Secrets and variables → Actions**:

- `GITHUB_TOKEN` — automatically provided by GitHub Actions, no setup needed.
- `NPM_TOKEN` — an npm automation token with publish access to the `@scope` org. Create one at npmjs.com under **Access Tokens → Generate New Token → Automation**. Add it as a repository secret named `NPM_TOKEN`. Without this, the publish step will fail silently on the version PR merge.

The workflow also sets `NPM_CONFIG_PROVENANCE=true`, which links the published package to the GitHub Actions run for supply chain transparency (requires the `id-token: write` permission, already included in the template).

## First-time Setup for a New Package

Before the workflow can publish a package that has never been on npm:

1. Make sure `"private": false` and `"publishConfig": { "access": "public" }` are in `package.json`.
2. Add the `NPM_TOKEN` secret to the repo (see above).
3. The first time the version PR is merged, the workflow publishes the package. npm will create the package entry automatically — no manual `npm publish` needed.

If the package name is scoped (e.g., `@aliou/pi-my-extension`) and the scope is new to your npm account, you may need to create the scope first at npmjs.com or run `npm publish --access public` once manually to register it.

## Installation

Users install extensions with:

```bash
pi install @scope/pi-my-extension
```

Pi reads the `pi` key from the package's `package.json` to discover extensions, skills, themes, and prompts.

## Dependency Management in Monorepos

If publishing from a monorepo that contains both public and private packages:

**Critical rule**: Public packages cannot depend on private workspace packages. This will break when users try to install your package from npm.

In the pi-extensions monorepo, this is enforced by:
- Pre-commit hook that blocks commits with invalid dependencies
- CI check that prevents merging bad dependencies
- `pnpm run check:public-deps` validates all dependencies

When adding a workspace dependency to a `package.json`:
1. Check if the dependency is public (`"private": false` or `"publishConfig": { "access": "public" }`).
2. If the dependency is private, either make it public, make your package private, or remove the dependency.

## Pre-publish Checklist

- [ ] `"private": false` is set.
- [ ] `"publishConfig": { "access": "public" }` is set.
- [ ] `"files"` lists only what users need (`["src", "README.md"]`).
- [ ] `peerDependencies` version range is correct (`>=` minimum supported version).
- [ ] `@mariozechner/pi-tui` is in `peerDependencies` with `optional: true` in `peerDependenciesMeta` if imported at runtime.
- [ ] `prepare` script is `[ -d .git ] && husky || true`, not bare `husky`.
- [ ] `check:lockfile` script is present.
- [ ] `description` is clear and concise.
- [ ] `pi.extensions` paths are correct.
- [ ] `NPM_TOKEN` secret is set on the GitHub repo.
- [ ] `.github/workflows/publish.yml` is present.
- [ ] If in a monorepo: no dependency on private workspace packages (`pnpm run check:public-deps` if available).
- [ ] README documents what the extension does, required environment variables, and available tools/commands.
- [ ] If wrapping a third-party API: extension handles missing API key gracefully (notification, not crash).
- [ ] `pnpm typecheck` and `pnpm lint` pass.
