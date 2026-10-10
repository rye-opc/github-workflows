## github-workflows

Reusable GitHub Actions workflows for the `rye-opc` org.

### Notes

- **Public repo safety**: workflow source is public, but **secrets are not**. Avoid hardcoding credentials in YAML.
- **GITHUB_TOKEN permissions**: called workflows cannot elevate permissions. Make sure the *caller* workflow grants what you need (for example `packages: write` to push to GHCR).
- **Self-hosted runners**: do not run untrusted code (avoid using them for PRs from forks).

### Usage

Reference a workflow from another repository:

```yaml
jobs:
  deploy:
    uses: rye-opc/github-workflows/.github/workflows/deploy.staging.yaml@main
    with:
      app_name: my-app
    permissions:
      contents: read
      packages: write
    secrets:
      DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```


### Trunk-based releases (opt-in)

Repos can drop the `staging` / `release` branches and use:

| Event | Environment |
|---|---|
| push to `main` (PR merge) | staging |
| push tag `v*` | production (approval gate) |

Copy [`examples/deploy.trunk.yaml`](examples/deploy.trunk.yaml) into the app repo as
`.github/workflows/deploy.yaml`, set `app_name`, then cut releases with:

```sh
gh release create v1.2.3 --generate-notes
```

`deploy.production.yaml` inputs that make this safe (both default to off, so existing
branch-based callers are unaffected):

- `require_ancestor_of: main` — refuses to deploy a tag whose commit is not reachable from
  `main` (e.g. a tag on an unmerged feature branch).
- `reuse_existing_image: true` — skips the Docker build if an image for the commit SHA is
  already in GHCR (built by the staging deploy), so production ships the exact staging image.
  Falls back to a normal build if the image is missing.

When triggered by a tag, the image is additionally tagged with the git tag name
(`ghcr.io/rye-opc/<app>:v1.2.3`). The HelmRelease still pins the commit SHA.

Migration checklist per repo:

1. Replace the deploy workflow with the trunk example and merge it to `main`.
2. Confirm the `main` push deploys to staging.
3. Check no open PRs target `staging` / `release`, then delete those branches.
4. Tag the first release: `gh release create v1.0.0 --generate-notes`.

Hotfix: fix on a branch, merge the PR to `main`, tag the merge commit. If `main` holds
unreleased work you do not want shipped, branch from the last tag
(`git checkout -b hotfix/x v1.2.3`), merge the fix there, and tag that commit with
`require_ancestor_of` temporarily pointed at the hotfix branch, then merge the hotfix back to `main`.
