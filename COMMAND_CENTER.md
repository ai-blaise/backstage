# Command Center Fork

`ai-blaise/backstage` branch `command-center` is the source of truth for the Backstage command-center fork.

- Rebase cadence: monthly from `backstage/backstage` `master`.
- Package convention: `@backstage/*` packages publish with a `-cc.N` prerelease suffix on the `cc` dist-tag.
- Deployment consumer: `ai-blaise/platform` via `deploy/backstage/values.yaml` and the Backstage Argo CD application.
- Rebase mechanics pull source changes into `ai-blaise/backstage`; fork changes remain in the ai-blaise namespace.

## Publication pipeline

Every push to `command-center` (and any `workflow_dispatch`) triggers:

1. `.github/workflows/publish-cc.yml` — rewrites the six packages enumerated in `.github/cc-published-packages.txt` to `<base>-cc.<CC_REVISION>`, publishes them to GitHub Packages under `@backstage/*` on the `cc` dist-tag with `--access restricted`, then tags `cc-publish-<CC_REVISION>`.
2. `.github/workflows/publish-image.yml` — builds `packages/backend/Dockerfile` and pushes `ghcr.io/ai-blaise/backstage:command-center-<base-ver>-cc.<CC_REVISION>` plus `:command-center-latest`, attaches an SPDX-JSON SBOM, and best-effort Cosign-keyless signs (graceful skip if Sigstore is unwired).

`CC_REVISION = git rev-list --count <UPSTREAM_REBASE_BASE>..HEAD` where `UPSTREAM_REBASE_BASE` lives at the repo root and is updated by the monthly rebase pipeline.

## Consuming from command-center

Downstream repos consume the published artifacts as follows.

### npm packages from GitHub Packages

Authenticate npm to GitHub Packages and add the registry scope:

```
# ~/.npmrc or repo-local .npmrc
@backstage:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
```

Pin to the `cc` dist-tag or to a specific `-cc.N` version in the consumer's manifest:

```
yarn add @backstage/app-defaults@cc
yarn add @backstage/backend-defaults@1.2.3-cc.42
```

The complete list of packages republished from this fork lives at `.github/cc-published-packages.txt`.

### Container image from ghcr.io

The fork's backend image is published as `ghcr.io/ai-blaise/backstage:command-center-<base-ver>-cc.<CC_REVISION>` and `ghcr.io/ai-blaise/backstage:command-center-latest`. Pin `deploy/backstage/values.yaml` (`ai-blaise/platform`) to an immutable `-cc.<N>` tag; never deploy from `-latest`.

Cosign verification (when Sigstore is wired):

```
cosign verify ghcr.io/ai-blaise/backstage:command-center-<base-ver>-cc.<CC_REVISION> \
  --certificate-identity-regexp 'https://github.com/ai-blaise/backstage/.*' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com
```
