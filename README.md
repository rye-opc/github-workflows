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

